# TIL - Unity Camera Projection (투영 방식)

## 핵심 개념

- 3D 공간의 오브젝트를 2D 화면에 표현하는 방식
- Unity 카메라 컴포넌트의 **Projection** 설정으로 선택
- **Orthographic** (정사영) 과 **Perspective** (원근 투영) 두 가지

## 수학적 배경

```
Orthographic (정사영, 직교 투영)
  투영선이 모두 평행 + 투영면에 수직
  → 거리와 무관하게 크기 보존
  → 수학의 정사영(orthogonal projection) 개념과 동일

Perspective (원근 투영)
  투영선이 카메라(시점) 한 점에서 퍼져나감
  → 멀수록 작게, 가까울수록 크게
  → 소실점(Vanishing Point)이 생김
```

```
Orthographic              Perspective

| | | | | |               \    |    /
| | | | | |                \   |   /
| | | | | |                 \  |  /
────────────                 \ | /
   카메라                   카메라(시점)

투영선이 평행               투영선이 한 점에서 퍼짐
```

## Orthographic (정사영)

### 특징

- 카메라와의 거리에 관계없이 오브젝트 크기 일정
- 원근감 없음 — 평면적으로 보임
- Z축 위치는 렌더링 순서(앞/뒤)에만 영향

### 설정값

- **Size** — 카메라에 보이는 세로 범위의 절반 크기. Size=5이면 세로 10유닛이 화면에 표시됨

### 적합한 용도

- 2D 게임 (플랫포머, 러닝 게임, RPG)
- 탑다운 게임
- UI 카메라
- 전략 시뮬레이션

## Perspective (원근 투영)

### 특징

- 멀리 있을수록 작게, 가까울수록 크게 보임
- 원근감 있음 — 현실감 있는 3D 표현
- 소실점 발생

### 설정값

- **Field of View (FOV)** — 시야각 (기본 60도). 넓을수록 더 넓은 범위가 보이지만 왜곡 증가
- **Near / Far Clip Plane** — 렌더링 범위. 이 범위 밖의 오브젝트는 렌더링 안 됨

### 적합한 용도

- 3D 게임 전반
- FPS / TPS
- 3D 플랫포머

## 비교 요약

| 항목        | Orthographic      | Perspective   |
| ----------- | ----------------- | ------------- |
| 수학 개념   | 정사영            | 원근 투영     |
| 크기 변화   | 거리와 무관, 일정 | 멀수록 작아짐 |
| 원근감      | 없음              | 있음          |
| 주요 설정값 | Size              | Field of View |
| 적합한 게임 | 2D, 탑다운        | 3D            |
| 소실점      | 없음              | 있음          |

## Clipping Plane (클리핑 평면)

두 방식 공통으로 적용되는 렌더링 범위 설정

- **Near Clip Plane** — 이 거리보다 가까운 오브젝트는 렌더링 안 됨
- **Far Clip Plane** — 이 거리보다 먼 오브젝트는 렌더링 안 됨
- 불필요한 렌더링을 줄여 성능 최적화에 활용

## 2D 게임에서 Perspective 쓰면 생기는 문제

2D 게임에서 실수로 Perspective를 쓰면

- Z축 위치에 따라 오브젝트 크기가 달라져서 의도치 않은 원근감 발생
- 스프라이트 크기가 Z값에 따라 달라 보임
- 러닝 게임에서 바닥 타일이 거리에 따라 크기가 달라 보이는 문제

## 게임 개발 관점에서

- **2D + Orthographic 조합**: Unity에서 2D 프로젝트 생성 시 기본으로 Orthographic으로 설정됨
- **Render Texture**: 한 카메라를 Orthographic, 다른 카메라를 Perspective로 설정해서 미니맵 같은 효과를 만들 수 있음
- **FOV 연출**: Perspective에서 FOV를 런타임에 변경하면 줌인/줌아웃 효과. 긴장감 연출에 활용

## 의문점 / 더 알아볼 것

- 투영 행렬(Projection Matrix) — Orthographic과 Perspective가 내부적으로 4x4 행렬로 표현되는 방식
- Oblique Projection — 비스듬한 각도의 정사영. 아이소메트릭 게임에 활용
- Cinemachine — 카메라 움직임을 자동화하는 Unity 패키지
