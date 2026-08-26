# new 키워드

## 핵심 개념

C#에서 `new`는 두 가지 완전히 다른 용도로 사용됨

1. **객체 생성** — Heap에 새 인스턴스를 만드는 연산자
2. **멤버 숨기기(Hiding)** — 부모 클래스의 멤버를 자식에서 숨기는 한정자

## 1. 객체 생성 연산자

### 기본 동작

- `new 타입()`을 호출하면 Heap에 메모리 할당 후 생성자 실행
- 참조 타입(class)은 Heap에 객체 생성, Stack에 참조(주소) 저장
- 값 타입(struct)은 `new`를 써도 Stack에 생성됨

### 타입별 동작

```
class Enemy → Heap에 생성
new Enemy() → Heap 할당 → GC 대상

struct Vector2 → Stack에 생성
new Vector2(1, 0) → Stack 할당 → GC 대상 아님

int x = new int() → Stack. 0으로 초기화됨 (= int x = 0과 동일)
```

### C# 9.0 이후 — new() 타입 추론

좌변에서 타입을 알 수 있으면 우변의 타입명 생략 가능

```
Enemy enemy = new Enemy();  // 기존
Enemy enemy = new();        // C# 9.0+
```

## 2. 멤버 숨기기 (Method Hiding)

### override와의 차이

```
override — 부모의 virtual 메서드를 재정의. 다형성 동작
new      — 부모의 메서드를 숨김. 다형성 동작 안 함
```

### 동작 비교

```csharp
class Parent
{
    public virtual void Hello() => Console.WriteLine("Parent");
}

class ChildA : Parent
{
    public override void Hello() => Console.WriteLine("ChildA");  // 재정의
}

class ChildB : Parent
{
    public new void Hello() => Console.WriteLine("ChildB");  // 숨기기
}

Parent a = new ChildA();
Parent b = new ChildB();

a.Hello();  // "ChildA" — override는 실제 타입 기준
b.Hello();  // "Parent" — new는 참조 타입 기준, 부모 메서드 호출
```

### 핵심 차이

```
부모 타입으로 참조할 때
  override → 자식 메서드 호출 (런타임 다형성)
  new      → 부모 메서드 호출 (컴파일 타임 결정)

자식 타입으로 참조할 때
  override → 자식 메서드 호출
  new      → 자식 메서드 호출
```

### new를 쓰는 경우

- 부모 메서드가 virtual이 아닌데 자식에서 같은 이름으로 다른 동작을 구현하고 싶을 때
- 의도적으로 부모 구현과 분리하고 싶을 때
- 하지만 대부분의 경우 **override가 더 올바른 선택**

### new 없이 같은 이름 사용 시

```csharp
class Child : Parent
{
    public void Hello() { }  // new 없이 부모와 같은 이름
    // 컴파일 경고 발생 — new 키워드로 명시할 것을 권장
}
```

new를 명시하지 않아도 동작은 하지만 컴파일러가 경고를 발생시킴. 의도적임을 명확히 하려면 new 키워드 명시

## 요약

| 용도           | 문법                       | 설명                             |
| -------------- | -------------------------- | -------------------------------- |
| 객체 생성      | `new Enemy()`              | Heap에 인스턴스 생성             |
| 배열 생성      | `new int[5]`               | 배열 생성                        |
| 타입 추론 생성 | `new()`                    | C# 9.0+, 좌변 타입 추론          |
| 멤버 숨기기    | `public new void Method()` | 부모 메서드를 숨김               |
| 제약 조건      | `where T : new()`          | 제네릭에서 기본 생성자 필요 조건 |

## 게임 개발 관점에서

- **Unity에서 new 남용 주의**: `Update()`에서 `new`로 객체를 매 프레임 생성하면 GC Spike 원인이 됨. 오브젝트 풀링으로 대체
- **멤버 숨기기 활용**: MonoBehaviour를 상속한 클래스에서 부모의 non-virtual 메서드와 같은 이름이 필요할 때 사용. 하지만 혼란을 줄 수 있으므로 가급적 다른 이름 권장
- **struct와 new**: Vector2, Vector3 같은 Unity struct는 new를 써도 Stack에 생성되어 GC 부담 없음

## 의문점 / 더 알아볼 것

- `new()` 제약과 `Activator.CreateInstance()` 차이
- 값 타입에서 `new`를 쓸 때와 안 쓸 때의 차이 (초기화 여부)
