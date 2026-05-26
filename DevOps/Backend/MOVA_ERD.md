# Mova ERD

`backend/apps/mova` ORM 기준 **Mova DB** 테이블 구조입니다.  
회원(`users`)은 **Secom** 모듈·`users` 테이블이며, **`SECOM_DATABASE_URL`을 비우면 Mova와 동일 DB**에 둡니다.  
이때 `chat` / `reviews` / `picks`의 `user_id`는 **`users.id` FK** (비로그인 `chat`·`picks`는 `user_id` NULL).

## DB 이름·역할 (정리)

| 구분 | 설명 |
|------|------|
| **접두어 `mova_` 없음** | 테이블명은 `movies`, `actors`처럼 **짧은 영문 단수**만 씀. `mova_movies` 같은 중복 접두어는 쓰지 않음. |
| **Mova DB** | 영화·랭킹·태그·AI 의도·리뷰·상호작용. env: `MOVA_DATABASE_URL` 또는 `DATABASE_URL`. |
| **Secom DB** | **회원가입·로그인** (`users`) + **프로필** (`members`) + **그룹** (`groups`, `member_groups`). |
| **`users`는 Secom만** | 인증·`role`(admin/user). 상세 프로필·취향은 **`members`** 1:1. Mova AI는 **`assistants`**. |
| **PK 규칙** | 모든 테이블 PK 컬럼명은 `id` (int 자동 증가). 비즈니스 키는 `slug`, `username` 등 별도 UNIQUE. |

Mermaid `erDiagram`은 속성·관계 라벨의 **따옴표·괄호·슬래시** 등에서 파싱 오류가 날 수 있습니다. 필드 설명은 아래 표를 참고하세요.

## 전체 ERD (영화 카탈로그 + 채팅 + 회원)

기존 9테이블 레이아웃에 `members` · `groups` · `member_groups` · `assistants`를 같은 스타일로 합친 다이어그램입니다.

![Mova ERD](./mova-erd.png)

편집용 Mermaid 소스: [`mova-erd.mmd`](./mova-erd.mmd) — Obsidian에서도 아래 블록으로 동일하게 렌더됩니다.

```mermaid
---
config:
  er:
    diagramPadding: 56
    layoutDirection: TB
---
erDiagram
    MOVIES ||--o{ CHARACTERS : casts
    ACTORS ||--o{ CHARACTERS : appears_in
    MOVIES ||--o{ TAGS : tagged
    CHARACTERS ||--o| TAGS : cast_keyword
    MOVIES ||--o{ RANKINGS : ranked
    MOVIES ||--o{ REVIEWS : receives

    CHAT ||--o{ PICKS : recommends
    MOVIES ||--o{ PICKS : picked_in
    USERS ||--|| MEMBERS : profile
    MEMBERS ||--o{ MEMBER_GROUPS : belongs
    GROUPS ||--o{ MEMBER_GROUPS : has
    ASSISTANTS ||--o{ CHAT : answers
    MEMBERS ||--o{ CHAT : member_searches
    USERS ||--o{ REVIEWS : user_actions
    USERS ||--o{ CHAT : searches
    USERS ||--o{ PICKS : user_actions

    MOVIES {
        int id PK
        varchar slug UK
        varchar title
        varchar release_year
        float rating
        text poster_url
        varchar platform
        jsonb genres
    }

    ACTORS {
        int id PK
        varchar name
        varchar role_type
        text profile_photo_url
    }

    CHARACTERS {
        int id PK
        int movie_id FK
        int actor_id FK
    }

    TAGS {
        int id PK
        int movie_id FK
        int character_id FK
        varchar tag_kind
        varchar slug
        varchar label
        text description
    }

    RANKINGS {
        int id PK
        int rank
        int movie_id FK
        varchar badge
        date ranked_at
    }

    MEMBERS {
        int id PK
        int user_id UK
        varchar gender
        varchar age_group
        int birth_year
        jsonb preferred_genres
    }

    GROUPS {
        int id PK
        varchar code UK
        varchar name
        text description
    }

    MEMBER_GROUPS {
        int id PK
        int member_id FK
        int group_id FK
    }

    ASSISTANTS {
        int id PK
        varchar slug UK
        varchar display_name
        text system_prompt
        varchar default_model
        bool is_active
    }

    CHAT {
        int id PK
        int user_id FK
        int member_id FK
        int assistant_id FK
        text raw_message
        varchar refined_query
        jsonb keywords
        varchar intent_type
        jsonb search_filters
        int hit_count
        timestamptz last_used_at
        timestamptz created_at
    }

    REVIEWS {
        int id PK
        int user_id FK
        int movie_id FK
        varchar action_type
        timestamptz action_at
        float rating
        text body
    }

    PICKS {
        int id PK
        int chat_id FK
        int user_id FK
        int movie_id FK
        int pick_rank
        varchar hook
        varchar title_snapshot
        timestamptz batch_at
    }

    USERS {
        int id PK
        varchar role
        varchar username UK
        varchar nickname
        varchar email
    }
```

### 그룹 코드 (시드)

| code | name | 용도 |
|------|------|------|
| `platform_admin` | 플랫폼 관리자 | Secom·import API 등 |
| `mova_admin` | Mova 관리자 | 카탈로그·랭킹·태그 |
| `mova_member` | Mova 회원 | 일반 가입 시 자동 부여 |

### 성별·연령대 코드 (`members`)

| gender | 설명 |
|--------|------|
| `male` | 남성 |
| `female` | 여성 |
| `other` | 기타 |
| `undisclosed` | 미입력 |

| age_group | 설명 |
|-----------|------|
| `10s` ~ `50s` | 10대 ~ 50대 |
| `60s_plus` | 60대 이상 |
| `undisclosed` | 미입력 |

## DB 범위

| DB | env 변수 | 테이블 |
|----|----------|--------|
| Mova | `MOVA_DATABASE_URL` 또는 `DATABASE_URL` | `movies`, `actors`, `characters`, `tags`, `rankings`, **`assistants`**, `chat`, `picks`, `reviews` |
| Secom | `SECOM_DATABASE_URL` (미설정 시 Mova와 **동일 URL**) | `users`, **`members`**, **`groups`**, **`member_groups`** — `chat`·`reviews`·`picks`.`user_id` FK |

## 관계

| 관계 | 카디널리티 | 설명 |
|------|------------|------|
| MOVIES → CHARACTERS | 1:N | 영화–배우·감독 연결 (`movie_id` → `movies.id`, CASCADE) |
| ACTORS → CHARACTERS | 1:N | 동일 중간 테이블 (`actor_id` → `actors.id`, CASCADE) |
| MOVIES ↔ ACTORS | N:M | `characters` 경유, `(movie_id, actor_id)` UNIQUE |
| MOVIES → TAGS | 1:N | 영화 키워드 (`tag_kind`: mood / genre / cast) |
| CHARACTERS → TAGS | 1:0..1 | `tag_kind=cast` 일 때 `character_id` FK — 영화–인물 연결을 검색 키워드로 노출 |
| TAGS (slug) | (논리 그룹) | mood: 같은 `slug`로 여러 영화에 동일 감성 태그 · genre: `genre-{장르}` · cast: `cast-{이름}` |
| MOVIES → RANKINGS | 1:N | HOT 랭킹 (`rank`, `ranked_at`) 조합 UNIQUE |
| MOVIES → REVIEWS | 1:N | 찜·시청·클릭·별점 리뷰 (`action_type`, `movie_id` FK) |
| USERS → REVIEWS | 1:N | `reviews.user_id` → `users.id` FK (`ON DELETE CASCADE`) |
| USERS → CHAT | 1:N | `chat.user_id` → `users.id` FK (`ON DELETE SET NULL`, 비로그인 NULL) |
| CHAT → PICKS | 1:N | AI가 한 번에 추천한 작품 (보통 3행, `batch_at`으로 묶음) |
| MOVIES → PICKS | 1:N | 추천된 `movie_id` FK |
| USERS → PICKS | 1:N | `picks.user_id` → `users.id` FK (`ON DELETE SET NULL`) |
| USERS → MEMBERS | 1:1 | `members.user_id` → `users.id` FK UNIQUE (`ON DELETE CASCADE`) |
| MEMBERS → MEMBER_GROUPS | 1:N | 회원이 속한 그룹 연결 |
| GROUPS → MEMBER_GROUPS | 1:N | 그룹에 포함된 회원 |
| MEMBERS ↔ GROUPS | N:M | `member_groups` 교차 테이블, `(member_id, group_id)` UNIQUE |
| MEMBERS → CHAT | 1:N | `chat.member_id` → `members.id` (`ON DELETE SET NULL`) |
| ASSISTANTS → CHAT | 1:N | `chat.assistant_id` → `assistants.id` — 응답 AI 페르소나 |
| REVIEWS (action_type=review) | 1:1 per user+movie | 별점·감상평 — `(user_id, movie_id)` partial UNIQUE |

**다이어그램에 선 없음 (DB FK·교차 테이블 아님, 앱 검색만):**

| 연결 | 설명 |
|------|------|
| CHAT → TAGS | `keywords`·`search_filters.must`로 `tags` 검색 (use) |
| CHAT → ACTORS | `search_filters`·`characters` 조인으로 배우 검색 (use) |
| CHAT → MOVIES | `search_filters`·`keywords`로 `movies` 조회 (use). 결과 FK는 **`picks`만** |

### 채팅 추천 흐름 (예: "오늘 우울하니까 재미있는 영화 틀어줘")

```text
사용자 메시지
    → IntentExtraction (refined_query, keywords 예: "재미있는", "우울")
    → chat 저장 (keywords, intent_type, search_filters JSONB)
    → SearchRepository: intent_type·search_filters(must AND) 또는 keywords로 tags·actors·movies 검색
    → Gemini: [태그·DB 카탈로그] 목록에서 3편 picks
    → movies 테이블에 제목 저장·메타 보강
```

| 단계 | 테이블 | 설명 |
|------|--------|------|
| 1. 의도 추출 | (메모리) | "재미있는", "우울" 등 키워드 분리 — **자동으로 tags 행을 만들지는 않음** |
| 2. 의도 이력 | `chat` | `keywords`, `refined_query`, `intent_type`, `search_filters` 저장 |
| 3. 태그 매칭 | `tags` → `movies` | `tags.label` ILIKE `%재미있는%` 등 (`GET /mova/search`와 동일) |
| 4. 추천 | `movies` | Gemini가 3편 제목 반환 후 `movies` upsert |
| 5. 추천 기록 | `picks` | `chat_id` + `movie_id` + 순위·hook 저장 (데이터화) |

**전제:** "재미있는"으로 찾으려면 DB `tags`에 그 `label`(또는 비슷한 문구)이 **어떤 영화에든** 붙어 있어야 합니다. 없으면 검색 결과가 비고 Gemini만으로 추천합니다.

**카탈로그 시드:** `backend/scripts/seed_mova_recommendation_catalog.py` — mood·genre 태그, **cast 태그**(`characters` 연결 후 `character_id` FK).

### tags vs chat (저장 구조)

| | `tags` | `chat` |
|--|--------|--------|
| **역할** | 작품별 **감성 라벨** (`movie_id` FK) | 사용자 **채팅 의도·분류** 로그 |
| **영화 연결** | `movie_id` FK | 없음 — `picks.movie_id`로 결과만 연결 |
| **분류** | 없음 | `intent_type` (`filter_and` / `similar_person` / `mood`) |
| **AND 조건** | 없음 | `search_filters.must` (actors, genres, keywords) |
| **검색 바** | `GET /mova/search` | `POST /mova/chat` — `search_by_filters` 또는 키워드 검색 |

`CHAT`과 `tags`·`actors`·`movies`의 연결은 ERD에 그리지 않습니다. DB FK가 아니라 **`search_filters`·`keywords`로 검색하는 use**이며, 채팅 결과만 `picks`로 `movies`에 연결됩니다.

기존 DB에 `intent_type`·`search_filters`가 없으면 `backend/scripts/add_chat_intent_columns.py` 1회 실행.

기존 DB에 `user_id` FK가 없으면 (동일 DB 전제) `backend/scripts/add_mova_user_id_fk.py` 1회 실행.

기존 DB에 `members`·`assistants`가 없으면 `backend/scripts/add_members_and_assistants.py` 1회 실행.

## 제약·인덱스

| 테이블 | 제약 |
|--------|------|
| `movies` | `slug` UNIQUE |
| `actors` | `(name, role_type)` UNIQUE — `uq_actors_name_role` |
| `characters` | `(movie_id, actor_id)` UNIQUE |
| `tags` | `(movie_id, slug)` UNIQUE · `character_id` UNIQUE — cast · `character_id` → `characters.id` FK |
| `rankings` | `(rank, ranked_at)` UNIQUE — `uq_rankings_rank_date` |
| `reviews` | `user_id` → `users.id` FK · `action_type=review` 시 `(user_id, movie_id)` partial UNIQUE |
| `chat` | `user_id` → `users.id` · `member_id` → `members.id` · `assistant_id` → `assistants.id` (nullable) · `intent_type` 인덱스 |
| `picks` | `user_id` → `users.id` FK (nullable) |
| `users` (Secom) | `username` UNIQUE · `role` = `admin` \| `user` |
| `members` | `user_id` UNIQUE → `users.id` ON DELETE CASCADE |
| `groups` | `code` UNIQUE |
| `member_groups` | `(member_id, group_id)` UNIQUE — `uq_member_groups_member_group` |
| `assistants` | `slug` UNIQUE |

## 필드 설명

### movies

| 필드 | 설명 |
|------|------|
| slug | URL·검색용 식별자 (예: `interstellar`, `tmdb-550`, `kofic-20139882`) |
| title | 작품 제목 |
| release_year | 개봉 연도 문자열 |
| rating | 평균 별점 (리뷰 upsert 시 갱신) |
| poster_url | 포스터 URL (TMDB enrich 가능) |
| platform | OTT 힌트 (`netflix`, `disney` 등, nullable) |
| genres | 장르 배열 JSONB |


### actors

| 필드 | 설명 |
|------|------|
| name | 인물 이름 |
| role_type | `director` 또는 `actor` |
| profile_photo_url | 프로필 이미지 URL |


### characters

| 필드 | 설명 |
|------|------|
| movie_id | `movies.id` |
| actor_id | `actors.id` |


### tags (영화 키워드: 감성·장르·등장인물)

| 필드 | 설명 |
|------|------|
| movie_id | `movies.id` FK |
| character_id | `characters.id` FK — `tag_kind=cast` 일 때만 (nullable) |
| tag_kind | `mood` 감성 · `genre` 장르 · `cast` 등장인물 |
| slug | `mood`: 공유 slug · `genre`: `genre-{장르}` · `cast`: `cast-{이름}` |
| label | 검색·표시 라벨 (감성 문구, 장르명, 배우 이름) |
| description | 태그 설명 |

기존 DB: `add_tags_actor_kind.py` → `add_tags_character_id.py` 순 실행 후 `seed_mova_recommendation_catalog.py`로 cast 태그를 `character_id` 기준으로 채우기.


### rankings

| 필드 | 설명 |
|------|------|
| rank | 순위 1~10 |
| movie_id | `movies.id` |
| badge | `NEW` 등 뱃지 (nullable) |
| ranked_at | 랭킹 기준일 (KOFIC 일간 박스오피스 등) |


### picks (AI 채팅 추천 작품 기록)

| 필드 | 설명 |
|------|------|
| chat_id | `chat.id` — 어떤 검색/채팅 의도에서 나온 추천인지 |
| user_id | `users.id` FK (로그인 시) |
| movie_id | `movies.id` — 추천된 작품 |
| pick_rank | 해당 응답 안 순위 1~3 |
| hook | AI 한 줄 추천 이유 |
| title_snapshot | 추천 시점 제목 (스냅샷) |
| batch_at | 같은 응답에서 나온 3편 묶음 시각 |

사용자가 **클릭·찜**한 선택은 `reviews` (`action_type`)로 별도 기록 가능.


### chat (AI 검색·채팅 의도 로그)

| 필드 | 설명 |
|------|------|
| user_id | `users.id` FK (로그인 시). 비회원은 NULL — 전역 인기 의도로 집계 |
| member_id | `members.id` FK — 로그인 회원 프로필 (nullable) |
| assistant_id | `assistants.id` FK — 응답한 AI 페르소나 (기본 `mova-concierge`) |
| raw_message | 사용자 원문 (채팅/랜딩 검색 입력) |
| refined_query | AI·규칙으로 정제한 검색 문구 — `tags.label`과 유사하지만 **작품 FK 없음** |
| keywords | 추출 키워드 JSONB 배열 (배우·장르·분위기 등 **전부**, 최대 24개) |
| intent_type | `filter_and` · `similar_person` · `mood` — 검색·추천 분류 (DB FK 없음, 앱 해석) |
| search_filters | JSONB — 아래 구조. `actors`/`movies`/`tags`와 **FK 없음**, 검색 시 use |
| hit_count | 동일 의도 재사용 횟수 |

`search_filters` 예시:

```json
{
  "must": { "actors": ["전지현"], "genres": ["스릴러"], "keywords": [] },
  "similar_to": { "actors": ["전지현"] },
  "match_mode": "all"
}
```

| `intent_type` | `match_mode` | 검색 의미 |
|---------------|--------------|-----------|
| `filter_and` | `all` | `must` 조건 **전부 AND** (교집합) |
| `similar_person` | `any` | `similar_to.actors` 출연작 위주 (넓게) |
| `mood` | `any` | `keywords`·`refined_query` 위주 (OR 검색) |
| last_used_at | 마지막 사용 시각 |
| created_at | 최초 저장 시각 |


### reviews (반응·별점 리뷰 단일 테이블)

| 필드 | 설명 |
|------|------|
| user_id | `users.id` FK |
| movie_id | `movies.id` |
| action_type | `favorite`, `watched`, `click`, `not_interested`, **`review`** |
| action_at | 반응·리뷰 시각 (API 리뷰 응답의 `created_at`과 동일) |
| rating | 별점 1~5 (`action_type=review`일 때) |
| body | 감상평 (`review`일 때) |

## ORM 매핑

| 테이블 | 모델 | 경로 |
|--------|------|------|
| `movies` | `MovaMovie` | `mova/app/models/movies_model.py` |
| `actors` | `MovaActor` | `mova/app/models/actors_model.py` |
| `characters` | `MovaCharacter` | `mova/app/models/characters_model.py` |
| `tags` | `MovaTag` | `mova/app/models/tags_model.py` |
| `rankings` | `MovaRanking` | `mova/app/models/rankings_model.py` |
| `chat` | `MovaChat` | `mova/app/models/chat_model.py` |
| `picks` | `MovaPick` | `mova/app/models/picks_model.py` |
| `reviews` | `MovaReview` | `mova/app/models/reviews_model.py` |
| `assistants` | `MovaAssistant` | `mova/app/models/assistants_model.py` |
| `users` | `User` | `secom/app/models/user_model.py` |
| `members` | `Member` | `secom/app/models/member_model.py` |
| `groups` | `Group` | `secom/app/models/group_model.py` |
| `member_groups` | `MemberGroup` | `secom/app/models/member_group_model.py` |

공통 PK 규칙은 [`ENTITY_RULE.md`](ENTITY_RULE.md)를 따릅니다 (`id` int 자동 증감).

### actors vs assistants

| | `actors` | `assistants` |
|--|----------|----------------|
| 의미 | 실제 인물(배우·감독) | AI 채팅 상대(페르소나) |
| 연결 | `characters` → `movies` | `chat` 의도·추천 응답 |
| 검색 | 출연·감독 키워드 | UI·프롬프트·모델 설정 |

## Secom — users · members · groups

| 테이블 | 역할 |
|--------|------|
| **users** | 로그인·비밀번호·`role`(admin/user) |
| **members** | 성별·연령대·선호 장르 등 Mova용 프로필 (`user_id` 1:1) |
| **groups** | `platform_admin`, `mova_admin`, `mova_member` |
| **member_groups** | 회원 ↔ 그룹 N:M |

마이그레이션: `backend/scripts/add_members_and_assistants.py`

### users (인증)

| 필드 | 설명 |
|------|------|
| role | `admin` 또는 `user` (기본 `user`) |
| username | 로그인 ID (UNIQUE) |
| password_hash | 비밀번호 해시 |
| nickname | 표시 이름 |
| email | 이메일 |
| created_at | 가입 시각 |

### members (회원 상세)

| 필드 | 설명 |
|------|------|
| user_id | `users.id` FK UNIQUE |
| gender | `male` \| `female` \| `other` \| `undisclosed` |
| age_group | `10s` \| `20s` \| `30s` \| `40s` \| `50s` \| `60s_plus` \| `undisclosed` |
| birth_year | 출생 연도 (nullable) |
| preferred_genres | 선호 장르 JSONB 배열 |
| bio | 한 줄 소개 (선택) |
| created_at / updated_at | 생성·수정 시각 |

### groups (권한 그룹)

| 필드 | 설명 |
|------|------|
| code | `platform_admin`, `mova_admin`, `mova_member` 등 UNIQUE |
| name | 표시 이름 |
| description | 그룹 설명 |

### member_groups (회원 ↔ 그룹)

| 필드 | 설명 |
|------|------|
| member_id | `members.id` FK |
| group_id | `groups.id` FK |

### assistants (Mova AI 상대)

| 필드 | 설명 |
|------|------|
| slug | `mova-concierge` 등 |
| display_name | UI 표시명 (예: Mova AI 컨시어지) |
| avatar_url | 아바타 이미지 URL |
| system_prompt | Gemini 시스템 지시 |
| default_model | `flash15` 등 |
| is_active | 사용 여부 |
