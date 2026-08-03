# static 키워드

## 관련 노트

→ [[Data-Segment]] — static이 적재되는 메모리 영역
→ [[ValueType-vs-ReferenceType]] — 값 타입/참조 타입과 메모리

## 핵심 개념

- **static**: 인스턴스가 아닌 **타입 자체**에 속하는 멤버를 선언하는 키워드
- 클래스의 인스턴스를 몇 개 만들어도 static 멤버는 **단 하나만 존재**
- 프로그램 시작 시 메모리에 올라가고 종료 시까지 유지됨

## 메모리 적재 위치

```
프로세스 메모리
┌─────────────────┐
│   Code 영역     │  ← 실행 코드
├─────────────────┤
│   Data 영역     │  ← static 변수/필드
│                 │     프로그램 시작 시 적재
│                 │     종료 시까지 유지
├─────────────────┤
│   Heap 영역     │  ← new로 생성한 객체
│                 │     (static 참조 타입 필드가
│                 │      가리키는 객체는 여기)
├─────────────────┤
│   Stack 영역    │  ← 지역변수, 함수 호출
└─────────────────┘
```

### 값 타입 vs 참조 타입 static 필드

```csharp
class GameManager
{
    static int score = 0;           // int(값 타입) → Data 영역에 직접 저장
    static List<Enemy> enemies;     // List 참조 → Data 영역
                                    // 실제 List 객체 → Heap
}
```

```
Data 영역
├─ score: 0          ← 값 자체가 Data 영역에
└─ enemies: 0x3A10   ← Heap의 주소(참조)만 Data 영역에

Heap 영역
└─ 0x3A10: List<Enemy> 객체
```

## 문법별 정리

### static 필드

```csharp
class Counter
{
    public static int count = 0;  // 모든 인스턴스가 공유
    public int id;                // 인스턴스마다 독립

    public Counter()
    {
        count++;          // 생성할 때마다 공유 카운터 증가
        id = count;
    }
}

Counter a = new Counter();  // count = 1, a.id = 1
Counter b = new Counter();  // count = 2, b.id = 2
Counter c = new Counter();  // count = 3, c.id = 3

Console.WriteLine(Counter.count);  // 3 — 클래스 이름으로 접근
```

### static 메서드

```csharp
class MathHelper
{
    // 인스턴스 없이 호출 가능
    public static int Clamp(int value, int min, int max)
    {
        return Math.Max(min, Math.Min(max, value));
    }
}

int hp = MathHelper.Clamp(150, 0, 100);  // 100
```

- static 메서드 안에서는 **인스턴스 멤버(this) 접근 불가**
- static 멤버끼리만 접근 가능

### static 클래스

```csharp
// 인스턴스 생성 자체가 불가능한 클래스
// 모든 멤버가 반드시 static이어야 함
static class CombatFormulas
{
    public static float CalcDamage(int power, int defense)
        => Math.Max(0, power - defense);

    public static float CalcCoinProb(int speed, int sanity)
        => 0.5f + (speed * 0.05f) + (sanity * 0.01f);
}

// 사용
float dmg = CombatFormulas.CalcDamage(50, 20);  // 30
```

- 유틸리티 함수 모음, 상수 집합에 적합
- 인스턴스 생성 불가 → `new CombatFormulas()` 컴파일 에러

### static 생성자

```csharp
class BossData
{
    public static readonly Dictionary<string, int> PatternTable;

    // 타입이 처음 사용될 때 딱 한 번 자동 호출
    // 매개변수 없음, 접근 제한자 없음
    static BossData()
    {
        PatternTable = new Dictionary<string, int>
        {
            { "공격", 60 },
            { "방어", 30 },
            { "광폭화", 10 }
        };
    }
}
```

- 언제 호출되는지 정확히 보장되지 않음 (타입 첫 사용 전에 자동 호출)
- 예외 발생 시 타입 자체를 사용할 수 없게 됨 → 간단하게만 사용 권장

### static readonly vs const

```csharp
class Constants
{
    // const — 컴파일 타임 상수. Data 영역 아닌 코드에 직접 삽입됨
    public const int MAX_PLAYERS = 4;

    // static readonly — 런타임 상수. Data 영역에 적재
    public static readonly int MAX_ENEMIES = GetMaxEnemies();

    private static int GetMaxEnemies() => 100;
}
```

| 항목           | const              | static readonly          |
| -------------- | ------------------ | ------------------------ |
| 결정 시점      | 컴파일 타임        | 런타임 (static 생성자)   |
| 메모리 위치    | 코드에 직접 삽입   | Data 영역                |
| 변경 가능      | 불가               | static 생성자에서만 가능 |
| 참조 타입 가능 | 불가 (string 제외) | 가능                     |

## 초기화 순서

```csharp
class Example
{
    static int a = 10;         // ① 필드 선언과 동시에 초기화
    static int b;

    static Example()           // ② static 생성자 실행
    {
        b = a * 2;             // b = 20
    }
}
```

**순서**: 필드 초기화 → static 생성자 실행 → 첫 사용

## 주의사항 — 메모리 해제 안 됨

```csharp
static class Cache
{
    // 게임 종료까지 메모리에 남음
    static List<Texture> loadedTextures = new List<Texture>();

    public static void Add(Texture t) => loadedTextures.Add(t);
    // Clear()를 명시적으로 호출하지 않으면 계속 누적
}
```

- static 멤버는 GC 대상이 아님 → 참조하는 객체도 GC가 수거 못 함
- 대용량 데이터를 static으로 들고 있으면 메모리 누수 위험

## 게임 개발 관점에서

- **싱글톤 패턴**: Unity에서 가장 흔한 static 활용

```csharp
  public class GameManager : MonoBehaviour
  {
      public static GameManager Instance { get; private set; }
      // Instance는 Data 영역에 주소 저장
      // 실제 GameManager 객체는 Heap

      private void Awake()
      {
          if (Instance != null) { Destroy(gameObject); return; }
          Instance = this;
          DontDestroyOnLoad(gameObject);
      }
  }

  // 어디서든 접근
  GameManager.Instance.StartGame();
```

- **씬 전환 시 데이터 유지**: static 변수는 씬이 바뀌어도 Data 영역에 그대로 유지됨. `DontDestroyOnLoad` 없이 데이터를 유지하는 가벼운 방법
- **[SerializeField]와 static 주의**: static 필드는 Unity 인스펙터에 표시되지 않음. `[SerializeField]`를 붙여도 무시됨

## 의문점 / 더 알아볼 것

- .NET CLR에서 static 필드가 정확히 어느 메모리 영역에 저장되는지 (AppDomain별 관리)
- `ThreadStatic` 어트리뷰트 — 스레드마다 별도의 static 변수를 갖는 방법
- Unity에서 도메인 리로드(Domain Reload) 시 static 변수가 초기화되는 타이밍
