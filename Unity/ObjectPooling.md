# TIL - Unity 오브젝트 풀링 (Object Pooling)

## 핵심 개념

- Instantiate/Destroy 반복으로 발생하는 **GC Spike를 방지**하는 패턴
- 미리 오브젝트를 생성해두고 필요할 때 꺼내 쓰고, 필요 없어지면 반납하는 방식
- 같은 오브젝트가 반복적으로 생성/파괴되는 구조에 적합

## 기존 방식 vs 풀링 방식

```
기존 방식
필요할 때 → Instantiate (Heap 할당)
필요 없을 때 → Destroy (GC 대상)
→ 반복될수록 GC Spike 발생 위험

풀링 방식
게임 시작 시 → 오브젝트 N개 미리 생성 후 비활성화
필요할 때 → 풀에서 꺼내기 + SetActive(true)
필요 없을 때 → SetActive(false) + 풀에 반납
→ Heap 할당 없음, GC 부담 없음
```

## 풀링이 효과적인 경우

- 총알, 이펙트처럼 자주 생성/파괴되는 오브젝트
- 러닝 게임의 바닥 타일처럼 같은 오브젝트가 반복 재사용되는 경우
- 적 오브젝트처럼 일정 개수가 계속 유지되는 경우

## 풀링이 필요 없는 경우

- 게임 전체에서 한두 번만 생성되는 오브젝트
- 씬 전환처럼 어차피 전체가 초기화되는 경우

## OnEnable / OnDisable과 풀링

풀에서 꺼낼 때 `SetActive(true)` → OnEnable 자동 호출
풀에 반납할 때 `SetActive(false)` → OnDisable 자동 호출

오브젝트 재사용 시 초기화(위치, HP, 상태 등)를 OnEnable에서 처리하면
Instantiate 없이도 새 오브젝트처럼 동작하게 만들 수 있음

## Unity 내장 풀 (Unity 2021+)

`UnityEngine.Pool` 네임스페이스에서 공식 풀 API 제공

- `ObjectPool<T>` — 가장 많이 쓰는 범용 풀
- `CollectionPool<T>` — List, Dictionary 등 컬렉션 풀
- 직접 구현하지 않아도 되는 공식 솔루션

## 러닝 게임 적용 구조

```
게임 시작
  → 바닥 타일 N개 미리 생성 + 비활성화 → 풀에 보관

바닥 타일 필요할 때 (새 타일 배치)
  → 풀에서 꺼내기 → 위치 설정 → SetActive(true)

바닥 타일 범위 벗어날 때 (OnTriggerExit)
  → SetActive(false) → 풀에 반납
```

Instantiate/Destroy가 전혀 없어서 런타임 Heap 할당 없음

## 주의사항

- **풀 크기 설정**: 풀이 너무 작으면 꺼낼 오브젝트가 없을 때 결국 Instantiate 필요. 최대 동시 사용 개수를 고려해서 크기 설정
- **반납 누락**: 오브젝트를 꺼내고 반납을 안 하면 풀이 고갈됨. 반드시 사용 후 반납 로직 보장
- **상태 초기화**: 재사용 시 이전 상태(위치, 속도, HP 등)가 남아있지 않도록 OnEnable에서 초기화 필수

## 게임 개발 관점에서

- **GC Spike 체감**: 총알처럼 초당 수십 개씩 생성/파괴되는 오브젝트에서 풀링 적용 전후 차이가 극명하게 남. Unity Profiler의 GC Alloc 항목으로 확인 가능
- **프리팹과 조합**: 풀에서 꺼낼 오브젝트의 원본은 프리팹으로 관리. 게임 시작 시 프리팹을 N번 Instantiate해서 풀 초기화

## 의문점 / 더 알아볼 것

- `ObjectPool<T>`의 maxSize 초과 시 동작 (기본적으로 초과분은 Destroy)
- 비동기 풀링 — 오브젝트를 백그라운드에서 미리 생성해두는 방식
- Addressables와 풀링 조합 — 에셋을 동적 로드하면서 풀링하는 구조
