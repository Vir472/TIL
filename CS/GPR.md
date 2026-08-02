# GPR (General Purpose Register)

## 핵심 개념

- **범용 레지스터(General Purpose Register)**: 특정 용도에 국한되지 않고 다양한 목적으로 사용할 수 있는 레지스터
- 산술/논리 연산의 피연산자와 결과를 임시 저장
- 개수가 많을수록 메모리 접근 없이 레지스터 안에서 연산 가능 → 성능 향상

## 아키텍처별 GPR

```
x86-64 (64bit, PC/서버)
├─ RAX, RBX, RCX, RDX  — 기본 범용 레지스터
├─ RSI, RDI            — 원래 문자열/인덱스용, 지금은 범용
├─ R8 ~ R15            — 64bit에서 추가된 레지스터
└─ (총 16개)

ARM64 (모바일, Apple Silicon)
├─ X0 ~ X30            — 64bit 범용 레지스터
└─ (총 31개 — x86보다 많음)
```

ARM이 레지스터가 더 많아서 메모리 접근을 줄이고 성능을 올릴 수 있는 구조

## 동작 예시

```csharp
int a = 10;
int b = 20;
int c = a + b;
```

컴파일 후 내부적으로

```
MOV RAX, 10    // RAX(GPR) ← 10
MOV RBX, 20    // RBX(GPR) ← 20
ADD RAX, RBX   // RAX ← RAX + RBX (결과 30)
MOV [c], RAX   // 메모리(c 변수)에 결과 저장
```

## 함수 호출 규약 (Calling Convention)

GPR은 함수 인자 전달에도 사용됨

```
x86-64 (Windows)
  첫 번째 인자 → RCX
  두 번째 인자 → RDX
  세 번째 인자 → R8
  네 번째 인자 → R9
  나머지 → Stack

x86-64 (Linux/macOS)
  인자 순서 → RDI, RSI, RDX, RCX, R8, R9
```

## 게임 개발 관점에서

- 직접 다룰 일 없음 — 컴파일러가 자동 할당
- **레지스터 압박(Register Pressure)**: 연산이 복잡해서 GPR 수보다 많은 변수를 동시에 다뤄야 하면 일부를 메모리(Stack)에 내보내야 함 → 성능 저하. 컴파일러 최적화가 이를 최소화
- **Unity Burst Compiler**: SIMD 연산을 위해 XMM/YMM 레지스터(GPR의 확장)를 활용해 여러 float을 동시에 처리. Vector3 연산 최적화의 원리

## 의문점 / 더 알아볼 것

- 컴파일러의 레지스터 할당(Register Allocation) 알고리즘
- SIMD 레지스터(XMM, YMM, ZMM)가 GPR과 다른 점
