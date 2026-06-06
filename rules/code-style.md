# Code Style

## Import Order

```python
# 1. 표준 라이브러리
import logging
from datetime import datetime

# 2. 서드파티
from fastapi import APIRouter, HTTPException
from sqlalchemy import select
from sqlalchemy.orm import Session

# 3. 내부 모듈 (absolute)
from pozzetti.place.entities import Place

# 4. 상대 경로 (현재 패키지)
from .service import get
```

## Type Annotations

```python
# PEP 604 union 문법 사용 (X | None, X | Y)
def search(
    *,
    session: Session,
    query: str,
    size: int = 20,
) -> list[Place]: ...

# FastAPI 의존성 주입은 Annotated 로
DbSession = Annotated[Session, Depends(get_db)]
```

## Error Handling

```python
# HTTP 에러: views.py 에서 HTTPException raise
raise HTTPException(
    status_code=status.HTTP_404_NOT_FOUND,
    detail=[{"msg": "RESTAURANT NOT FOUND"}]
)

# 비즈니스 로직 에러: service.py / flows.py 에서 ValueError raise
raise ValueError("Invalid award grade")
```

## Comment Style

- 한국어 docstring 권장
- 복잡한 플로우는 번호 매긴 한국어 인라인 주석으로 설명
- 복잡한 쿼리는 `### Query: ... ###` 헤더로 표시
- 테스트 docstring은 GIVEN/WHEN/THEN 구조

```python
def get(*, session: Session, place_id: str) -> Place | None:
    """
    ID 로 단일 place 조회.

    :param session: SQLAlchemy 세션
    :param place_id: 조회할 place ID
    :return: Place 객체, 없으면 None
    """
```

## Pydantic Models

```python
class PozzettiBaseModel(BaseModel):
    model_config = ConfigDict(
        alias_generator=AliasGenerator(
            validation_alias=to_camel,
            serialization_alias=to_camel,
        ),
        populate_by_name=True,
    )
```
- API I/O는 camelCase, 내부 코드는 snake_case
