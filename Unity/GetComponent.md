# Unity 컴포넌트 접근 방법

## 핵심 개념

- Unity에서 컴포넌트나 오브젝트에 접근하는 방법은 여러 가지
- 상황에 따라 적합한 방법이 다름
- **성능을 위해 캐싱(Awake에서 한 번만 가져오기)이 중요**

## 자기 자신의 컴포넌트 접근

### GetComponent\<T\>()

- 자신에게 붙어있는 컴포넌트를 가져오는 가장 기본적인 방법
- 없으면 null 반환
- 매 프레임 호출하면 성능 저하 → Awake에서 캐싱 필수

```
나쁜 패턴 — Update에서 매 프레임 GetComponent 호출
좋은 패턴 — Awake에서 한 번만 가져와서 변수에 저장
```

### TryGetComponent\<T\>(out T component)

- GetComponent와 같지만 없을 때 null 대신 bool로 결과 반환
- null 체크가 더 안전하고 명시적
- Unity 권장 방식

### GetComponents\<T\>()

- 같은 타입의 컴포넌트가 여러 개 붙어있을 때 배열로 전부 가져옴

### GetComponentInChildren\<T\>()

- 자식 오브젝트들 중에서 컴포넌트를 찾아 반환
- 비활성화된 자식도 검색하려면 `GetComponentInChildren<T>(true)`

### GetComponentInParent\<T\>()

- 부모 오브젝트들 중에서 컴포넌트를 찾아 반환

## 다른 오브젝트의 컴포넌트 접근

### 인스펙터에서 직접 연결 (가장 권장)

- `[SerializeField]`로 변수를 노출하고 인스펙터에서 드래그해서 연결
- 런타임 탐색 비용 없음 — 가장 빠르고 안전
- 참조가 명확해서 유지보수 쉬움

```
[SerializeField] private PlayerController player;
[SerializeField] private GameManager gameManager;
```

### FindObjectOfType\<T\>()

- 씬 전체에서 특정 타입의 컴포넌트를 가진 오브젝트를 탐색
- 씬 전체를 순회하므로 **매우 느림** — Awake/Start에서만 사용
- Update에서 절대 사용 금지

### FindObjectsByType\<T\>() (Unity 2023+)

- FindObjectOfType의 개선 버전
- 정렬 방식을 지정할 수 있고 성능이 더 좋음
- `FindObjectsByType<Enemy>(FindObjectsSortMode.None)`

### GameObject.Find("이름")

- 이름으로 씬 전체에서 오브젝트 탐색
- 가장 느린 방법 중 하나 — 이름 변경 시 버그 위험
- 가급적 사용 자제. 인스펙터 연결로 대체 권장

### Transform.Find("이름")

- 자식 오브젝트 중에서 이름으로 탐색
- 씬 전체가 아닌 자식만 탐색해서 GameObject.Find보다 빠름

## 싱글톤 패턴으로 접근

GameManager처럼 씬 어디서든 접근해야 하는 오브젝트에 주로 사용

- static Instance로 어디서든 `GameManager.Instance`로 접근
- FindObjectOfType 없이 직접 참조 가능

## 성능 비교

| 방법                   | 속도      | 권장 사용 시점    |
| ---------------------- | --------- | ----------------- |
| 인스펙터 직접 연결     | 매우 빠름 | 항상 권장         |
| GetComponent (캐싱)    | 빠름      | Awake에서 한 번만 |
| TryGetComponent        | 빠름      | Awake에서 한 번만 |
| GetComponentInChildren | 보통      | Awake에서 한 번만 |
| FindObjectOfType       | 느림      | Awake/Start에서만 |
| GameObject.Find        | 매우 느림 | 가급적 사용 자제  |

## 컴포넌트 추가 / 제거

- `gameObject.AddComponent<T>()` — 런타임에 컴포넌트 추가
- `Destroy(GetComponent<T>())` — 컴포넌트 제거

## 게임 개발 관점에서

- **캐싱 원칙**: 컴포넌트 참조는 Awake에서 한 번만 가져와 변수에 저장. Update에서 GetComponent 호출은 성능 저하의 주원인
- **인스펙터 연결 우선**: 코드로 탐색하는 것보다 인스펙터에서 드래그로 연결하는 게 빠르고 안전. 탐색 비용이 없고 참조가 명확함
- **null 체크**: GetComponent 결과는 항상 null 가능성 있음. TryGetComponent나 null 체크로 안전하게 처리

## 의문점 / 더 알아볼 것

- `GetComponentInChildren` vs `ComponentInChildren` 성능 차이
- `FindObjectsOfType<T>()` — 같은 타입 모든 오브젝트 배열로 반환
- Interface로 GetComponent하는 방법 — `GetComponent<IDamageable>()`
