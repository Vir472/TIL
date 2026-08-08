# TIL - C# virtual / abstract / override

## 다형성 (Polymorphism)

- 객체지향의 4대 원칙(캡슐화, 상속, 다형성, 추상화) 중 하나
- **"같은 타입으로 참조해도 실제 타입에 따라 다르게 동작한다"**
- 구체적으로는 런타임에 어떤 메서드를 호출할지 결정되는 **런타임 다형성**

```csharp
// 부모 타입 하나로 여러 자식을 다룰 수 있음
List<Character> units = new List<Character> { new Warrior(), new Mage(), new Boss() };

foreach (var unit in units)
{
    unit.Attack();  // 각자 다른 Attack()이 호출됨 — 다형성
}
```

세 키워드의 역할

- `virtual` — 다형성을 **허용**
- `abstract` — 다형성을 **강제**
- `override` — 다형성을 **실현**

## 핵심 개념

세 키워드 모두 **다형성(Polymorphism)** 을 구현하기 위한 도구

- `virtual` — 부모가 "자식이 바꿔도 돼"라고 허용
- `override` — 자식이 "부모 것 대신 내 걸로 쓸게"라고 재정의
- `abstract` — 부모가 "자식이 반드시 구현해야 해"라고 강제

## virtual

```csharp
class Character
{
    public virtual void Attack()  // 자식이 재정의 가능
    {
        Console.WriteLine("기본 공격");
    }
}

class Warrior : Character
{
    public override void Attack()  // 재정의 선택 가능
    {
        Console.WriteLine("검 공격");
    }
}

class Mage : Character
{
    // override 안 하면 부모 메서드 그대로 사용
}
```

- 기본 구현이 있음 — 자식이 override 안 해도 부모 것이 호출됨
- override는 선택사항

## override

```csharp
class Boss : Character
{
    public override void Attack()
    {
        base.Attack();              // 부모 메서드 호출 가능
        Console.WriteLine("추가 패턴");
    }
}
```

- `base.메서드()` 로 부모 구현을 재사용하면서 확장 가능
- `virtual` 또는 `abstract`가 붙은 메서드만 override 가능
- 반환 타입과 시그니처가 완전히 일치해야 함

## abstract

```csharp
abstract class CombatUnit  // abstract 클래스 — 인스턴스화 불가
{
    public abstract void TakeTurn();  // 구현 없음 — 자식이 반드시 구현해야 함

    public void TakeDamage(int dmg)   // 일반 메서드는 그대로 상속
    {
        hp -= dmg;
    }
}

class Sinner : CombatUnit
{
    public override void TakeTurn()  // 반드시 구현해야 함 — 안 하면 컴파일 에러
    {
        Console.WriteLine("스킬 선택");
    }
}

CombatUnit unit = new CombatUnit();  // 컴파일 에러 — abstract 클래스 인스턴스화 불가
CombatUnit unit = new Sinner();      // OK — 자식 클래스는 가능
```

## 세 키워드 비교

| 항목        | virtual | abstract          | override    |
| ----------- | ------- | ----------------- | ----------- |
| 기본 구현   | 있음    | 없음              | 자식이 작성 |
| 자식의 구현 | 선택    | 필수              | -           |
| 클래스 제한 | 없음    | abstract 클래스만 | -           |
| 인스턴스화  | 가능    | 불가              | -           |
| base 호출   | -       | 불가 (구현 없음)  | 가능        |

## 다형성 동작 — 중요

```csharp
Character c1 = new Warrior();
Character c2 = new Mage();
Character c3 = new Boss();

// 부모 타입으로 참조해도 실제 타입의 메서드 호출
c1.Attack();  // "검 공격"
c2.Attack();  // "기본 공격" (override 안 했으므로)
c3.Attack();  // "기본 공격" + "추가 패턴"
```

virtual/override 없이 `new` 키워드로 숨기면 다형성 동작 안 함

## sealed override — 더 이상 재정의 금지

```csharp
class Boss : Character
{
    public sealed override void Attack()  // 이 클래스 이하에서 재정의 불가
    {
        Console.WriteLine("보스 공격");
    }
}

class FinalBoss : Boss
{
    public override void Attack() { }  // 컴파일 에러
}
```
