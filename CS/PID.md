# Process ID (PID)

## 핵심 개념

- **PID(Process ID)**: OS가 프로세스를 식별하기 위해 부여하는 고유 번호
- 프로세스가 생성될 때 OS가 자동으로 할당하고, 종료되면 반환됨
- [[PCB]]에 저장되는 핵심 식별 정보

## 기본 특성

```
프로세스 생성
    ↓
OS가 PID 할당 (사용 중이지 않은 번호 중 선택)
    ↓
프로세스 종료
    ↓
PID 반환 → 나중에 다른 프로세스에 재사용 가능
```

- PID는 **같은 시점에 유일**함. 단, 프로세스가 종료된 후엔 같은 번호가 재사용될 수 있음
- **PPID(Parent PID)**: 자신을 생성한 부모 프로세스의 PID. 프로세스는 트리 구조로 관리됨

## 프로세스 트리 구조

```
PID 1 (init / systemd)        ← 최초 프로세스, 모든 프로세스의 조상
├─ PID 512 (sshd)
│   └─ PID 1024 (bash)
│        └─ PID 2048 (game_server)
└─ PID 600 (Unity)
    └─ PID 601 (렌더 워커)
```

## C#에서 PID 확인

```csharp
using System.Diagnostics;

// 현재 프로세스 PID
int myPid = Process.GetCurrentProcess().Id;
Console.WriteLine($"현재 PID: {myPid}");

// 실행 중인 전체 프로세스 목록
Process[] processes = Process.GetProcesses();
foreach (var p in processes)
{
    Console.WriteLine($"{p.ProcessName} — PID: {p.Id}");
}

// 특정 이름의 프로세스 찾기
Process[] unity = Process.GetProcessesByName("Unity");
```

## 게임 개발 관점에서

- **직접 쓸 일은 드뭄**: 게임 로직에서 PID를 직접 다루는 경우는 거의 없음
- **멀티프로세스 게임 구조**: 런처(PID A) → 게임 클라이언트(PID B) → 안티치트(PID C) 형태로 각각 별도 프로세스로 실행. 런처가 게임 클라이언트의 PID를 알고 있어야 모니터링/종료 가능
- **크래시 리포터**: 게임 크래시 시 별도 프로세스(크래시 리포터)가 죽은 프로세스의 PID를 기록해 덤프 파일과 함께 서버로 전송
- **안티치트**: 외부 프로세스가 게임 프로세스 PID에 접근해 메모리를 읽으려는 시도를 탐지 → 핵/치트 방지의 핵심 원리

## 의문점 / 더 알아볼 것

- PID 네임스페이스(Linux) — 컨테이너(Docker)에서 프로세스마다 독립적인 PID 공간을 갖는 원리
- Windows에서 PID가 항상 4의 배수인 내부적인 이유
- iOS 샌드박스가 PID 접근을 차단하는 구체적인 메커니즘
