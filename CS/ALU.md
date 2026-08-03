# 연산장치 (ALU, Arithmetic Logic Unit)

## 핵심 개념

- **산술논리연산장치(Arithmetic Logic Unit)**: 산술 연산과 논리 연산을 수행하는 CPU의 핵심 회로
- 제어장치의 신호를 받아 실제 연산을 처리하는 "계산기"
- 모든 연산의 최종 실행 주체

## ALU가 수행하는 연산

```
산술 연산 (Arithmetic)
├─ 덧셈 (ADD)
├─ 뺄셈 (SUB) — 보수기 + 가산기 조합
├─ 곱셈 (MUL)
└─ 나눗셈 (DIV)

논리 연산 (Logic)
├─ AND
├─ OR
├─ NOT
├─ XOR
└─ 비트 시프트 (<<, >>)

비교 연산
└─ CMP — 두 값을 빼서 결과를 버리고 Flag Register만 업데이트
```

## 동작 흐름

```
① 제어장치 → ALU에 연산 종류 신호 전달
② 레지스터(GPR, AC)에서 피연산자 입력
③ ALU 내부 회로에서 연산 수행
④ 결과 → AC(누산기) 또는 GPR에 저장
⑤ 연산 상태 → Flag Register 업데이트
```

## 게임 개발 관점에서

- 직접 다룰 일 없음 — 모든 연산(+, -, \*, /, &&, ||, &, |)이 결국 ALU로 처리됨
- **정수 vs 부동소수점**: 일반 ALU는 정수 연산 담당. 부동소수점(float, double) 연산은 별도의 **FPU(Floating Point Unit)** 가 처리. 현대 CPU는 ALU와 FPU가 통합됨
- **SIMD ALU**: 현대 CPU의 ALU는 여러 데이터를 한 번에 처리하는 벡터 연산(SIMD) 지원. Unity Burst Compiler가 Vector3 연산을 SIMD로 최적화하는 원리
- **비트 연산 활용**: 게임에서 상태 플래그를 비트로 관리할 때(`status & FLAG_ALIVE`) ALU의 AND 연산이 직접 사용됨

## 의문점 / 더 알아볼 것

- FPU(Floating Point Unit)와 ALU의 차이 — 부동소수점 연산 방식
- SIMD ALU — XMM/YMM 레지스터로 4~8개 float을 동시에 처리하는 구조
