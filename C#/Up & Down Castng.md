# 업캐스팅 / 다운캐스팅

## 핵심 개념

- **업캐스팅(Upcasting)**: 자식 타입 → 부모 타입으로 변환
- **다운캐스팅(Downcasting)**: 부모 타입 → 자식 타입으로 변환
- 타입 계층 구조에서 위아래로 이동하는 것

```
Character (부모)
├─ Warrior (자식)
└─ Mage (자식)

업캐스팅 ↑  Warrior → Character
다운캐스팅 ↓ Character → Warrior
```

## 업캐스팅 (Upcasting)

### 특징

- 자식을 부모 타입으로 참조
- **암묵적으로 가능** — 별도 캐스팅 문법 불필요
- 항상 안전 — 실패할 가능성 없음
- 부모 타입으로 참조하면 부모에 정의된 멤버만 접근 가능

```csharp
Warrior warrior = new Warrior();
Character c = warrior;  // 업캐스팅 — 자동으로 변환
c.Attack();             // virtual/override면 Warrior의 Attack 호출 (다형성)
c.UseShield();          // 컴파일 에러 — Character에 없는 멤버
```

### 활용 — 다형성의 핵심

```csharp
// 다양한 자식 타입을 부모 타입 하나로 관리
List<Character> party = new List<Character>
{
    new Warrior(),
    new Mage(),
    new Archer()
};

foreach (var c in party)
    c.Attack();  // 각자 다른 Attack() 호출 — 다형성
```

## 다운캐스팅 (Downcasting)

### 특징

- 부모 타입을 자식 타입으로 변환
- **명시적 캐스팅 필요**
- 실패 가능 — 실제 타입이 다르면 런타임 에러 발생

```csharp
Character c = new Warrior();  // 업캐스팅

Warrior w = (Warrior)c;  // 다운캐스팅 — 명시적 캐스팅
w.UseShield();           // Warrior 전용 멤버 접근 가능

Mage m = (Mage)c;  // 런타임 에러! — 실제 타입이 Warrior이므로
```

## 안전한 다운캐스팅 방법

### as 연산자

- 변환 실패 시 예외 대신 **null 반환**
- 참조 타입에만 사용 가능

```csharp
Character c = new Warrior();

Warrior w = c as Warrior;  // 성공 → Warrior 반환
Mage m = c as Mage;        // 실패 → null 반환 (예외 없음)

if (w != null)
    w.UseShield();
```

### is 연산자

- 특정 타입인지 bool로 확인

```csharp
if (c is Warrior)
    Console.WriteLine("전사입니다");
```

### is 패턴 매칭 (C# 7.0+)

- 타입 확인과 변수 할당을 한 번에

```csharp
if (c is Warrior w)
{
    w.UseShield();  // 여기서 w는 Warrior 타입으로 사용 가능
}
```

### switch 패턴 매칭

```csharp
switch (c)
{
    case Warrior w:
        w.UseShield();
        break;
    case Mage m:
        m.CastSpell();
        break;
    default:
        c.Attack();
        break;
}
```

## as vs (T) 직접 캐스팅 비교

| 항목         | (T) 직접 캐스팅             | as 연산자           |
| ------------ | --------------------------- | ------------------- |
| 실패 시      | InvalidCastException 예외   | null 반환           |
| 성능         | 약간 빠름                   | 약간 느림           |
| 값 타입 사용 | 가능                        | 불가 (참조 타입만)  |
| 권장 상황    | 반드시 성공한다고 확신할 때 | 실패 가능성 있을 때 |

## 인터페이스와 캐스팅

인터페이스도 동일하게 업캐스팅/다운캐스팅 적용됨

```csharp
Warrior w = new Warrior();
IDamageable d = w;          // 업캐스팅 (인터페이스로)
Warrior w2 = d as Warrior;  // 다운캐스팅
```

## 게임 개발 관점에서

- **충돌 처리**: OnTriggerEnter에서 충돌한 오브젝트가 특정 타입인지 확인할 때 is/as 패턴이 핵심

```
  충돌한 오브젝트 as IDamageable → null이 아니면 TakeDamage 호출
```

- **GetComponent와 조합**: GetComponent가 반환한 컴포넌트를 더 구체적인 타입으로 다운캐스팅해서 전용 기능 사용
- **다형성 리스트 관리**: `List<CombatUnit>`으로 아군/적을 함께 관리하다가 특정 연산이 필요할 때 is/as로 구체적인 타입 접근

## 의문점 / 더 알아볼 것

- 박싱/언박싱 — 값 타입과 object 간의 업캐스팅/다운캐스팅
- 공변성/반공변성(out/in) — 제네릭 타입 간의 캐스팅 허용 규칙
