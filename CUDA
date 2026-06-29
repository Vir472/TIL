# CUDA

## 핵심 개념

- **CUDA (Compute Unified Device Architecture)**: NVIDIA에서 만든 GPU 병렬 컴퓨팅 플랫폼 및 프로그래밍 모델
- GPU를 그래픽 외의 범용 연산(GPGPU)에 활용할 수 있게 해주는 인터페이스

## 동작 방식

- CPU(Host)가 GPU(Device)에 연산을 맡기는 구조
- **커널(Kernel)**: GPU에서 실행되는 함수 단위
- 수천 개의 스레드가 동시에 커널을 실행

```
CPU (Host)
  └─ 데이터 전송 → GPU 메모리 (VRAM)
       └─ 커널 실행 (수천 스레드 병렬)
            └─ 결과 반환 → CPU
```

## AI분야에서 활용

PyTorch, TensorFlow 등 딥러닝 프레임워크가 내부적으로 CUDA를 사용함
`model.to('cuda')` 한 줄이 이 구조를 활용하는 것

## 게임 개발에서의 활용

직접적으로 CUDA를 게임 로직에 쓰는 경우는 드물지만 간접적으로 깊게 연결됨

- **PhysX**: NVIDIA의 물리 엔진이 내부적으로 CUDA를 활용. Unity/Unreal에서 GPU 가속 물리 연산에 사용
- **AI NPC**: 게임 내 대규모 NPC 행동 시뮬레이션을 GPU에서 병렬로 처리할 때 CUDA 기반
- **Compute Shader와의 차이**: 게임 엔진 안에서 GPU 연산은 주로 Compute Shader로 처리. CUDA는 엔진 외부(AI, 툴 개발)에서 주로 사용

## 의문점 / 더 알아볼 것

- CUDA는 NVIDIA 전용인데, AMD GPU에서는 어떻게 대응하나? (ROCm)
- Unity Physics와 PhysX의 CUDA 가속 차이는?
