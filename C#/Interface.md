# TIL - C# Interface (인터페이스)

## 핵심 개념

- **"이 기능을 반드시 구현해야 한다"는 계약(Contract)을 정의하는 타입**
- 구현 없이 시그니처만 선언 (C# 8.0부터 기본 구현 가능)
- 클래스는 하나의 부모만 상속 가능하지만 인터페이스는 **여러 개 구현 가능**

## abstract class vs interface

| 항목        | abstract class        | interface        |
| ----------- | --------------------- | ---------------- |
| 인스턴스화  | 불가                  | 불가             |
| 기본 구현   | 가능                  | C# 8.0+부터 가능 |
| 필드        | 가능                  | 불가             |
| 생성자      | 가능                  | 불가             |
| 접근 제한자 | 지정 가능             | 기본 public      |
| 다중 구현   | 불가 (단일 상속)      | 가능             |
| 용도        | 공통 기능 + 계층 구조 | 기능 계약만 정의 |

## 기본 문법

```csharp
interface IDamageable
{
    void TakeDamage(int damage);  // 반드시 구현해야 함
    int Hp { get; }               // 프로퍼티도 선언 가능
}

class Enemy : IDamageable
{
    public int Hp { get; private set; } = 100;

    public void TakeDamage(int damage)
    {
        Hp -= damage;
    }
}
```

- 인터페이스 이름은 관례상 **I**로 시작
- 구현 클래스는 인터페이스의 모든 멤버를 반드시 구현해야 함
- 구현하지 않으면 컴파일 에러

## 다중 구현

```csharp
interface IDamageable { void TakeDamage(int dmg); }
interface IHealable   { void Heal(int amount); }
interface IMovable    { void Move(Vector2 dir); }

// 여러 인터페이스 동시 구현 가능
class Player : IDamageable, IHealable, IMovable
{
    public void TakeDamage(int dmg) { ... }
    public void Heal(int amount) { ... }
    public void Move(Vector2 dir) { ... }
}
```

C#은 클래스 다중 상속이 불가능하지만 인터페이스는 여러 개 구현 가능

## 인터페이스 타입으로 참조

```csharp
IDamageable target = new Enemy();  // 인터페이스 타입으로 참조
target.TakeDamage(10);             // IDamageable 멤버만 접근 가능

// 다양한 타입을 같은 인터페이스로 다룰 수 있음
List<IDamageable> damageables = new List<IDamageable>
{
    new Enemy(),
    new Boss(),
    new BreakableWall()  // 적이 아니어도 IDamageable이면 같이 관리 가능
};

foreach (var d in damageables)
    d.TakeDamage(10);
```

## 인터페이스 상속

인터페이스끼리도 상속 가능

```csharp
interface IAttackable : IDamageable
{
    void Attack();  // IDamageable의 TakeDamage + 추가로 Attack도 구현 필요
}
```

## C# 8.0+ 기본 구현

인터페이스에 기본 구현을 제공할 수 있음

```csharp
interface ILoggable
{
    void Log(string message) => Console.WriteLine(message);  // 기본 구현
    void LogError(string message);  // 구현 필수
}
```

- 구현 클래스가 override하지 않으면 기본 구현이 사용됨
- abstract class의 virtual 메서드와 비슷하지만 필드를 가질 수 없다는 차이

## 명시적 구현

같은 메서드 이름이 여러 인터페이스에 있을 때 충돌 해결

```csharp
interface IA { void Do(); }
interface IB { void Do(); }

class MyClass : IA, IB
{
    void IA.Do() => Console.WriteLine("A");  // IA 전용
    void IB.Do() => Console.WriteLine("B");  // IB 전용
}

// 사용 시 인터페이스 타입으로 캐스팅해서 접근
IA a = new MyClass();
a.Do();  // "A"
```

## 게임 개발 관점에서

- **GetComponent와 조합**: Unity에서 `GetComponent<IDamageable>()`처럼 인터페이스로 컴포넌트를 가져올 수 있음. 구체적인 타입을 몰라도 기능만으로 접근 가능
- **충돌 처리**: OnTriggerEnter에서 충돌한 오브젝트가 IDamageable을 구현하고 있는지 확인해서 처리하는 패턴이 일반적

```
  충돌한 오브젝트가 IDamageable이면 → TakeDamage 호출
  아니면 → 무시
```

- **느슨한 결합**: 구체적인 클래스 대신 인터페이스에 의존하면 나중에 구현체를 바꿔도 사용하는 쪽 코드를 수정할 필요 없음

## 의문점 / 더 알아볼 것

- `is` / `as` 패턴 — 인터페이스 구현 여부 런타임 체크
- Dependency Injection — 인터페이스 기반 의존성 주입 패턴
- `IComparable<T>`, `IEnumerable<T>` — C# 내장 인터페이스 활용
