# 상속 (Inheritance)

## 핵심 개념

- 기존 클래스(부모/기반 클래스)의 멤버를 새 클래스(자식/파생 클래스)가 물려받는 것
- 코드 재사용 + 계층 구조 표현
- C#은 **단일 상속만 지원** — 클래스는 하나의 부모만 가질 수 있음 (인터페이스는 여러 개 가능)

## 기본 문법

```csharp
// 부모 클래스
class Character
{
    public string name;
    public int hp;

    public Character(string name, int hp)
    {
        this.name = name;
        this.hp = hp;
    }

    public void TakeDamage(int dmg)
    {
        hp -= dmg;
        Console.WriteLine($"{name}이 {dmg} 피해를 받음. 남은 HP: {hp}");
    }
}

// 자식 클래스 — Character를 상속
class Enemy : Character
{
    public int reward;

    public Enemy(string name, int hp, int reward)
        : base(name, hp)  // 부모 생성자 호출
    {
        this.reward = reward;
    }

    public void DropLoot()
    {
        Console.WriteLine($"{reward} 골드 드롭");
    }
}

// 사용
Enemy slime = new Enemy("슬라임", 30, 10);
slime.TakeDamage(10);  // 부모 메서드 사용 가능
slime.DropLoot();      // 자식 메서드
```

## base 키워드

부모 클래스의 멤버에 접근할 때 사용

```csharp
class Boss : Enemy
{
    public Boss(string name, int hp, int reward)
        : base(name, hp, reward)  // Enemy 생성자 호출
    { }

    // 부모 메서드를 확장
    public new void TakeDamage(int dmg)
    {
        Console.WriteLine("보스는 피해 감소!");
        base.TakeDamage(dmg / 2);  // 부모의 TakeDamage 호출
    }
}
```

## 접근 제한자와 상속

```csharp
class Parent
{
    public int a;       // 어디서든 접근 가능
    protected int b;    // 자식 클래스까지 접근 가능
    private int c;      // 자식 클래스도 접근 불가
    internal int d;     // 같은 어셈블리 내 접근 가능
}

class Child : Parent
{
    void Test()
    {
        a = 1;  // OK
        b = 2;  // OK — protected는 자식까지 허용
        c = 3;  // 컴파일 에러 — private은 자식도 불가
    }
}
```

## virtual / override — 다형성의 핵심

부모의 메서드를 자식이 재정의할 때

```csharp
class Character
{
    public virtual void Attack()  // virtual — 자식이 재정의 가능
    {
        Console.WriteLine("기본 공격");
    }
}

class Warrior : Character
{
    public override void Attack()  // override — 재정의
    {
        Console.WriteLine("검으로 베기");
    }
}

class Mage : Character
{
    public override void Attack()
    {
        Console.WriteLine("마법 발사");
    }
}

// 다형성 — 부모 타입으로 자식을 다룰 수 있음
Character c1 = new Warrior();
Character c2 = new Mage();

c1.Attack();  // "검으로 베기" — 실제 타입 기준으로 호출
c2.Attack();  // "마법 발사"
```

## virtual vs new — 혼동 주의

```csharp
class Parent
{
    public virtual void Speak() => Console.WriteLine("Parent");
}

class Child : Parent
{
    // override — 다형성 동작 (권장)
    public override void Speak() => Console.WriteLine("Child(override)");
}

class Child2 : Parent
{
    // new — 부모 메서드를 숨김 (hiding), 다형성 동작 안 함
    public new void Speak() => Console.WriteLine("Child2(new)");
}

Parent p1 = new Child();
Parent p2 = new Child2();

p1.Speak();  // "Child(override)" — 자식 메서드 호출
p2.Speak();  // "Parent" — new는 부모 타입으로 참조 시 부모 메서드 호출됨
```

- `override`: 런타임에 실제 타입 기준으로 메서드 결정 (가상 디스패치)
- `new`: 컴파일 타임에 참조 타입 기준으로 메서드 결정 (메서드 숨기기)

## sealed — 상속/오버라이드 금지

```csharp
// sealed 클래스 — 더 이상 상속 불가
sealed class FinalBoss : Enemy
{
    // ...
}

class SuperBoss : FinalBoss { }  // 컴파일 에러

// sealed 메서드 — 더 이상 오버라이드 불가
class Boss : Enemy
{
    public sealed override void TakeDamage(int dmg)
    {
        // ...
    }
}
```

## 게임 개발 관점에서

- **Unity MonoBehaviour 상속 구조**:

```
  Object
    └─ Component
         └─ Behaviour
              └─ MonoBehaviour  ← 우리가 상속받는 클래스
                   └─ PlayerController, EnemyAI, GameManager ...
```

`Update()`, `Start()` 등이 virtual로 정의되어 있어 override 없이도 동작하는 이유

- **상속보다 컴포지션(Composition)**: 깊은 상속 계층은 유지보수가 어려워짐. Unity가 컴포넌트 방식을 쓰는 이유도 상속 대신 조합을 선호하기 때문. 실무에서는 3단계 이상 상속은 지양하는 편

## 의문점 / 더 알아볼 것

- C#의 다형성이 내부적으로 vtable(가상 함수 테이블)로 구현되는 방식
- 인터페이스 기본 구현(C# 8+)이 추상 클래스와 겹치는 부분
- Unity의 `[RequireComponent]`, `[AddComponentMenu]` 어트리뷰트와 상속의 관계
