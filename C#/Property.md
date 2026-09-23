# 프로퍼티 (Property)

## 핵심 개념

- **프로퍼티**: 필드처럼 쓰이지만 실제로는 메서드인 멤버
- 외부에는 값처럼 노출하고, 내부에서는 값을 읽고 쓰는 과정에 **코드를 끼워 넣을 수 있음**
- 캡슐화를 지키면서도 `obj.Hp = 50` 같은 자연스러운 문법을 유지하는 장치임
- `get` 접근자와 `set` 접근자로 구성됨

```csharp
public class Player
{
    private int hp;             // 백킹 필드 (실제 저장소)

    public int Hp               // 프로퍼티 (통로)
    {
        get { return hp; }
        set { hp = value; }     // value는 대입된 값을 가리키는 예약어
    }
}
```

## 왜 필드를 그냥 쓰지 않는가

필드를 public으로 열면 **값이 바뀌는 순간에 개입할 수 없음**.

```mermaid
graph LR
    A["player.Hp = 50"] --> B["set_Hp(50) 호출"]
    B --> C["범위 제한 · 이벤트 발행 · 로그"]
    C --> D["백킹 필드 hp = 50"]
```

- 프로퍼티는 대입과 저장 사이에 **가로챌 지점**을 만들어줌
- 나중에 검증 로직을 추가해도 이 클래스를 쓰는 외부 코드는 한 줄도 고치지 않아도 됨
- 필드를 public으로 열었다가 나중에 프로퍼티로 바꾸면 외부 코드는 그대로 컴파일되지만, 리플렉션·직렬화·바이너리 호환성이 깨짐

## 컴파일 결과

프로퍼티는 언어가 제공하는 문법 설탕이고, IL 수준에서는 메서드임.

```csharp
public int Hp { get; set; }
```

- `get_Hp()`와 `set_Hp(int value)` 두 메서드로 컴파일됨
- 자동 구현 프로퍼티는 컴파일러가 숨은 백킹 필드(`<Hp>k__BackingField`)도 함께 만듦
- 대부분 JIT가 인라이닝하므로 단순 프로퍼티는 필드 접근과 성능 차이가 거의 없음

## 문법 정리

| 형태          | 코드                                              | 용도                               |
| ------------- | ------------------------------------------------- | ---------------------------------- |
| 자동 구현     | `public int Hp { get; set; }`                     | 로직이 필요 없을 때                |
| 읽기 전용     | `public int MaxHp { get; }`                       | 생성자에서만 대입 가능             |
| 초기화 전용   | `public int Id { get; init; }`                    | 객체 초기화 시점까지만 대입 (C# 9) |
| 접근자 분리   | `public int Hp { get; private set; }`             | 읽기는 공개, 쓰기는 내부만         |
| 계산 프로퍼티 | `public bool IsDead => hp <= 0;`                  | 저장하지 않고 매번 계산            |
| 표현식 본문   | `public int Hp { get => hp; set => hp = value; }` | 짧게 쓰기                          |

**접근자 분리가 실전에서 가장 유용함**

```csharp
public int Hp { get; private set; }   // 외부는 읽기만, 변경은 클래스 내부에서만
```

- 외부에서 HP를 마음대로 바꾸는 사고를 컴파일 단계에서 막음

## 필드 · 프로퍼티 · 메서드 선택 기준

| 상황                                | 선택         |
| ----------------------------------- | ------------ |
| 값을 그냥 저장만 함, 외부 노출 없음 | private 필드 |
| 값처럼 읽히고 비용이 거의 없음      | 프로퍼티     |
| 계산이 무겁거나 부수 효과가 있음    | 메서드       |
| 호출할 때마다 결과가 달라짐         | 메서드       |

- 주의 — 프로퍼티는 읽는 쪽이 "싸다"고 가정함. 무거운 연산을 프로퍼티에 숨기면 `if (obj.Something)` 한 줄이 성능 문제가 됨

## 게임 개발 관점에서

**유니티 인스펙터에 노출되지 않음**

- 유니티 직렬화는 **필드만** 대상으로 함. 프로퍼티는 `public`이어도 인스펙터에 뜨지 않음
- `[SerializeField]`도 필드 전용임
- 해결 — 필드를 직렬화하고 프로퍼티는 접근 통로로 둠

```csharp
[SerializeField] private int maxHp;          // 인스펙터에 노출
public int MaxHp => maxHp;                   // 외부에는 읽기 전용으로 제공
```

**setter에서 이벤트를 발행하는 패턴**

- 값이 바뀔 때 UI를 갱신하는 구조를 프로퍼티로 깔끔하게 만들 수 있음

```csharp
public event Action<int> OnHpChanged;

private int hp;
public int Hp
{
    get => hp;
    set
    {
        int clamped = Mathf.Clamp(value, 0, maxHp);
        if (clamped == hp) return;           // 같은 값이면 알리지 않음
        hp = clamped;
        OnHpChanged?.Invoke(hp);
    }
}
```

- 범위 제한과 변경 알림이 한곳에 모임
- HP를 건드리는 모든 코드가 자동으로 같은 규칙을 따르게 됨

**구조체를 반환하는 프로퍼티는 복사본을 줌**

- 유니티 입문자가 반드시 한 번 겪는 지점임

```csharp
transform.position.x = 5f;    // ❌ 컴파일 에러
```

- `position`은 필드가 아니라 `Vector3`(구조체)를 반환하는 **프로퍼티**임
- 반환된 것은 복사본이라 거기에 대입해도 원본이 바뀌지 않음. 그래서 컴파일러가 아예 막음

```csharp
// ✅ 통째로 새 값을 대입
Vector3 pos = transform.position;
pos.x = 5f;
transform.position = pos;
```

**유니티 내장 프로퍼티의 숨은 비용**

- `Camera.main`은 내부적으로 태그로 오브젝트를 찾음. 매 프레임 호출하면 비쌈
- `gameObject`, `transform`도 프로퍼티라 네이티브 쪽 조회가 일어남
- 대응 — `Awake`에서 한 번 받아 필드에 캐싱

```csharp
private Camera cam;
private Transform tr;

void Awake()
{
    cam = Camera.main;
    tr = transform;
}
```

**`ref`/`out`으로 넘길 수 없음**

- 프로퍼티는 메서드라 변수의 주소가 없음
- `ref` 인자가 필요하면 백킹 필드를 직접 넘겨야 함

## 의문점 / 더 알아볼 것

- 인덱서(Indexer) — `this[int i]` 형태의 프로퍼티
- `init` 접근자와 `record` — 불변 객체를 만드는 최신 문법
- 유니티가 프로퍼티를 직렬화하지 않는 이유 — 직렬화 시스템이 C++ 쪽에 있다는 점과의 관계
- `Mathf.Clamp` 같은 검증을 프로퍼티에 넣는 것과 별도 메서드로 빼는 것의 설계 트레이드오프
- 프로퍼티를 인터페이스에 선언하는 방법과 그 의미
- 자동 구현 프로퍼티의 백킹 필드에 `[SerializeField]`를 붙이는 우회 방법과 그 위험성
