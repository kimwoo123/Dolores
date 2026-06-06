# Dolores

나의 작은 클로드 — 개인용 Claude Code 설정 모음.

프로젝트와 무관하게 어디서든 통하는 **범용 행동 규칙**과 **작업 규칙**을 모아둡니다.

## 구성

```
CLAUDE.md        # 전역 행동 규칙 (think before coding / simplicity / surgical changes / goal-driven)
rules/
├── code-style.md        # import 순서, 타입 힌트, 에러 처리, 주석/Pydantic 스타일
├── naming.md            # 모델·DTO·함수·상수 네이밍 컨벤션
├── testing.md           # 테스트 디렉토리 구조, 격리, factory 패턴
└── planning-workflow.md # 계획 문서 → 테스트 → 구현 워크플로
```

## 사용

- **전역 적용**: `CLAUDE.md` 를 `~/.claude/CLAUDE.md` 로 복사하면 모든 프로젝트에 적용됩니다.
- **프로젝트별 규칙**: `rules/` 의 파일을 프로젝트 `.claude/rules/` 로 가져와 import 블록에 한 줄 추가해 사용합니다.
