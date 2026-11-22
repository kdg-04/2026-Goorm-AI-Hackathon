# AI 기반 한글 문서 작성 비서

<sub>AI Mobile Document Assistant</sub>

> **"매번 같은 칸을 다시 채운다."**
>
> 한글 문서를 업로드하면 AI가 문서의 빈칸과 문항을 읽고,
> 저장된 개인정보와 이전 답변을 바탕으로 문서를 빠르게 작성하도록 돕는 서비스.
> 2026 구름(goorm) AI 해커톤 **최우수상 수상작**.

<p align="center">
  <a href="my-code/"><b>내가 쓴 코드</b></a>
  &nbsp;·&nbsp;
  <a href="docs/service-spec.pdf">서비스 명세서</a>
  &nbsp;·&nbsp;
  <a href="docs/team-roles.pdf">역할분담 가이드</a>
  &nbsp;·&nbsp;
  <a href="https://github.com/2026AigroomHackerton/backend">팀 백엔드 저장소</a>
  &nbsp;·&nbsp;
  <a href="https://github.com/2026AigroomHackerton/frontend">팀 프론트엔드 저장소</a>
</p>

| | |
|---|---|
| **프로젝트명** | AI 기반 한글 문서 작성 비서 |
| **행사** | 2026 구름(goorm) AI 해커톤 |
| **일자** | 2026년 5월 9일 ~ 10일 |
| **결과** | 🏆 **최우수상** |
| **팀** | 람머스구름 · 5인 (팀장 1 · 프론트엔드 2 · 백엔드 2) |
| **내 역할** | **백엔드 1** — 문서 업로드 · 목록 · 상세 · 저장 · 아카이브 API |
| **개발 방식** | 해커톤 MVP · 바이브코딩 보조 · 단일 데모 사용자 기준 |

> ### ⚠️ 이 저장소에 대하여
>
> 이 저장소는 **기획 문서와 개인 기여 기록**이다.
>
> 5인 팀 프로젝트이며 전체 코드는 팀 조직 저장소에 있다.
> 팀원의 작업까지 옮기지 않도록, **내가 대부분을 쓴 파일 3개만** [`my-code/`](my-code/)에 그대로 옮겼다(실행은 팀 저장소에서).
>
> - **[2026AigroomHackerton/backend](https://github.com/2026AigroomHackerton/backend)** — FastAPI · SQLite
> - **[2026AigroomHackerton/frontend](https://github.com/2026AigroomHackerton/frontend)** — React · TypeScript
>
> 팀원 이름은 기획 문서에도 역할(팀장 / 프론트1 / 백엔드1 …)로만 표기되어 있다.

---

## 1. 문제

한글 문서 양식은 **매번 같은 칸을 다시 채운다.**

- 학교·공공기관·기업 지원서 등에서 한글 양식을 반복적으로 작성해야 한다
- 이름, 전화번호, 주소, 이메일 같은 기본 정보를 **매번 직접 입력**한다
- 지원동기·자기소개·활동계획처럼 비슷한 답변을 **과거 파일에서 찾아 재작성**한다
- 작성했던 문서를 체계적으로 보관하지 않으면 나중에 다시 찾기 어렵다
- AI가 참고할 때마다 한글 파일을 다시 열어 분석하면 **속도와 비용 면에서 비효율적**이다

---

## 2. 해결 방식

### 2-1. 규칙 기반 + AI 혼합

AI가 한글 파일을 직접 조작하지 않는다.
**문서에서 필요한 정보를 이해하고 어떤 값을 넣으면 좋을지 추천하는 역할**만 맡는다.

| 처리 대상 | 방식 |
|---|---|
| 명확한 개인정보 항목 (성명·전화번호·이메일) | **규칙 기반 우선** |
| 표현이 다양한 항목 (신청자·대표자·소속정보) | AI 분류 보조 |
| 서술형 문항 (지원동기·자기소개) | AI 추천 |
| 답변 재사용 | DB 검색 + AI 재작성 |

실제 문서 반영은 **사용자의 최종 승인 절차**를 거친다.

### 2-2. AI 참조용 텍스트 캐싱 — 설계의 핵심

AI가 추천할 때마다 한글 파일을 열어 파싱하면 느리고 비용이 든다.
그래서 업로드·수정 시점에 텍스트를 미리 추출해 DB에 저장한다.

```
HWPX 업로드 → XML 텍스트 추출 → document_texts 저장
                                      ↓
                        AI 추천 시 원본이 아니라 DB 텍스트를 조회
```

| 저장 대상 | 위치 | 용도 |
|---|---|---|
| 원본 한글 파일 | `documents.file_path` | 다운로드 · 편집 원본 · 버전 관리 |
| 추출 텍스트 | `document_texts.extracted_text` | AI 분석 · 검색 |
| 문항-답변 데이터 | `answer_histories` | 이전 답변 재사용 |
| 추천 로그 | `ai_recommendations` | 무엇을 근거로 추천했는지 추적 |

**"AI 추천 → 파일 열기 → 텍스트 추출 → 분석"** 을 반복하지 않고
**"AI 추천 → DB 조회 → 재작성"** 으로 바꾸는 구조다.

### 2-3. 권한을 처음부터 설계에 넣음

그룹 문서 기능을 고려해 권한 모델을 초기 설계에 포함했다.
MVP에서는 실제 인증을 생략하되, 구조는 확장 가능하도록 남겼다.

`owner` · `admin` · `editor` · `viewer` 네 단계이며, 삭제 권한은 별도 플래그(`can_delete`)로 분리했다.

---

## 3. 내가 만든 것 — 백엔드 1

역할분담 문서상 담당 범위는 다음과 같다.

```
백엔드 1 │ 문서 업로드 / 목록 / 상세 / 저장 / 아카이브 API
         │ routers/documents.py, routers/archive.py,
           services/document_service.py, repositories/document_repository.py
```

[팀 저장소](https://github.com/2026AigroomHackerton/backend/commits?author=kdg-04)에 남은 커밋 8개다.
최종본 기준으로 내가 쓴 줄은 `archive.py` 94% · `document_repository.py` 82% · `document_service.py` 69%이고,
이 세 파일은 [`my-code/`](my-code/)에서 바로 읽을 수 있다(함수별로 누가 썼는지도 정리).

| 커밋 | 내용 |
|---|---|
| `backend1_첫_작업` | 담당 라우터 · 서비스 · 리포지토리 초기 구성 |
| `chore` | 문서 업로드 최대 크기 100MB로 상향 |
| `fix` | `_connect` 반환 타입을 `Iterator[sqlite3.Connection]`로 수정 |
| `fix` | **에러 응답도 공통 포맷(`{success, data}`)을 지키도록 수정** |
| `feat` | 문서 목록 · 단일 조회 API + 스키마 보강 |
| `feat` | 문서 텍스트 수정 API + **버전 이력 테이블** |
| `refactor` | PUT 본문을 Pydantic `BaseModel`로 단순화, 404를 `NOT_FOUND`로 통일 |
| `feat` | 아카이브 통합 조회 API + 라우터 등록 |

**응답 포맷을 성공과 실패 양쪽에서 통일한 것**이 이 작업에서 가장 신경 쓴 부분이다.
프론트엔드 2명이 같은 규칙으로 응답을 다룰 수 있어야 통합할 때 어긋나지 않는다.
역할분담 문서에도 *"API 응답 구조를 바꿀 때는 프론트엔드 담당자에게 공유한다"*가 원칙으로 적혀 있다.

---

## 4. 기술 스택

| 영역 | 선택 | 이유 |
|---|---|---|
| 프론트엔드 | React · TypeScript · Tailwind CSS | 컴포넌트 기반 UI, 타입 안정성 |
| 백엔드 | **FastAPI** | 비동기 처리, 자동 API 문서, Python AI 연동에 적합 |
| 데이터베이스 | **SQLite** | 해커톤 규모에서 별도 DB 서버 없이 빠른 구현 |
| 문서 처리 | Python `zipfile` + XML 파싱 | HWPX 내부 XML에서 텍스트 추출 |
| AI | LLM API | 문항 분석, 필드 분류, 답변 재작성 |

**HWPX를 우선 지원 대상으로 잡은 것**이 범위 판단의 핵심이었다.
HWP는 바이너리 구조라 수정·저장이 어려워 확장 기능으로 미뤘다.

---

## 5. 저장소 구성

```
2026-Goorm-AI-Hackathon/
├── README.md              이 문서
├── my-code/               내가 대부분을 쓴 백엔드 파일 3개 (팀 저장소 최종본 그대로)
│   ├── README.md          출처 · 내 줄 비율 · 함수별 작성자
│   └── app/               routers/archive.py · repositories/document_repository.py · services/document_service.py
└── docs/
    ├── service-spec.pdf   서비스 명세서 (12p) — 문제 정의 · API 17개 · DB 12테이블 · 권한 설계
    └── team-roles.pdf     협업 역할분담 가이드 (11p) — 파트별 담당 범위와 수정 금지 규칙
```

전체 소스 코드와 실행 방법은 위 **팀 조직 저장소**에 있다.

---

## 6. 현재 상태

해커톤 종료 시점에서 개발이 멈춰 있다. 실제 서비스 개발은 이어지지 않았다.

- 회원가입·로그인은 구현 범위에서 제외했고, `demo_user_id = 1` 단일 데모 사용자 기준으로 검증했다
- HWP 직접 편집, 실시간 협업 편집, 상용 수준의 개인정보 암호화는 확장 과제로 남겼다
- 그룹 권한은 구조만 설계하고 UI와 더미 데이터로 흐름을 검증했다

---

## 7. 참고

- 문서에 등장하는 개인정보 항목은 모두 예시이며, 해커톤에서는 실제 개인정보 대신 마스킹된 샘플 데이터를 사용했다
- 이 저장소는 개인 포트폴리오 목적으로 공개한다
