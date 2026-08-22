# ScriptableObject

## 핵심 개념

- **데이터를 에셋 파일로 저장하고 관리하는 Unity 전용 클래스**
- MonoBehaviour와 달리 게임오브젝트에 부착하지 않음 — 독립적인 에셋 파일(.asset)로 존재
- 코드에 데이터를 하드코딩하거나 static으로 들고 있지 않고, 에디터에서 데이터를 관리할 수 있게 해줌

## MonoBehaviour와의 차이

| 항목          | MonoBehaviour    | ScriptableObject           |
| ------------- | ---------------- | -------------------------- |
| 부착 대상     | 게임오브젝트     | 없음 (에셋 파일)           |
| 생명주기 함수 | Awake, Update 등 | OnEnable, OnDisable 정도만 |
| 용도          | 게임 로직        | 데이터 보관                |
| 씬 의존성     | 씬에 종속        | 씬과 무관                  |
| 인스펙터 편집 | 가능             | 가능                       |

## 왜 쓰냐면

데이터를 관리하는 방법은 여러 가지인데 각각 단점이 있음

- **코드에 하드코딩** → 수정할 때마다 재컴파일 필요. 기획자가 수정 불가
- **static 변수** → 씬 전환 후에도 남아있고 메모리에 항상 올라가 있음
- **JSON/XML 파일** → 로드 로직이 필요하고 에디터에서 바로 편집 불가
- **ScriptableObject** → 에디터에서 바로 편집 가능, 여러 오브젝트가 공유 가능, 씬과 무관하게 존재

## 생성 방법

```csharp
[CreateAssetMenu(fileName = "EnemyData", menuName = "Data/EnemyData")]
public class EnemyData : ScriptableObject
{
    public string enemyName;
    public int maxHp;
    public float moveSpeed;
    public int rewardGold;
}
```

- `[CreateAssetMenu]` 어트리뷰트로 에디터 메뉴에서 생성 가능
- Project 창에서 우클릭 → Create → Data → EnemyData

## 활용 방법

- 생성된 .asset 파일을 인스펙터에서 직접 편집
- 스크립트에서 `[SerializeField] EnemyData data;`로 참조해서 사용
- 여러 오브젝트가 같은 ScriptableObject를 참조하면 데이터 공유 가능

## 활용 예시

### 게임 데이터 관리

```
EnemyData_Slime.asset   → 이름: 슬라임, HP: 30, 속도: 2
EnemyData_Boss.asset    → 이름: 보스,   HP: 500, 속도: 1
ItemData_Sword.asset    → 이름: 검,     공격력: 20
```

데이터마다 에셋 파일을 만들어서 관리

### 게임 설정값

```
GameSettings.asset → 중력: 9.8, 최대 속도: 10, 점프력: 5
```

한 곳에서 수정하면 참조하는 모든 오브젝트에 반영

### 이벤트 시스템

ScriptableObject로 이벤트를 만들어서 씬 간 통신에 활용하는 패턴도 있음

## static과의 차이

```
static 변수
  → 코드 안에 존재. 에디터에서 편집 불가
  → 프로그램 종료까지 메모리에 상주
  → 씬 전환 후에도 유지

ScriptableObject
  → 에셋 파일로 존재. 에디터에서 편집 가능
  → 참조할 때만 메모리에 올라옴
  → 여러 씬에서 공유 가능
```

## 게임 개발 관점에서

- **기획자 협업**: 프로그래머가 ScriptableObject 틀을 만들면 기획자가 코드 없이 인스펙터에서 직접 데이터 입력 가능. 밸런싱 작업이 빨라짐
- **데이터 공유**: 같은 종류의 적 오브젝트 100개가 하나의 EnemyData를 공유하면 메모리 절약
- **프리팹과 조합**: 프리팹에 ScriptableObject 참조를 연결해두면 프리팹 자체는 그대로 두고 데이터만 교체해서 다른 종류의 오브젝트처럼 동작시킬 수 있음

## 주의사항

- **런타임 데이터 저장 금지**: ScriptableObject는 에디터에서 수정한 값이 에셋에 저장되지만, 런타임에 값을 바꾸면 플레이 종료 후 원래대로 돌아옴. 런타임 데이터 저장에는 적합하지 않음
- **빌드 포함 확인**: Resources 폴더에 넣거나 직접 참조하지 않으면 빌드에 포함 안 될 수 있음

## 의문점 / 더 알아볼 것

- ScriptableObject 기반 이벤트 시스템 (Ryan Hipple 패턴)
- Addressables와 ScriptableObject 조합 — 동적 로드와 데이터 관리
- `[CreateAssetMenu]` 없이 코드로 ScriptableObject 인스턴스 생성하는 방법
