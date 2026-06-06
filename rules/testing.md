# Test Patterns

## 디렉토리 구조

테스트는 도메인별 디렉토리로 묶고, 그 안에 대상 레이어별 파일로 분리합니다 — `tests/<도메인>/test_<layer>.py`.

```
tests/
├── conftest.py             # 공용 fixture / factory 등록
├── factories.py            # factory-boy 팩토리
└── <도메인>/
    ├── test_views.py       # views.py (라우터) 테스트
    ├── test_entities.py    # entities.py (스키마 관련 함수) 테스트
    └── test_service.py     # service.py (쿼리/비즈니스 로직) 테스트
```

예) `account` 도메인 테스트를 작성한다면:

```
tests/account/test_views.py    # account/views.py 테스트
tests/account/test_entities.py # account/entities.py 테스트
tests/account/test_service.py  # account/service.py 테스트
```

- 파일명은 대상 레이어와 1:1 — `service.py` → `test_service.py`, `views.py` → `test_views.py`, `entities.py` → `test_entities.py`
- 도메인이 늘어나면 디렉토리만 추가 (`tests/place/`, `tests/note/`, …)

## 격리 (Postgres)

[tests/conftest.py](../../tests/conftest.py) 가 제공:

- `engine` (session scope): 빈 테스트 DB `pozzetti_test` 재생성 + `alembic upgrade head` (확장·인덱스 포함)
- `db` (function scope): 트랜잭션 안에서 실행 후 **롤백** — service 의 `commit()` 은 savepoint 로 흡수되어 테스트 간 격리
- `client`: `get_db` 를 테스트 세션으로 오버라이드한 `TestClient`
- `place_factory`: Place ORM 행 생성 헬퍼 (비기본 상태만 인자로)

```python
# BDD-style docstring
class TestSearch:
    path = "/places"

    def test_korean_partial_match(self, client, place_factory):
        """
        GIVEN: 이름에 "스시"가 포함된 place 가 색인되어 있음
        WHEN: GET /places?q=스시 호출
        THEN: 200 응답과 매칭 결과 반환
        """
        place_factory(name="스시코우지")
        resp = client.get(self.path, params={"q": "스시"})
        assert resp.status_code == 200
```

## Factory 패턴

- 기본 상태는 fixture, 비기본 상태만 팩토리에 인자로 전달
- ORM 행은 `place_factory` 가 `db` 세션에 add+flush — `client` 와 같은 트랜잭션이라 요청에서 바로 보임

```python
def test_filter(self, client, place_factory):
    place_factory(name="강남집", sido="서울", sigungu="강남구")
    place_factory(name="마포집", sido="서울", sigungu="마포구")
    resp = client.get("/places", params={"sido": "서울", "sigungu": "강남구"})
    assert {p["sigungu"] for p in resp.json()} == {"강남구"}
```
