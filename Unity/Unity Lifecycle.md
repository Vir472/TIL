# Unity 생명주기 (Lifecycle)

## 핵심 개념

- Unity의 MonoBehaviour를 상속한 스크립트는 **정해진 순서**로 함수가 자동 호출됨
- 개발자가 직접 호출하지 않아도 Unity 엔진이 적절한 시점에 호출
- 각 단계마다 역할이 다르기 때문에 올바른 함수에서 작업해야 함

## 전체 생명주기 흐름

```
오브젝트 생성
    │
    ▼
① Awake          ← 컴포넌트 초기화
    │
    ▼
② OnEnable       ← 오브젝트 활성화 시
    │
    ▼
③ Start          ← 첫 프레임 직전 초기화
    │
    ▼
④ FixedUpdate    ← 물리 프레임마다 (고정 주기)
    │
    ▼
⑤ Update         ← 렌더 프레임마다 (가변 주기)
    │
    ▼
⑥ LateUpdate     ← Update 이후
    │
    ▼
⑦ OnDisable      ← 오브젝트 비활성화 시
    │
    ▼
⑧ OnDestroy      ← 오브젝트 파괴 시
```

## 각 함수 상세

### Awake

- 오브젝트가 생성되는 순간 호출 — Start보다 먼저
- 오브젝트가 비활성화 상태여도 호출됨
- **자기 자신의 컴포넌트 참조, 변수 초기화**에 사용
- 싱글톤 Instance 설정은 여기서

### Start

- 첫 번째 Update 직전에 한 번만 호출
- 모든 오브젝트의 Awake가 완료된 후 실행됨
- **다른 오브젝트의 컴포넌트 참조, 씬 전체 초기화**에 사용
- Awake에서 다른 오브젝트를 참조하면 초기화 순서 문제 발생 가능 → Start에서 처리

### Awake vs Start 차이

```
Awake  — 자기 자신 초기화. 다른 오브젝트가 준비 안 됐을 수 있음
Start  — 모든 Awake가 끝난 후. 다른 오브젝트 참조 안전
```

### FixedUpdate

- 고정 주기 호출 (기본 초당 50회)
- Rigidbody 이동, 물리 힘 적용은 여기서
- `Time.fixedDeltaTime` 사용

### Update

- 매 렌더 프레임마다 호출 (가변 주기)
- 입력 감지, 타이머, 일반 게임 로직에 사용
- `Time.deltaTime` 곱하기 필수

### LateUpdate

- 모든 Update가 끝난 후 호출
- 카메라 추적, 애니메이션 후처리에 사용
- 캐릭터가 Update에서 이동한 뒤 카메라가 따라가야 하므로 LateUpdate가 적합

### OnEnable / OnDisable

- OnEnable — 오브젝트/컴포넌트가 활성화될 때마다 호출. 이벤트 구독에 사용
- OnDisable — 오브젝트/컴포넌트가 비활성화될 때마다 호출. 이벤트 구독 해제에 사용
- 오브젝트 풀링에서 재사용할 때마다 호출됨
- 이벤트 구독은 OnEnable, 해제는 OnDisable이 정형화된 패턴

### OnDestroy

- `Destroy(gameObject)` 호출 시 또는 씬 전환 시 실행
- 외부 등록 해제, 리소스 정리에 사용

## 충돌 관련 함수

| 함수               | 호출 시점             |
| ------------------ | --------------------- |
| OnCollisionEnter2D | 충돌 시작             |
| OnCollisionStay2D  | 충돌 중               |
| OnCollisionExit2D  | 충돌 종료             |
| OnTriggerEnter2D   | 트리거 진입           |
| OnTriggerStay2D    | 트리거 안에 있는 동안 |
| OnTriggerExit2D    | 트리거 벗어남         |

- Collision — 물리적 충돌 (Rigidbody + Collider 필요)
- Trigger — 통과 가능한 감지 영역 (Collider의 Is Trigger 체크)
- 3D는 뒤에 2D 없이 동일한 이름 사용

## 함수별 용도 요약

| 함수        | 호출 시점      | 주요 용도                          |
| ----------- | -------------- | ---------------------------------- |
| Awake       | 생성 즉시      | 자신의 컴포넌트 참조, 변수 초기화  |
| Start       | 첫 프레임 직전 | 타 오브젝트 참조, 게임 시작 초기화 |
| FixedUpdate | 고정 주기      | Rigidbody 이동, 물리 처리          |
| Update      | 매 프레임      | 입력 감지, 게임 로직               |
| LateUpdate  | Update 이후    | 카메라 추적, 후처리                |
| OnEnable    | 활성화 시      | 이벤트 구독                        |
| OnDisable   | 비활성화 시    | 이벤트 구독 해제                   |
| OnDestroy   | 파괴 시        | 외부 등록 해제, 정리               |

## 게임 개발 관점에서

- **초기화 순서 문제**: A가 Awake에서 B를 참조하는데 B가 아직 Awake를 안 했으면 null 에러 발생. 이런 경우 참조를 Start로 옮기거나 Script Execution Order를 설정
- **오브젝트 풀링과 OnEnable**: 풀에서 꺼낼 때 `SetActive(true)` → OnEnable 호출. 재사용 시 초기화를 여기서 처리
- **이벤트 구독 해제 필수**: OnDisable이나 OnDestroy에서 `-=` 해제 안 하면 파괴된 오브젝트의 메서드가 호출되어 에러 발생

## 의문점 / 더 알아볼 것

- Script Execution Order — 여러 스크립트의 Awake/Start 실행 순서를 수동으로 지정하는 방법
- `void Reset()` — 인스펙터에서 컴포넌트를 Reset할 때 호출되는 에디터 전용 함수
- 코루틴과 생명주기 — `StartCoroutine`이 OnDisable 시 자동 중단되는 동작
