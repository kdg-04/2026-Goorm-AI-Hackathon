# 내가 쓴 코드 — 백엔드 1

팀 백엔드 저장소 [2026AigroomHackerton/backend](https://github.com/2026AigroomHackerton/backend)의 최종본
([`4d94db8`](https://github.com/2026AigroomHackerton/backend/tree/4d94db8abd72e2d2dbc29b082ff7f04c3e95c895), 2026-05-10)에서
**내가 대부분을 쓴 파일 3개만** 옮겼다. 파일 내용은 한 글자도 고치지 않았다.

> - 일부만 옮겼기 때문에 **이 폴더만으로는 실행되지 않는다.** 실행 구조(`main.py`, 스키마, 다른 라우터)는 팀 저장소에 있다.
> - 파일 안에는 팀원이 쓰거나 나중에 고친 줄도 섞여 있다. 아래에 함수 단위로 나눠 적었다.
> - 커밋 이력은 가져오지 않았다. 내 커밋 8개는 [팀 저장소](https://github.com/2026AigroomHackerton/backend/commits?author=kdg-04)에 그대로 있다.

## 파일

| 파일 | 하는 일 | 내가 쓴 줄 | 원본 |
|---|---|---|---|
| [`app/routers/archive.py`](app/routers/archive.py) | 보관함 화면용 API. 내 문서를 폴더별로 묶고 외부 연동 자리를 함께 돌려줌 | 197 / 209 (94%) | [blame](https://github.com/2026AigroomHackerton/backend/blame/4d94db8abd72e2d2dbc29b082ff7f04c3e95c895/app/routers/archive.py) |
| [`app/repositories/document_repository.py`](app/repositories/document_repository.py) | 문서·본문·버전 이력 테이블을 읽고 쓰는 SQL | 341 / 416 (82%) | [blame](https://github.com/2026AigroomHackerton/backend/blame/4d94db8abd72e2d2dbc29b082ff7f04c3e95c895/app/repositories/document_repository.py) |
| [`app/services/document_service.py`](app/services/document_service.py) | 업로드 검증·DB 초기화·본문 수정 순서를 담당하는 서비스 계층 | 497 / 720 (69%) | [blame](https://github.com/2026AigroomHackerton/backend/blame/4d94db8abd72e2d2dbc29b082ff7f04c3e95c895/app/services/document_service.py) |

줄 수는 최종본에 `git blame -w`(공백 변경 제외)를 돌려 센 값이다. 내가 쓰고 팀원이 나중에 고친 줄은 팀원 것으로 셌다.

## 함수별로 누가 썼나

| 파일 | 내가 쓴 것 | 팀원과 나눠 쓴 것 | 팀원이 쓴 것 |
|---|---|---|---|
| `archive.py` | `get_archive`, `_group_local_documents_by_folder`, `_error_response` | `_success_response` | — |
| `document_repository.py` | `_connect`, `get_document_with_latest_text`, `document_exists_for_user`, `find_document_text_id`, `insert_document_text`, `update_document_text`, `get_max_version_no`, `insert_document_version`, `update_document_updated_at` | `list_active_documents_by_user` | `list_document_versions`, `get_document_text_record` |
| `document_service.py` | `_init_db`, `_connect`, `upload_document`, `update_document_text`, 예외 클래스 4개 | `list_documents`, `get_document` | `get_document_file_info`, `replace_document_file`, `reindex_document` |

## 먼저 볼 곳

1. **본문 수정 + 버전 이력** — `document_service.update_document_text`
   문서 확인 → 마지막 버전 번호 + 1 → 본문 저장 → 이력 한 줄 추가 → 수정 시각 갱신.
   서비스는 순서와 분기만 맡고, 각 단계는 저장소 함수 하나씩(`get_max_version_no`, `insert_document_version` …)으로 나눴다.
2. **실패 응답도 같은 모양으로** — `archive._error_response`
   성공이든 실패든 `{success, data, message, error}` 네 칸으로 돌려준다. 프론트엔드 2명이 응답을 한 가지 방식으로만 다루게 하려는 것.
3. **보관함 묶기** — `archive._group_local_documents_by_folder`
   문서를 `folder_id`별로 묶고, 미분류(`None`)를 맨 위에 둔다. 서비스 함수가 `folder_id`를 빼고 돌려주는 상황이라,
   그 작업에서는 서비스·저장소 파일을 고치지 않기로 해서, 라우터에서 저장소를 바로 부른 이유를 파일 맨 위 주석에 적어 두었다.

## 개발 방식

해커톤 이틀 동안 AI 코딩 도구를 보조로 썼다(팀 공식 방식, 역할분담 문서에 "바이브코딩 보조"로 적혀 있음).
