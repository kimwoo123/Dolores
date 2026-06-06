# Naming Conventions

| Target | Rule | Example |
|--------|------|---------|
| 도큐먼트 모델 | PascalCase | `Restaurant`, `Tag` |
| Pydantic DTO | PascalCase + suffix | `RestaurantRead`, `RestaurantCreate` |
| Enum | PascalCase | `AwardGrade`, `SourceType` |
| Function | snake_case | `get_by_id()`, `search()` |
| Constant | UPPER_SNAKE_CASE | `INDEX_ALIAS`, `DEFAULT_PAGE_SIZE` |

**도메인 접두사 금지** — `service.py` 함수명에 도메인 이름을 반복하지 않습니다. 도메인은 모듈 경로로 이미 표현됩니다.

```python
# src/pozzetti/restaurant/service.py
def get(...) -> Restaurant | None: ...        # ✅
def search(...) -> list[Restaurant]: ...      # ✅

def get_restaurant(...) -> Restaurant | None: ...  # ❌ (모듈 경로에 이미 restaurant가 있음)
```

## service 호출 컨벤션

- **현재 도메인**: 필요한 함수를 심볼로 직접 import 해 그대로 호출.

  ```python
  # src/pozzetti/restaurant/views.py
  from .service import get, search

  place = get(session=db_session, place_id=place_id)
  ```

- **외부 도메인**: `service` 모듈을 `<도메인>_service` 별칭으로 import 해 한정 호출.

  ```python
  # src/pozzetti/signon_session/views.py
  from pozzetti.account import service as account_service

  account = account_service.verify_credentials(session=db_session, ...)
  ```

**라우트 핸들러는 도메인 접미사를 붙입니다** — service 함수는 도메인 접두사를 떼고(`create`, `get`), **라우트 핸들러는 `<동작>_<도메인>` 형태**로 둡니다(`create_note`, `get_note`, `delete_note`). 

```python
from .service import create, get     # service 함수: 접두사 없음
@router.post("/notes")
def create_note(...):                # 핸들러: _note 접미사
    note = create(session=db_session, ...)
```

예외: 동작명이 이미 도메인을 함의해 접미사가 어색한 경우는 그대로 둡니다(인증: `signup`, `signin`, `signout`, `me`).
