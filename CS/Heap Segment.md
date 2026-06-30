# 힙 영역 (Heap Segment)

## 핵심 개념

- 프로세스 메모리의 4가지 영역(Code, Data, Heap, Stack) 중 하나
- **동적으로 할당되는 메모리** 공간. 프로그램 실행 중에 크기가 정해지는 데이터를 저장
- C#에서 `new`로 생성하는 객체(클래스 인스턴스)가 저장되는 곳

## "동적"인 이유

- Stack은 함수 호출 시점에 크기가 컴파일 타임에 정해짐 (지역변수 개수, 크기 다 알고 있음)
- Heap은 런타임에 "이만큼 필요해" 하고 요청하면 그때 할당됨 → 프로그램 실행 흐름에 따라 크기가 계속 변함
- 그래서 객체를 몇 개 만들지 모르는 상황(사용자 입력, 게임 중 생성되는 적·아이템 등)에 Heap을 씀

## Stack vs Heap 비교

| 항목        | Stack                     | Heap                                |
| ----------- | ------------------------- | ----------------------------------- |
| 할당 속도   | 매우 빠름 (포인터만 이동) | 상대적으로 느림 (빈 공간 탐색 필요) |
| 크기        | 작음, 정해져 있음         | 큼, 유동적                          |
| 생존 기간   | 함수 종료 시 자동 소멸    | 명시적 해제 or GC가 처리할 때까지   |
| 접근 방식   | LIFO (후입선출)           | 임의 접근 가능                      |
| 단편화      | 없음                      | 발생 가능 (Fragmentation)           |
| 저장 데이터 | 지역변수, 값 타입(struct) | 객체(class), 동적 할당 데이터       |

## C#에서 Heap에 올라가는 것

```csharp
class Enemy  // class = 참조 타입 → Heap에 저장
{
    public int hp;
}

struct Vector2Like  // struct = 값 타입 → Stack에 저장
{
    public float x, y;
}

void Example()
{
    Enemy e = new Enemy();        // Heap에 객체 생성, e는 Stack에서 그 주소를 가리킴
    Vector2Like v = new Vector2Like(); // Stack에 직접 저장됨
    List<int> list = new List<int>();  // List 객체 자체는 Heap
}
```

핵심은 **class는 Heap, struct는 Stack**이라는 것. 정확히는 "참조 타입은 Heap, 값 타입은 Stack"이 원칙

## C#의 가비지 컬렉터(GC)가 Heap을 관리하는 방식

C#은 Heap 메모리를 수동으로 해제(`free`)하지 않음. 대신 GC가 자동으로 정리함

```
1. Mark (표시)
   → 현재 사용 중인(참조되고 있는) 객체를 추적해서 표시

2. Sweep (제거)
   → 표시되지 않은(아무도 참조 안 하는) 객체를 메모리에서 제거

3. Compact (압축) — .NET GC가 추가로 수행
   → 남은 객체들을 한쪽으로 모아서 빈 공간을 합침 (단편화 방지)
```

- .NET GC는 **세대별 수집(Generational GC)** 방식을 씀

## 게임 개발 관점에서 — 왜 Heap이 성능 이슈의 핵심인가

- **GC Spike 문제**: GC가 동작하는 순간 메인 스레드가 멈춤 → 그 프레임에서 버벅임(Hitch) 발생. 60FPS 게임에서 한 프레임이라도 밀리면 바로 체감됨
- **Heap 할당이 많아지는 안 좋은 패턴들**:

```csharp
  void Update()
  {
      Enemy e = new Enemy();           // 매 프레임 Heap 할당 — 최악
      var list = new List<int>();      // 매 프레임 새 컬렉션 생성 — 안 좋음
      string s = "Score: " + score;    // 문자열 연결도 매번 새 객체 생성
  }
```

- **해결 패턴들**:
  | 기법 | 설명 |
  |------|------|
  | 오브젝트 풀링 | 자주 생성/삭제되는 객체(총알, 이펙트)를 미리 만들어두고 재사용 |
  | struct 활용 | 값 타입으로 바꿔서 Heap 할당 자체를 피함 |
  | StringBuilder | 문자열 연결 시 매번 새 문자열 생성 방지 |
  | 캐싱 | `Update()` 밖에서 한 번만 생성하고 재사용 |
  | NonAlloc API | Unity의 `Physics.RaycastNonAlloc` 등 — 매번 새 배열 생성 안 함 |
- **Unity Profiler에서 확인 가능**: `GC Alloc` 항목으로 프레임당 Heap 할당량을 직접 확인할 수 있음. 0B에 가깝게 유지하는 게 목표

## 의문점 / 더 알아볼 것

- .NET의 Gen 0/1/2 GC가 정확히 어떤 기준으로 객체를 승급(Promotion)시키는지
- Unity의 IL2CPP + Boehm GC는 Mono의 GC와 어떻게 다른지
- 오브젝트 풀링을 구현할 때 `UnityEngine.Pool` API와 직접 구현하는 것의 차이
