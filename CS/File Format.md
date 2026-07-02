# 파일 포맷 (File Format)

## 핵심 개념

- 파일에 데이터를 저장하는 **구조와 규칙**
- 같은 데이터라도 포맷에 따라 저장 방식, 크기, 읽는 방법이 달라짐
- 크게 **텍스트(Text)** 와 **바이너리(Binary)** 로 나뉨

## 텍스트 vs 바이너리

| 항목      | 텍스트 포맷                | 바이너리 포맷         |
| --------- | -------------------------- | --------------------- |
| 저장 방식 | 사람이 읽을 수 있는 문자열 | 0과 1의 이진 데이터   |
| 크기      | 상대적으로 큼              | 작음                  |
| 읽기      | 메모장으로 열 수 있음      | 전용 파서 필요        |
| 속도      | 파싱 느림                  | 파싱 빠름             |
| 예시      | JSON, XML, CSV, YAML       | PNG, MP3, FBX, .bytes |

## 주요 포맷들

### 데이터 직렬화

```
JSON  - 웹/API에서 표준. 사람이 읽기 쉬움
XML   - 구조적이지만 장황함. 레거시 시스템에 많이 남아있음
CSV   - 표 형태 데이터. 엑셀과 호환
YAML  - JSON보다 가독성 좋음. 설정 파일에 많이 쓰임
```

### 이미지

```
PNG  - 무손실 압축, 투명도 지원 → UI, 스프라이트
JPG  - 손실 압축, 파일 작음 → 배경, 사진
WebP - PNG/JPG 대체재, 더 작은 용량
```

### 3D/게임 에셋

```
FBX  - 3D 모델, 애니메이션 표준 포맷 (Maya, Blender → Unity 임포트)
OBJ  - 단순 3D 메시. 애니메이션 없음
GLTF - 웹 표준 3D 포맷. "3D의 JPEG"라 불림
```

### 오디오

```
WAV  - 무손실, 파일 큼 → 짧은 효과음
MP3  - 손실 압축, 파일 작음 → 배경음악
OGG  - MP3 대안. Unity 권장 오디오 포맷
```

## 직렬화 (Serialization)

파일 포맷과 직결되는 개념

- **직렬화**: 메모리의 객체를 파일/네트워크로 전송 가능한 형태(바이트)로 변환
- **역직렬화**: 반대로 파일/바이트를 다시 객체로 복원

```csharp
// C# JSON 직렬화 예시
[System.Serializable]
public class SaveData
{
    public int score;
    public string playerName;
}

// 직렬화 (객체 → JSON 문자열)
string json = JsonUtility.ToJson(saveData);

// 역직렬화 (JSON 문자열 → 객체)
SaveData loaded = JsonUtility.FromJson<SaveData>(json);
```

## 게임 개발 관점에서

- **Unity 에셋 포맷**: Unity는 임포트한 에셋을 내부적으로 `.meta` 파일과 함께 자체 포맷으로 변환해 저장. FBX를 임포트하면 Unity가 최적화된 내부 포맷으로 재가공함
- **세이브 데이터**: 게임 저장은 주로 JSON(개발/디버깅 편함) 또는 바이너리(용량 작고 치트 방지)로 구현
- **ScriptableObject**: Unity에서 게임 데이터(아이템, 스킬 스탯 등)를 에디터에서 관리하는 에셋 포맷. 내부적으로 YAML 기반의 `.asset` 파일로 저장됨

## 의문점 / 더 알아볼 것

- Unity의 AssetBundle과 Addressable은 내부적으로 어떤 포맷을 쓰는지
- Protocol Buffers(protobuf)가 JSON 대비 빠른 이유 (바이너리 직렬화)
