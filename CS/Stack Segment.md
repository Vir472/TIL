# TIL - 스택 영역 (Stack Segment)

## 핵심 개념

- 프로세스 메모리의 4가지 영역(Code, Data, Heap, Stack) 중 하나
- **지역변수**와 **함수 호출 정보**가 저장되는 공간
- LIFO(Last In, First Out) 구조 — 마지막에 들어온 게 먼저 나감
- 컴파일 타임에 크기가 정해져 있어서 할당/해제가 매우 빠름

## 저장되는 것들

```csharp
void Attack()
{
    int damage = 10;        // 지역변수 → Stack
    float multiplier = 1.5f; // 지역변수 → Stack
    // Attack() 종료 시 위 변수들 자동 소멸
}
```

- 지역변수 (값 타입)
- 함수의 리턴 주소 (함수 끝나고 어디로 돌아갈지)
- 매개변수
- 참조 타입 변수의 **주소값** (객체 자체는 Heap, 주소는 Stack)

## 동작 방식s

```
함수 호출 시 → Stack Frame 쌓임
함수 종료 시 → Stack Frame 제거 (Stack Pointer만 이동)

[Main()]
[Attack()]      ← 현재 실행 중
[CalculateDmg()]
```

중간에 구멍이 생길 수 없는 구조라 단편화 발생 안 함 (→ [[힙 영역]] 참고)

## Stack Overflow

재귀 함수가 종료 조건 없이 계속 호출되면 Stack이 가득 참 → 크래시

```csharp
void Infinite()
{
    Infinite(); // 종료 조건 없는 재귀 → Stack Overflow
}
```

## 게임 개발 관점에서

- **빠르다**: Heap 할당과 달리 Stack은 포인터 이동만으로 할당/해제 → GC 부담 없음
- **struct가 Stack에 저장되는 이유**: Unity의 Vector3, Color 같은 자주 쓰는 타입이 struct인 이유. GC 없이 빠르게 쓰고 버릴 수 있음
- **깊은 재귀 주의**: 복잡한 트리 탐색, 재귀 알고리즘 구현 시 Stack Overflow 위험. 게임 AI에서 재귀 깊이 제한 필요

## 의문점 / 더 알아볼 것

- Unity Job System의 워커 스레드는 각자 별도의 Stack을 가지는지
- Stack 크기는 OS/플랫폼마다 다른지 (모바일 vs PC)
