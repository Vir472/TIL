# TIL - Unity 스크립트 기본 구조

## 기본 구조

Unity에서 스크립트를 생성하면 자동으로 만들어지는 기본 형태

```csharp
using UnityEngine;          // Unity 핵심 기능
using System.Collections;   // Coroutine 등

public class PlayerController : MonoBehaviour
{
    void Start() { }
    void Update() { }
}
```

## 구성요소 설명

### using 지시문

- 네임스페이스를 가져오는 선언
- 매번 전체 경로를 쓰지 않아도 되게 해줌
- 자주 쓰는 네임스페이스
  | 네임스페이스 | 포함 내용 |
  |------------|---------|
  | `UnityEngine` | GameObject, Transform, Rigidbody, Input 등 Unity 핵심 |
  | `UnityEngine.UI` | Canvas, Text, Button 등 UI 관련 |
  | `UnityEngine.SceneManagement` | SceneManager (씬 전환) |
  | `System.Collections` | IEnumerator (Coroutine) |
  | `System.Collections.Generic` | List\<T\>, Dictionary\<K,V\> 등 제네릭 컬렉션 |
  | `System` | Math, DateTime 등 C# 기본 기능 |

### MonoBehaviour

- Unity 스크립트의 기반 클래스
- 상속해야 생명주기 함수(Awake, Start, Update 등) 사용 가능
- 상속해야 게임오브젝트에 컴포넌트로 부착 가능
- MonoBehaviour를 상속하지 않으면 일반 C# 클래스 — 게임오브젝트에 부착 불가

```
MonoBehaviour 상속 O → 게임오브젝트에 부착 가능, 생명주기 함수 사용 가능
MonoBehaviour 상속 X → 일반 클래스, 데이터/로직 전용 (부착 불가)
```

### 클래스 이름과 파일 이름

- **반드시 일치해야 함** — 파일명 `PlayerController.cs` → 클래스명 `PlayerController`
- 일치하지 않으면 Unity가 스크립트를 인식하지 못해 컴포넌트로 부착 불가

## 인스펙터에 노출하는 방법

```
public 변수          → 인스펙터에 자동 노출
[SerializeField]     → private이지만 인스펙터에 노출
private 변수         → 인스펙터에 안 보임
[HideInInspector]   → public이지만 인스펙터에서 숨김
```

권장 패턴은 `[SerializeField] private` — 외부 접근은 막으면서 인스펙터에서 편집 가능

## 자주 쓰는 어트리뷰트

코드 위에 `[]`로 붙이는 메타 정보

| 어트리뷰트                      | 설명                                           |
| ------------------------------- | ---------------------------------------------- |
| `[SerializeField]`              | private 필드를 인스펙터에 노출                 |
| `[HideInInspector]`             | public 필드를 인스펙터에서 숨김                |
| `[Header("제목")]`              | 인스펙터에서 구분선과 제목 표시                |
| `[Tooltip("설명")]`             | 인스펙터에서 마우스 올리면 설명 표시           |
| `[Range(min, max)]`             | 인스펙터에서 슬라이더로 범위 제한              |
| `[RequireComponent(typeof(T))]` | 이 스크립트가 특정 컴포넌트를 필요로 함을 명시 |

## MonoBehaviour 없는 일반 클래스

게임 로직이나 데이터 구조는 굳이 MonoBehaviour를 상속할 필요 없음

```
MonoBehaviour 상속
  → 게임오브젝트에 부착
  → 생명주기 함수 사용 가능
  → new로 직접 생성 불가 (AddComponent로만 생성)

일반 클래스
  → 순수 C# 클래스
  → 데이터 모델, 유틸리티, 계산 로직에 적합
  → new로 자유롭게 생성 가능
```

## 네임스페이스 직접 만들기

프로젝트가 커지면 클래스 이름 충돌 방지를 위해 네임스페이스 사용

```csharp
namespace LimbusSimulator.Combat
{
    public class BattleSimulator : MonoBehaviour { }
}

namespace LimbusSimulator.UI
{
    public class BattleUI : MonoBehaviour { }
}
```

## 게임 개발 관점에서

- **파일 하나 = 클래스 하나**: Unity에서 스크립트 파일 하나에 MonoBehaviour 클래스 하나가 관례. 여러 클래스를 한 파일에 넣을 수 있지만 유지보수가 어려워짐
- **[RequireComponent] 활용**: `[RequireComponent(typeof(Rigidbody2D))]`를 붙이면 이 스크립트를 부착할 때 Rigidbody2D가 없으면 자동으로 추가해줌. 의존 관계를 명시적으로 표현
- **[Header], [Tooltip]**: 협업할 때 인스펙터를 문서화하는 효과. 기획자가 인스펙터를 볼 때 어떤 값인지 바로 알 수 있게 해줌

## 의문점 / 더 알아볼 것

- `partial class` — 하나의 클래스를 여러 파일에 나눠서 작성하는 방법
- Assembly Definition — 스크립트를 별도 어셈블리로 분리해 컴파일 시간 단축
- `ExecuteInEditMode` / `ExecuteAlways` — 플레이 모드 아닐 때도 스크립트 실행
