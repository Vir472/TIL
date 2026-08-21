# 애니메이션 시스템

## 핵심 개념

Unity 애니메이션은 세 가지 요소가 조합되어 동작함

- **Sprite** — 실제 이미지 소스
- **Animation Clip** — 동작 하나를 정의한 파일
- **Animator Controller** — 여러 동작 간 전환 조건을 관리하는 상태머신

## 구성 요소

### Sprite

- 스프라이트 시트에서 Sprite Editor로 잘라낸 개별 프레임
- 애니메이션의 가장 기본 재료

### Animation Clip (.anim)

- 프레임들을 타임라인에 순서대로 배치한 "동작 하나"
- 걷기, 점프, 대기, 공격 각각 별도 Clip으로 만듦
- Samples(FPS)로 재생 속도 조정
- Loop Time 체크 시 반복 재생

### Animator Controller (.controller)

- 여러 Animation Clip을 연결하고 전환 조건을 관리
- 내부적으로 **상태머신(State Machine)** 구조
- 각 상태(State) = Animation Clip 하나
- 상태 간 화살표(Transition)에 전환 조건 설정

```
[Idle] ──isRunning=true──▶ [Run]
[Run]  ──isRunning=false──▶ [Idle]
[Run]  ──isJumping=true───▶ [Jump]
[Jump] ──isGrounded=true──▶ [Run]
```

### Animator 컴포넌트

- 오브젝트에 부착하는 컴포넌트. Controller를 연결해서 실제로 동작시킴
- 코드에서 파라미터를 바꿔주면 Controller가 알아서 상태 전환

## 파라미터 종류

Animator Controller에서 전환 조건으로 사용하는 변수

| 타입    | 용도                                   |
| ------- | -------------------------------------- |
| Bool    | 지속적인 상태 (달리는 중, 땅에 있는지) |
| Int     | 숫자로 구분되는 상태 (공격 타입)       |
| Float   | 수치 기반 조건 (이동 속도)             |
| Trigger | 순간적인 이벤트 (점프, 공격 시작)      |

## 코드에서 애니메이션 제어

코드가 애니메이션을 직접 재생하는 게 아니라 **파라미터만 바꿔주면** Animator Controller가 알아서 전환

- `animator.SetBool("isRunning", true)`
- `animator.SetTrigger("jump")`
- `animator.SetFloat("speed", moveSpeed)`
- `animator.SetInteger("attackType", 2)`

## 2D 스프라이트 애니메이션 설정 순서

```
① 스프라이트 시트 임포트 (Sprite Mode: Multiple)
② Sprite Editor로 프레임 분리
③ Animation 창에서 Clip 생성 (프레임 드래그)
④ Animator Controller에서 상태 연결 + 조건 설정
⑤ 오브젝트에 Animator 컴포넌트 부착 + Controller 연결
⑥ 스크립트에서 파라미터 제어
```

## 게임 개발 관점에서

- **Any State**: Animator Controller에서 어떤 상태에서든 특정 상태로 전환할 수 있는 특수 상태. 피격, 사망처럼 언제든 발생할 수 있는 동작에 활용
- **Sub-State Machine**: 상태가 많아지면 그룹으로 묶어서 정리하는 기능
- **Avatar / Humanoid**: 3D 캐릭터 애니메이션에서 뼈대(본) 구조를 정의하는 개념. 2D에서는 불필요

## 의문점 / 더 알아볼 것

- Blend Tree — Float 값에 따라 여러 애니메이션을 자연스럽게 섞는 기능 (걷기→달리기 속도 전환)
- Animation Rigging — 런타임에 뼈대를 코드로 제어하는 기능
- Spine / DragonBones — Unity 외부의 2D 골격 애니메이션 툴과의 연동
