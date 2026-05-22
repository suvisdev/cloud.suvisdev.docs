# Suvisdev 규칙 (요약·인덱스)

> **용도:** Suvisdev 제품 UI·프론트 작업 시 에이전트가 따를 **스코프(범위)** 규칙의 진입점이다.  
> **전문:** [`docs/DevOps/Frontend/SUVISDEV_RULES.md`](DevOps/Frontend/SUVISDEV_RULES.md)

---

## 한 줄 요약

**사용자가 추가로 지시하지 않으면, 위치 이동·추가 생성·전체 구조 변경을 하지 않는다.**  
지시가 **특정 대상만**이면 그 오브젝트만, **전체·구조**를 명시하면 그 범위 안에서만 넓게 수정한다.

---

## 스코프 빠른 표

| 사용자 의도 (예) | 에이전트 동작 |
|------------------|---------------|
| 「로고 위치 옮겨줘」「검색창만 작게」 | **해당 요소(+필요한 직접 래퍼)만** |
| 「전체적으로 구조 이쁘게」「헤더 다시 짜줘」 | **명시된 영역·페이지** 내 구조 변경 허용 |
| (애매함) | **좁게** 가정 → 필요 시 질문 |

---

## 문서 위치

| 경로 | 내용 |
|------|------|
| **이 파일** `docs/SUVISDEV_RULES.md` | 요약·인덱스 |
| **상세** `docs/DevOps/Frontend/SUVISDEV_RULES.md` | 판별 예시·체크리스트·금지/허용 목록 |

---

## Cursor 참조

```
@docs/SUVISDEV_RULES.md
@docs/DevOps/Frontend/SUVISDEV_RULES.md
```

백엔드 작업은 별도로 [`docs/DevOps/Backend/PYTHON_FASTAPI_RULES.md`](DevOps/Backend/PYTHON_FASTAPI_RULES.md) 를 따른다.
