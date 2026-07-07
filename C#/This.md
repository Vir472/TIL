# this 키워드

## 핵심 개념

- **현재 인스턴스(자기 자신)를 가리키는 참조**
- 클래스나 struct 안에서 "나 자신"을 명시적으로 참조할 때 사용
- 컴파일러가 대부분 자동으로 처리하지만, 명시적으로 써야 하는 상황이 있음

## 1. 매개변수와 필드 이름이 겹칠 때

가장 흔한 사용 케이스

```csharp
class Enemy
{
    public int hp;
    public string name;

    public Enemy(int hp, string name)
    {
        this.hp = hp;       // this.hp = 필드, hp = 매개변수
        this.name = name;   // 이름이 같을 때 구분하기 위해 this 사용
    }
}

// this 없이 쓰면?
public Enemy(int hp, string name)
{
    hp = hp;    // 매개변수 = 매개변수 (필드에 할당 안 됨 — 버그!)
}
```

## 2. 현재 인스턴스를 다른 메서드에 전달할 때

```csharp
class Character
{
    public string name;

    public void Register()
    {
        GameManager.RegisterCharacter(this);  // 자기 자신을 넘김
    }
}

class GameManager
{
    public static void RegisterCharacter(Character c)
    {
        Console.WriteLine(c.name + " 등록됨");
    }
}
```

## 3. 메서드 체이닝 (Fluent Interface)

`this`를 반환해서 메서드를 연속으로 호출하는 패턴

```csharp
class CharacterBuilder
{
    private string name;
    private int hp;
    private int speed;

    public CharacterBuilder SetName(string name)
    {
        this.name = name;
        return this;  // 자기 자신 반환
    }

    public CharacterBuilder SetHp(int hp)
    {
        this.hp = hp;
        return this;
    }

    public CharacterBuilder SetSpeed(int speed)
    {
        this.speed = speed;
        return this;
    }
}

// 사용 시
var builder = new CharacterBuilder()
    .SetName("이상봉")
    .SetHp(100)
    .SetSpeed(5);   // 메서드 체이닝 가능
```

## 4. this() — 생성자 체이닝

다른 생성자를 호출해서 코드 중복 줄이기

```csharp
class Enemy
{
    public string name;
    public int hp;
    public int speed;

    // 기본 생성자
    public Enemy(string name, int hp, int speed)
    {
        this.name = name;
        this.hp = hp;
        this.speed = speed;
    }

    // 속도 기본값을 가진 생성자 — this()로 위 생성자 호출
    public Enemy(string name, int hp) : this(name, hp, 5)
    {
        // 추가 초기화 없이 위 생성자에 위임
    }

    // 이름만 받는 생성자
    public Enemy(string name) : this(name, 100)
    {
        // hp 기본값 100, speed 기본값 5
    }
}

// 사용
var e1 = new Enemy("슬림");           // hp=100, speed=5
var e2 = new Enemy("보스", 500);      // speed=5
var e3 = new Enemy("엘리트", 300, 8); // 모두 지정
```

## 5. 인덱서 (Indexer)

`this[index]` 형태로 클래스를 배열처럼 사용 가능하게 하는 문법

```csharp
class SkillSlot
{
    private string[] skills = new string[4];

    // 인덱서 정의
    public string this[int index]
    {
        get => skills[index];
        set => skills[index] = value;
    }
}

// 사용
var slot = new SkillSlot();
slot[0] = "베기";
slot[1] = "방어";
Console.WriteLine(slot[0]);  // "베기"
```

## struct에서 this

struct에서 `this`는 값 타입이므로 인스턴스 자체를 복사본으로 가리킴

```csharp
struct Point
{
    public int x, y;

    // struct에서 this 재할당 가능 (class에서는 불가)
    public void Reset()
    {
        this = new Point { x = 0, y = 0 };  // struct에서만 가능
    }
}
```

## 요약

| 사용 케이스                  | 예시                   |
| ---------------------------- | ---------------------- |
| 필드/매개변수 이름 충돌 해결 | `this.hp = hp`         |
| 자기 자신을 인수로 전달      | `Method(this)`         |
| 메서드 체이닝                | `return this`          |
| 생성자 체이닝                | `: this(...)`          |
| 인덱서 정의                  | `public T this[int i]` |

## 게임 개발 관점에서

- **MonoBehaviour 생성자**: Unity에서 MonoBehaviour는 생성자를 직접 쓰지 않고 `Awake()`/`Start()`를 쓰는데, 이 안에서 `this`로 자기 자신을 다른 매니저에 등록하는 패턴이 흔함

```csharp
  var character = new CharacterBuilder()
      .SetName("이상봉")
      .SetSkills(skillList)
      .SetSanity(45)
      .Build();
```

- **인덱서 활용**: 스킬 슬롯, 덱 슬롯처럼 순서가 있는 데이터를 클래스로 감쌀 때 인덱서로 배열처럼 접근 가능하게 하면 코드가 직관적으로 됨

## 의문점 / 더 알아볼 것

- `this` 캡처와 람다식 — 람다 안에서 `this`를 쓰면 클로저가 생기는 구조
- Unity에서 `this`를 null 체크하는 특이한 패턴 (`if (this == null)`)이 존재하는 이유
