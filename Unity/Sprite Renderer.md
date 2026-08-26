# Sprite Renderer

## 핵심 개념

- 2D 스프라이트를 화면에 렌더링하는 컴포넌트
- 2D 게임오브젝트에 이미지를 표시하려면 반드시 필요
- Sprite, Color, Sorting Layer 등을 제어

## 주요 프로퍼티

### Sprite

- 표시할 스프라이트 이미지 지정
- 코드에서 `spriteRenderer.sprite = 다른스프라이트`로 런타임에 변경 가능
- 애니메이션은 이 Sprite 값을 프레임마다 바꾸는 방식으로 동작

### Color

- 스프라이트에 색상을 곱해서 표현
- 기본값 흰색(1,1,1,1) — 원본 이미지 그대로
- 알파값으로 투명도 조절 가능
- 피격 시 빨간색으로 깜빡이는 연출에 활용

### Flip

- X / Y 축으로 스프라이트를 뒤집는 설정
- 왼쪽 이동 시 `flipX = true`로 캐릭터 방향 전환
- 별도의 좌우 반전 스프라이트를 만들 필요 없음

### Sorting Layer / Order in Layer

- 여러 스프라이트의 렌더링 순서(앞/뒤) 결정
- Sorting Layer — 레이어 그룹 (Background, Default, UI 등)
- Order in Layer — 같은 레이어 안에서 순서. 숫자가 클수록 앞에 렌더링

```
Sorting Layer: Background, Order: 0  → 가장 뒤
Sorting Layer: Default,    Order: 0  → 중간
Sorting Layer: Default,    Order: 1  → Default보다 앞
Sorting Layer: UI,         Order: 0  → 가장 앞
```

### Draw Mode

- **Simple** — 기본값. Transform Scale로 크기 조절
- **Sliced** — 9-Slice. 모서리는 그대로, 중앙만 늘어남 (UI 버튼 등)
- **Tiled** — 이미지를 반복해서 채움 (바닥 타일 등)

## 코드에서 자주 쓰는 패턴

```
// 방향 전환
spriteRenderer.flipX = moveDirection < 0;

// 피격 연출
spriteRenderer.color = Color.red;

// 투명도 조절
Color c = spriteRenderer.color;
c.a = 0.5f;
spriteRenderer.color = c;

// 스프라이트 교체
spriteRenderer.sprite = newSprite;
```

## 게임 개발 관점에서

- **Color.a로 페이드 효과**: 알파값을 서서히 줄이면 오브젝트가 서서히 사라지는 연출 가능. Coroutine과 조합
- **flipX로 방향 전환**: 러닝 게임에서 장애물이 좌우 방향이 있다면 flipX로 간단하게 처리
- **Sorting Layer 설계**: 배경 → 지형 → 캐릭터 → 이펙트 → UI 순서로 레이어를 미리 설계해두면 렌더링 순서 문제를 예방

## 의문점 / 더 알아볼 것

- Sprite Atlas — 여러 스프라이트를 하나의 텍스처로 합쳐서 Draw Call 줄이는 방법
- Material — SpriteRenderer에 커스텀 셰이더를 적용해 특수 효과 구현
- GPU Instancing과 SpriteRenderer 조합
