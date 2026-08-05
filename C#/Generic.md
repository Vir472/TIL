# C# Generic (제네릭)

## 핵심 개념

- **타입을 매개변수처럼 받는 기능** — 코드 작성 시점에 타입을 확정하지 않고, 사용 시점에 타입을 지정
- 코드 중복 없이 여러 타입에 대해 동일한 로직을 재사용 가능
- 컴파일 타임에 타입이 확정되므로 타입 안전성 보장

## 왜 필요한가

제네릭 없이 여러 타입을 다루려면

```csharp
// object 사용 — boxing/unboxing 발생, 타입 안전성 없음
ArrayList list = new ArrayList();
list.Add(10);      // int → object로 boxing
int x = (int)list[0];  // 다시 unboxing + 형변환 필요
list.Add("hello"); // 실수로 다른 타입 넣어도 컴파일 에러 없음
```

제네릭 사용

```csharp
List<int> list = new List<int>();
list.Add(10);      // boxing 없음
list.Add("hello"); // 컴파일 에러 — 타입 안전
int x = list[0];   // 형변환 불필요
```

## 기본 문법

### 제네릭 클래스

```csharp
class Stack<T>
{
    private T[] items = new T[100];
    private int top = 0;

    public void Push(T item) => items[top++] = item;
    public T Pop() => items[--top];
}

// 사용
Stack<int> intStack = new Stack<int>();
Stack<string> strStack = new Stack<string>();
```

### 제네릭 메서드

```csharp
T GetFirst<T>(T[] array) => array[0];

int first = GetFirst(new int[] { 1, 2, 3 });       // T = int로 추론
string str = GetFirst(new string[] { "a", "b" });   // T = string으로 추론
```

### 제네릭 인터페이스

```csharp
interface IRepository<T>
{
    T GetById(int id);
    void Save(T item);
}

class EnemyRepository : IRepository<Enemy>
{
    public Enemy GetById(int id) { ... }
    public void Save(Enemy item) { ... }
}
```

## 타입 제약 (where)

T에 들어올 수 있는 타입을 제한하는 키워드

| 제약                   | 의미                                 |
| ---------------------- | ------------------------------------ |
| `where T : class`      | 참조 타입만 허용                     |
| `where T : struct`     | 값 타입만 허용                       |
| `where T : new()`      | 매개변수 없는 생성자 필요            |
| `where T : 부모클래스` | 특정 클래스를 상속한 타입만 허용     |
| `where T : 인터페이스` | 특정 인터페이스를 구현한 타입만 허용 |

```csharp
// T가 CombatUnit을 상속한 타입이어야 함
class BattleSimulator<T> where T : CombatUnit, new()
{
    public T CreateUnit() => new T();  // new() 제약 덕분에 인스턴스 생성 가능
    public void Simulate(T unit) => unit.TakeTurn();  // CombatUnit 메서드 호출 가능
}
```

## 여러 타입 매개변수

```csharp
class Pair<TKey, TValue>
{
    public TKey Key { get; }
    public TValue Value { get; }

    public Pair(TKey key, TValue value)
    {
        Key = key;
        Value = value;
    }
}

var pair = new Pair<string, int>("보스HP", 5000);
```

## 공변성 / 반공변성 (out / in)

인터페이스와 델리게이트에서 타입 계층 간 변환을 허용하는 기능

```csharp
// out — 공변성: 더 구체적인 타입을 더 일반적인 타입으로 사용 가능
IEnumerable<Enemy> enemies = new List<Boss>();  // Boss는 Enemy의 자식

// in — 반공변성: 더 일반적인 타입을 더 구체적인 타입 위치에 사용 가능
Action<Enemy> action = (Enemy e) => { };
Action<Boss> bossAction = action;  // Boss는 Enemy의 자식이므로 허용
```

- `out T` — 반환에만 사용. 읽기 전용
- `in T` — 입력에만 사용. 쓰기 전용

## 게임 개발 관점에서

- **Unity의 GetComponent\<T\>()**: 제네릭 메서드의 대표적 활용. `GetComponent<Rigidbody>()`처럼 타입을 지정해 컴포넌트를 가져옴
- **오브젝트 풀링**: 여러 타입의 오브젝트를 하나의 풀 클래스로 관리할 때 제네릭이 필수

```csharp
  class ObjectPool<T> where T : MonoBehaviour, new()
  {
      private Queue<T> pool = new Queue<T>();
      public T Get() => pool.Count > 0 ? pool.Dequeue() : new T();
      public void Return(T obj) => pool.Enqueue(obj);
  }
```

- **ScriptableObject 제네릭 패턴**: 베이스 ScriptableObject를 제네릭으로 만들어 다양한 데이터 에셋을 일관된 구조로 관리

## 의문점 / 더 알아볼 것

- 제네릭 특수화(Generic Specialization) — .NET은 값 타입마다 별도 코드를 생성하는 방식
- `default(T)` — T의 기본값을 반환하는 키워드 (값 타입이면 0, 참조 타입이면 null)
- C# 11의 `where T : allows ref struct` 제약
