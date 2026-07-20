# Thread Status (스레드 상태)

## 관련 노트

→ [[TCB]] — 스레드 상태가 저장되는 자료구조
→ [[process-status]] — 프로세스 상태와 비교

## 핵심 개념

- 스레드도 프로세스처럼 생성부터 종료까지 여러 **상태(State)** 를 거침
- 프로세스 상태와 유사하지만 스레드는 같은 프로세스 안에서 상태가 독립적으로 관리됨
- 현재 상태는 [[TCB]]에 저장됨

## 스레드 상태 종류

### 5가지 기본 상태

```
        생성
         ↓
       New (생성)
         ↓ start() 호출
      Runnable (실행 가능) ←──────────────────┐
         │                                    │
         │ 스케줄러 선택                        │ 타임아웃 /
         ↓                                    │ yield()
      Running (실행 중) ───────────────────────┘
         │
         ├─── I/O 대기 / sleep() / wait() ──→ Blocked (차단)
         │                                         │
         │                                         │ I/O 완료 /
         │                                         │ notify() / 시간 만료
         │                                   Runnable로 복귀
         │
         └─── 종료 ──→ Terminated (종료)
```

| 상태           | 설명                                               |
| -------------- | -------------------------------------------------- |
| **New**        | 스레드 객체가 생성됐지만 아직 시작 안 된 상태      |
| **Runnable**   | 실행 준비 완료. CPU를 기다리거나 실행 중인 상태    |
| **Running**    | CPU를 점유하고 실제로 실행 중인 상태               |
| **Blocked**    | I/O 대기, sleep, lock 대기 등으로 일시 중단된 상태 |
| **Terminated** | 실행이 끝난 상태. 재시작 불가                      |

## 프로세스 상태와의 차이

| 항목              | 프로세스       | 스레드                                   |
| ----------------- | -------------- | ---------------------------------------- |
| 상태 관리         | PCB            | TCB                                      |
| Waiting 진입 원인 | I/O 요청       | I/O, sleep(), wait(), lock 대기 등 다양  |
| 종료 후           | 좀비 상태 가능 | 바로 Terminated (좀비 없음)              |
| 독립성            | 완전 독립      | 같은 프로세스 내 다른 스레드에 영향 가능 |
| 생성 비용         | 무거움         | 가벼움                                   |

## Blocked 상태의 세부 원인

```
Blocked
├─ I/O 대기        — 파일 읽기/쓰기, 네트워크 응답 대기
├─ sleep()         — 지정한 시간만큼 대기
├─ wait()          — 다른 스레드의 notify() 신호 대기
├─ lock 대기       — 다른 스레드가 점유한 lock 해제 대기
└─ join()          — 다른 스레드가 종료될 때까지 대기
```

## C#에서의 스레드 상태

C#은 `System.Threading.ThreadState` 열거형으로 스레드 상태를 표현

```csharp
Thread t = new Thread(() => { Thread.Sleep(1000); });

Console.WriteLine(t.ThreadState); // Unstarted (New)
t.Start();
Console.WriteLine(t.ThreadState); // Running or WaitSleepJoin (Blocked)
t.Join();
Console.WriteLine(t.ThreadState); // Stopped (Terminated)
```

| C# ThreadState  | 의미                                           |
| --------------- | ---------------------------------------------- |
| `Unstarted`     | New — 아직 Start() 호출 안 됨                  |
| `Running`       | Running                                        |
| `WaitSleepJoin` | Blocked — wait/sleep/join으로 대기 중          |
| `Stopped`       | Terminated                                     |
| `Suspended`     | 일시 정지 (Deprecated)                         |
| `Background`    | 백그라운드 스레드 여부 (다른 상태와 조합 가능) |

## 게임 개발 관점에서

- **Unity 메인 스레드**: 항상 Running 또는 Runnable 상태를 반복. `Update()` 루프가 매 프레임 실행됨
- **백그라운드 스레드 Blocked 활용**: 네트워크 응답 대기, 파일 로딩 시 백그라운드 스레드가 Blocked 상태로 전환 → 메인 스레드는 영향 없이 계속 실행 가능. 비동기 로딩의 원리
- **lock 대기로 인한 Blocked 주의**: 여러 스레드가 같은 자원에 접근할 때 lock 경쟁이 심해지면 스레드들이 Blocked 상태에 머무는 시간이 늘어나 성능 저하
- **Task와 스레드 상태**: `Task.Run()`으로 실행한 작업도 내부적으로 스레드 상태를 거침. `await`로 대기할 때 해당 스레드는 Blocked가 아니라 스레드 풀로 반환됨 — async/await가 스레드를 블로킹하지 않는 이유

## 의문점 / 더 알아볼 것

- `await`가 스레드를 Blocked로 만들지 않고 스레드 풀로 반환하는 내부 구조 (상태 머신)
- Unity Job System의 워커 스레드 상태 전이 방식
- 스레드가 Terminated 된 후 ThreadPool이 재사용하는 방식
