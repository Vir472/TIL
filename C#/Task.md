# Task

## 핵심 개념

- `System.Threading.Tasks` 네임스페이스에 있는 **.NET 클래스**
- OS도 Unity도 아닌 **C# (.NET)** 이 제공하는 비동기 작업 단위
- 스레드를 직접 다루지 않고 "이 작업을 백그라운드에서 실행해줘"라고 요청하는 고수준 추상화

## Task가 생겨난 이유

### 기존 Thread의 문제점

```csharp
// Thread 직접 사용 — 번거롭고 위험
Thread t = new Thread(() =>
{
    var result = HeavyWork();
    // 결과를 메인 스레드로 어떻게 전달하지?
    // 예외가 나면 어떻게 잡지?
    // 취소하려면?
});
t.Start();
t.Join();  // 끝날 때까지 블로킹
```

Thread를 직접 쓰면 이런 문제가 있었음

- **결과값 반환 불편**: 반환값이 없어서 공유 변수나 콜백으로 우회
- **예외 처리 어려움**: 백그라운드 스레드의 예외가 메인 스레드로 전파 안 됨
- **취소 어려움**: 강제 종료(`Thread.Abort`)는 위험하고 불안정
- **스레드 생성 비용**: 매번 OS 스레드를 새로 만드는 건 무거움
- **콜백 지옥**: 비동기 작업이 중첩되면 코드가 극도로 복잡해짐

### 해결책으로 등장한 Task (.NET 4.0, 2010)

```
Thread (저수준)
  └─ ThreadPool (스레드 재사용)
       └─ Task (고수준 추상화) ← 여기
            └─ async/await (문법 설탕)
```

Task는 ThreadPool 위에서 동작 → 매번 새 스레드를 만들지 않고 풀에서 재사용

## Thread vs Task 비교

| 항목        | Thread                | Task                             |
| ----------- | --------------------- | -------------------------------- |
| 제공 주체   | .NET (OS 스레드 래핑) | .NET (ThreadPool 추상화)         |
| 스레드 생성 | 매번 새로 생성        | ThreadPool에서 재사용            |
| 반환값      | 없음                  | `Task<T>`로 반환값 가능          |
| 예외 처리   | 어려움                | `try/catch`로 자연스럽게         |
| 취소        | `Thread.Abort` (위험) | `CancellationToken`으로 안전하게 |
| 연속 작업   | 직접 구현 필요        | `ContinueWith`, `await`로 간단   |
| 권장 여부   | 저수준 제어 필요 시만 | 대부분의 경우 권장               |

## 기본 사용

```csharp
// 반환값 없는 Task
Task task = Task.Run(() =>
{
    Console.WriteLine("백그라운드 실행");
});
await task;  // 완료까지 대기

// 반환값 있는 Task<T>
Task<int> task = Task.Run(() =>
{
    return 42;  // 결과값 반환
});
int result = await task;  // 결과값 받기
```

## async / await — Task를 편하게 쓰는 문법

```csharp
// Task만 쓸 때 (콜백 중첩 — 읽기 어려움)
Task.Run(() => LoadData())
    .ContinueWith(t => ProcessData(t.Result))
    .ContinueWith(t => SaveData(t.Result));

// async/await 사용 (동기 코드처럼 읽힘)
async Task DoWork()
{
    var data = await LoadDataAsync();
    var processed = await ProcessDataAsync(data);
    await SaveDataAsync(processed);
}
```

- `async`: 이 메서드가 비동기임을 선언
- `await`: Task가 완료될 때까지 "여기서 잠깐 기다렸다가 계속 실행해줘" — 스레드를 블로킹하지 않음

## await가 스레드를 블로킹하지 않는 이유

```
// 일반 블로킹
Thread.Sleep(1000);  // 1초 동안 이 스레드가 아무것도 못 함

// await
await Task.Delay(1000);
// 1초 대기하는 동안 이 스레드는 다른 작업 처리 가능
// 1초 후 여기서 다시 재개
```

await를 만나면 현재 메서드의 실행을 일시 중단하고 스레드를 반환 → 다른 작업에 쓰임 → 완료 후 다시 이어서 실행

## 취소 — CancellationToken

```csharp
CancellationTokenSource cts = new CancellationTokenSource();

Task task = Task.Run(() =>
{
    for (int i = 0; i < 1000; i++)
    {
        cts.Token.ThrowIfCancellationRequested();  // 취소 요청 시 예외
        DoWork(i);
    }
}, cts.Token);

// 외부에서 취소
cts.Cancel();
```

## 계층 정리

```
OS
└─ 커널 스레드 (실제 CPU에서 돌아가는 단위)

.NET CLR
├─ Thread        — OS 커널 스레드를 1:1로 래핑
├─ ThreadPool    — 스레드를 미리 만들어두고 재사용
└─ Task          — ThreadPool 위의 고수준 추상화
     └─ async/await — Task를 동기 코드처럼 쓰는 문법

Unity
└─ Job System    — Unity 자체 멀티스레드 솔루션 (Task와 별개)
```

## 게임 개발 관점에서

- **Unity에서 Task 사용 가능한 이유**: Unity가 .NET(Mono/IL2CPP)을 런타임으로 사용하기 때문. Unity가 제공하는 게 아니라 .NET이 깔려있어서 쓸 수 있는 것
- **Unity 메인 스레드 제약 주의**: Task로 백그라운드 작업을 돌려도 Unity API 호출은 메인 스레드에서만 가능
- **UniTask**: Unity에 최적화된 Task 대체 라이브러리. Unity의 프레임 단위 타이밍과 잘 맞고 GC 할당이 없어서 게임에서 선호됨

```csharp
  async void RunSimulation()
  {
      var winRate = await Task.Run(() => SimulateBattle(10000));
      UpdateUI(winRate);  // 여기는 메인 스레드로 복귀
  }
```

## 의문점 / 더 알아볼 것

- async/await가 내부적으로 상태 머신(State Machine)으로 변환되는 구조
- `Task.Run` vs `Task.Factory.StartNew` 차이
- UniTask가 일반 Task 대비 GC 할당이 없는 이유
