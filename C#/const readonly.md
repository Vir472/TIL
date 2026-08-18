# TIL - C# const / readonly

## 관련 노트

→ [[csharp-static]] — static readonly와의 관계
→ [[TIL-Data-Segment]] — 상수가 적재되는 메모리 영역

## 핵심 개념

둘 다 **변경 불가능한 값**을 선언하는 키워드지만 결정 시점과 동작 방식이 다름

- `const` — 컴파일 타임에 값이 확정
- `readonly` — 런타임에 값이 확정 (생성자에서 한 번만 할당 가능)

## const

```csharp
class CombatConstants
{
    public const int MAX_SPEED = 45;
    public const float COIN_BASE_PROB = 0.5f;
    public const string VERSION = "1.0.0";
}

// 사용 — 클래스 이름으로 접근
int speed = CombatConstants.MAX_SPEED;
```

- 선언과 동시에 반드시 초기화
- **컴파일 시 값이 코드에 직접 삽입됨** — 별도 메모리 할당 없음
- 기본 타입(int, float, bool, string 등)과 enum만 사용 가능
- 암묵적으로 static — `static const`는 불필요

## readonly

```csharp
class Enemy
{
    public readonly int id;
    public readonly string enemyName;

    public Enemy(int id, string name)
    {
        this.id = id;             // 생성자에서만 할당 가능
        this.enemyName = name;
    }
}

Enemy e = new Enemy(1, "슬라임");
e.id = 2;  // 컴파일 에러 — 생성자 밖에서 수정 불가
```

- 생성자 또는 선언 시점에 초기화
- 런타임에 값이 결정되므로 참조 타입, 복잡한 객체도 사용 가능
- 인스턴스마다 다른 값을 가질 수 있음

## const vs readonly 비교

| 항목               | const                   | readonly           |
| ------------------ | ----------------------- | ------------------ |
| 값 결정 시점       | 컴파일 타임             | 런타임 (생성자)    |
| 메모리             | 코드에 직접 삽입        | Data/Heap 영역     |
| static 여부        | 항상 static             | 인스턴스 or static |
| 사용 가능 타입     | 기본 타입, enum, string | 모든 타입          |
| 인스턴스별 다른 값 | 불가                    | 가능               |
| 초기화 위치        | 선언 시점만             | 선언 or 생성자     |

## static readonly

`const`의 한계를 보완하는 가장 많이 쓰이는 패턴

```csharp
class BossPatternDB
{
    // const 불가 — Dictionary는 기본 타입이 아님
    public static readonly Dictionary<string, int> Patterns
        = new Dictionary<string, int>
        {
            { "공격", 60 },
            { "방어", 30 },
            { "광폭화", 10 }
        };

    // const 가능 — 기본 타입
    public const int MAX_PHASE = 3;
}
```

## const의 주의사항 — 어셈블리 간 버전 문제

```csharp
// Library.dll
public class Config
{
    public const int VERSION = 1;  // 컴파일 시 1이 코드에 삽입됨
}

// Game.exe (Library.dll 참조)
Console.WriteLine(Config.VERSION);  // 빌드 시 1이 직접 삽입됨
```

Library.dll에서 VERSION을 2로 바꿔도 Game.exe를 **재빌드하지 않으면** 여전히 1 출력
→ 외부에 공개되는 값은 `const` 대신 `static readonly` 권장

## 게임 개발 관점에서

- **게임 상수**: 변경될 가능성이 없는 수학 상수나 단순 숫자는 `const`

```csharp
  public const float PI = 3.14159f;
  public const int TILE_SIZE = 32;
```

- **게임 설정값**: 나중에 바뀔 수 있거나 복잡한 타입은 `static readonly`

```csharp
  public static readonly Vector2 GRAVITY = new Vector2(0, -9.8f);
  public static readonly Color DAMAGE_COLOR = Color.red;
```

- **인스턴스 고유 ID**: 생성 시 한 번만 할당되는 값은 `readonly`

```csharp
  class CombatUnit
  {
      public readonly int unitId;
      public CombatUnit(int id) => unitId = id;
  }
```

## 의문점 / 더 알아볼 것

- `const`가 코드에 직접 삽입되는 구체적인 IL 코드 확인 방법
- C# 11의 `required` 키워드와 readonly 조합
- `record` 타입의 불변성과 readonly의 관계
