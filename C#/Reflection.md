# 리플렉션 (Reflection)

## 핵심 개념

- **리플렉션**: 실행 중에 타입 정보를 조사하고 조작하는 기능
- 컴파일 시점에 몰랐던 타입의 필드·메서드를 이름(문자열)으로 찾아 읽고 호출할 수 있음
- 가능한 이유는 어셈블리에 **메타데이터**가 함께 들어있기 때문임. 코드(IL)뿐 아니라 "어떤 타입에 어떤 멤버가 있는지"가 같이 저장됨

```mermaid
graph LR
    A["소스 코드"] --> B["컴파일"]
    B --> C["어셈블리<br/>IL + 메타데이터"]
    C --> D["런타임에<br/>Type 객체로 조회"]
    D --> E["멤버 목록 읽기<br/>동적 생성 · 호출"]
```

## Type 얻기

```csharp
Type t1 = typeof(Player);              // 컴파일 시점에 타입을 알 때
Type t2 = player.GetType();            // 인스턴스에서 실제 타입을 얻을 때
Type t3 = Type.GetType("Game.Player"); // 문자열로 찾을 때 (실패하면 null)
```

- `typeof`는 선언된 타입, `GetType()`은 **실제 런타임 타입**을 줌
- 업캐스팅된 상태에서 진짜 타입을 알아내는 데 `GetType()`이 쓰임

## 멤버 조회

```csharp
Type t = typeof(Player);

FieldInfo[] fields   = t.GetFields();
MethodInfo[] methods = t.GetMethods();
PropertyInfo[] props = t.GetProperties();

// private까지 보려면 BindingFlags 필요
FieldInfo hp = t.GetField("hp",
    BindingFlags.NonPublic | BindingFlags.Instance);
```

- 인자 없이 부르면 **public 멤버만** 나옴
- `BindingFlags`로 `NonPublic`, `Static`, `Instance` 등을 조합해 범위를 지정함

## 동적 생성과 호출

```csharp
// 인스턴스 생성
object obj = Activator.CreateInstance(typeof(Player));

// 필드 읽기 / 쓰기
int value = (int)hp.GetValue(obj);
hp.SetValue(obj, 100);

// 메서드 호출
MethodInfo m = t.GetMethod("TakeDamage");
m.Invoke(obj, new object[] { 10 });
```

- 접근 제한자를 무시하고 private 멤버에도 접근할 수 있음
- 캡슐화를 뚫는 행위이므로 일반 로직에는 쓰지 말 것. 도구·프레임워크 영역의 기능임

## 어트리뷰트 읽기

리플렉션의 가장 흔한 용도임. 어트리뷰트는 **리플렉션으로 읽지 않으면 아무 의미가 없음**.

```csharp
[AttributeUsage(AttributeTargets.Method)]
public class CommandAttribute : Attribute
{
    public string Name;
    public CommandAttribute(string name) => Name = name;
}

// 이 어트리뷰트가 붙은 메서드를 전부 찾기
foreach (var method in t.GetMethods())
{
    var attr = method.GetCustomAttribute<CommandAttribute>();
    if (attr != null) Register(attr.Name, method);
}
```

- 코드에 표시만 해두면 시스템이 알아서 수집하는 구조를 만들 수 있음
- 치트 콘솔, 이벤트 자동 등록 같은 것이 이 방식으로 구현됨

## 비용

- 직접 호출 대비 **수십~수백 배 느림**. 메타데이터 탐색과 인자 배열 생성, 박싱이 겹침
- `Invoke`는 인자를 `object[]`로 받으므로 값 타입이 박싱됨
- 완화 방법
  - `MemberInfo`를 매번 찾지 말고 **캐싱**. 조회 비용이 호출 비용보다 큼
  - `Delegate.CreateDelegate`로 델리게이트를 만들어두면 직접 호출에 가까워짐
  - Expression Tree나 소스 제너레이터로 아예 코드를 생성하는 방법도 있음

## 게임 개발 관점에서

**인스펙터가 리플렉션으로 동작함**

- `[SerializeField]`, `[Header]`, `[Range]`는 전부 어트리뷰트임
- 유니티 에디터가 리플렉션으로 필드와 어트리뷰트를 훑어서 인스펙터 UI를 그림
- 커스텀 에디터나 프로퍼티 드로어를 만들 때 직접 쓰게 됨

**직렬화**

- `JsonUtility`, Newtonsoft.Json, 세이브/로드 시스템이 리플렉션 기반임
- 클래스에 필드를 추가하면 저장 코드를 고치지 않아도 되는 이유가 이것임

**IL2CPP 코드 스트리핑 주의**

- IL2CPP 빌드는 **참조되지 않는 코드를 제거함**
- 리플렉션으로만 접근하는 타입·멤버는 "아무도 안 쓴다"고 판단되어 빌드에서 사라질 수 있음
- 에디터에서는 잘 되는데 빌드에서만 터지는 대표적 원인임
- 대응 — `link.xml`로 보존 대상을 지정하거나 `[Preserve]` 어트리뷰트를 붙임

**런타임 남발 금지**

- `Update`나 매 프레임 도는 경로에서 리플렉션을 쓰면 프레임을 그대로 날림
- 초기화 시점에 한 번 훑어 캐싱하고, 이후에는 캐싱된 델리게이트만 호출하는 구조로 짤 것

**적합한 자리**

- 에디터 툴, 디버그 콘솔, 세이브/로드, 데이터 테이블 로딩처럼 **개발 편의와 초기화 영역**
- 게임플레이 로직에는 쓰지 않는 것이 기본임

## 의문점 / 더 알아볼 것

- `Activator.CreateInstance`와 `new`의 성능 차이, 그리고 제네릭 제약 `where T : new()`
- Expression Tree — 런타임에 코드를 만들어 리플렉션 비용을 없애는 방법
- 소스 제너레이터 — 컴파일 시점에 코드를 생성해 리플렉션 자체를 없애는 최신 방향
- AOT(IL2CPP) 환경에서 동작하지 않는 리플렉션 기능은 무엇인가
- `link.xml` 작성 규칙과 스트리핑 레벨 설정
- 유니티가 프로퍼티를 직렬화하지 않는 이유 — 리플렉션으로는 읽을 수 있는데도 제외하는 배경
- DI 컨테이너(VContainer, Zenject)가 리플렉션을 쓰는 방식과 그 비용
