# 박싱 / 언박싱 (Boxing / Unboxing)

## 핵심 개념

- **박싱(Boxing)**: 값 타입 → 참조 타입(object)으로 변환. Heap에 새 객체 생성
- **언박싱(Unboxing)**: 참조 타입(object) → 값 타입으로 변환. 명시적 캐스팅 필요
- 둘 다 **성능 비용**이 발생 — Heap 할당, GC 부담, 타입 검사

## 메모리 관점

### 박싱 발생 시

```
Stack              Heap
┌────────┐         ┌──────────────┐
│ int 42 │  ──▶   │ object { 42 }│  ← Heap에 새로 할당
└────────┘         └──────────────┘

값 타입이 Heap으로 복사되면서 object로 감싸짐
```

### 언박싱 발생 시

```
Heap                    Stack
┌──────────────┐        ┌────────┐
│ object { 42 }│  ──▶  │ int 42 │  ← 값을 꺼내서 Stack에 복사
└──────────────┘        └────────┘

타입 검사 후 값을 꺼내는 과정에서 비용 발생
```

## 박싱이 발생하는 경우

### object 타입에 값 타입 할당

```csharp
int x = 42;
object obj = x;   // 박싱 — Heap 할당 발생
int y = (int)obj; // 언박싱 — 명시적 캐스팅 필요
```

### 제네릭 없는 컬렉션 사용

```csharp
ArrayList list = new ArrayList();
list.Add(10);       // int → object 박싱
list.Add(3.14f);    // float → object 박싱
int x = (int)list[0]; // object → int 언박싱
```

### 인터페이스로 값 타입 참조

```csharp
interface IFoo { void Do(); }
struct MyStruct : IFoo { public void Do() { } }

IFoo foo = new MyStruct(); // 박싱 발생
```

### 문자열 포맷

```csharp
int score = 100;
string s = "점수: " + score;        // 박싱 발생
string s2 = string.Format("{0}", score); // 박싱 발생
string s3 = $"점수: {score}";       // C# 6.0+ 보간 문자열은 내부적으로 최적화
```

## 박싱/언박싱의 비용

```
박싱 1회 비용
├─ Heap 메모리 할당
├─ 값 복사 (Stack → Heap)
└─ GC 추적 대상 추가

언박싱 1회 비용
├─ 타입 검사 (실제 타입이 맞는지 확인)
├─ 값 복사 (Heap → Stack)
└─ 잘못된 타입이면 InvalidCastException
```

가끔 한 번 발생하는 건 무시할 수 있지만, 반복문이나 매 프레임 실행되는 코드에서 반복 발생하면 GC Spike 원인이 됨

## 제네릭으로 박싱 방지

### 제네릭 컬렉션 사용

```
비제네릭 (박싱 발생)
ArrayList  → 내부적으로 object[] 배열
Hashtable  → 키/값 모두 object

제네릭 (박싱 없음)
List<int>             → 내부적으로 int[] 배열
Dictionary<string, int> → 타입 지정으로 박싱 없음
```

제네릭 컬렉션은 컴파일 타임에 타입이 확정되어 있어서 object로 변환할 필요가 없음

### 제네릭 메서드

```csharp
// 박싱 발생 — object로 받음
void Print(object value) => Console.WriteLine(value);
Print(42);      // 박싱

// 박싱 없음 — 제네릭으로 받음
void Print<T>(T value) => Console.WriteLine(value);
Print(42);      // 박싱 없음. T = int로 컴파일 타임에 확정
```

### 제네릭이 박싱을 막는 원리

```
List<int> 내부
  → int[] 배열로 구현
  → int를 그대로 저장. object 변환 없음

ArrayList 내부
  → object[] 배열로 구현
  → int를 넣으면 반드시 object로 박싱
```

.NET은 값 타입으로 제네릭을 사용할 때 해당 타입 전용 코드를 별도로 생성해서 박싱 자체가 발생하지 않도록 최적화

## Nullable과 박싱

```csharp
int? x = 42;
object obj = x;   // 박싱 — 특수 동작
                  // null이면 null로 박싱
                  // 값이 있으면 값 자체(int)로 박싱 (Nullable 래퍼 없이)
```

## 게임 개발 관점에서

- **ArrayList 대신 List\<T\> 항상 사용**: Unity에서도 동일. 비제네릭 컬렉션은 박싱 때문에 사용 자제
- **인터페이스 매개변수 주의**: 값 타입을 인터페이스로 받는 메서드는 박싱 발생. 제네릭 제약 `where T : IFoo`로 대체하면 박싱 없음
- **Unity Profiler GC Alloc**: 매 프레임 박싱이 발생하면 GC Alloc 수치가 올라가는 게 Profiler에서 보임. 0B 목표
- **string.Format vs 보간 문자열**: `string.Format("{0}", value)`는 박싱 발생. `$"{value}"`는 최신 C#에서 최적화되어 있음

## 의문점 / 더 알아볼 것

- `IEquatable<T>` — 값 타입의 Equals 비교 시 박싱 방지하는 인터페이스
- `Span<T>` — Heap 할당 없이 메모리를 다루는 고성능 타입
- 구조체 인터페이스 박싱과 `default interface implementation` 관계
