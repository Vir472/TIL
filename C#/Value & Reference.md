# 값 타입 vs 참조 타입 (Value Type vs Reference Type)

## 핵심 개념

- C#의 모든 타입은 **값 타입(Value Type)** 또는 **참조 타입(Reference Type)** 중 하나
- 가장 큰 차이는 **"변수에 뭐가 저장되느냐"**
  - 값 타입: 변수 자체에 **데이터(값)** 가 저장됨
  - 참조 타입: 변수에 **Heap에 있는 객체의 주소(참조)** 가 저장됨

  ## 값 타입 (Value Type)

### 해당하는 타입들

```csharp
// 기본 내장 타입
int, float, double, bool, char, byte, long, short

// struct
struct Vector2 { public float x, y; }

// enum
enum Direction { Up, Down, Left, Right }
```

### 특징 — 복사 시 값 자체가 복사됨

```csharp
int a = 10;
int b = a;   // a의 값이 복사됨
b = 99;

Console.WriteLine(a);  // 10 — a는 그대로
Console.WriteLine(b);  // 99 — b만 바뀜
```

```csharp
struct Point
{
    public int x, y;
}

Point p1 = new Point { x = 1, y = 2 };
Point p2 = p1;   // 값 전체가 복사됨
p2.x = 99;

Console.WriteLine(p1.x);  // 1 — p1은 그대로
Console.WriteLine(p2.x);  // 99
```

## 참조 타입 (Reference Type)

### 해당하는 타입들

```csharp
// class
class Enemy { public int hp; }

// 배열 (내용물이 값 타입이어도 배열 자체는 참조 타입)
int[] arr = new int[5];

// string (특수한 참조 타입 — 아래 별도 설명)
string name = "Player";

// delegate, interface, object
```

### 특징 — 복사 시 주소(참조)가 복사됨

```csharp
class Enemy
{
    public int hp;
}

Enemy e1 = new Enemy { hp = 100 };
Enemy e2 = e1;   // 주소가 복사됨 — 같은 객체를 가리킴

e2.hp = 0;

Console.WriteLine(e1.hp);  // 0 — 같은 객체이므로 e1도 바뀜
Console.WriteLine(e2.hp);  // 0
```

같은 Heap 객체를 두 변수가 가리키는 구조이기 때문에 한쪽을 수정하면 양쪽 다 바뀜

## null의 의미

- 참조 타입만 `null`이 될 수 있음 → "아무 객체도 가리키지 않는다"는 뜻
- 값 타입은 기본적으로 `null` 불가 (항상 어떤 값이 있어야 함)

```csharp
Enemy e = null;   // OK — 참조 타입
int x = null;     // 컴파일 에러 — 값 타입

// 값 타입을 null 가능하게 하려면 Nullable 사용
int? x = null;    // OK — Nullable<int>
```

## boxing / unboxing

값 타입을 참조 타입처럼 다뤄야 할 때 발생하는 변환

```csharp
int a = 42;
object obj = a;   // boxing — 값 타입을 Heap에 올림 (object는 참조 타입)
int b = (int)obj; // unboxing — 다시 값 타입으로 꺼냄

// 왜 문제냐면 — boxing은 Heap 할당을 만들어냄
// GC 부담 증가
```

```csharp
// 흔한 boxing 실수
ArrayList list = new ArrayList();
list.Add(42);     // int → object로 boxing 발생
// → List<int> 같은 제네릭 컬렉션을 써야 boxing 없음
```

## string이 특수한 이유

`string`은 참조 타입이지만 **불변(Immutable)**이라 값 타입처럼 동작하는 것처럼 보임

```csharp
string s1 = "hello";
string s2 = s1;
s2 = "world";   // s2에 새 문자열 객체 할당

Console.WriteLine(s1);  // "hello" — s1은 그대로
Console.WriteLine(s2);  // "world"

// Enemy처럼 내부 필드를 수정하는 게 아니라
// 아예 새 문자열 객체가 만들어지기 때문
```

그래서 문자열 연결(`+`)을 반복하면 매번 새 객체가 Heap에 생성됨 → `StringBuilder` 쓰는 이유

## 메서드 파라미터로 전달할 때

```csharp
// 값 타입 전달 — 복사본이 전달됨
void AddOne(int x) { x += 1; }

int n = 5;
AddOne(n);
Console.WriteLine(n);  // 5 — 원본 그대로

// 참조 타입 전달 — 같은 객체의 주소가 전달됨
void KillEnemy(Enemy e) { e.hp = 0; }

Enemy enemy = new Enemy { hp = 100 };
KillEnemy(enemy);
Console.WriteLine(enemy.hp);  // 0 — 원본이 바뀜
```

### ref / out 키워드

값 타입을 원본 그대로 전달하고 싶을 때

```csharp
// ref — 값 타입도 참조로 전달 (읽기/쓰기 가능, 초기화 필요)
void AddOne(ref int x) { x += 1; }

int n = 5;
AddOne(ref n);
Console.WriteLine(n);  // 6 — 원본이 바뀜

// out — 메서드 안에서 반드시 값을 할당해야 함 (초기화 불필요)
bool TryParse(string s, out int result)
{
    // result에 반드시 값 할당
    result = int.Parse(s);
    return true;
}
```

## 게임 개발 관점에서

- **Vector2, Vector3은 struct (값 타입)**: Unity의 Vector3는 struct로 구현되어 있음. 함수에 전달할 때마다 복사가 일어나지만, 작은 크기라 성능 영향 미미. 오히려 Heap 할당 없어서 GC 부담이 없음
- **Component는 class (참조 타입)**: `GetComponent<Rigidbody>()`로 가져온 컴포넌트는 참조 타입. 변수에 캐싱해두면 같은 객체를 계속 참조하는 것

```csharp
// 나쁜 패턴 — 매 프레임 GetComponent 호출 (검색 비용 + 참조 타입 반환)
void Update()
{
    GetComponent<Rigidbody>().velocity = Vector3.zero;
}

// 좋은 패턴 — Start에서 캐싱
Rigidbody rb;
void Start() { rb = GetComponent<Rigidbody>(); }
void Update() { rb.velocity = Vector3.zero; }
```

- **struct 남용 주의**: struct는 전달 시 복사가 일어나므로, 필드가 많은 큰 구조체를 자주 전달하면 오히려 복사 비용이 class보다 높아질 수 있음. 일반적으로 **16바이트 이하의 작은 데이터**에 struct가 적합

## 요약

|             | 값 타입                     | 참조 타입                            |
| ----------- | --------------------------- | ------------------------------------ |
| 저장 위치   | Stack                       | Heap (주소는 Stack)                  |
| 복사 시     | 값 자체 복사                | 주소 복사 (같은 객체 공유)           |
| null 가능   | 기본 불가 (`?`로 가능)      | 가능                                 |
| 대표 키워드 | `struct`, `enum`, 기본 타입 | `class`, `interface`, 배열, `string` |
| GC 대상     | 아님                        | 대상                                 |

## 의문점 / 더 알아볼 것

- C# 9.0+의 `record struct` / `record class`는 어떻게 다른지
- `in` 키워드 (읽기 전용 ref) — 큰 struct를 복사 없이 전달할 때 활용
- Unity DOTS(Data-Oriented Technology Stack)에서 struct 중심 설계를 쓰는 이유
