# Box Collider 2D

## 핵심 개념

- 사각형 형태의 2D 충돌 영역을 정의하는 컴포넌트
- 실제 스프라이트 모양과 별개로 충돌 영역만 담당
- Rigidbody2D와 조합해서 물리 충돌 또는 트리거 감지에 사용

## 주요 파라미터

### Edit Collider

- 인스펙터에서 버튼 클릭 시 씬 뷰에서 충돌 영역을 직접 드래그로 조절 가능
- 스프라이트보다 약간 작게 설정하는 경우가 많음 (발 부분 등 세밀한 조정)

### Offset

- 충돌 영역의 중심점을 오브젝트 중심에서 얼마나 이동할지 설정
- X, Y 값으로 조정
- 스프라이트 중심과 충돌 영역 중심이 다를 때 활용

### Size

- 충돌 영역의 가로(X), 세로(Y) 크기
- Transform의 Scale과는 독립적으로 설정 가능
- 스프라이트 크기와 다르게 충돌 영역만 크거나 작게 만들 수 있음

### Is Trigger

- 체크 해제 — 물리적 충돌. 오브젝트가 통과 못 함. OnCollision 계열 함수 호출
- 체크 — 감지 전용. 오브젝트가 통과 가능. OnTrigger 계열 함수 호출
- 러닝 게임 범위 감지 컨테이너처럼 "영역 안에 들어왔는지"만 알면 될 때 Is Trigger 사용

### Used By Effector

- Physics Effector 컴포넌트(부력, 바람, 컨베이어 벨트 효과 등)와 연동할 때 체크

### Used By Composite

- Composite Collider 2D와 합쳐서 사용할 때 체크
- Tilemap의 여러 타일 Collider를 하나로 합칠 때 활용

### Auto Tiling

- Sprite Renderer의 Draw Mode가 Tiled일 때 스프라이트 크기에 맞게 Collider 크기 자동 조정

### Edge Radius

- 사각형 모서리에 둥근 처리를 추가
- 0이면 완전한 직각 모서리
- 값을 주면 모서리가 둥글어져서 미끄러지듯 넘어가는 물리 처리 가능

## Collider 크기와 Transform Scale 관계

```
Transform Scale (2, 2, 1) + Collider Size (1, 1)
→ 실제 충돌 영역 크기 = (2, 2)

Transform Scale (1, 1, 1) + Collider Size (2, 2)
→ 실제 충돌 영역 크기 = (2, 2)
```

결과는 같지만 Scale로 키우면 자식 오브젝트에도 영향 → Collider Size로 조정하는 게 더 안전한 경우도 있음

## 시각화

인스펙터에서 초록색 선이 실제 충돌 영역

- 스프라이트와 충돌 영역이 일치하는지 씬 뷰에서 확인 가능
- Play 모드에서도 표시되므로 디버깅에 활용

## 게임 개발 관점에서

- **발 부분만 충돌**: 캐릭터 스프라이트 전체가 아니라 발 부분에만 작은 Collider를 달아서 플랫폼 위에 서는 판정을 정밀하게 조정하는 패턴이 흔함
- **Is Trigger로 감지 범위**: 공격 범위, 아이템 획득 범위, 러닝 게임 컨테이너처럼 "닿았는지 감지"만 필요한 곳에 Is Trigger 활용
- **Offset 활용**: 스프라이트의 피벗 위치와 Collider 중심이 맞지 않을 때 Offset으로 보정
- **여러 Collider 조합**: 하나의 오브젝트에 Box Collider 여러 개를 붙여서 복잡한 충돌 형태를 만들 수 있음. 머리용 Collider + 몸통용 Collider 따로 달기

## 의문점 / 더 알아볼 것

- Composite Collider 2D — 여러 Collider를 하나로 합쳐서 성능 최적화
- Physics2D.BoxCast — Collider 없이 코드로 사각형 범위 내 오브젝트를 감지하는 방법
- Layer Collision Matrix — 특정 레이어끼리만 충돌하도록 설정
