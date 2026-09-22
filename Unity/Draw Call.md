# TIL - Draw Call

## 핵심 개념

- **Draw Call**: CPU가 GPU에게 "이 메시를 이 머티리얼로 그려라"고 보내는 렌더링 명령 단위
- Draw Call 자체보다 **Draw Call을 준비하는 CPU의 오버헤드**가 성능 병목의 주원인
- Draw Call이 많아질수록 CPU가 바빠지고 GPU는 오히려 놀게 되는 역설적인 상황 발생

## 동작 흐름

```
CPU
① 씬에서 렌더링할 오브젝트 수집
② 각 오브젝트의 메시, 머티리얼, 트랜스폼 정보 준비
③ GPU에 Draw Call 전송 (시스템 버스 통해)

GPU
④ Draw Call 수신
⑤ 버텍스 셰이더 실행
⑥ 픽셀 셰이더 실행
⑦ 화면에 출력
```

오브젝트 100개 = Draw Call 최소 100번 → CPU가 100번 준비하고 100번 전송

## 왜 Draw Call이 많으면 느린가

```
Draw Call 1번당 발생하는 CPU 작업
├─ 렌더 상태 변경 (머티리얼, 셰이더 설정)
├─ 상수 버퍼 업데이트 (트랜스폼, 색상 등)
├─ 드라이버 호출 (DirectX / OpenGL / Metal API)
└─ GPU 커맨드 버퍼에 명령 기록

→ Draw Call 하나당 CPU 비용이 고정으로 발생
→ 오브젝트가 아무리 단순해도 이 비용은 동일
```

## 최적화 기법

### Static Batching

씬 로드 시 움직이지 않는 오브젝트들을 하나의 메시로 합쳐두는 방식

- 같은 머티리얼을 쓰는 정적 오브젝트들을 미리 병합
- 런타임 비용 없음 — 대신 메모리 사용량 증가
- Unity에서 오브젝트에 Static 체크박스로 활성화

### Dynamic Batching

런타임에 조건이 맞는 오브젝트들을 자동으로 묶어 한 번에 그리는 방식

- 버텍스 수가 적은 오브젝트에만 적용 (Unity 기준 900 버텍스 이하)
- 같은 머티리얼을 써야 함
- CPU에서 합치는 작업이 있어서 오히려 느려질 수 있음

### GPU Instancing

같은 메시 + 같은 머티리얼인 오브젝트를 하나의 Draw Call로 여러 개 그리는 방식

- 나무, 풀, 군중처럼 동일한 오브젝트가 대량으로 필요할 때 효과적
- 위치/색상 등 인스턴스별 데이터는 별도로 전달 가능
- 머티리얼에서 Enable GPU Instancing 체크로 활성화

### SRP Batcher (Unity)

셰이더가 같으면 머티리얼이 달라도 묶어서 처리하는 Unity 전용 최적화

- 기존 배칭은 머티리얼까지 같아야 했지만 SRP Batcher는 셰이더만 같으면 됨
- URP/HDRP에서 기본 활성화

## 확인 방법

```
Unity Stats 창 (Game 뷰 → Stats)
  Batches  — 실제 Draw Call 수
  Saved by batching — 배칭으로 줄어든 Draw Call 수

Frame Debugger (Window → Analysis → Frame Debugger)
  프레임당 Draw Call을 하나씩 순서대로 확인 가능
  어떤 오브젝트가 Draw Call을 많이 쓰는지 파악
```

## 게임 개발 관점에서

- **모바일 기준**: 모바일은 드라이버 오버헤드가 크기 때문에 Draw Call에 더 민감. 일반적으로 100 이하를 목표로 함
- **PC 기준**: 수천 개도 감당 가능하지만 여전히 줄이는 게 좋음
- **텍스처 아틀라스**: 여러 스프라이트를 하나의 텍스처에 모으면 머티리얼이 통일되어 배칭 가능 → Draw Call 감소

## 의문점 / 더 알아볼 것

- Indirect Draw — CPU 개입 없이 GPU가 직접 Draw Call을 생성하는 방식
- Vulkan / Metal / DX12에서 Draw Call 오버헤드가 줄어든 이유 (멀티스레드 커맨드 버퍼)
