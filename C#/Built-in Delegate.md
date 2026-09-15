# 내장 델리게이트 (Func / Action / Predicate)

## 개념

- `delegate` 타입을 매번 선언하는 대신 .NET이 미리 선언해둔 제네릭 델리게이트 사용
- `Action`, `Func`, `Predicate` 세 가지

```csharp
// 직접 선언
delegate int Operation(int a, int b);
Operation op = Add;

// 내장 델리게이트 — 타입 선언 불필요
Func<int, int, int> op = Add;
```

## Action — 반환값 없음

```csharp
Action                      // void Method()
Action<int>                 // void Method(int)
Action<string, float>       // void Method(string, float)
// 매개변수 최대 16개
```

```csharp
Action<string> log = msg => Console.WriteLine(msg);
log("피격!");

Action onHit = PlaySound;
onHit += SpawnEffect;   // 멀티캐스트 동일하게 동작
```

## Func — 반환값 있음

마지막 타입 파라미터가 반환 타입

```csharp
Func<int>                   // int Method()
Func<int, int>              // int Method(int)
Func<int, int, int>         // int Method(int, int)
Func<Enemy, bool>           // bool Method(Enemy)
```

```csharp
Func<int, int, int> add = (a, b) => a + b;
Console.WriteLine(add(3, 4));  // 7

Func<float> getDeltaTime = () => Time.deltaTime;
```

## Predicate\<T\> — bool 반환 전용

`Func<T, bool>`과 시그니처는 같지만 타입이 달라 서로 대입 불가

```csharp
Predicate<Enemy> isDead = e => e.Hp <= 0;
enemies.RemoveAll(isDead);        // List<T>.RemoveAll → Predicate<T>
var dead = enemies.Where(isDead); // ❌ LINQ → Func<T,bool>, 컴파일 에러
```

- `List<T>.Find`, `FindAll`, `RemoveAll`, `Array.Find` → `Predicate<T>`
- LINQ (`Where`, `Any`, `First`) → `Func<T, bool>`

## 정리 표

| 델리게이트 | 반환 | 시그니처 | 용도 |
| --- | --- | --- | --- |
| `Action` | void | `void ()` | 이벤트 콜백, 알림 |
| `Action<T>` | void | `void (T)` | 값 전달 콜백 |
| `Func<TResult>` | O | `TResult ()` | 지연 평가, 팩토리 |
| `Func<T, TResult>` | O | `TResult (T)` | 변환, 선택자 |
| `Func<T, bool>` | bool | `bool (T)` | LINQ 필터 |
| `Predicate<T>` | bool | `bool (T)` | List/Array 검색 |
| `Comparison<T>` | int | `int (T, T)` | 정렬 기준 |

---

# 익명 메서드와 람다

델리게이트에 넣을 메서드를 따로 정의하지 않고 그 자리에서 작성하는 문법

```csharp
// 1) 이름 있는 메서드
Func<int, int, int> f1 = Add;

// 2) 익명 메서드 (C# 2.0)
Func<int, int, int> f2 = delegate (int a, int b) { return a + b; };

// 3) 람다식 (C# 3.0~)
Func<int, int, int> f3 = (a, b) => a + b;
Func<int, int, int> f4 = (a, b) =>
{
    var result = a + b;
    return result;          // 블록 본문은 return 필요
};
```

## 클로저 캡처

- 람다는 바깥 스코프의 변수를 참조로 캡처 (값 복사 아님)
- 선언 시점이 아닌 호출 시점의 값 사용

```csharp
int damage = 10;
Action attack = () => Console.WriteLine(damage);
damage = 999;
attack();   // 999
```

`for` 루프에서 반복 변수를 캡처하는 경우

```csharp
// ❌ 모든 버튼이 i == 3 출력
for (int i = 0; i < 3; i++)
    buttons[i].onClick.AddListener(() => Debug.Log(i));

// ✅ 루프 내부에서 복사본 생성 후 캡처
for (int i = 0; i < 3; i++)
{
    int index = i;
    buttons[index].onClick.AddListener(() => Debug.Log(index));
}
```

`foreach`의 반복 변수는 C# 5.0부터 매 반복마다 새로 생성되어 해당 문제 없음

---

# event 키워드

델리게이트 필드를 `public`으로 열면 외부에서 통째로 덮어쓰거나 직접 호출 가능

```csharp
public Action OnDeath;       // 외부에서 = 대입, 직접 호출 가능
public event Action OnDeath; // 외부에서 += / -= 만 가능
```

| | 외부 `+=` `-=` | 외부 `=` 대입 | 외부 호출 |
| --- | --- | --- | --- |
| `public Action` | O | O | O |
| `public event Action` | O | X | X |

```csharp
public class Player
{
    public event Action<int> OnHpChanged;

    private int hp;
    public void TakeDamage(int amount)
    {
        hp -= amount;
        OnHpChanged?.Invoke(hp);   // 구독자 없으면 null
    }
}

// 구독 측
player.OnHpChanged += UpdateHpBar;
```

---

# 게임 개발 활용 패턴

## 1. 이벤트 기반 UI 갱신

발행자가 구독자를 알 필요 없음

```csharp
// Player는 HP 변경 사실만 알림
public event Action<int, int> OnHpChanged;  // (current, max)

// UI가 구독
void OnEnable()  => player.OnHpChanged += Refresh;
void OnDisable() => player.OnHpChanged -= Refresh;
```

## 2. 콜백 / 완료 통지

```csharp
public void PlayAnimation(string clip, Action onComplete)
{
    StartCoroutine(PlayRoutine(clip, onComplete));
}

PlayAnimation("Attack", () => Debug.Log("공격 끝"));
```

## 3. 전략 패턴 / 상태 머신

```csharp
Dictionary<SkillType, Action<Enemy>> skills = new()
{
    { SkillType.Fire,  e => e.TakeDamage(50) },
    { SkillType.Slow,  e => e.ApplySlow(3f)  },
};

skills[type](target);   // if / switch 분기 제거
```

```csharp
Action currentState;
void Update() => currentState?.Invoke();

void SetState(Action state) => currentState = state;
SetState(IdleState);
```

## 4. UnityEvent 비교

| | C# `event` | `UnityEvent` |
| --- | --- | --- |
| 인스펙터 노출 | X | O |
| 성능 | 빠름 | 느림 (리플렉션 기반) |
| 직렬화 | X | O |
| 용도 | 코드 간 통신 | 인스펙터에서 연결하는 버튼 / 트리거 |

---

# 주의사항

## 구독 해제 누락 시 메모리 누수

- 발행자가 구독자의 참조를 보유하므로 구독자가 파괴되어도 GC가 수거 불가
- Unity에서는 파괴된 오브젝트 참조로 `MissingReferenceException` 발생
- `static` 이벤트는 씬 전환 후에도 살아있어 더 위험

```csharp
void OnEnable()  => EventBus.OnGameOver += HandleGameOver;
void OnDisable() => EventBus.OnGameOver -= HandleGameOver;   // 쌍으로 작성
```

## 람다로 구독하면 해제 불가

```csharp
// ❌ -= 할 대상 참조 불가
manager.OnTick += () => Debug.Log("tick");

// ✅ 변수에 담거나 이름 있는 메서드 사용
Action handler = () => Debug.Log("tick");
manager.OnTick += handler;
manager.OnTick -= handler;
```

## null 체크는 ?.Invoke()

구독자가 없으면 델리게이트는 `null`이므로 그냥 호출 시 `NullReferenceException` 발생

```csharp
OnHpChanged?.Invoke(hp);
```

## 멀티캐스트 중 예외 발생 시 뒤쪽 구독자 호출 안 됨

```csharp
foreach (Action d in OnHit.GetInvocationList())
{
    try { d(); }
    catch (Exception e) { Debug.LogException(e); }
}
```

## 반환값 있는 멀티캐스트는 마지막 값만 남음

```csharp
Func<int> f = () => 1;
f += () => 2;
Console.WriteLine(f());  // 2
```

멀티캐스트는 `Action` / `void` 전용으로 사용
