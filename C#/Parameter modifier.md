# C# 매개변수 한정자 (Parameter Modifier)

## 핵심 개념

- 메서드에 매개변수를 전달하는 **방식**을 제어하는 키워드
- 기본적으로 값 타입은 복사본, 참조 타입은 참조가 전달됨
- 한정자로 이 기본 동작을 변경할 수 있음

## 종류 요약

| 한정자   | 방향   | 초기화 필요 | 설명                             |
| -------- | ------ | ----------- | -------------------------------- |
| (없음)   | 입력   | 필요        | 기본값. 복사본 전달              |
| `ref`    | 입출력 | 필요        | 참조 전달. 원본 수정 가능        |
| `out`    | 출력   | 불필요      | 메서드 안에서 반드시 할당해야 함 |
| `in`     | 입력   | 필요        | 읽기 전용 참조 전달. 수정 불가   |
| `params` | 입력   | -           | 가변 개수의 인자를 배열로 받음   |

## ref

호출자의 변수를 직접 참조로 전달 — 메서드 안에서 수정하면 원본이 바뀜

```csharp
void AddHp(ref int hp, int amount)
{
    hp += amount;  // 원본 변수 직접 수정
}

int playerHp = 50;
AddHp(ref playerHp, 30);
Console.WriteLine(playerHp);  // 80 — 원본이 바뀜
```

- 호출 시에도 `ref` 키워드 명시 필요
- 전달 전에 반드시 초기화되어 있어야 함

## out

메서드가 값을 **출력**하기 위한 한정자. 여러 값을 반환할 때 주로 사용

```csharp
bool TryGetEnemy(int id, out Enemy enemy)
{
    enemy = enemyDB.Find(id);  // 메서드 안에서 반드시 할당
    return enemy != null;
}

// 사용
if (TryGetEnemy(1, out Enemy e))
{
    e.TakeDamage(10);
}

// C# 7.0+ — 인라인 선언 가능
if (TryGetEnemy(1, out var e))  // var로 타입 추론
```

- 전달 전 초기화 불필요 — 메서드 안에서 반드시 할당해야 함
- `ref`와 달리 단방향(출력 전용)

## in

읽기 전용 참조 전달 — 복사 없이 전달하되 수정은 불가

```csharp
void PrintStats(in CombatStats stats)
{
    Console.WriteLine(stats.hp);
    stats.hp = 100;  // 컴파일 에러 — 수정 불가
}

CombatStats s = new CombatStats { hp = 50 };
PrintStats(in s);
```

- 큰 struct를 복사 없이 전달할 때 성능 최적화 목적
- `ref`와 구조는 같지만 수정 불가

## ref vs out vs in 비교

```
ref — 읽고 쓰기 모두 가능. 초기화 필수
out — 쓰기 전용 (출력). 초기화 불필요
in  — 읽기 전용 (입력). 초기화 필수. 복사 비용 절감
```

## params

인자 개수가 가변적일 때 배열로 묶어서 받는 한정자

```csharp
int Sum(params int[] numbers)
{
    int total = 0;
    foreach (var n in numbers) total += n;
    return total;
}

Sum(1, 2, 3);          // 3개
Sum(1, 2, 3, 4, 5);    // 5개
Sum(new int[] { 1, 2}); // 배열로도 전달 가능
```

- 반드시 마지막 매개변수여야 함
- 하나의 메서드에 하나만 사용 가능

## 게임 개발 관점에서

- **out — TryGetComponent**: Unity에서 가장 흔하게 만나는 out 패턴

```csharp
  if (TryGetComponent<Rigidbody>(out var rb))
  {
      rb.AddForce(Vector3.up);
  }
```

- **ref — 시뮬레이션 상태 수정**: 전투 시뮬레이터에서 상태 구조체를 ref로 넘기면 복사 없이 직접 수정 가능

```csharp
  void ApplyDamage(ref CombatStats stats, int dmg)
  {
      stats.hp -= dmg;
      stats.sanity -= 5;
  }
```

- **in — 대형 struct 전달**: readonly struct나 큰 데이터 구조체를 읽기 전용으로 넘길 때 복사 비용 절감. 림버스 프로젝트에서 보스 패턴 데이터처럼 크고 변경이 없는 struct에 적합
- **params — 유틸 메서드**: 가변 인자가 필요한 로그, 디버그 출력 등에 활용

## 의문점 / 더 알아볼 것

- `ref return` — 메서드가 참조 자체를 반환하는 방식
- `ref struct`와 `in` 한정자의 조합 (Span\<T\> 등)
- C# 12의 `ref readonly` 매개변수
