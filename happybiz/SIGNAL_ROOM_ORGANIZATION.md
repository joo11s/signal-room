# HAPPYBIZ SIGNAL ROOM — Organization

> 네이버 이커머스 자동화 6-agent 파이프라인

## 전체 구조

```
main.py (오케스트레이터)
    │
    ▼
research-agent     ← 시장 조사 & 구매자 프로파일
    │
    ▼
strategy-agent     ← 트렌드/채널/키워드/offer 전략
    │
    ▼
plan-agent ↔ margin-agent   (핑퐁 최대 2회)
    │
    ▼
blog-agent         ← 네이버 블로그 콘텐츠 작성
    │
    ▼
qa-agent           ← DoD 5가지 검수 + 핀포인트 재실행
```

## Agent 역할 요약

| Agent | 역할 | 도구 | Output |
|-------|------|------|--------|
| research-agent | 시장 조사, 구매자 프로파일 | WebSearch, WebFetch | buyer_profile, top_seller_analysis, market_saturation |
| strategy-agent | 전략 수립 (트렌드/채널/키워드) | WebSearch, WebFetch | trend_direction, channel_strategy, keyword_priority |
| plan-agent | 월간 실행 캘린더 & 광고 계획 | Read, Write | monthly_calendar, ad_plan, execution_priority |
| margin-agent | 수익성 계산, 3단계 시나리오 | Read, Write | cost_breakdown, profit_scenarios, execution_verdict |
| blog-agent | 블로그 콘텐츠 작성 (5-gate QA) | Read, Write, Edit, WebSearch | blog_title, blog_content, quality_score |
| qa-agent | 전체 DoD 검수 + 재실행 | Read, Grep | pass, scores, failed_items, rerun_agents |

## DoD (Definition of Done) 5가지

1. **구매자 프로파일 반영** — blog 톤앤매너가 research buyer_profile과 일치
2. **트렌드 방향 일치** — 콘텐츠 방향이 strategy trend_direction과 정합
3. **CPC 키워드 3개 이상** — 핵심 키워드 최소 3개 본문에 자연스럽게 포함
4. **수익 시나리오-CTA 일치** — margin 시나리오 go_decision=true & CTA URL 정확
5. **블로그 품질 90점 이상** — blog-agent 5-gate 합산 90점+

## 데이터 흐름

```
research → buyer_profile ──────────────────────────────► blog (톤앤매너)
research → market_saturation ──► strategy
research → top_seller_analysis ─► strategy

strategy → keyword_priority ────────────────────────────► blog (키워드)
strategy → trend_direction ─────► plan
strategy → offer_positioning ───► plan + blog (CTA)
strategy → channel_strategy ────► plan + margin

plan → monthly_calendar ◄──────► margin (핑퐁)
plan → ad_plan ─────────────────────────────────────────► margin (비용)
plan → execution_priority ──────────────────────────────► blog (스케줄)

margin → profit_scenarios ──────────────────────────────► qa (DoD4)
margin → execution_verdict ─────────────────────────────► qa (DoD4)

blog → blog_content ────────────────────────────────────► qa (DoD 1,2,3,5)
blog → quality_score ───────────────────────────────────► qa (DoD5)

qa → rerun_agents ──────────────────────────────────────► 핀포인트 재실행
```

## 핑퐁 & 재실행 정책

- **plan ↔ margin 핑퐁**: 최대 2회. 수익 가능한 플랜 확정
- **qa 미달 재실행**: 미달 항목 해당 agent만 재실행, 최대 2회/항목
- **전체 재실행 금지**: 불필요한 전체 파이프라인 재실행 방지

## 사용법

```bash
python main.py \
  --product "무선 이어폰" \
  --keyword "블루투스이어폰추천" \
  --budget 500000 \
  --month 2026-06
```

## 출력 구조

```
output/
└── result_YYYYMMDD_HHMMSS.json
    ├── metadata
    ├── research   (buyer_profile, top_seller_analysis, market_saturation)
    ├── strategy   (trend_direction, channel_strategy, keyword_priority)
    ├── plan_margin (monthly_calendar, ad_plan, profit_scenarios)
    ├── blog       (blog_title, blog_content, quality_score)
    └── qa         (pass, scores, failed_items)
```

## Claude API 비용 구조

| Agent | 모델 | 예상 토큰/실행 |
|-------|------|---------------|
| research-agent | sonnet | ~8K |
| strategy-agent | sonnet | ~6K |
| plan-agent | sonnet | ~5K |
| margin-agent | sonnet | ~4K |
| blog-agent | sonnet | ~10K (재작성 시 ×1.5) |
| qa-agent | sonnet | ~3K (재실행 시 추가) |
| **합계** | | **~36K / 실행** |

## 버전 히스토리

- v1.0.0 (2026-06-08): 초기 구조 — 6 agents + orchestrator
