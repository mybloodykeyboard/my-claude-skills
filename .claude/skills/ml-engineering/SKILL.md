---
name: ml-engineering
description: General-purpose ML engineering guidance for tabular regression/classification tasks. Activate when working on training/evaluating ML models in a project — running EDA, choosing models, designing train/test/CV splits, picking metrics, handling skewed targets, avoiding leakage, structuring ensembles, and reporting results. Provides defaults that complement work-style (terse) and code-review (issue-grouped review) skills — this skill is the *engineering* layer (how to build), not the review layer (what to critique).
---

# ml-engineering

ML 엔지니어링 기본 원칙. Jungwoo가 tabular ML 학습/평가 코드를 작성하거나 리뷰할 때 적용. work-style 톤(반말·개조식) 따르고, code-review skill과 중복되지 않게 **빌드 방향**에만 집중.

## 언제 활성화되나

- "회귀 모델 만들어", "예측 모델 학습해" 등 ML 학습 작업
- EDA, feature engineering, 모델 비교, 평가 작업
- 학습 파이프라인 설계 / 재현성 점검
- 학습 결과 해석 / 모델 선택

리뷰성 작업이면 code-review skill이 우선. 이 skill은 **새로 만들 때** 적용.

## 데이터 점검 (학습 전 default)

매번 학습 코드 짜기 전 30초 안에 끝내는 체크:
- `df.shape`, `df.dtypes`, `df.isnull().sum()`
- target 분포: `describe()`, `quantile([0.01,0.5,0.95,0.99])`, 0/음수 비율
- target 범위가 여러 order of magnitude → **log1p 변환** default
- categorical 컬럼의 cardinality, rare value 비율
- feature ↔ target 상관 (continuous: corr, categorical: groupby mean)
- **feature 분포의 outlier도 점검** — 정상 척도(예: 평점 1~10) 벗어난 값은 데이터 오류일 가능성. clip 또는 NaN 처리.

## Subgroup-conditional 관계 점검

corr이 0에 가까운 feature를 그냥 버리지 말 것. **다른 변수로 conditioning하면 강한 패턴이 드러날 수 있음** (Simpson's paradox).

방법:
- 의심되는 binary/categorical feature로 데이터 split → 각 subgroup 내에서 corr 재계산
- cross-tab: `df.pivot_table(index=feature_bin, columns=group_flag, values=target, aggfunc='median')`
- 두 subgroup에서 부호가 반대거나 한쪽만 강한 관계면 → **interaction feature** 추가 (`feature × group_flag`).

GBM이 이걸 implicit하게 학습할 수 있지만, 명시적 interaction이 importance 상위 진입하면 ±k% 적중률 등 보조 metric에서 lift 받는 경우 종종 있음.

**실전 예** (movie-recsys 과제):
- `관객평가 vs 총관객수` 전체 Pearson corr -0.008 (무의미해 보임)
- 그런데 평가 max=471 outlier 1건 있음 → clip 후 corr +0.063 (부호 뒤집힘. 1건 outlier가 통계 망가뜨림 — 항상 outlier 점검 먼저)
- `다양성영화` 플래그로 split: 대중영화 corr +0.186 (Spearman +0.192), 독립영화 corr +0.148. 양 그룹 모두 양의 관계지만 합치면 0에 가까움 (두 그룹의 관객수 절대 수준 차이로 cancel out — Simpson's paradox)
- Median 관객수로 효과 크기 보강: 대중영화 평가 0~5 → 9~10에서 **4.6배 증가**, 독립영화는 plateau
- `관객평가 × (1-다양성영화)` interaction이 feature importance 상위 진입 (raw 관객평가 단독보다 강한 신호)

## Split 전략 (필수)

- **train/test split 명시적**: 절대 CV만 의존하지 말 것. 최종 holdout 따로.
- test set은 **1회만** 사용. 튜닝 중 보면 leakage.
- CV는 train 내부에서. shuffle=True, `random_state` 고정.
- 시계열 데이터면 `TimeSeriesSplit`, group ID 있으면 `GroupKFold`.
- 만약 데이터 < 1000행: 5-fold도 너무 작아 LOO 또는 nested CV.

## Leakage 체크리스트

- target 변환은 fold 안에서 (예: log mean encoding을 train에서만 fit).
- StandardScaler / Encoder는 train fold에 fit, test fold에 transform.
- 시계열: test가 train보다 미래여야. `train_test_split`에 `shuffle=False`.
- Target의 후행 정보가 feature에 섞였는지 도메인 확인 (예: 영화 관객수 예측에 "최종 평점" 들어가면 의심).

## Target 변환

- 분포가 long-tail / log-normal → **log1p(target)** 권장 (raw는 squared loss가 큰 값에 휘둘림).
- 학습은 log scale, 평가/보고는 **양쪽 다**. R²(raw)와 R²(log)는 의미가 다름.
- log→raw 역변환 시 **Duan's smearing** 고려: `E[Y|X] ≈ exp(E[logY|X]) * mean(exp(residuals))`. residual 분산이 작으면 영향 미미하지만 클 땐 큰 차이.
- 0 또는 음수 target 있으면 log 불가 → log1p 또는 Yeo-Johnson.

## 모델 선택 default

Tabular regression/classification:
1. **Baseline**: Ridge/Logistic (변환된 feature로). 모델 capacity 부족이 문제인지 확인용.
2. **GBM**: LightGBM 또는 XGBoost. tabular 대부분 SOTA. n_estimators 1000~2000 + early stopping.
3. **RandomForest**: GBM 비교용 robustness 체크.
4. **MLP**: 데이터 < 10만행이면 비추. GBM이 거의 항상 이김.
5. **Stacking/Ensemble**: weighted avg를 먼저. stacking은 robust한 base 모델 3개 이상 있을 때만.

## 잔차 패턴별 처방

학습 후 잔차를 quantile 또는 그룹별로 분석. 패턴에 따라 처방이 다름:

- **Regression to mean** (큰 값 underpredict / 작은 값 overpredict):
  - tabular 모델의 흔한 실패 모드. squared loss + log target에서 종종 발생.
  - 처방 A: target을 4-tier 등으로 이산화 → tier classifier 학습 → tier 확률을 feature로 concat (효과 약함, 같은 feature 재학습)
  - 처방 B: **Mixture of Experts** — tier별 expert regressor + soft probability blending. 데이터 작은 tier는 overlap(±1 tier)으로 학습. base ensemble과 blend하면 robust.
  - 처방 C: quantile regression — 큰 영화엔 q=0.7~0.9 예측, 작은 영화엔 q=0.3 예측. 비대칭 loss 직접 타격.
  - 한계: 만약 oracle tier (정답 tier 사용)로도 R² 거의 그대로면 → **데이터에 tier-별 추가 신호 없음** = 더 깊은 feature engineering / 외부 데이터 필요. 더 짜내는 것 무의미.

- **Heteroscedastic noise** (큰 값에서 분산 ↑):
  - log/Yeo-Johnson 변환으로 분산 안정화.
  - 또는 GBM에 quantile 또는 Huber loss.

- **Bias on specific subgroup** (특정 카테고리에서 systematic 오차):
  - 해당 카테고리 indicator + interaction 추가.
  - 그룹별 학습률/모델 분리 검토.

**핵심 진단 도구**:
- 잔차 분위수별 mean bias / RMSE: regression to mean 즉시 보임.
- 상위 N% 큰 값이 SS_residual의 몇 %인가: R²(raw) 한계 진단.
- Oracle 실험: 정답 정보를 일부 알려줘도 R² 못 올라가면 → 데이터의 본질적 ceiling. 더 안 짜내고 보고하는 게 정답.

LGBM 권장 default (tabular regression):
```
n_estimators=2000, num_leaves=15, learning_rate=0.03,
min_child_samples=20, subsample=0.9, colsample_bytree=0.9,
reg_lambda=5.0, random_state=42
```
- **early stopping callback 50 rounds 거의 항상 켜라**. 안 켜면 train R² 0.99 / val 0.79 같은 큰 gap 흔함.
- 과적합 신호 (val loss 증가) → num_leaves↓ (15→7), min_child_samples↑ (20→50), reg_lambda↑.
- 데이터 < 5000행이면 num_leaves 15, depth 4~5가 default. 31은 데이터 1만+ 이상에서.

**진단 루틴**: 학습 후 train R² vs val R² gap 항상 확인.
- gap > 0.1: 과적합. reg 강화.
- gap < 0.05: underfit 가능. capacity 늘리기.
- gap 줄여도 val 변화 없으면 → **데이터 ceiling**. 더 짜내려 하지 말고 보고.

## 메트릭

Regression:
- **R²**: variance explained. 기본. 단 long-tail에선 큰 값이 R² 지배.
- **MAPE**: `mean(|y-y_hat|/|y|)`. 작은 y에 0 가까우면 폭발. `max(|y|, eps)`로 clip.
- **MdAPE**: median APE. 이상치 robust.
- **RMSE**: scale 의존, 절대 비교용.
- **±k% 적중률**: 비즈니스 친화적 지표. MAPE보다 직관적.

**단일 메트릭 보고 금지**. 최소 R² + MAPE + MdAPE 세트. log target이면 R²(log)도 함께.

**평가 기준에 메트릭이 여러 개면 dual-track 학습 고려**:
- 예: R² (squared loss로 최적화) + ±k% 적중률 (relative error 최적화)
- squared loss는 outlier에 끌려 평균 회귀. ±k% 적중률엔 불리.
- **L1 loss on log target** ≈ relative error 최소화 → median-aligned 예측, ±k% 적중에 유리.
- R²-optimized track + ±k%-optimized track 별도 학습 후 blend (0.5/0.5 default).
- 한 트랙이 다른 메트릭을 망치지 않게 양쪽 모두 보고.

Classification: imbalanced면 accuracy 금지. F1, AUPRC, calibration error 같이.

## 재현성

매 학습 스크립트에:
- seed 고정 (`random_state=42` 등 1곳에서 상수로)
- `np.random.seed`, `random.seed`, GPU면 `torch.manual_seed` 등도
- `requirements.txt` 또는 `pyproject.toml` 의존성 버전 명시
- 결과 파일 (scores/predictions) 산출 + timestamp 로깅

## 보고 형식

학습 후 출력:
```
[split] train=N1 test=N2 features=F
[CV summary]  (각 모델 R²/MAPE/MdAPE/within_10pct)
[ensemble weights]
[TEST set scores]  (model별 + ensemble 한 줄씩)
[saved] results/*.csv
```

표는 1줄/모델 — bullet/장황한 설명 금지. 결론은 별도로 (보고서/REPORT.md).

## 흔한 함정 (자주 잡는 것)

- 작은 데이터 (< 5000행)에 깊은 트리·딥러닝 → 과적합. simple model + 강한 regularization.
- 메트릭 1개 보고 → 분포·이상치 가려짐.
- 학습 R²와 test R² 비교 안 하고 일반화 주장.
- target = 1 같은 마커값 무시 (도메인에서 "데이터 없음" 표시일 수 있음). 학습 전 도메인 확인 필수.
- categorical 변수를 ordinal로 잘못 인코딩 (예: 장르 1~10 정수). one-hot 또는 target encoding으로.

## 산출물 구조 (default)

프로젝트가 ML 위주면:
```
src/
  features.py    # feature engineering (build_features 함수)
  train.py       # 학습 + 평가, CLI entrypoint
results/
  cv_scores.csv
  test_scores.csv
  predictions.csv
  run.log        # 각 실행 메타 (timestamp, seed, scores)
REPORT.md        # 접근/결과/한계 1장 요약
```

사용자가 다른 구조 명시하면 따른다.

## 다른 skill과의 관계

- **work-style**: 항상 우선. 톤(반말), 길이(짧게), 금기(자동 README 생성) 적용.
- **code-review**: 이 skill은 **빌드 단계**, code-review는 **검토 단계**. 동료 코드 들어오면 code-review로 전환.
- **proposal-writing**: ML 결과를 외부 제안서에 쓸 때는 proposal-writing이 우선 (존대어, 비즈니스 어조).

이 skill은 ML 작업의 default 가이드. 사용자 지시가 더 구체적이면 그쪽이 우선.
