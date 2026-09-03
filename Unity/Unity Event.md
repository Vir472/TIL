# UnityEvent

## 핵심 개념

- Unity가 제공하는 이벤트 시스템
- C# delegate/event 기반이지만 **인스펙터에서 시각적으로 연결** 가능
- 코드 수정 없이 에디터에서 이벤트에 메서드를 연결할 수 있음

## C# event vs UnityEvent 비교

| 항목          | C# event      | UnityEvent                  |
| ------------- | ------------- | --------------------------- |
| 설정 방식     | 코드에서 +=   | 코드 또는 인스펙터에서 연결 |
| 인스펙터 노출 | 불가          | 가능                        |
| 성능          | 빠름          | 약간 느림                   |
| 직렬화        | 불가          | 가능 (씬/프리팹에 저장)     |
| 디버깅        | 어려움        | 인스펙터에서 바로 확인 가능 |
| 주요 용도     | 런타임 이벤트 | UI 버튼, 인스펙터 연동      |

## 기본 사용법

### 선언

```csharp
using UnityEngine.Events;

public class Enemy : MonoBehaviour
{
    public UnityEvent OnDeath;          // 매개변수 없는 이벤트
    public UnityEvent<int> OnDamaged;   // int 매개변수 있는 이벤트
}
```

`[SerializeField]`나 `public`으로 선언하면 인스펙터에 표시됨

### 호출

```csharp
OnDeath.Invoke();          // 이벤트 발생
OnDamaged.Invoke(30);      // 매개변수와 함께 발생
```

### 코드에서 구독

```csharp
enemy.OnDeath.AddListener(OnEnemyDeath);
enemy.OnDeath.RemoveListener(OnEnemyDeath);

// 람다도 가능
enemy.OnDamaged.AddListener(dmg => UpdateUI(dmg));
```

## 인스펙터에서 연결

UnityEvent의 가장 큰 장점 — 코드 수정 없이 에디터에서 연결

```
인스펙터에서 UnityEvent 필드를 보면
+ 버튼으로 리스너 추가
오브젝트 드래그 → 메서드 선택
→ 이벤트 발생 시 해당 메서드 자동 호출
```

기획자나 디자이너가 코드 없이 이벤트 연결 가능

## 제네릭 UnityEvent

매개변수 타입을 지정하는 방법

```csharp
// UnityEvent<T> — 최대 4개 타입 매개변수 지원
UnityEvent<int>           // 매개변수 1개
UnityEvent<int, string>   // 매개변수 2개

// 인스펙터에 노출하려면 직렬화 가능한 클래스로 감싸야 함
[System.Serializable]
public class IntEvent : UnityEvent<int> { }

public IntEvent OnScoreChanged;
```

## UI Button과의 연동

Unity UI Button 컴포넌트의 OnClick이 UnityEvent 기반

```
Button 컴포넌트
  └─ OnClick (UnityEvent)
       → 인스펙터에서 메서드 연결
       → 버튼 클릭 시 자동 호출
```

Button.onClick.AddListener()로 코드에서도 연결 가능

## 주의사항

- **null 체크**: UnityEvent는 선언만 해도 null이 아니지만 Invoke 전에 리스너가 없으면 아무 동작 안 함. C# event와 달리 null 체크 불필요
- **람다 구독 해제 불가**: 람다로 AddListener하면 RemoveListener로 제거 불가. 메서드로 분리해서 구독해야 해제 가능
- **성능**: 매 프레임 Invoke하는 용도로는 적합하지 않음. 버튼 클릭, 사망, 스테이지 클리어처럼 드물게 발생하는 이벤트에 적합

## 게임 개발 관점에서

- **UI 연동**: 점수 변경, 체력 변경 같은 게임 상태 변화를 UnityEvent로 발행하면 UI가 직접 구독해서 갱신. 게임 로직과 UI 분리 가능
- **기획자 협업**: 인스펙터에서 이벤트 연결을 기획자가 직접 할 수 있어서 프로그래머 의존도 감소
- **씬 저장**: C# event는 씬에 저장이 안 되지만 UnityEvent는 직렬화되어 씬/프리팹에 저장됨. 에디터를 껐다 켜도 연결이 유지

## 의문점 / 더 알아볼 것

- `UnityAction` — UnityEvent의 내부 델리게이트 타입
- `EventSystem` — Unity UI의 입력 이벤트 처리 시스템
- ScriptableObject 기반 이벤트 시스템 — Ryan Hipple 패턴. 씬 간 이벤트 통신
