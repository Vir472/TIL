# TIL - Unity 오브젝트 생성과 삭제

## 오브젝트 생성

### Instantiate

씬에 오브젝트를 생성하는 기본 메서드

| 오버로드                                            | 설명                        |
| --------------------------------------------------- | --------------------------- |
| `Instantiate(original)`                             | 원본을 복제해서 생성        |
| `Instantiate(original, position, rotation)`         | 위치와 회전 지정해서 생성   |
| `Instantiate(original, parent)`                     | 부모 오브젝트 지정해서 생성 |
| `Instantiate(original, position, rotation, parent)` | 위치, 회전, 부모 모두 지정  |
| `Instantiate<T>(original)`                          | 제네릭으로 반환 타입 지정   |

- 프리팹뿐 아니라 씬에 있는 오브젝트도 복제 가능
- 생성된 오브젝트는 Awake → OnEnable → Start 순으로 생명주기 시작
- Heap에 메모리 할당 발생 → 반복 생성 시 GC 부담

### new GameObject()

코드로 빈 게임오브젝트를 직접 생성하는 방법

- `new GameObject("이름")`으로 생성
- `AddComponent<T>()`로 컴포넌트를 붙여서 구성
- 프리팹 없이 순수 코드로만 오브젝트 구성할 때 사용
- 실무에서는 Instantiate 방식이 훨씬 많이 쓰임

## 오브젝트 삭제

### Destroy

오브젝트 또는 컴포넌트를 삭제하는 기본 메서드

| 오버로드                       | 설명                               |
| ------------------------------ | ---------------------------------- |
| `Destroy(gameObject)`          | 오브젝트 즉시(현재 프레임 끝) 삭제 |
| `Destroy(gameObject, float t)` | t초 후에 삭제                      |
| `Destroy(component)`           | 특정 컴포넌트만 삭제               |

- 실제 삭제는 현재 프레임 끝에 일어남 — 호출 직후 바로 사라지지 않음
- 삭제 시 OnDisable → OnDestroy 순으로 호출됨
- 자식 오브젝트도 함께 삭제됨

### DestroyImmediate

- 즉시 삭제 (프레임 끝 기다리지 않음)
- **에디터 스크립트 전용** — 게임 런타임에서 사용 금지
- 런타임에서 쓰면 예상치 못한 오류 발생 가능

### 오브젝트 비활성화 vs 삭제

```
SetActive(false) — 오브젝트를 숨기고 비활성화. 메모리에 유지
Destroy()        — 오브젝트를 완전히 제거. 메모리 해제

오브젝트 풀링 → SetActive(false/true) 반복
완전히 필요 없는 오브젝트 → Destroy
```

## 씬 전환 시 오브젝트 유지

### DontDestroyOnLoad

- 씬이 전환되어도 오브젝트가 파괴되지 않고 유지됨
- GameManager, AudioManager처럼 씬 간 유지가 필요한 싱글톤에 사용
- 씬 전환 시 중복 생성 방지 로직과 함께 사용 필수

```
씬 전환 시 기본 동작 — 현재 씬의 모든 오브젝트 Destroy
DontDestroyOnLoad 적용 — 씬 전환 후에도 유지
```

## null 체크 주의사항

Destroy 후에도 C# 참조 변수는 null이 아닌 것처럼 보이는 Unity 특수 동작

```
Destroy(enemy);
// 이 시점에서 enemy 변수는 C# 참조가 남아있음
// 하지만 Unity 내부에서는 파괴된 상태
// enemy == null은 true (Unity가 == 연산자를 오버로드함)
// enemy는 null이지만 (string)enemy.name 접근 시 에러
```

Destroy 후에는 참조 변수를 null로 명시적으로 설정하는 게 안전

## 주요 활용 패턴

### 일정 시간 후 삭제

이펙트, 총알처럼 일정 시간 후 사라져야 하는 오브젝트에 활용

```
Destroy(gameObject, 3f);  // 3초 후 삭제
```

### 생성과 동시에 컴포넌트 접근

Instantiate의 반환값으로 바로 컴포넌트에 접근 가능

```
var enemy = Instantiate(enemyPrefab, spawnPos, Quaternion.identity);
enemy.GetComponent<Enemy>().Init(difficulty);

// 제네릭으로 더 간결하게
var enemy = Instantiate<Enemy>(enemyPrefab, spawnPos, Quaternion.identity);
enemy.Init(difficulty);
```

### 부모 지정 생성

UI 요소처럼 특정 부모 아래에 생성해야 할 때

```
Instantiate(itemPrefab, inventoryPanel.transform);
```

## 게임 개발 관점에서

- **Instantiate/Destroy 반복 자제**: 총알, 이펙트, 바닥 타일처럼 반복 생성/파괴되는 오브젝트는 오브젝트 풀링으로 대체. GC Spike 방지
- **Destroy(gameObject, t) 활용**: 이펙트처럼 일정 시간 후 사라지는 오브젝트에 딜레이 파라미터 활용하면 Coroutine 없이 간단하게 처리
- **DontDestroyOnLoad 중복 방지**: 씬을 재로드하면 DontDestroyOnLoad 오브젝트가 중복 생성될 수 있음. Awake에서 Instance가 이미 있으면 자신을 Destroy하는 로직 필수

## 의문점 / 더 알아볼 것

- `Addressables.InstantiateAsync` — 에셋을 비동기로 로드하면서 생성하는 방법
- `PoolManager` 직접 구현 vs `UnityEngine.Pool.ObjectPool<T>` 사용 비교
- `Object.Instantiate` vs `GameObject.Instantiate` 차이
