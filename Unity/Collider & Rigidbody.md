# Collider / Rigidbody / 충돌 함수

## Collider 종류

### 2D Collider

| 컴포넌트              | 형태               | 용도                         |
| --------------------- | ------------------ | ---------------------------- |
| Box Collider 2D       | 사각형             | 바닥, 벽, 플랫폼             |
| Circle Collider 2D    | 원형               | 공, 캐릭터                   |
| Capsule Collider 2D   | 캡슐형             | 캐릭터 (머리/발 처리에 유리) |
| Polygon Collider 2D   | 다각형             | 불규칙한 형태의 지형         |
| Edge Collider 2D      | 선분               | 얇은 바닥, 경사면            |
| Composite Collider 2D | 여러 Collider 합성 | Tilemap과 함께 사용          |

### 3D Collider

| 컴포넌트         | 형태             | 용도                      |
| ---------------- | ---------------- | ------------------------- |
| Box Collider     | 직육면체         | 박스형 오브젝트           |
| Sphere Collider  | 구형             | 공, 캐릭터                |
| Capsule Collider | 캡슐형           | 캐릭터 (가장 많이 쓰임)   |
| Mesh Collider    | 메시 형태 그대로 | 복잡한 지형. 성능 비용 큼 |

### Is Trigger

Collider에 Is Trigger 체크 시 물리 충돌 없이 감지만 함

- 체크 해제 — 물리적으로 막힘. OnCollision 계열 함수 호출
- 체크 — 통과 가능. OnTrigger 계열 함수 호출
- 바닥 밖을 감지하는 컨테이너처럼 "범위 감지"용으로 사용할 때 Trigger가 적합

## Rigidbody 종류

### Rigidbody2D (2D)

| 항목                 | 설명                                                      |
| -------------------- | --------------------------------------------------------- |
| Body Type: Dynamic   | 물리 엔진 완전 적용. 중력, 힘, 충돌 반응                  |
| Body Type: Kinematic | 스크립트로 직접 제어. 물리 힘 받지 않음. 충돌 감지는 가능 |
| Body Type: Static    | 움직이지 않는 오브젝트. 바닥, 벽에 사용                   |
| Gravity Scale        | 중력 배율. 0이면 중력 없음                                |
| Constraints          | 특정 축 이동/회전 고정                                    |

### Kinematic 활용

- Dynamic으로 하면 물리 힘에 반응해서 예상치 못하게 움직일 수 있음
- Kinematic은 스크립트로만 움직이면서 충돌 감지도 가능

## 충돌 감지 함수

### Collision 계열 (물리 충돌)

물리적으로 막히는 충돌. 두 오브젝트 모두 Collider 필요, 하나 이상 Rigidbody 필요

| 함수                            | 호출 시점         |
| ------------------------------- | ----------------- |
| OnCollisionEnter2D(Collision2D) | 충돌 시작 순간    |
| OnCollisionStay2D(Collision2D)  | 충돌 중 매 프레임 |
| OnCollisionExit2D(Collision2D)  | 충돌 끝나는 순간  |

### Trigger 계열 (감지 전용)

Is Trigger 체크된 Collider와의 접촉. 통과 가능

| 함수                         | 호출 시점                       |
| ---------------------------- | ------------------------------- |
| OnTriggerEnter2D(Collider2D) | 트리거 영역 진입 순간           |
| OnTriggerStay2D(Collider2D)  | 트리거 안에 있는 동안 매 프레임 |
| OnTriggerExit2D(Collider2D)  | 트리거 영역 벗어나는 순간       |

- 3D는 함수 이름 뒤에 2D 없이 동일
- 매개변수 타입도 다름 — Collision2D vs Collider2D

### 충돌한 오브젝트 식별

```
Collision2D.gameObject   — 충돌한 오브젝트
Collision2D.collider     — 충돌한 Collider
Collision2D.contacts     — 충돌 지점 정보

Collider2D.gameObject    — 트리거에 닿은 오브젝트
Collider2D.tag           — 태그로 구분
Collider2D.CompareTag()  — 태그 비교
```

## 충돌 감지 조건 정리

두 오브젝트 간에 충돌/트리거 감지가 되려면 조건이 필요

| 조건           | Collision       | Trigger        |
| -------------- | --------------- | -------------- |
| Collider 필요  | 양쪽 모두       | 양쪽 모두      |
| Rigidbody 필요 | 하나 이상       | 하나 이상      |
| Is Trigger     | 둘 다 체크 해제 | 하나 이상 체크 |

## 의문점 / 더 알아볼 것

- Object Pooling — 오브젝트를 매번 생성/파괴하지 않고 재사용하는 패턴
- Layer Collision Matrix — 특정 레이어끼리만 충돌하도록 설정하는 방법
- Physics2D.OverlapBox — Collider 없이 코드로 특정 영역 내 오브젝트를 감지하는 방법
