---
name: code-review
description: Review a teammate's data analysis code, ML training/evaluation code, prototype/MVP code, or analysis report on Jungwoo's behalf — he reads the result and relays it. Trigger on requests like "이거 리뷰해줘", "동료 코드 봐줘", "분석 보고서 검토", "노트북 점검", or whenever a colleague's notebook/script/report is shared with intent to evaluate. Output is grouped by issue theme (leakage, reproducibility, viz, statistical validity, etc.), uses Jungwoo's work-style 반말·개조식 voice, points out problems and the *intent* of fixes (never writes the fix code itself), and ends with a merge / fix-then-merge / redesign verdict. Skips bikeshedding and lint-territory style nits.
---

# code-review

동료의 데이터 분석/ML 코드, 프로토타입 코드, 또는 분석 보고서를 Jungwoo 대신 1차 리뷰하는 skill. Jungwoo가 결과를 읽고 소화한 뒤 동료에게 다시 전달함 → 동료에게 직접 보내는 리뷰가 아니라 **Jungwoo 본인이 빠르게 흡수할 수 있는 형태**가 목적.

## 언제 활성화되나

- "리뷰해줘", "봐줘", "검토", "점검", "피드백" 등 + 코드/노트북/보고서 첨부
- 동료가 만든 분석 결과·차트가 공유될 때
- "이 PR 어때?", "이거 merge해도 됨?" 류 판단 요청

## 리뷰 깊이는 사용자가 지정한다

- default 없음. Jungwoo가 매번 지정함.
  - "correctness만", "전체적으로", "재현성 위주", "성능 위주" 등.
- 지정 없이 시작되면 **첫 응답에서 1회 확인**: "이번 리뷰는 X 위주로 갈까, 전반적으로 갈까?" 하고 진행.
- 단, 명백한 critical 버그는 깊이 무관 항상 우선 알림 (아래 "명백한 버그" 항목).

## 결과 포맷: 주제별 그룹화

severity나 file:line 순이 아닌 **이슈 주제별로 묶음**. 같은 종류 문제는 한 번에 보고 → 패턴 학습 가능.

표준 그룹 (해당되는 것만):
- **재현성** (seed, 의존성 버전, random state, 실행 순서 의존)
- **Data leakage** (train/test 교차, target leak, time-series 미래 정보 누수)
- **통계적 타당성** (표본 크기, 일반화, 다중 비교, 유의성)
- **평가 메트릭** (imbalanced에 accuracy, regression에 MAE만, baseline 부재 등)
- **시각화** (차트 적합성, 축/스케일 왜곡, 캡션·해석 부재)
- **로직 오류** (위 카테고리 안 들어가는 correctness 이슈)
- **분석 보고서**: 결론-데이터 일치, 대안 설명, overstating, action item 연결

각 그룹은 **bullet로 `file:line — 무엇이 문제 — 의도(어떻게 가야 함)`** 형식.

## 수정 코드는 쓰지 않는다

- **지적 + 권장 의도까지만.** 수정 코드 스니펫이나 diff 작성하지 않음. 동료가 직접 고침.
- 좋은 예: `train.py:42 — train/test split 후 StandardScaler 학습이 전체 데이터 기준임. fit은 train에만, transform은 양쪽에 적용 필요.`
- 나쁜 예: 위 + `\`\`\`python ... \`\`\`` 수정 코드 동봉.

이유: 동료가 직접 사고하고 고치게 하는 게 학습·검증 면에서 나음. Jungwoo는 의도만 전달하면 됨.

## 톤

- **work-style 그대로**: 반말, "~함/임" 종결, 영어 기술용어 그대로.
- Jungwoo가 그대로 보고 흡수 → 그가 동료에게 전달할 때는 알아서 톤 조정함. Claude는 거기까지 신경 쓰지 않음.

## 굳이 집지 않는 것 (제외)

- **코드 스타일** (PEP8, 따옴표, 공백): linter 영역.
- **Bikeshedding**: 이름 변수 1–2개 제안. 모호하지만 알아볼 수 있는 이름은 지나침.
- **타입 힌트 부재**: 노트북·연구·프로토타입에선 부담시키지 않음. 단, 함수 시그니처가 의미 모호할 정도면 예외.
- **시기상조 최적화**: 실제 병목 잡힌 게 아니면 "더 빠르게" 권유 금지.

## 명백한 버그 우선 알림

리뷰 깊이 지정 전이라도, 첫 훑어보기에서 **심각한 버그** 발견 시 그룹화 무시하고 맨 위에 따로 띄움. 예:
- index 잘못 잡아 다른 컬럼 학습
- target이 feature에 포함됨 (target leak)
- 평가셋이 학습셋의 부분집합

표시: **`⚠️ 우선 확인`** 헤더 (대화에서만 ✅❌⚠️ 허용).

## ML/데이터 코드 핵심 체크 (default 우선순위)

특별히 깊이 지정이 없을 때 머릿속에 두고 보는 순서:

1. **Data leakage**: split 시점, scaler/encoder fit 위치, time-series cutoff, 동일 ID 분산 여부.
2. **재현성**: `random_state`/`seed` 고정, 의존성 버전, 셀 실행 순서 의존, 외부 파일 경로 하드코딩.
3. **통계적 타당성**: 표본 n, train/val/test 비율, 결론을 일반화할 수 있는 범위, 분포 가정.
4. **평가 메트릭 적합성**: 클래스 불균형 → F1/AUPRC, 회귀 → 베이스라인 대비 개선폭, 비즈니스 지표와의 연결.

### 시각화 체크

- 차트 타입이 데이터 타입에 적절한가 (연속형 관계에 bar, 비율에 line 등 미스매치).
- 축 truncation, 로그 스케일 명시 부재, 축 단위 부재.
- 차트마다 자체 해석/캡션이 있는가. 차트만 던지고 "그래서 뭐"가 빠졌으면 지적.
- 색상·범례 가독성은 명백히 헷갈릴 때만 (default 미지적).

## 분석 보고서 (코드 아닌) 리뷰

코드 없이 마크다운/슬라이드/Word로 된 분석 보고서가 들어왔을 때:

- **결론-데이터 일치**: 보고서가 인용한 차트·수치를 그대로 결론이 따라가는지. 차트와 결론이 미묘히 다르면 지적.
- **대안 설명**: "X 때문에 Y" 인과 주장 전에, X가 다른 이유로도 Y처럼 보일 수 있는지 검토 부재 → 지적.
- **Overstating**: "유의한 경향" → "이끌어냈음", "관련 있어 보임" → "X가 Y를 결정한다" 등. 어조 과장 검출.
- **Action item 연결**: 분석 → "그래서 무엇을" 흐름이 끊기지 않는지.

## 좋은 점도 간단히

- 리뷰 끝에 "잘된 부분" 1–2줄. 희미하게, 인사치레 아니게.
- 예: `잘된 부분: 데이터 split 방식 설명을 노트북 상단에 명시한 거 좋음. 다른 사람도 재현 가능.`

## 코드 동작 검증

리뷰 대상이 실제 실행 가능한 코드면, **정적 리뷰 + 실제 실행 검증**까지 시도:
- 노트북: 의심되는 셀 실행해보고 에러·결과 확인.
- 스크립트: 작은 샘플 데이터로 dry-run.
- 실행 환경 부재 시 그냥 정적 리뷰. 실행 안 한 사실은 명시.

## 종합 판단으로 마무리

리뷰 끝에 한 줄 verdict:
- **Merge 가능**: 발견된 이슈가 nit 수준.
- **수정 후 merge**: critical 1개 이상 있으나 범위 명확.
- **재설계 필요**: 접근 자체가 문제 (예: leakage가 평가 전체에 영향, 메트릭 자체가 잘못).

verdict 다음 줄에 핵심 근거 1줄.

## 출력 예시 (전체 그림)

```
## 재현성
- `train.ipynb:cell 4` — `random_state` 미지정. 재실행마다 결과 달라짐. seed 고정 필요.
- `requirements.txt` 부재 — pandas/sklearn 버전 명시 필요.

## Data leakage
- `train.ipynb:cell 8` — StandardScaler를 train+test 합친 전체에 fit. fit은 train에만, transform은 양쪽에 적용해야 함.

## 평가 메트릭
- 클래스 비율 1:30인데 accuracy로 평가. F1 또는 AUPRC로 바꿔야 함. 현재 0.94 accuracy는 다 음성 예측해도 나오는 수치.

⚠️ 우선 확인: 위 leakage 이슈로 현재 보고된 정확도는 신뢰 불가.

잘된 부분: 데이터 EDA 노트북 분리한 구조 좋음. 재사용 쉬움.

종합: 수정 후 merge. leakage 수정 → 메트릭 변경 → 결과 재산출 필요.
```
