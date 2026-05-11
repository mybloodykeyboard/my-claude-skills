---
name: company-context
description: Load Jungwoo's company, product, differentiation, track record, and target customer information when drafting external-facing artifacts (proposals, pitch decks, customer-facing docs) or when Jungwoo asks "우리 회사/제품/사례/차별화" related questions. The proposal-writing skill explicitly references this skill — activate it whenever a proposal is being drafted, restructured, or reviewed. Information is split across topical files (company.md, products.md, differentiation.md, track-record.md, customers.md); read only the files relevant to the task rather than loading all of them.
---

# company-context

Reference store for company-specific facts that Claude needs when helping with external artifacts (proposals, pitches, customer comms). This is **information**, not heuristics — the value comes from the data in the linked files.

## When to load

- Drafting/restructuring/reviewing a proposal (always — `proposal-writing` references this)
- "우리 회사/제품/차별화/사례" 류 질문
- Customer-facing 문서·메일·발표자료 작성
- 자기소개·about us 섹션 작성

## File index

Read **only the files relevant to the current task**. Don't preload all of them.

| File | Use when |
|---|---|
| `company.md` | About-us 섹션, 회사 개요, 인력 규모, 미션 |
| `products.md` | 제품·솔루션 라인업, 각 제품의 핵심 기능 |
| `differentiation.md` | 기술 차별화, "우리만 할 수 있는 것", 특허·논문·자체 엔진 |
| `track-record.md` | 도메인별 구축 사례, 고객사, 성과 (제안서에서 가장 강력한 무기) |
| `customers.md` | 주력 고객 도메인, 자주 만나는 RFP 패턴, 페르소나 |

## Usage protocol

1. Task가 들어오면 먼저 어느 파일이 필요한지 판단.
   - 제안서 차별화 섹션 → `differentiation.md` + `track-record.md`
   - 회사 소개 슬라이드 → `company.md` + `products.md`
   - RFP 도메인 분석 → `customers.md` + `track-record.md`
2. 해당 파일을 읽고 **인용**. 직접 사실을 만들지 않음 (work-style 금기: "확실하지 않은 것을 확실한 듯 말하기").
3. 파일에 정보가 없으면 **추측 금지**. Jungwoo에게 1회 묻고, 답을 받으면 해당 파일에 추가 권유.

## File freshness

- 정보는 시간이 지나면 낡음 (인력 규모, 트랙레코드, 고객사 변경).
- 사용 시점에 정보가 의심스러우면 Jungwoo에게 확인 요청.
- 새 정보가 들어오면 해당 파일에 추가/갱신.

## Status (현재)

초기 채움 + 1차 보강 완료. 출처:
- `★랩큐소개_실적중심_260429.pdf` (2026-04-29) — 회사 전반·트랙레코드.
- `AI정책보좌관_V5_랩큐.pdf` (2026 Proposal) — AI 정책 보좌관 제품.
- `차세대 지능형LLM 서비스 솔루션_FPGA결과 포함_260401.pptx` — RPU/FPGA 상세 + 고양시 정량 성과.

상태:
- [x] `company.md`
- [x] `products.md` — AI 정책 보좌관, 차세대 LLM 행정/민원 솔루션 추가
- [x] `differentiation.md` — RPU(FPGA) 정량 벤치마크, 하이브리드 LLM, 고도화 RAG, 보이스 AI 보강
- [x] `track-record.md` — 고양시 정량 성과 보강
- [x] `customers.md` — 지자체장 페르소나·6개 지자체 시그니처 시나리오 추가

미해결: 현재 없음.
