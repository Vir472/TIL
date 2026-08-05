# C# 델리게이트 (Delegate)

## 핵심 개념

- **메서드를 변수처럼 저장하고 전달할 수 있는 타입**
- "어떤 메서드를 호출할지"를 런타임에 결정할 수 있게 해줌
- 특정 시그니처(반환 타입 + 매개변수)를 가진 메서드만 담을 수 있음

## 기본 문법

```csharp
// 1. 델리게이트 타입 선언
delegate int Operation(int a, int b);

// 2. 시그니처가 일치하는 메서드 정의
int Add(int a, int b) => a + b;
int Multiply(int a, int b) => a * b;

// 3. 델리게이트 변수에 메서드 할당
Operation op = Add;
Console.WriteLine(op(3, 4));  // 7

op = Multiply;
Console.WriteLine(op(3, 4));  // 12
```

## 멀티캐스트 (Multicast)

하나의 델리게이트 변수에 여러 메서드를 연결할 수 있음

```csharp
delegate void OnHit();

void PlaySound() => Console.WriteLine("타격음");
void SpawnEffect() => Console.WriteLine("이펙트 생성");
void ShakeCamera() => Console.WriteLine("카메라 흔들림");

OnHit onHit = PlaySound;
onHit += SpawnEffect;   // 메서드 추가
onHit += ShakeCamera;

onHit();  // 세 메서드가 순서대로 모두 호출됨

onHit -= SpawnEffect;   // 메서드 제거
```

반환값이 있는 멀티캐스트는 마지막 메서드의 반환값만 남으므로 보통 `void`에서 사용

## 내장 제네릭 델리게이트 — Action, Func

매번 델리게이트를 직접 선언하지 않아도 되는 내장 타입

| 타입            | 설명                                                 |
| --------------- | ---------------------------------------------------- |
| `Action`        | 반환값 없는 메서드. `Action<T>`, `Action<T1, T2>` 등 |
| `Func<TResult>` | 반환값 있는 메서드. 마지막 타입이 반환 타입          |
| `Predicate<T>`  | `bool`을 반환하는 메서드. `Func<T, bool>`과 동일     |

```csharp
Action<int> takeDamage = (dmg) => hp -= dmg;
Func<int, int, int> add = (a, b) => a + b;
Predicate<Enemy> isAlive = (e) => e.hp > 0;
```

## 람다식 (Lambda)

델리게이트에 익명 메서드를 간결하게 작성하는 문법

```csharp
// 기존 메서드 방식
delegate int Operation(int a, int b);
int Add(int a, int b) => a + b;
Operation op = Add;

// 람다식으로 한 번에
Operation op = (a, b) => a + b;

// Action, Func과 조합
Action<string> print = msg => Console.WriteLine(msg);
Func<int, bool> isPositive = x => x > 0;
```

## 이벤트 (event)

델리게이트를 외부에서 직접 호출하지 못하도록 캡슐화한 것

```csharp
class Enemy
{
    // event 키워드 — 외부에서 += / -= 만 가능, 직접 호출 불가
    public event Action OnDeath;

    public void TakeDamage(int dmg)
    {
        hp -= dmg;
        if (hp <= 0)
            OnDeath?.Invoke();  // null 체크 후 호출
    }
}

// 구독
enemy.OnDeath += () => DropLoot();
enemy.OnDeath += () => PlayDeathEffect();

// 외부에서 직접 호출 불가
enemy.OnDeath();      // 컴파일 에러
enemy.OnDeath = null; // 컴파일 에러
```

델리게이트와 이벤트의 차이
| 항목 | delegate | event |
|------|----------|-------|
| 외부 호출 | 가능 | 불가 |
| 외부에서 = 할당 | 가능 | 불가 |
| 외부에서 += / -= | 가능 | 가능 |

## 게임 개발 관점에서

- **Unity UnityEvent**: Unity에서 제공하는 이벤트 시스템. 내부적으로 델리게이트 기반이며 인스펙터에서 연결 가능
- **옵저버 패턴**: 적 사망, 스테이지 클리어, 아이템 획득 등의 게임 이벤트를 델리게이트/이벤트로 구현하면 발생 객체와 반응 객체를 느슨하게 연결 가능
- **콜백 패턴**: 비동기 작업 완료 후 호출할 메서드를 델리게이트로 전달

```csharp
  void LoadData(Action<BossPattern> onComplete)
  {
      var data = FetchFromServer();
      onComplete?.Invoke(data);
  }
```

## 의문점 / 더 알아볼 것

- 클로저(Closure) — 람다가 외부 변수를 캡처할 때 Heap에 올라가는 구조
- `EventHandler<TEventArgs>` — .NET 표준 이벤트 패턴
- Unity에서 이벤트 구독 해제를 깜빡하면 발생하는 메모리 누수 패턴
