# TIL - Flag Register (플래그 레지스터)

## 핵심 개념

- **플래그 레지스터(Flag Register)**: ALU 연산 결과의 상태를 비트 단위로 저장하는 레지스터
- **Status Register**, **Condition Code Register(CCR)** 이라고도 불림
- 각 비트가 하나의 상태(플래그)를 나타냄
- 조건 분기 명령어(`if`, `while` 등)가 내부적으로 이 레지스터를 확인해서 분기를 결정

## 주요 플래그

| 플래그         | 약자 | 설명                                                                       |
| -------------- | ---- | -------------------------------------------------------------------------- |
| Zero Flag      | ZF   | 연산 결과가 0이면 1. `if (x == 0)` 분기에 활용                             |
| Carry Flag     | CF   | 덧셈에서 최상위 비트 올림(Carry) 발생 시 1. 부호 없는 정수 오버플로우 감지 |
| Sign Flag      | SF   | 연산 결과가 음수이면 1. 최상위 비트(부호 비트)와 동일                      |
| Overflow Flag  | OF   | 부호 있는 정수 연산에서 오버플로우 발생 시 1                               |
| Parity Flag    | PF   | 결과의 하위 8비트에서 1의 개수가 짝수이면 1                                |
| Interrupt Flag | IF   | 외부 인터럽트 허용 여부. 1이면 인터럽트 허용                               |

## 조건 분기와의 관계

```csharp
int x = 5;
if (x == 0)  // 내부적으로
{
    // ...
}
```

컴파일 후

```
CMP RAX, 0    // RAX - 0 연산 (결과는 버리고 Flag만 업데이트)
              // RAX = 5이므로 결과 = 5 ≠ 0 → ZF = 0
JE  label     // Jump if Equal — ZF == 1이면 점프
              // ZF = 0이므로 점프 안 함 → if 블록 실행 안 함
```

## Zero Flag 동작 예시

```
5 - 5 = 0  → ZF = 1  (결과가 0)
5 - 3 = 2  → ZF = 0  (결과가 0이 아님)
```

## Carry Flag vs Overflow Flag

```
부호 없는(Unsigned) 오버플로우 → Carry Flag
  예) 255 + 1 = 256 (8bit 범위 초과) → CF = 1

부호 있는(Signed) 오버플로우 → Overflow Flag
  예) 127 + 1 = -128 (8bit 부호 있는 범위 초과) → OF = 1
```

## 게임 개발 관점에서

- 직접 다룰 일 없음 — 컴파일러와 CPU가 자동 관리
- **정수 오버플로우 버그**: `int`의 최대값(2,147,483,647)에 1을 더하면 음수가 됨. 내부적으로 OF가 1이 되는 상황. 게임에서 점수, 데미지가 int 범위를 초과할 때 발생하는 버그와 연결
- **C# checked 키워드**: 정수 오버플로우를 명시적으로 감지하는 문법. 내부적으로 OF를 확인해 예외를 던짐

```csharp
  checked
  {
      int max = int.MaxValue;
      int result = max + 1;  // OverflowException 발생
  }
```

## 의문점 / 더 알아볼 것

- Direction Flag(DF) — 문자열 연산 방향을 제어하는 플래그
- x86의 EFLAGS/RFLAGS 레지스터 전체 비트 구조
