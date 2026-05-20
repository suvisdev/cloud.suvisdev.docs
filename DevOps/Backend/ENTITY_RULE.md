# 엔티티·테이블 규칙 (Cursor)

> **용도:** `backend/` 에서 ORM 모델·테이블·마이그레이션을 추가·수정할 때 `@docs/DevOps/Backend/ENTITY_RULE.md` 로 참조한다.  
> 레이어·API 규칙은 [`PYTHON_FASTAPI_RULES.md`](PYTHON_FASTAPI_RULES.md) 를 함께 본다.

---

## 핵심 원칙

**이 프로젝트의 모든 테이블은 반드시 `int` 타입의 자동 증감 기본 키를 갖고, 컬럼 이름은 `id` 로 통일한다.**

| 항목 | 규칙 |
|------|------|
| 기본 키(PK) | `id` (단일 컬럼) |
| 타입 | `INTEGER` / `int` |
| 증가 | DB 자동 증감 (`SERIAL` / `IDENTITY` / `autoincrement=True`) |
| 이름 | `user_id`, `title_pk` 등 **PK 이름 변경 금지** — 항상 `id` |

- 비즈니스 식별자(`slug`, `username`, UUID 등)는 **별도 컬럼 + UNIQUE** 로 둔다. PK를 대체하지 않는다.
- FK는 참조 테이블의 **`id`** 를 가리킨다. (예: `title_id: int` → `mova_titles.id`)

---

## 1. 기본 키 정의 (SQLModel — 권장 템플릿)

신규 모델은 아래 패턴을 **그대로** 사용한다.

```python
from typing import Optional

from sqlmodel import Field, SQLModel


class Example(SQLModel, table=True):
    __tablename__ = "examples"

    # 1. 시스템 내부용 자동 증감 고유 번호 (기본 키)
    id: Optional[int] = Field(
        default=None,
        primary_key=True,
        sa_column_kwargs={"name": "id"},  # DB 컬럼명: id
    )
```

**의미**

- `default=None` — INSERT 시 DB가 번호를 부여 (애플리케이션에서 id를 넣지 않음)
- `primary_key=True` — 기본 키
- `sa_column_kwargs={"name": "id"}` — 물리 컬럼명을 `id` 로 고정

---

## 2. 기본 키 정의 (SQLAlchemy 2.0 `Mapped` — 본 저장소 현행)

`database.Base` 를 쓰는 기존 앱(`secom`, `mova` 등)은 동일 의미로 아래 형태를 쓴다.

```python
from sqlalchemy.orm import Mapped, mapped_column

from database import Base


class Example(Base):
    __tablename__ = "examples"

    id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
```

SQLModel 템플릿과 **DB 결과는 동일**하다: `int` PK, 컬럼명 `id`, 자동 증감.

---

## 3. 외래 키·관계

```python
# 자식 테이블
title_id: Mapped[int] = mapped_column(ForeignKey("mova_titles.id"), nullable=False, index=True)
```

- FK 컬럼 이름은 `{참조엔티티}_id` 형태를 쓸 수 있다. (예: `title_id`, `user_id`)
- **PK 이름은 항상 `id`** 이다. FK 이름과 혼동하지 않는다.

---

## 4. 금지·지양

| ❌ 금지 | ✅ 대신 |
|--------|--------|
| PK를 `slug`, `username`, UUID로만 두기 | `id` PK + `slug` UNIQUE |
| PK 컬럼명을 `user_id`, `pk`, `seq` 등으로 변경 | PK는 항상 `id` |
| 복합 PK만으로 테이블 설계 (예외 없이 신규 금지) | surrogate `id` + 필요 시 UNIQUE 제약 |
| 애플리케이션에서 `id` 를 수동 할당해 INSERT | `id` 는 DB 자동 증감에 맡김 |

---

## 5. 참고 구현 (본 저장소)

다음 모델은 모두 `id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)` 패턴을 따른다.

- `backend/apps/mova/app/models/base.py` — `MovaModel` (공통 `id` PK)
- `backend/apps/mova/app/models/audience_model.py` — `chat_intents`
- `backend/apps/mova/app/models/movies_model.py` — `movies`
- `backend/apps/mova/app/models/actors_model.py` — `actors`
- `backend/apps/mova/app/models/users_model.py` — `users` (Mova 취향 프로필)

새 테이블 추가 시 위 파일과 **동일한 PK 규칙**을 적용하고, `database.py` 의 `create_tables` 에 모델 import를 등록한다.

---

## 6. Cursor 에이전트 지시문

```
docs/DevOps/Backend/ENTITY_RULE.md 를 읽은 뒤 모델을 작성하세요.

- 모든 테이블 PK: int 자동 증감, 컬럼명 id
- SQLModel: Field(default=None, primary_key=True, sa_column_kwargs={"name": "id"})
- SQLAlchemy Mapped: mapped_column(primary_key=True, autoincrement=True)
- slug/username 등은 UNIQUE 보조 키로만 사용
```

---

## 7. 체크리스트 (PR·리뷰 전)

- [ ] 테이블에 `id` int PK가 있는가?
- [ ] PK 이름이 `id` 가 아닌 다른 이름이 아닌가?
- [ ] 비즈니스 키는 UNIQUE·인덱스로만 분리했는가?
- [ ] FK가 참조 테이블의 `id` 를 가리키는가?
- [ ] `create_tables` / Alembic에 모델이 반영되었는가?
