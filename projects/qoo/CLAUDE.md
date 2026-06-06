# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 로컬 실행

빌드 과정 없음. 정적 파일 서버만 있으면 됩니다:

```bash
npx serve src
# → http://localhost:3000
```

`DEBUG_QUADTREE`는 `localhost`에서 자동 활성화되어 QuadTree 경계를 지도에 시각화합니다 ([src/store/constants.js](src/store/constants.js):20).

## 기술 스택

- **Vanilla JS ES Modules** — 번들러 없음, Import Map으로 절대경로 해석 (`src/index.html`에 정의)
- **Naver Maps JavaScript API v3** — `naver` 전역 객체로 접근 (CDN 로드)
- **Cloudflare Pages** 배포: [qoo.kimwooseok.com](https://qoo.kimwooseok.com)

## 아키텍처 개요

### 데이터 흐름

```
JSON 파일 (src/statics/) → data/*.data.js (파싱/변환) → layersConfig (store/layers.js)
→ toggleLayer() (services/layer.js) → buildQuadTree() → processVisibleMarkers()
→ clusterMarkers() → 마커/클러스터 마커 풀에서 생성 → Naver Maps API
```

### 전역 상태 (`src/store/index.js`)

`state` 객체가 유일한 전역 상태. `state.layers[layerKey]`는 각 레이어의 런타임 상태를 보유:
- `isActive`, `isLoaded`, `data[]` — 레이어 데이터 및 상태
- `quadTree` — 공간 인덱스 (뷰포트 쿼리용)
- `markers: Map<idx, marker>`, `markerPool[]` — 개별 마커 DOM 풀링
- `clusters: Map<clusterId, marker>`, `clusterPool[]` — 클러스터 마커 DOM 풀링
- `clusterContents: Map<clusterId, Set<idx>>` — 애니메이션 전환 시 이전 클러스터 포함 인덱스 추적

### 클러스터링 알고리즘 (`src/services/cluster.js`)

1. `buildQuadTree()` — 레이어 로드 시 데이터 전체를 QuadTree에 삽입 (위경도 좌표계)
2. 지도 `idle` 이벤트마다 현재 뷰포트를 `Rectangle`로 변환해 `quadTree.query()` 호출
3. 쿼리된 포인트들을 현재 줌의 **Web Mercator 픽셀 좌표**로 변환
4. 60px 반경 내 포인트들을 하나의 클러스터로 그루핑 (중심은 무게중심으로 갱신)
5. 줌 레벨 > `CLUSTER_MAX_ZOOM`(14)이면 클러스터링 안 함 — 모두 개별 마커

### 애니메이션 전환 (`src/services/layer.js`)

`processVisibleMarkers()`에서 이전 상태와 새 상태를 비교해 세 가지 전환을 애니메이션 처리:
- 개별 마커 → 클러스터: `clusterContents`로 목적지 클러스터 탐색 후 fly-to
- 클러스터 → 개별 마커: 이전 클러스터 위치에서 확산
- 클러스터 → 클러스터: 흡수되는 클러스터가 새 클러스터 위치로 fly-to

### 레이어 추가 방법

1. `src/data/`에 `*.data.js` 작성 (JSON 로드 + 좌표/등급 정규화)
2. `src/statics/`에 JSON 데이터 파일 추가
3. `src/store/layers.js`에 레이어 키 및 `loader`, `color`, `zIndex` 등록
4. `src/store/sidebar.js`에 사이드바 그룹 정의 추가

### 데이터 형식

각 레스토랑 항목은 `MinimalRestaurant` 타입([src/store/constants.js](src/store/constants.js):10):
```js
{ name, lat, lng, source, sourceId, award }
```
일부 레거시 데이터는 `gps: { latitude, longitude }` 형태 — `extractCoordinates()`가 양쪽 모두 처리.

## 커밋 컨벤션

형식: `type(scope): 제목`
본문은 불릿 리스트로 변경 이유 및 주요 내용 기술. Co-Authored-By 불필요.

```
feat(avpn): AVPN 인증 레이어 추가

- AVPN 한국 인증점 데이터 크롤링 및 레이어 등록
- crawl/avpn.py, src/statics/avpn_korea.json, src/data/avpn.data.js 추가
```

**타입**: `feat` / `fix` / `refactor` / `style` / `docs` / `chore`

**커밋 단위**: 논리적으로 독립된 변경 단위로 분리. 여러 레이어를 동시에 추가할 때는 레이어별로 분리.
공유 파일(`constants.js`, `layers.js`, `sidebar.js`)에 변경이 섞인 경우, 중간 상태로 편집 후 순서대로 커밋.

## `crawl/` 및 `sandbox/`

Python 크롤러 스크립트들 (배포와 무관). `src/statics/`의 JSON 정적 데이터를 생성하는 데 사용됨.


# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.