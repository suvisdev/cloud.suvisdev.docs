# James FastAPI — 초기화 컨텍스트

> Cursor가 이 프로젝트의 FastAPI 구조를 기억하기 위한 참조 문서입니다.

---

## 📁 파일 위치

```
project/
├── james.py        ← FastAPI 앱 진입점 (이 파일)
└── walter.py       ← 데이터 처리 클래스 (Walter)
```

---

## 🧱 클래스 구조

### `James`
- 현재 빈 클래스 (`__init__` 만 존재)
- 향후 비즈니스 로직 또는 상태 관리 담당 예정

### `Walter`
- 데이터 로딩 및 처리 담당
- 주요 메서드:
  - `get_data()` : 데이터 로딩
  - `head_records()` : 상위 레코드 반환 (JSON 직렬화 가능 형태)

---

## ⚡ FastAPI 앱 (`app`)

```python
app = FastAPI(title="Titanic James API")
```

| 엔드포인트 | 메서드 | 설명 |
|-----------|--------|------|
| `/`       | GET    | 초기화 확인 — `{"message": "FAST API 초기화 성공"}` 반환 |
| `/data`   | GET    | `Walter().head_records()` 결과 반환 |

---

## 🔌 Import 전략

패키지 내부 실행과 단독 실행 모두 지원:

```python
try:
    from .walter import Walter   # 패키지로 import 시
except ImportError:
    from walter import Walter    # 단독 실행 시
```

---

## ▶️ 실행 방법

### uvicorn으로 서버 실행
```bash
uvicorn james:app --reload
```

### 단독 스크립트 실행
```bash
python james.py
# 출력: 제임스가 메인이다.
```

---

## 🗒️ 현재 상태 요약

- [x] FastAPI 앱 초기화 완료
- [x] `/` 루트 엔드포인트 등록
- [x] `/data` 엔드포인트 — Walter 연동 완료
- [ ] James 클래스 내부 로직 미구현 (예정)
- [ ] 인증, 에러 핸들링 미적용

---

*Last updated: 2026-05-07*
