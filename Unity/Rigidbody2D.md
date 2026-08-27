# Rigidbody2D

## 핵심 개념

- 게임오브젝트에 물리 엔진을 적용하는 컴포넌트
- 중력, 힘, 충돌 반응 등 물리 시뮬레이션을 담당
- FixedUpdate 주기로 물리 연산 처리

## Body Type

물리 동작 방식을 결정하는 가장 핵심 설정

| Body Type | 설명                                                   | 적합한 용도                            |
| --------- | ------------------------------------------------------ | -------------------------------------- |
| Dynamic   | 물리 엔진 완전 적용. 중력, 힘, 충돌 반응 모두 받음     | 캐릭터, 물리 영향 받는 오브젝트        |
| Kinematic | 스크립트로만 제어. 물리 힘 받지 않음. 충돌 감지는 가능 | 이동 플랫폼, 러닝 게임 바닥 타일       |
| Static    | 완전히 고정. 물리 연산 최소화                          | 바닥, 벽처럼 절대 안 움직이는 오브젝트 |

## Dynamic 전용 파라미터

### Material

- 물리 머티리얼 설정. 마찰력(Friction)과 탄성(Bounciness) 정의
- 기본값 None — 마찰 있고 튕김 없음
- 얼음 바닥, 고무공 등 표면 특성이 필요할 때 사용

### Mass

- 오브젝트의 질량 (기본값 1)
- 질량이 클수록 힘에 덜 반응하고 충돌 시 상대를 더 많이 밀어냄
- 중력 가속도에는 영향 없음 (낙하 속도는 동일)

### Linear Drag

- 이동 시 저항력 (공기 저항 개념)
- 값이 클수록 이동 속도가 빠르게 감소
- 0이면 저항 없음. 빙판처럼 미끄러지는 효과

### Angular Drag

- 회전 시 저항력
- 값이 클수록 회전이 빠르게 멈춤
- 2D 게임에서 오브젝트가 충돌 후 계속 빙글빙글 도는 걸 방지

### Gravity Scale

- 중력 배율 (기본값 1)
- 0이면 중력 없음
- 음수면 위로 떠오름
- Dynamic Body Type에서 중력을 끄고 싶을 때 0으로 설정

### Collision Detection

물리 충돌 감지 방식

| 옵션       | 설명                            | 적합한 경우                        |
| ---------- | ------------------------------- | ---------------------------------- |
| Discrete   | 기본값. 프레임 단위로 충돌 감지 | 일반적인 경우                      |
| Continuous | 연속적으로 충돌 감지            | 빠르게 움직이는 오브젝트 (총알 등) |

Discrete 방식은 오브젝트가 너무 빠르면 충돌을 뚫고 지나가는 터널링(Tunneling) 현상 발생 가능 → 빠른 오브젝트는 Continuous 사용

### Sleeping Mode

- 오브젝트가 움직임을 멈추면 물리 연산을 중단해서 성능 최적화
- Never Sleep — 항상 물리 연산 (입력에 즉각 반응 필요할 때)
- Start Awake — 시작부터 활성 상태
- Start Asleep — 시작은 비활성, 힘이 가해지면 활성화

### Interpolate

물리 프레임과 렌더 프레임 사이의 움직임을 보간해서 부드럽게 표현

| 옵션        | 설명                                               |
| ----------- | -------------------------------------------------- |
| None        | 보간 없음. 기본값                                  |
| Interpolate | 이전 프레임 기준으로 보간 — 부드럽지만 약간의 지연 |
| Extrapolate | 다음 프레임 예측 보간 — 빠르지만 예측 오차 가능    |

캐릭터처럼 부드러운 이동이 중요하면 Interpolate 권장

## Constraints

특정 축의 이동/회전을 고정하는 설정

| 옵션              | 설명          |
| ----------------- | ------------- |
| Freeze Position X | X축 이동 고정 |
| Freeze Position Y | Y축 이동 고정 |
| Freeze Rotation Z | Z축 회전 고정 |

2D 게임에서 캐릭터가 충돌 시 옆으로 쓰러지지 않게 **Freeze Rotation Z**를 체크하는 게 일반적

## Kinematic 전용 파라미터

### Use Full Kinematic Contacts

- 체크 시 Kinematic 오브젝트도 모든 충돌 콜백 수신
- 기본값 해제 — Dynamic과의 충돌만 감지

### Use Auto Mass

- 연결된 Collider 크기를 기반으로 질량을 자동 계산

## 게임 개발 관점에서

- **러닝 게임 바닥 타일**: Kinematic + Freeze All로 설정하면 스크립트로만 이동하면서 물리 간섭 없음
- **플랫포머 캐릭터**: Dynamic + Gravity Scale 1~3 + Freeze Rotation Z가 기본 설정. Linear Drag로 공중 제어감 조정
- **빠른 투사체**: Dynamic + Continuous Collision Detection으로 터널링 방지
- **Gravity Scale 0 vs Kinematic**: 중력만 끄고 싶으면 Gravity Scale 0, 물리 힘 전체를 끄고 싶으면 Kinematic이 맞음

## 의문점 / 더 알아볼 것

- Physics Material 2D — 마찰력과 탄성을 커스텀하는 에셋
- AddForce / AddTorque — 코드로 힘과 회전력을 가하는 메서드
- ForceMode2D — Force(지속적), Impulse(순간적) 차이
