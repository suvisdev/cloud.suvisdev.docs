# Python / FastAPI 백엔드 규칙 (Cursor)

> **용도:** `backend/` 작업 시 `@docs/DevOps/Backend/PYTHON_FASTAPI_RULES.md` 로 참조한다.  
> 진입 전 `docs/README.md` 인덱스를 먼저 확인한다.

---

## 1. 레이어 구조 (필수)

신규 API·도메인 기능은 **Controller → Service → Repository** 순서를 따른다.

| 레이어 | 역할 | 위치 예 |
|--------|------|---------|
| **Controller** | HTTP 진입, 스키마 수신, Service 호출, 로깅 | `{app}/app/controllers/*_controller.py` |
| **Service** | 유스케이스·트랜잭션 조율 (얇게 유지) | `{app}/app/services/*_service.py` |
| **Repository** | DB 접근, SQLAlchemy 쿼리 | `{app}/app/repositories/*_repository.py` |
| **Schema** | Pydantic 요청/응답 | `{app}/app/schemas/*_schema.py` |
| **Model** | SQLAlchemy ORM (`id` int PK — [`ENTITY_RULE.md`](ENTITY_RULE.md)) | `{app}/app/models/*_model.py` |

**금지**

- `main.py` 라우트 핸들러에서 직접 SQL 작성 (기존 레거시 제외, 신규는 레이어 분리)
- Controller에서 Repository 직접 호출 (Service 경유)
- Repository에서 HTTPException 발생 (도메인 예외 → 상위에서 HTTP 변환)

**참고 구현:** `backend/apps/secom/app/` (`UserController`, `UserService`, `UserRepository`)

---

## 2. 앱 모듈 구조

```
backend/apps/
├── main.py              # FastAPI 앱, 라우트 등록, lifespan
├── database.py          # Neon async engine, create_tables, get_db
├── secom/               # 인증·회원
├── mova/                # 영화 DB·검색·채팅
├── titanic/             # 타이타닉 API
└── ...
```

- 새 도메인은 `apps/<이름>/app/` 아래에 controllers, services, repositories, schemas, models를 둔다.
- `database.py`의 `create_tables`에 새 모델 import를 등록한다.
- 테이블 PK·컬럼명 규칙: [`ENTITY_RULE.md`](ENTITY_RULE.md) — 모든 테이블 `id: int` 자동 증감 PK.

---

## 3. 비동기·DB

- DB 작업은 **`async def`** + `AsyncSession` (Neon PostgreSQL).
- Repository에서 `get_session_factory()` → `async with factory() as session:` 패턴 사용.
- 중복·무결성 오류는 `UserRepositoryError`처럼 **도메인 예외**로 올리고, `main.py`에서 `HTTPException`으로 변환.

---

## 4. 스키마·로깅

- 요청 본문은 **Pydantic `BaseModel`** (`Field`, 검증).
- 민감 로그는 `log_summary()` 등으로 **필드 단위 요약** (기존 `LoginSchema`, `UserSchema` 패턴).
- 각 레이어 진입 시 `logger.info("[ClassName] method — %s", ...)` 형태 유지.

---

## 5. 라우트 등록

- 공통 앱: `backend/apps/main.py`
- Controller 인스턴스 생성 후 라우트에서 `Depends(get_db)` 등 주입.
- CORS·환경 변수는 기존 설정을 따른다 (`DATABASE_URL`, `GEMINI_API_KEY` 등).

---

## 6. Cursor 에이전트 지시문 (백엔드)

```
docs/README.md, PYTHON_FASTAPI_RULES.md, ENTITY_RULE.md 를 읽은 뒤 코드를 작성하세요.

- Controller → Service → Repository 레이어를 지키세요.
- ORM 모델 PK는 int 자동 증감 `id` (ENTITY_RULE.md).
- Pydantic 스키마와 async Repository 패턴을 사용하세요.
- 요청 범위 밖 리팩터·과도한 추상화를 하지 마세요.
- 참고: backend/apps/secom/app/
```

---

## 7. 체크리스트 (PR·리뷰 전)

- [ ] `docs/README.md` 및 본 규칙을 반영했는가?
- [ ] 새 기능이 Controller → Service → Repository로 분리되었는가?
- [ ] Model PK가 `id` int 자동 증감인가? ([`ENTITY_RULE.md`](ENTITY_RULE.md))
- [ ] Schema / Model / Repository import가 `create_tables`에 반영되었는가?
- [ ] async 세션 사용·예외 처리가 기존 패턴과 일치하는가?
- [ ] `main.py` 변경이 라우트 등록 범위에 한정되는가?
