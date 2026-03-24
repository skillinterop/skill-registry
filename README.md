# skill-registry

인터옵 생태계를 위한 스킬 패키지 저장소.

## 개요

이 저장소는 스킬 패키지만 저장하는 **리프 레지스트리**입니다. 허브가 아닙니다 -- 허브는 [`registry-hub`](https://github.com/skillinterop/registry-hub)입니다. 소비자 도구는 이 저장소의 `index.jsonld`를 읽어 스킬을 발견하고, 각 항목의 `url`로 실제 `SKILL.md`를 해석합니다.

## 디렉토리 구조

```
skill-registry/
├── index.jsonld               # 모든 스킬 항목이 포함된 공개 카탈로그
├── schemas/
│   ├── index.schema.json      # index.jsonld 검증용 JSON Schema
│   └── manifest.schema.json   # 레거시 manifest 계약 (Phase 3 전까지 유지)
├── skills/
│   └── <skill-name>/
│       └── SKILL.md           # 스킬 문서
├── README.md
└── .gitignore
```

## 등록된 스킬 목록

| 정규 ID | 이름 | 버전 | 설명 |
|---------|------|------|------|
| `skill/org/workmux-router@1.0.0` | workmux-router | 1.0.0 | workmux와 git worktree를 활용한 자연어 작업 실행기 및 라우터 |

## 카탈로그 형식

`index.jsonld` 파일은 JSON-LD `DataCatalog` 구조를 따릅니다:

```json
{
  "@type": "DataCatalog",
  "name": "Skill Registry",
  "dataset": [
    {
      "@type": "SoftwareSourceCode",
      "identifier": "skill/org/workmux-router@1.0.0",
      "url": "./skills/workmux-router/SKILL.md",
      "skillinterop:status": "active",
      "skillinterop:channel": "stable"
    }
  ]
}
```

## 새 스킬 추가 방법

1. `skills/<skill-name>/` 디렉토리를 생성합니다
2. 스킬 문서인 `SKILL.md`를 추가합니다
3. `index.jsonld`의 `dataset` 배열에 새 항목을 추가합니다
4. `schemas/index.schema.json` 기준으로 `index.jsonld`를 검증합니다
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
