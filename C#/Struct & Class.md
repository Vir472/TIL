# TIL - C# struct vs class

## 핵심 개념

- 둘 다 데이터와 메서드를 묶는 사용자 정의 타입
- 가장 큰 차이는 [Value & Reference](Value & Reference.md) 참고
- 언제 뭘 쓸지 판단하는 기준이 중요

## 문법 비교

```csharp
// struct
struct Point
{
    public int x;
    public int y;

    public Point(int x, int y)  // 생성자 가능
    {
        this.x = x;
        this.y = y;
    }

    public float Distance() => MathF.Sqrt(x * x + y * y);  // 메서드 가능
}

// class
class Enemy
{
    public int hp;
    public string name;

    public Enemy(int hp, string name)
    {
        this.hp = hp;
        this.name = name;
    }

    public void TakeDamage(int dmg) => hp -= dmg;
}
```

## 핵심 차이 정리

| 항목               | struct            | class           |
| ------------------ | ----------------- | --------------- |
| 타입 분류          | 값 타입           | 참조 타입       |
| 메모리 위치        | Stack (기본)      | Heap            |
| 복사 시            | 값 전체 복사      | 주소(참조) 복사 |
| null 가능          | 기본 불가         | 가능            |
| 상속               | 불가              | 가능            |
| 인터페이스 구현    | 가능              | 가능            |
| 기본 생성자        | C# 10 이전 불가   | 가능            |
| GC 대상            | 아님 (Stack 해제) | 대상            |
| 소멸자(Destructor) | 불가              | 가능            |

## 상속 불가 — struct의 중요한 제약

```csharp
struct Animal { }
struct Dog : Animal { }  // 컴파일 에러 — struct는 상속 불가

class Animal { }
class Dog : Animal { }   // OK
```

단, 인터페이스 구현은 struct도 가능

```csharp
interface IDamageable { void TakeDamage(int dmg); }

struct Obstacle : IDamageable  // OK
{
    public int durability;
    public void TakeDamage(int dmg) => durability -= dmg;
}
```

## 복사 동작 차이 — 실수하기 쉬운 부분

```csharp
// struct — 독립적인 복사본
struct HpData { public int value; }

HpData a = new HpData { value = 100 };
HpData b = a;
b.value = 0;

Console.WriteLine(a.value);  // 100 — 원본 그대로
Console.WriteLine(b.value);  // 0

// class — 같은 객체 공유
class HpData { public int value; }

HpData a = new HpData { value = 100 };
HpData b = a;
b.value = 0;

Console.WriteLine(a.value);  // 0 — 원본도 바뀜!
Console.WriteLine(b.value);  // 0
```

## C# 버전별 struct 변화

```
C# 6  이전 : 필드 초기값 지정 불가, 매개변수 없는 생성자 불가
C# 10      : 필드 초기값 지정 가능, 매개변수 없는 생성자 가능
C# 11      : ref struct 개선, required 멤버 지원
```

```csharp
// C# 10 이후 가능
struct Point
{
    public int x = 0;   // 초기값 지정 가능
    public int y = 0;

    public Point() { }  // 매개변수 없는 생성자 가능
}
```

## readonly struct

불변 struct — 모든 필드가 읽기 전용

```csharp
readonly struct Vector2
{
    public readonly float x;
    public readonly float y;

    public Vector2(float x, float y)
    {
        this.x = x;
        this.y = y;
    }

    // readonly struct의 메서드는 자동으로 readonly
    public float Magnitude() => MathF.Sqrt(x * x + y * y);
}
```

- 불변이 보장되므로 컴파일러가 방어적 복사(defensive copy)를 생략해 성능 최적화 가능
- Unity의 Vector2, Vector3가 readonly struct로 구현된 이유

## ref struct

Stack에만 존재할 수 있는 struct. Heap에 올라가는 것 자체를 금지

```csharp
ref struct StackOnlyData
{
    public int value;
}

// Heap에 올릴 수 없음
class Wrapper
{
    StackOnlyData data;  // 컴파일 에러
}
```

- `Span<T>`, `ReadOnlySpan<T>`가 ref struct로 구현됨
- GC 압박 없이 메모리를 다루기 위한 고성능 패턴에 활용

## 언제 struct를 쓰고 언제 class를 쓰냐면

**struct가 적합한 경우**

- 크기가 작은 데이터 (일반적으로 16바이트 이하)
- 불변(Immutable) 데이터
- 값 의미론이 자연스러운 경우 (좌표, 색상, 범위 등)
- GC 부담을 줄여야 하는 성능 크리티컬한 코드
  **class가 적합한 경우**
- 상속이 필요한 경우
- 크기가 크거나 필드가 많은 경우
- null을 표현해야 하는 경우
- 동일 객체를 여러 곳에서 공유해야 하는 경우 (참조 의미론)
- 소멸자나 복잡한 리소스 관리가 필요한 경우

## 게임 개발 관점에서

- **Unity에서 struct 활용 예시**: `Vector2`, `Vector3`, `Quaternion`, `Color`, `Rect` 모두 struct. 자주 생성/소멸되는 데이터를 GC 없이 처리하기 위함
- **class를 써야 하는 Unity 타입**: `MonoBehaviour`, `ScriptableObject`, `GameObject` 등 Unity 오브젝트 계층 구조에 속하는 건 모두 class — 상속 구조가 필수이기 때문
- **struct 남용 주의**: struct가 크면 복사 비용이 오히려 class보다 높아짐. 필드 4~5개 이상이면 class를 고려

## 의문점 / 더 알아볼 것

- `record struct` vs `record class` (C# 10) — 불변 데이터 타입의 새로운 선택지
- `Span<T>`가 ref struct인 이유와 실제 활용 패턴
- Unity DOTS에서 struct 중심 설계(ECS)가 캐시 효율에 미치는 영향
