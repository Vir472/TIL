# TIL - C# Nullable

## 관련 노트

→ [[TIL-ValueType-vs-ReferenceType]] — 값 타입/참조 타입과 null
→ [[csharp-static]] — static과 null 초기화

## 핵심 개념

- **Nullable**: 원래 null이 될 수 없는 **값 타입**을 null 가능하게 만드는 기능
- 참조 타입은 기본적으로 null 가능 — Nullable이 필요 없음
- 값 타입은 기본적으로 null 불가 → `?` 키워드로 null 허용

## 왜 필요한가

"값이 없음"을 표현해야 할 때 사용

```csharp
int hp;         // 반드시 어떤 값이 있어야 함. "없음" 표현 불가
int? hp = null; // "아직 HP가 설정되지 않았음"을 표현 가능
```

DB에서 데이터를 가져올 때 레코드가 없거나, 게임 세이브 데이터가 없을 때처럼 "값이 존재하지 않는 상태"를 명확하게 표현하는 용도

## 문법

`T?`는 `Nullable<T>`의 축약 문법 — 완전히 동일

```csharp
int?     nullableInt    = null;
float?   nullableFloat  = null;
bool?    nullableBool   = null;
```

## 메모리 구조

```csharp
int  x = 10;   // 4바이트 (값만)
int? y = 10;   // 5바이트 (값 4바이트 + HasValue 1바이트)
```

내부적으로 `bool hasValue`와 `T value` 두 필드로 구성된 struct
null이면 hasValue = false, 값이 있으면 hasValue = true

## 주요 멤버

### HasValue / Value

- `HasValue` — null 여부를 bool로 반환
- `Value` — 실제 값 반환. **null인데 접근하면 예외 발생**
- `GetValueOrDefault(기본값)` — null이면 기본값 반환, 값이 있으면 그 값 반환
  Value에 접근하기 전에 반드시 HasValue 확인 필요

## 연산자

| 연산자 | 이름             | 설명                                                 |
| ------ | ---------------- | ---------------------------------------------------- |
| `??`   | null 병합        | 왼쪽이 null이면 오른쪽 값 사용. `score ?? 0`         |
| `??=`  | null 병합 할당   | null이면 오른쪽 값으로 초기화. `score ??= 0`         |
| `?.`   | null 조건        | null이면 호출 자체를 건너뜀. `enemy?.TakeDamage(10)` |
| `?[]`  | null 조건 인덱서 | null이면 인덱서 접근을 건너뜀. `arr?[0]`             |

`?.`는 체이닝 가능 — `enemy?.Status?.Hp` (중간에 null이면 전체가 null)

## Nullable 참조 타입 (C# 8.0+)

C# 8.0부터 참조 타입에도 null 안전성 분석 도입

- 프로젝트에서 활성화하면 참조 타입도 기본적으로 null 불가로 취급
- `string?`처럼 명시해야 null 가능
- 컴파일러가 null 역참조 가능성을 정적 분석으로 경고 → 런타임 NullReferenceException을 컴파일 타임에 잡을 수 있음

## null 체크 패턴

```csharp
// 전통적 방식
if (enemy != null) { ... }

// is 패턴 (C# 7.0+) — 더 명확한 의도 표현
if (enemy is not null) { ... }

// switch 패턴
string message = score switch
{
    null   => "기록 없음",
    < 50   => "낮음",
    >= 50  => "높음"
};
```

## 게임 개발 관점에서

- **NullReferenceException 방지**: Unity 개발에서 가장 흔한 런타임 에러. `?.`와 `??`를 적극 활용하면 방어 코드를 간결하게 작성 가능
- **세이브 데이터**: 플레이 기록이 없을 때 `int? lastStage = null`로 표현하고 `lastStage ?? 1`로 기본값 처리
- **선택적 컴포넌트**: `GetComponent<Rigidbody>()`가 없으면 null 반환 → `rb?.AddForce(...)` 패턴으로 안전하게 처리
- **Unity 권장 패턴**: null 체크보다 `TryGetComponent<T>(out var comp)`를 선호 — 컴포넌트 존재 여부와 참조를 한 번에 처리

## 의문점 / 더 알아볼 것

- `Nullable<T>`가 박싱될 때 null이면 null로, 값이 있으면 값 자체로 박싱되는 특수 동작
- C# 8.0 Nullable Reference Type과 Unity 버전별 지원 현황
- `[NotNull]`, `[MaybeNull]` 어트리뷰트 — 정적 분석 힌트
