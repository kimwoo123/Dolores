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

호출부에서는 `from pozzetti.restaurant import service as restaurant_service` 후 `restaurant_service.get(...)` 형태로 사용합니다.
