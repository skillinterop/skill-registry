# skill-registry

인터옵 생태계를 위한 스킬 패키지 저장소.

## 개요

이 저장소는 스킬 패키지만 저장하는 **리프 레지스트리**입니다. 허브가 아닙니다 -- 허브는 [`registry-hub`](https://github.com/skillinterop/registry-hub)입니다. 래퍼 도구들은 이 레지스트리의 `manifest.json`을 참조하여 스킬을 검색하고 설치합니다.

## 디렉토리 구조

```
skill-registry/
├── manifest.json              # 모든 스킬 항목이 포함된 레지스트리 매니페스트
├── schemas/
│   └── manifest.schema.json   # 매니페스트 검증용 JSON Schema
├── skills/
│   └── <skill-name>/
│       ├── SKILL.md           # 스킬 문서
│       └── metadata.json      # 스킬 메타데이터
├── README.md
└── .gitignore
```

## 등록된 스킬 목록

| 정규 ID | 이름 | 버전 | 설명 |
|---------|------|------|------|
| `skill/org/workmux-router@1.0.0` | workmux-router | 1.0.0 | workmux와 git worktree를 활용한 자연어 작업 실행기 및 라우터 |

## 매니페스트 형식

`manifest.json` 파일은 LeafManifest 구조를 따릅니다:

```json
{
  "registryType": "skill",
  "namespace": "org",
  "version": "0.1.0",
  "channel": "experimental",
  "generatedAt": "2026-03-20T07:00:00Z",
  "items": [...]
}
```

## 새 스킬 추가 방법

1. `skills/<skill-name>/` 디렉토리를 생성합니다
2. 스킬 문서인 `SKILL.md`를 추가합니다
3. 스킬 메타데이터인 `metadata.json`을 추가합니다
4. `manifest.json`에 새 스킬 항목을 추가합니다
5. 변경 사항으로 PR을 생성합니다

## 정규 ID 형식

모든 스킬은 다음과 같은 정규 ID 형식을 사용합니다:

```
skill/<namespace>/<name>@<version>
```

예시: `skill/org/workmux-router@1.0.0`

## 관련 저장소

- [`registry-hub`](https://github.com/skillinterop/registry-hub) -- 리프 레지스트리들을 통합하는 최상위 허브
- [`cao-profile-registry`](https://github.com/skillinterop/cao-profile-registry) -- CAO 프로필 리프 레지스트리
- [`reprogate-registry`](https://github.com/skillinterop/reprogate-registry) -- ReproGate 리프 레지스트리

## 라이선스

MIT
