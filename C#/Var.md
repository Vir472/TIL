# C# var 키워드

## 핵심 개념

- **타입 추론(Type Inference)** 키워드 — 컴파일러가 우변을 보고 타입을 자동으로 결정
- **동적 타입이 아님** — 런타임이 아니라 컴파일 타임에 타입이 확정됨
- 한 번 타입이 정해지면 다른 타입으로 바꿀 수 없음

## 기본 사용

```csharp
var hp = 100;           // int로 추론
var name = "Player";    // string으로 추론
var speed = 3.5f;       // float으로 추론
var enemy = new Enemy(); // Enemy로 추론

// 위 코드는 아래와 완전히 동일
int hp = 100;
string name = "Player";
float speed = 3.5f;
Enemy enemy = new Enemy();
```

컴파일 후에는 `var`가 실제 타입으로 대체됨 — 런타임 성능 차이 없음

## dynamic과의 차이 — 중요

`var`와 혼동하기 쉬운 `dynamic`은 완전히 다른 개념

```csharp
// var — 컴파일 타임에 타입 확정
var x = 10;
x = "hello";   // 컴파일 에러 — int로 확정됐으므로

// dynamic — 런타임에 타입 결정
dynamic y = 10;
y = "hello";   // OK — 런타임에 타입 바뀜
y.NonExistent(); // 컴파일은 되지만 런타임 에러 발생 가능
```

| 항목           | var              | dynamic                     |
| -------------- | ---------------- | --------------------------- |
| 타입 결정 시점 | 컴파일 타임      | 런타임                      |
| 타입 변경      | 불가             | 가능                        |
| IntelliSense   | 동작함           | 동작 안 함                  |
| 성능           | 일반 타입과 동일 | 느림 (런타임 타입 확인)     |
| 용도           | 타입 추론 편의   | COM interop, 동적 언어 연동 |

## var를 써야 하는 경우

```csharp
// 1. 타입명이 길고 우변에서 명확할 때
var dict = new Dictionary<string, List<int>>();
// Dictionary<string, List<int>> dict = new Dictionary<string, List<int>>(); 보다 깔끔

// 2. LINQ 결과 — 타입이 복잡해서 직접 쓰기 어려움
var result = enemies
    .Where(e => e.hp > 0)
    .Select(e => new { e.name, e.hp });
// 익명 타입은 var 없이 쓸 방법이 없음

// 3. for문 루프 변수
foreach (var enemy in enemyList)
{
    enemy.TakeDamage(10);
}
```

## var를 쓰지 말아야 하는 경우

```csharp
// 1. 우변만 봐서는 타입을 알 수 없을 때
var result = GetData();      // GetData()가 뭘 반환하는지 모름 — 가독성 저하
int result = GetData();      // 명확함

// 2. 숫자 리터럴 — 의도한 타입과 다를 수 있음
var x = 10;    // int로 추론 — long이 필요하다면?
var y = 3.14;  // double로 추론 — float이 필요하다면?
float y = 3.14f;  // 명시적으로 쓰는 게 안전

// 3. null 초기화 — 타입 추론 불가
var x = null;  // 컴파일 에러 — 타입을 알 수 없음
```

## C# 9.0 — new() 타입 추론 (반대 방향)

`var`와 반대로 좌변에서 타입을 알고 우변을 생략하는 방식

```csharp
// 기존
Enemy enemy = new Enemy();

// C# 9.0 이후 — 좌변 타입을 알고 있으면 우변 타입 생략 가능
Enemy enemy = new();  // var의 반대 방향 추론
```

## 게임 개발 관점에서

- **LINQ와 조합**: 적 목록 필터링, 정렬, 변환 등 컬렉션 처리에 LINQ를 쓸 때 `var`가 코드를 훨씬 깔끔하게 만들어줌

```csharp
  var aliveEnemies = enemies.Where(e => e.hp > 0).ToList();
  var sortedByHp = enemies.OrderBy(e => e.hp);
```

- **foreach에서 자주 씀**: Unity에서 컴포넌트 리스트나 오브젝트 컬렉션 순회할 때

```csharp
  foreach (var collider in Physics.OverlapSphere(transform.position, range))
  {
      collider.GetComponent<Enemy>()?.TakeDamage(damage);
  }
```

- **남용 주의**: 팀 프로젝트에서 `var`를 과하게 쓰면 코드 리뷰 시 타입 파악이 어려워짐. 협업 환경에선 팀 컨벤션에 따르는 게 중요

## 요약

| 상황                      | var 사용 여부 |
| ------------------------- | ------------- |
| 타입명이 길고 우변이 명확 | 권장          |
| LINQ / 익명 타입          | 필수          |
| foreach 루프 변수         | 권장          |
| 우변만 보고 타입 불명확   | 비권장        |
| 숫자 리터럴 (타입 중요)   | 비권장        |
| null 초기화               | 불가          |

## 의문점 / 더 알아볼 것

- C# 12의 `var` 패턴 매칭 개선 사항
- LINQ의 익명 타입(`new { }`)이 `var` 없이는 쓸 수 없는 이유
