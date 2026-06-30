# Compute Shader

## 핵심 개념

- **Compute Shader**: GPU에서 그래픽 렌더링과 무관한 범용 연산을 수행하기 위한 셰이더
- 렌더링 파이프라인(버텍스 → 픽셀)에 묶이지 않고 GPU의 병렬 연산 능력만 활용

## 기존 셰이더와의 차이

| 항목       | 일반 셰이더 (Vertex/Fragment) | Compute Shader          |
| ---------- | ----------------------------- | ----------------------- |
| 목적       | 렌더링                        | 범용 연산               |
| 입출력     | 버텍스/픽셀                   | 버퍼, 텍스처 (자유로움) |
| 파이프라인 | 렌더링 파이프라인에 종속      | 독립적                  |

## Unity에서의 활용

- Unity에서 Compute Shader를 직접 작성해 GPU 연산을 활용 가능
- 활용 예: 파티클 시뮬레이션, 군중 AI, 절차적 지형 생성, 물리 시뮬레이션

```hlsl
// Unity Compute Shader 예시 구조
#pragma kernel CSMain

RWTexture2D<float4> Result;

[numthreads(8,8,1)]
void CSMain (uint3 id : SV_DispatchThreadID)
{
    Result[id.xy] = float4(id.x & id.y, (id.x & 15)/15.0, (id.y & 15)/15.0, 0.0);
}
```

## 게임 개발에서의 활용

게임에서 CPU로 처리하기엔 너무 무거운 연산을 GPU로 넘길 때 사용

| 활용 사례       | 설명                                              |
| --------------- | ------------------------------------------------- |
| 파티클 시스템   | 수만 개 파티클 위치/속도 연산을 GPU에서 병렬 처리 |
| GPU Skinning    | 캐릭터 애니메이션 본(Bone) 연산을 GPU로 오프로드  |
| 군중 시뮬레이션 | 수백~수천 NPC 이동 경로 동시 계산                 |
| 절차적 지형     | 노이즈 기반 지형 생성 연산                        |
| 포스트 프로세싱 | 블러, 글로우, 색보정 등 화면 전체 픽셀 연산       |

## 의문점 / 더 알아볼 것

- Compute Shader와 CUDA의 차이는? (CUDA는 NVIDIA 전용, Compute Shader는 DirectX/OpenGL 표준)
- GPU Instancing과 Compute Shader를 같이 쓰면 어떤 시너지가 있나?
