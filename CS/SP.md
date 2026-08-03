# SP (Stack Pointer)

## 핵심 개념

- **스택 포인터(Stack Pointer)**: 현재 Stack의 최상단(Top) 주소를 저장하는 레지스터
- 함수 호출/반환, 지역변수 할당/해제 시 자동으로 변경됨
- x86-64에서는 `RSP`, ARM64에서는 `SP`

## Stack과 SP의 관계

```
높은 주소
┌──────────────┐ ← Stack 시작 (높은 주소)
│   함수 A     │
│  지역변수들  │
├──────────────┤ ← SP (현재 Stack Top)
│   (빈 공간)  │
│              │
└──────────────┘ ← 낮은 주소
```

Stack은 높은 주소에서 낮은 주소 방향으로 자람 → SP는 아래로 이동

## 함수 호출 시 SP 변화

```
초기 SP = 0x7FFF80

함수 A 호출
  SP -= 필요한 공간  → SP = 0x7FFF60  (지역변수 공간 확보)
  지역변수들이 [0x7FFF60 ~ 0x7FFF80]에 저장

  함수 B 호출 (A 안에서)
    SP -= 필요한 공간 → SP = 0x7FFF40
    B의 지역변수들이 [0x7FFF40 ~ 0x7FFF60]에 저장

  함수 B 반환
    SP += B가 사용한 공간 → SP = 0x7FFF60  (B의 Stack Frame 해제)

함수 A 반환
  SP += A가 사용한 공간 → SP = 0x7FFF80  (A의 Stack Frame 해제)
```

## Stack Overflow와 SP

```
재귀 함수가 끝없이 호출됨
  → SP가 계속 낮은 주소로 이동
  → Stack 공간 한계에 도달
  → Stack Overflow 발생 → 크래시
```

SP가 허용 범위를 벗어나면 OS가 예외를 발생시킴

## 컨텍스트 스위칭에서 SP

```
프로세스 A 실행 중
  SP = 0x7FFF60  ← A의 현재 Stack Top
    ↓ 컨텍스트 스위칭
  SP 값 → A의 PCB에 저장
  B의 PCB에서 SP 복원 → 0x6FFF80
  B의 Stack에서 이어서 실행
```

SP가 정확히 복원되어야 함수 호출 흐름이 올바르게 이어짐

## BP(Base Pointer)와의 관계

```
SP — 현재 Stack Top (계속 변함)
BP — 현재 함수의 Stack Frame 시작점 (함수 안에서 고정)

함수 안에서 지역변수 접근:
  변수 x → [BP - 4]  (BP 기준 상대 주소로 접근)
  변수 y → [BP - 8]
```

SP는 계속 변하기 때문에 BP를 기준점으로 지역변수 주소를 계산

## 게임 개발 관점에서

- 직접 다룰 일 없음 — 컴파일러가 자동 관리
- **Stack Overflow**: Unity에서 종료 조건 없는 재귀 함수나 너무 깊은 재귀 탐색 시 SP가 Stack 한계를 벗어나 크래시. `StackOverflowException`
- **Stack 크기 제한**: 각 스레드마다 독립적인 Stack을 가지며 기본 크기가 정해져 있음 (Windows 기본 1MB, Linux 기본 8MB). 매우 큰 지역변수 배열 선언 시 SP가 한계를 넘을 수 있음

## 의문점 / 더 알아볼 것

- Red Zone — x86-64 Linux에서 SP 아래 128바이트를 함수 없이 사용할 수 있는 최적화 영역
- 각 플랫폼별 기본 Stack 크기와 Unity에서 스레드 Stack 크기를 설정하는 방법
