# 코딩 규칙 인덱스 (`docs/`)

> Cursor 에이전트는 **코드 작성·수정 전에 이 인덱스를 읽고**, 작업 영역에 맞는 규칙 문서를 **반드시** 참조한다.  
> 규칙은 `docs/` 밖의 추측이 아니라, 아래 경로의 Markdown을 **단일 근거(Single Source of Truth)** 로 삼는다.

---

## 읽는 순서 (에이전트)

1. **작업 루트의 `.cursorrules`** — 하네스·docs 필수 참조 여부
2. **본 파일** (`docs/README.md`) — 영역별 규칙 경로 확인
3. **해당 영역 규칙 MD** — 아래 표에서 1개 이상 **전문 읽기**
4. **도메인 컨텍스트** (해당 시) — 타이타닉·앱별 문서
5. 그 다음에만 `backend/` 또는 `frontend/` 코드 변경

규칙 문서를 읽지 않고 코드를 작성하지 않는다.

---

## 규칙 문서 목록

| 영역 | 경로 | 적용 대상 |
|------|------|-----------|
| **백엔드 (FastAPI)** | [`DevOps/Backend/PYTHON_FASTAPI_RULES.md`](DevOps/Backend/PYTHON_FASTAPI_RULES.md) | `backend/`, `backend/apps/` |
| **백엔드 (엔티티·PK)** | [`DevOps/Backend/ENTITY_RULE.md`](DevOps/Backend/ENTITY_RULE.md) | `backend/**/models/`, DB 스키마 |
| **프론트 (React)** | [`DevOps/Frontend/REACT_RULES.md`](DevOps/Frontend/REACT_RULES.md) | `frontend/` |
| **타이타닉 도메인** | [`타이타닉 개발/james_fastapi_context.md`](타이타닉%20개발/james_fastapi_context.md) | `backend/apps/titanic/` |

---

## Cursor에서 참조하는 방법

```
@docs/README.md
@docs/DevOps/Backend/PYTHON_FASTAPI_RULES.md
```

프론트 작업:

```
@docs/README.md
@docs/DevOps/Frontend/REACT_RULES.md
```

---

## 규칙 추가·변경 시

- 새 스택/도메인 규칙은 `docs/DevOps/<영역>/` 아래 MD로 추가한다.
- **본 인덱스 표를 반드시 갱신**한다.
- `backend/.cursorrules` 또는 `frontend/.cursorrules`에 새 경로를 반영한다.

---

## 상위 하네스 (프로젝트 공통)

- 저장소 루트 `CLAUDE.md` — Karpathy 정렬 네 원칙 (상세)
- `backend/.cursorrules` — 백엔드 Cursor 하네스 + **docs 필수**
- `backend/CURSOR.md` — Cursor 하네스 운영 설명
