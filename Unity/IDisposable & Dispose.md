# IDisposable과 Dispose (IDisposable & Dispose)

## 핵심 개념

- **문제**: [[GC]]는 언제 객체를 회수할지 알려주지 않음. 그래서 파일 핸들, 소켓, 네이티브 메모리처럼 **제때 놓아야 하는 자원**을 GC에 맡길 수 없음
- **IDisposable**: 해제 시점을 **개발자가 직접 정하는** 규약. `Dispose()` 메서드 하나만 가짐
- 소멸자(Finalizer)가 "언젠가" 정리하는 것과 달리, `Dispose()`는 **호출한 그 순간** 해제됨

```csharp
public interface IDisposable
{
    void Dispose();
}
```

## 소멸자의 한계

소멸자(`~ClassName()`)로는 자원 해제를 보장할 수 없음.

| 문제 | 내용 |
| --- | --- |
| 시점이 비결정적 | GC가 돌 때까지 실행되지 않음. 몇 초 뒤일 수도 있음 |
| 회수가 늦어짐 | 종결 큐를 거치므로 GC 사이클을 한 번 더 기다림 |
| 순서 보장 없음 | 객체 간 해제 순서를 제어할 수 없음 |
| 스레드가 다름 | 전용 종결자 스레드에서 실행됨 |
| 아예 안 불릴 수 있음 | 프로세스가 강제 종료되면 실행되지 않음 |

```mermaid
graph LR
    A["객체가 도달 불가 상태"] --> B{"Finalizer 있음?"}
    B -->|없음| C["이번 GC에서 회수"]
    B -->|있음| D["종결 큐에 등록"]
    D --> E["종결자 스레드가<br/>Finalizer 실행"]
    E --> F["다음 GC에서 회수"]
```

- 소멸자를 붙이는 것만으로 **회수가 한 사이클 늦어짐**. 성능 비용이 공짜가 아님
- 그래서 소멸자는 주 수단이 아니라 **마지막 안전망**으로만 씀

## using 문

`Dispose()`를 잊지 않게 해주는 문법임.

```csharp
using (var stream = new FileStream(path, FileMode.Open))
{
    // 사용
}   // 블록을 벗어나면 Dispose() 자동 호출

// C# 8 이상 — 선언형
using var stream = new FileStream(path, FileMode.Open);
// 스코프가 끝날 때 Dispose() 호출
```

- 컴파일하면 `try-finally`가 됨. **예외가 발생해도 반드시 해제됨**

```csharp
// using의 실제 컴파일 결과
var stream = new FileStream(path, FileMode.Open);
try
{
    // 사용
}
finally
{
    if (stream != null) stream.Dispose();
}
```

## 관리 자원과 비관리 자원

| 구분 | 예시 | 해제 주체 |
| --- | --- | --- |
| 관리 자원 | 일반 클래스 인스턴스, 배열 | GC가 알아서 함 |
| 비관리 자원 | 파일 핸들, 소켓, 네이티브 메모리, GPU 버퍼 | **직접 해제해야 함** |

- GC는 **관리 힙만** 알고 있음. OS가 쥔 핸들이나 네이티브 할당은 모름
- `IDisposable`이 필요한 이유가 전적으로 이 비관리 자원 때문임

## Dispose 패턴

비관리 자원을 직접 들고 있을 때의 표준 구현임.

```csharp
public class ResourceHolder : IDisposable
{
    private IntPtr nativeHandle;      // 비관리 자원
    private FileStream stream;        // 관리 자원 (자신도 IDisposable)
    private bool disposed;

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);    // 이미 정리했으니 소멸자를 건너뛰게 함
    }

    protected virtual void Dispose(bool disposing)
    {
        if (disposed) return;         // 중복 호출 방어

        if (disposing)
        {
            // 관리 자원 해제 — Dispose()로 들어온 경우만
            stream?.Dispose();
        }

        // 비관리 자원 해제 — 항상
        ReleaseHandle(nativeHandle);

        disposed = true;
    }

    ~ResourceHolder() => Dispose(false);   // 안전망
}
```

**왜 `disposing` 플래그가 있는가**

- `Dispose()`로 들어오면 `true` — 관리 자원도 아직 살아있으니 함께 정리함
- 소멸자로 들어오면 `false` — 다른 관리 객체가 이미 회수됐을 수 있어 **건드리면 안 됨**

**`GC.SuppressFinalize`의 역할**

- `Dispose()`가 이미 정리했으면 소멸자를 실행할 이유가 없음
- 종결 큐에서 빼서 회수 지연을 없앰

**지켜야 할 규칙**

- `Dispose()`는 **여러 번 불러도 안전해야** 함 (멱등성)
- Dispose 후 다른 메서드를 쓰면 `ObjectDisposedException`을 던질 것
- 비관리 자원이 없고 다른 `IDisposable`만 들고 있다면 **소멸자는 만들지 말 것**. 단순 `Dispose()`만으로 충분함

## 게임 개발 관점에서

**유니티에서 Dispose가 필요한 것들**

| 타입 | 비고 |
| --- | --- |
| `NativeArray<T>` 등 네이티브 컨테이너 | Job System. 해제 안 하면 에디터에 누수 경고가 뜸 |
| `ComputeBuffer` | GPU 버퍼. `Dispose()` 또는 `Release()` |
| `UnityWebRequest` | 내부 네이티브 자원 보유 |
| `CommandBuffer` | 렌더링 명령 버퍼 |
| `FileStream`, `StreamReader` | 세이브/로드 |
| `CancellationTokenSource` | 비동기 취소 |

**UnityEngine.Object는 Dispose가 아님**

- `GameObject`, `Texture2D`, `Material`, `Mesh`는 `Destroy()`로 해제함
- 네이티브 측에 실체가 있고 유니티가 별도로 관리하는 구조임
- 주의 — 코드로 만든 `Material`이나 `Texture2D`는 자동으로 사라지지 않음. 직접 `Destroy`할 것

**Job System에서 가장 자주 만남**

```csharp
var array = new NativeArray<float>(100, Allocator.TempJob);
try
{
    // Job 실행
}
finally
{
    array.Dispose();   // 반드시
}

// 또는
using var array = new NativeArray<float>(100, Allocator.TempJob);
```

- 네이티브 컨테이너는 GC 대상이 아니라 직접 할당한 메모리임. 해제하지 않으면 그대로 누수됨

**구독 해제를 Dispose로 묶는 패턴**

- 이벤트 구독 해제를 `IDisposable`로 감싸면 해제를 잊기 어려워짐
- R3(구 UniRx)의 구독이 `IDisposable`을 반환하는 이유임
- `CompositeDisposable`에 모아두고 `OnDestroy`에서 한 번에 해제하는 구조가 흔함

```csharp
private readonly CompositeDisposable disposables = new();

void OnDestroy() => disposables.Dispose();   // 모든 구독 한 번에 해제
```

- [[Event & Pub-Sub Pattern]]에서 다룬 누수 문제를 구조적으로 막는 방법임

## 의문점 / 더 알아볼 것

- `SafeHandle` — 비관리 핸들을 더 안전하게 다루는 표준 방식
- `IAsyncDisposable`과 `await using` — 비동기 해제가 필요한 경우
- `Allocator.Temp`, `TempJob`, `Persistent`의 수명 차이와 Dispose 필요 여부
- 유니티 Boehm GC에서 소멸자가 실제로 어떻게 처리되는가
- `using`을 매 프레임 쓰는 것이 성능에 미치는 영향 — 객체 생성 비용과 분리해서 보기
- Addressables의 핸들 Release와 Dispose 패턴의 관계
- `Dispose()`에서 예외가 발생하면 어떻게 되는가