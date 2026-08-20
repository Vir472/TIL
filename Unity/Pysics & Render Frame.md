# Unity 물리 프레임 vs 렌더 프레임

## 핵심 개념

Unity는 두 가지 독립적인 업데이트 루프를 가짐

- **렌더 프레임** — 화면에 그리는 주기 (가변)
- **물리 프레임** — 물리 연산을 처리하는 주기 (고정)

## 렌더 프레임 — Update()

```
렌더 프레임
├─ 주기: 가변 (기기 성능, 부하에 따라 달라짐)
├─ 고사양 PC  → 초당 200프레임도 가능
├─ 저사양 모바일 → 초당 30프레임 이하
└─ 호출 메서드: Update(), LateUpdate()
```

- 매 렌더링마다 호출 → 프레임 속도가 빠를수록 자주 호출됨
- `Time.deltaTime` — 이전 프레임과 현재 프레임 사이의 시간(초)
- `Time.deltaTime`을 곱해야 프레임 속도에 관계없이 일정한 속도 보장

## 물리 프레임 — FixedUpdate()

```
물리 프레임
├─ 주기: 고정 (기본 0.02초 = 초당 50회)
├─ 기기 성능과 무관하게 항상 일정
└─ 호출 메서드: FixedUpdate()
```

- Project Settings → Time → Fixed Timestep에서 변경 가능
- `Time.fixedDeltaTime` — 항상 Fixed Timestep과 동일한 고정값
- Rigidbody, 충돌 감지 등 물리 연산이 여기서 처리됨

## 두 루프가 분리된 이유

```
렌더 프레임이 빠를 때 (120FPS)
  물리 프레임(50Hz)보다 렌더가 더 자주 실행됨
  → 물리 결과를 보간해서 부드럽게 표시

렌더 프레임이 느릴 때 (20FPS)
  물리 프레임(50Hz)이 렌더보다 더 자주 실행됨
  → 한 렌더 프레임 사이에 물리가 여러 번 실행될 수 있음
```

물리를 렌더와 같은 루프에 묶으면 프레임 속도에 따라 물리 결과가 달라짐
→ 60FPS에서 점프 높이와 30FPS에서 점프 높이가 달라지는 버그 발생

## 실행 순서

```
한 프레임의 실행 순서

① FixedUpdate (물리 프레임이 도달한 경우, 여러 번 실행될 수 있음)
   └─ 물리 연산, Rigidbody 이동

② Update
   └─ 입력 처리, 게임 로직

③ LateUpdate
   └─ 카메라 추적, Update 이후 처리가 필요한 로직

④ 렌더링
   └─ 화면에 출력
```

## 어디서 뭘 해야 하는지

| 작업                | 사용 메서드 | 이유                                    |
| ------------------- | ----------- | --------------------------------------- |
| 키 입력 감지        | Update      | 렌더 프레임마다 확인해야 입력 누락 없음 |
| Rigidbody 이동/힘   | FixedUpdate | 물리 엔진과 같은 주기로 처리해야 정확   |
| transform.Translate | Update      | 물리 미사용 시. deltaTime 곱하기 필수   |
| 카메라 추적         | LateUpdate  | 캐릭터 이동 후 카메라가 따라가야 하므로 |
| 애니메이션 파라미터 | Update      | 렌더 프레임에 맞춰 갱신                 |

## GetKey를 FixedUpdate에 쓰면 안 되는 이유

```
렌더 프레임: 60FPS → Update 60번 호출
물리 프레임: 50Hz  → FixedUpdate 50번 호출

순간적인 키 입력(GetKeyDown)을 FixedUpdate에서 감지하면
렌더 프레임과 물리 프레임 주기가 달라서 입력을 놓칠 수 있음
```

입력은 Update에서 받고, 그 값을 변수에 저장해서 FixedUpdate에서 사용하는 패턴이 정석

```csharp
bool jumpPressed = false;

void Update()
{
    if (Input.GetKeyDown(KeyCode.Space))
        jumpPressed = true;  // Update에서 입력 감지
}

void FixedUpdate()
{
    if (jumpPressed)
    {
        rb.AddForce(Vector2.up * jumpForce, ForceMode2D.Impulse);
        jumpPressed = false;
    }
}
```

## Time 관련 주요 변수

| 변수                     | 설명                                                          |
| ------------------------ | ------------------------------------------------------------- |
| `Time.deltaTime`         | 이전 렌더 프레임과 현재 프레임 사이의 시간                    |
| `Time.fixedDeltaTime`    | Fixed Timestep 값 (기본 0.02초)                               |
| `Time.time`              | 게임 시작 후 경과 시간 (초)                                   |
| `Time.timeScale`         | 시간 배율. 0이면 일시정지, 0.5면 슬로우모션                   |
| `Time.unscaledDeltaTime` | timeScale 영향 받지 않는 deltaTime (UI, 일시정지 메뉴에 활용) |

## 게임 개발 관점에서

- **timeScale 활용**: 슬로우모션 연출, 일시정지 구현 시 `Time.timeScale`을 조정하면 물리와 Update 모두 영향받음. UI는 `unscaledDeltaTime` 사용
- **Fixed Timestep 조정**: 물리 정밀도가 중요한 게임(핀볼, 레이싱)은 Fixed Timestep을 낮춰 물리 프레임을 늘림. 대신 CPU 부하 증가

## 의문점 / 더 알아볼 것

- Interpolation 설정 — Rigidbody가 물리 프레임 사이를 보간해 렌더링을 부드럽게 하는 방법
- `WaitForFixedUpdate` — 코루틴에서 물리 프레임을 기다리는 방법
- Physics.Simulate — 물리를 수동으로 특정 시간만큼 시뮬레이션하는 방법
