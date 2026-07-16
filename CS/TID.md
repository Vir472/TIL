# Thread ID (TID)

## 관련 노트

→ [[TCB]] — TID가 저장되는 자료구조
→ [[process-id]] — PID와 비교

## 핵심 개념

- **TID(Thread ID)**: OS가 스레드를 식별하기 위해 부여하는 고유 번호
- PID가 프로세스를 식별하듯, TID는 프로세스 안의 스레드를 식별
- [[TCB]]에 저장되는 핵심 식별 정보

## PID와의 관계

```
프로세스 (PID: 1024)
├─ 메인 스레드  (TID: 1024)  ← 메인 스레드의 TID는 보통 PID와 동일
├─ 렌더 스레드 (TID: 1025)
├─ 오디오 스레드 (TID: 1026)
└─ 워커 스레드  (TID: 1027)
```

- 메인 스레드의 TID는 대부분의 OS에서 PID와 같은 값
- 같은 프로세스 안의 스레드들은 PID를 공유하고 TID만 다름

## OS별 TID 특성

### Linux

- Linux는 스레드를 프로세스와 동일한 구조(`task_struct`)로 관리
- TID = PID와 동일한 구조로 할당 — 커널 입장에선 스레드도 경량 프로세스
- `getpid()` → 프로세스 PID / `gettid()` → 현재 스레드 TID
- 같은 프로세스 내 스레드들은 서로 다른 TID를 가지지만 같은 TGID(Thread Group ID)를 공유 → TGID가 사실상 PID 역할

### Windows

- 프로세스와 스레드를 명확히 분리해서 관리
- TID는 PID와 독립적인 번호 체계 — 시스템 전체에서 유일
- 작업 관리자에서 스레드 탭으로 TID 확인 가능

### Android / iOS

- Linux/XNU 커널 기반이라 각각 Linux/macOS의 TID 구조를 따름

## C#에서 TID 확인

```csharp
using System.Threading;

// 현재 스레드 TID
int tid = Thread.CurrentThread.ManagedThreadId;
Console.WriteLine($"현재 스레드 ID: {tid}");

// 메인 스레드 여부 확인
bool isMain = Thread.CurrentThread.IsBackground == false
              && Thread.CurrentThread.ManagedThreadId == 1;
```

- `ManagedThreadId`: .NET이 관리하는 스레드 ID. OS의 실제 TID와 다를 수 있음
- .NET 스레드 풀에서 재사용되는 스레드는 ManagedThreadId가 바뀔 수 있음

## 게임 개발 관점에서

- **Unity 메인 스레드 확인**: 현재 코드가 메인 스레드에서 실행 중인지 TID로 확인 가능. Unity API 호출 전 메인 스레드 여부를 체크하는 방어 코드에 활용

```csharp
  // 메인 스레드 TID를 Start()에서 저장해두고 비교
  int mainThreadId;

  void Start()
  {
      mainThreadId = Thread.CurrentThread.ManagedThreadId;
  }

  void SomeMethod()
  {
      if (Thread.CurrentThread.ManagedThreadId != mainThreadId)
      {
          Debug.LogError("메인 스레드에서만 호출 가능!");
          return;
      }
  }
```

- **디버깅 시 TID 활용**: Unity Profiler나 디버거에서 스레드별 TID로 어느 스레드에서 병목이 생기는지 식별 가능

## 의문점 / 더 알아볼 것

- Linux의 TGID(Thread Group ID)와 TID의 정확한 관계
- .NET ManagedThreadId와 OS 실제 TID가 다른 이유
- Unity Job System의 워커 스레드들은 TID가 고정인지 재사용되는지
