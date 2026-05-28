# Vertical Slice Architecture for Machine Learning Systems

## Navigation

- Core idea and ML-specific motivation: sections 1-3
- When to use VSA in ML: section 4
- ML project layouts and top-level structure: sections 5-6
- Training, inference, command/query, and pipeline patterns: sections 8-14
- Slice-owned evaluation, monitoring, and batch/online serving: sections 15-19 and 27
- LLM, VLM, recommender, forecasting, CV, NLP, and anomaly examples: sections 20-26
- Dependency rules, testing, CI/CD, orchestration, and deployment: sections 29-34
- Data ownership, feature stores, versioning, and configuration: sections 35-38
- Anti-patterns, tradeoffs, decision matrix, and review checklist: sections 39-42
- Migration, naming, templates, and final agent rules: sections 48-59

## Purpose of this skill

Use this skill when designing, reviewing, refactoring, or generating machine learning software using Vertical Slice Architecture. This skill is focused only on ML systems and covers both training and inference.

The goal is to help agents design ML codebases where each business or ML capability owns its own data access, feature logic, model training, evaluation, inference, contracts, and tests, while shared infrastructure remains small, stable, and platform-oriented.

This skill is not a generic ML tutorial. It is an architecture guide for applying Vertical Slice Architecture to real ML projects.

---

## Source basis

This skill synthesizes ideas from:

- Jimmy Bogard's Vertical Slice Architecture writing, especially the idea of organizing around requests and coupling along the axis of change.
- MLOps guidance from Google Cloud, AWS SageMaker, MLflow, Feast, KServe, BentoML, TensorFlow Extended, Microsoft TDSP, and research literature on MLOps architecture.
- Production ML concepts such as training-serving skew, model registry, feature store, online serving, batch scoring, continuous training, monitoring, drift, lineage, and deployment governance.
- Practical project structures used in tabular ML, computer vision, NLP, recommender systems, time-series forecasting, LLM fine-tuning, and multimodal ML.

Primary reference URLs are listed in the bibliography at the end.

---

## 1. Core idea

Vertical Slice Architecture means organizing software around independent user-visible or business-relevant capabilities instead of technical layers.

In classic backend systems, a vertical slice might be:

```text
CreateOrder
CancelSubscription
GetCustomerProfile
UploadInvoice
```

In ML systems, a vertical slice is usually a full ML capability or workflow:

```text
predict_churn
rank_products
classify_invoice
train_fraud_model
evaluate_vlm_answers
serve_image_similarity
forecast_demand
fine_tune_support_agent
batch_score_credit_risk
```

A vertical slice should contain most things needed to deliver that capability:

```text
slice/
  api.py
  schemas.py
  dataset.py
  features.py
  train.py
  evaluate.py
  predict.py
  registry.py
  monitoring.py
  tests/
```

The slice should not depend on unrelated slices. It may depend on stable shared platform infrastructure such as storage, logging, registry clients, queues, object storage, config loading, tracing, and deployment helpers.

---

## 2. Why ML needs a special version of Vertical Slice Architecture

Machine learning systems are different from normal software systems because they contain at least four kinds of changing artifacts:

1. Code
2. Data
3. Model artifacts
4. Runtime behavior under changing real-world distributions

A normal backend feature changes mainly because business rules change. An ML feature changes because of:

- Input data schema changes
- Feature definitions changing
- Label definitions changing
- Model architecture changes
- Training hyperparameters changing
- Evaluation metrics changing
- Thresholds changing
- Latency or cost constraints changing
- Drift and retraining requirements changing
- Regulatory or explainability requirements changing
- Model registry and deployment policies changing

Because of this, a layered ML architecture often becomes painful:

```text
src/
  data/
  features/
  models/
  training/
  evaluation/
  inference/
  api/
```

When you change one business capability, you may need to edit files in every folder.

Vertical Slice Architecture changes the axis of organization:

```text
src/slices/
  churn_prediction/
  fraud_detection/
  invoice_classification/
  product_ranking/
```

Each capability owns its local details.

---

## 3. Main rule for ML vertical slices

Use this rule:

> If code changes mainly because one ML capability changes, keep it inside that capability slice.

Examples:

| Code or artifact | Usually belongs where? | Reason |
|---|---|---|
| Churn-specific feature engineering | `slices/churn_prediction/features.py` | It changes with churn modeling |
| Fraud-specific threshold policy | `slices/fraud_detection/thresholds.py` | It is domain behavior |
| Common S3 client | `platform/storage/s3.py` | It is infrastructure |
| Common MLflow wrapper | `platform/model_registry/mlflow.py` | It is infrastructure |
| Shared CustomerId value object | `shared_kernel/types.py` | Stable domain primitive |
| All project feature logic | Avoid | Creates fake slices |
| All training code in one global trainer | Usually avoid | Couples unrelated models |
| Generic PyTorch training loop utility | `platform/training/pytorch_loop.py` or `shared_ml/` | Reusable technical primitive |

---

## 4. When to use Vertical Slice Architecture in ML

Use VSA when the project has two or more distinct capabilities, products, workflows, or model-serving responsibilities.

Good fits:

```text
- Production ML APIs
- Batch scoring systems
- ML platforms with many models
- Recommender systems with multiple ranking/retrieval stages
- Fraud/risk systems
- Forecasting systems for multiple business domains
- LLM fine-tuning and evaluation platforms
- Multimodal inference services
- ML systems with separate training and serving workflows
- ML systems where each capability has different data, metrics, labels, and deployment rules
```

Weaker fits:

```text
- One notebook exploration
- One-off Kaggle-style project
- One tiny model with no production path
- Research code where boundaries are not understood yet
- Library projects where the product is the reusable algorithm itself
```

In early research, a standard data science layout can be simpler. As soon as the work turns into a production capability, VSA becomes more useful.

---

## 5. VSA compared with common ML project layouts

### 5.1 Layered ML layout

```text
src/
  data/
    load_transactions.py
    load_users.py
  features/
    churn_features.py
    fraud_features.py
  models/
    churn_model.py
    fraud_model.py
  training/
    train_churn.py
    train_fraud.py
  inference/
    predict_churn.py
    predict_fraud.py
  api/
    churn_routes.py
    fraud_routes.py
```

This looks organized, but a churn change crosses many layers.

### 5.2 Vertical slice ML layout

```text
src/
  platform/
    db/
    object_storage/
    model_registry/
    observability/

  slices/
    churn_prediction/
      api.py
      schemas.py
      dataset.py
      features.py
      train.py
      evaluate.py
      predict.py
      registry.py
      tests/

    fraud_detection/
      api.py
      schemas.py
      dataset.py
      features.py
      train.py
      evaluate.py
      predict.py
      registry.py
      tests/
```

A churn change is mostly local.

### 5.3 Hybrid layout

For larger ML systems, use a hybrid:

```text
src/
  platform/              # stable infrastructure
  shared_kernel/         # stable domain primitives
  shared_ml/             # stable technical ML utilities
  slices/                # business capabilities
```

This avoids duplication while keeping business-specific behavior local.

---

## 6. Recommended top-level structure

```text
ml-system/
  pyproject.toml
  README.md
  configs/
    local.yaml
    staging.yaml
    production.yaml

  src/
    app/
      main.py
      dependency_injection.py

    platform/
      config/
      storage/
      database/
      queue/
      observability/
      feature_store/
      model_registry/
      serving/
      orchestration/

    shared_kernel/
      ids.py
      errors.py
      result.py
      time.py

    shared_ml/
      metrics/
      calibration/
      dataset_versioning/
      pytorch_utils/
      sklearn_utils/
      serialization/

    slices/
      fraud_detection/
      churn_prediction/
      invoice_classification/
      product_recommendation/

  tests/
    contract/
    integration/
    e2e/

  pipelines/
    airflow/
    kubeflow/
    github_actions/

  infra/
    docker/
    k8s/
    terraform/
```

Use `src/slices` for domain or capability behavior. Use `platform` for infrastructure. Use `shared_ml` only for stable reusable algorithms or technical helpers.

---

## 7. What is a slice in ML?

A slice can represent different things depending on the system.

### 7.1 Prediction capability slice

```text
slices/churn_prediction/
  api.py
  schemas.py
  predict.py
  features.py
  model_loader.py
  thresholds.py
  explain.py
  tests/
```

Use when the runtime endpoint is the main capability.

### 7.2 Training capability slice

```text
slices/train_churn_model/
  command.py
  dataset.py
  features.py
  train.py
  evaluate.py
  register.py
  tests/
```

Use when training is an independent workflow.

### 7.3 Combined train and infer slice

```text
slices/churn_prediction/
  api.py
  command_train.py
  command_batch_score.py
  dataset.py
  features.py
  train.py
  evaluate.py
  predict.py
  registry.py
  monitoring.py
  tests/
```

Use when one team owns the full lifecycle for one capability.

### 7.4 Pipeline stage slice

Sometimes a slice is not a final business use case, but a productized internal workflow:

```text
slices/evaluate_model_candidate/
  command.py
  load_candidate.py
  benchmark.py
  quality_gates.py
  report.py
  tests/
```

This is valid if the workflow is an independently valuable operation.

---

## 8. Training and inference in the same slice

For production ML, training and inference should usually be close enough that you can verify consistency.

Example:

```text
slices/fraud_detection/
  contracts.py
  feature_spec.py
  dataset.py
  features.py
  train.py
  evaluate.py
  predict.py
  model_card.py
  registry.py
  tests/
```

Training and inference should share:

- Input schema definitions
- Feature definitions
- Label policy
- Preprocessing code
- Model artifact contract
- Evaluation thresholds
- Model registry metadata

They should not necessarily share the same runtime process. Training may run in a batch job, while inference may run in a FastAPI, KServe, BentoML, vLLM, or batch scoring worker.

The architectural goal is not same process. The goal is same slice ownership and same contracts.

---

## 9. Pattern: command and query split for ML

CQRS maps well to ML.

Commands change state:

```text
TrainModel
RegisterModelCandidate
PromoteModelVersion
ImportTrainingDataset
BackfillFeatures
TriggerRetraining
RunBatchScoring
```

Queries return answers:

```text
Predict
ExplainPrediction
GetModelMetrics
GetModelCard
GetPredictionHistory
GetFeatureValues
```

Example structure:

```text
slices/fraud_detection/
  commands/
    import_dataset.py
    train_model.py
    evaluate_candidate.py
    promote_model.py
    batch_score.py
  queries/
    predict.py
    explain.py
    get_metrics.py
  common/
    schemas.py
    features.py
    model_contract.py
```

Use command/query split when a slice becomes complex. For small slices, flat files are fine.

---

## 10. Pattern: train/evaluate/promote pipeline inside a slice

```text
slices/churn_prediction/
  train_pipeline.py
  dataset.py
  features.py
  train.py
  evaluate.py
  promote.py
  quality_gates.py
```

Example Python code:

```python
# src/slices/churn_prediction/train_pipeline.py
from .dataset import load_training_frame
from .features import build_feature_matrix
from .train import train_model
from .evaluate import evaluate_model
from .promote import register_candidate
from .quality_gates import assert_candidate_is_promotable


def run_churn_training(dataset_uri: str, run_id: str) -> str:
    frame = load_training_frame(dataset_uri)
    x_train, y_train, x_valid, y_valid = build_feature_matrix(frame)

    candidate = train_model(x_train, y_train, run_id=run_id)
    report = evaluate_model(candidate, x_valid, y_valid)

    assert_candidate_is_promotable(report)
    model_version = register_candidate(candidate, report)
    return model_version
```

Good properties:

- The full flow is visible in one place.
- Evaluation gates live with the business capability.
- Model registration metadata can include slice name and dataset version.
- Tests can run the pipeline with a small fixture dataset.

---

## 11. Pattern: inference endpoint inside a slice

```python
# src/slices/churn_prediction/api.py
from fastapi import APIRouter, Depends
from .schemas import ChurnPredictionRequest, ChurnPredictionResponse
from .predict import ChurnPredictor, get_predictor

router = APIRouter(prefix="/churn", tags=["churn"])


@router.post("/predict", response_model=ChurnPredictionResponse)
def predict_churn(
    request: ChurnPredictionRequest,
    predictor: ChurnPredictor = Depends(get_predictor),
) -> ChurnPredictionResponse:
    return predictor.predict(request)
```

```python
# src/slices/churn_prediction/predict.py
from .features import build_online_features
from .schemas import ChurnPredictionRequest, ChurnPredictionResponse
from .thresholds import classify_risk


class ChurnPredictor:
    def __init__(self, model, feature_client):
        self.model = model
        self.feature_client = feature_client

    def predict(self, request: ChurnPredictionRequest) -> ChurnPredictionResponse:
        features = build_online_features(request, self.feature_client)
        probability = float(self.model.predict_proba(features)[0, 1])
        risk = classify_risk(probability)
        return ChurnPredictionResponse(
            customer_id=request.customer_id,
            churn_probability=probability,
            risk=risk,
        )


def get_predictor() -> ChurnPredictor:
    # Wire this through dependency injection in real apps.
    raise NotImplementedError
```

The endpoint is thin. The slice owns feature construction, thresholding, prediction response, and tests.

---

## 12. Pattern: shared feature definition with local ownership

Training-serving skew happens when training-time feature computation differs from serving-time feature computation. A feature store can reduce this by making features consistently available for training and low-latency serving.

In VSA, the key question is ownership.

Bad:

```text
shared/features/all_features.py
```

Better:

```text
slices/churn_prediction/feature_spec.py
slices/fraud_detection/feature_spec.py
platform/feature_store/client.py
```

Example:

```python
# src/slices/churn_prediction/feature_spec.py
from dataclasses import dataclass


@dataclass(frozen=True)
class FeatureSpec:
    name: str
    dtype: str
    source: str
    freshness_minutes: int


CHURN_FEATURES = [
    FeatureSpec("days_since_last_login", "float", "user_activity", 60),
    FeatureSpec("subscription_age_days", "float", "billing", 1440),
    FeatureSpec("support_tickets_30d", "int", "support", 60),
]
```

```python
# src/slices/churn_prediction/features.py
from .feature_spec import CHURN_FEATURES


def build_training_features(feature_store, entity_df):
    return feature_store.get_historical_features(
        feature_specs=CHURN_FEATURES,
        entity_df=entity_df,
    )


def build_online_features(request, feature_store):
    return feature_store.get_online_features(
        feature_specs=CHURN_FEATURES,
        entity_id=request.customer_id,
    )
```

The infrastructure client is shared, but the business feature set is owned by the slice.

---

## 13. Pattern: model registry as infrastructure, model policy as slice logic

Model registry code should be shared infrastructure.

```text
platform/model_registry/mlflow_client.py
```

Model selection and promotion rules belong inside the slice.

```text
slices/churn_prediction/promote.py
slices/churn_prediction/quality_gates.py
```

Example:

```python
# src/slices/churn_prediction/quality_gates.py
from dataclasses import dataclass


@dataclass(frozen=True)
class EvaluationReport:
    auc: float
    brier_score: float
    latency_p95_ms: float
    calibration_error: float


def assert_candidate_is_promotable(report: EvaluationReport) -> None:
    if report.auc < 0.82:
        raise ValueError("AUC is below churn promotion threshold")
    if report.brier_score > 0.16:
        raise ValueError("Brier score is too high")
    if report.latency_p95_ms > 80:
        raise ValueError("p95 latency is too high")
    if report.calibration_error > 0.04:
        raise ValueError("Calibration error is too high")
```

This keeps business-specific quality gates near the model they control.

---

## 14. Pattern: model artifact contract

Every production ML slice should define the artifact contract it expects.

```python
# src/slices/fraud_detection/model_contract.py
from dataclasses import dataclass
from typing import Literal


@dataclass(frozen=True)
class FraudModelContract:
    slice_name: Literal["fraud_detection"]
    model_kind: Literal["xgboost", "lightgbm", "sklearn", "torch"]
    input_schema_version: str
    feature_spec_version: str
    output_schema_version: str
    threshold_policy_version: str
```

Register this metadata with the model version. At inference startup, validate the loaded model:

```python
# src/slices/fraud_detection/model_loader.py
from .model_contract import FraudModelContract


def validate_loaded_model(metadata: dict) -> FraudModelContract:
    contract = FraudModelContract(**metadata["contract"])
    if contract.slice_name != "fraud_detection":
        raise ValueError("Wrong model loaded for fraud_detection")
    if contract.input_schema_version != "v3":
        raise ValueError("Incompatible input schema")
    return contract
```

This prevents accidental cross-slice model usage.

---

## 15. Pattern: slice-owned evaluation

Evaluation metrics are not globally universal. A recommender, classifier, ranking model, forecast model, and LLM evaluator need different metrics.

Keep evaluation local unless it is truly generic.

```text
slices/product_ranking/evaluate.py
slices/demand_forecast/evaluate.py
slices/document_classifier/evaluate.py
slices/vlm_answer_quality/evaluate.py
```

Example:

```python
# src/slices/product_ranking/evaluate.py
from dataclasses import dataclass


@dataclass(frozen=True)
class RankingMetrics:
    ndcg_at_10: float
    recall_at_50: float
    coverage: float
    p95_latency_ms: float


def evaluate_ranking(model, validation_queries) -> RankingMetrics:
    # Pseudocode: implement retrieval/ranking metric calculation here.
    return RankingMetrics(
        ndcg_at_10=0.0,
        recall_at_50=0.0,
        coverage=0.0,
        p95_latency_ms=0.0,
    )
```

Do not force every ML capability through one global `metrics.py` unless that module contains only stable mathematical primitives.

---

## 16. Pattern: thin platform, fat slices

Platform modules should provide mechanisms, not business decisions.

Good platform code:

```python
# src/platform/model_registry/client.py
class ModelRegistry:
    def log_model(self, name: str, artifact_path: str, metadata: dict) -> str:
        raise NotImplementedError

    def load_model(self, name: str, alias: str):
        raise NotImplementedError
```

Bad platform code:

```python
# Bad: platform decides churn-specific production quality.
def should_promote_churn_model(metrics):
    return metrics["auc"] > 0.82
```

Better:

```text
platform/model_registry/client.py          # generic
slices/churn_prediction/quality_gates.py   # churn-specific
```

---

## 17. Pattern: slice-local notebooks

Notebooks are acceptable in ML, but keep them close to the capability.

```text
slices/churn_prediction/notebooks/
  2026-05-eda.ipynb
  2026-05-calibration-analysis.ipynb
```

Rules:

- Notebooks are exploration, not production source of truth.
- Production logic must move into `dataset.py`, `features.py`, `train.py`, `evaluate.py`, or similar files.
- Notebook outputs should record dataset version, code commit, and model artifact version.

---

## 18. Pattern: batch scoring slice

Batch scoring often differs from online prediction.

```text
slices/churn_batch_scoring/
  command.py
  dataset.py
  features.py
  predict_batch.py
  output_writer.py
  reconciliation.py
  tests/
```

Example:

```python
# src/slices/churn_batch_scoring/command.py
from .dataset import load_customers_to_score
from .features import build_batch_features
from .predict_batch import score_customers
from .output_writer import write_scores


def run_batch_scoring(score_date: str) -> None:
    customers = load_customers_to_score(score_date)
    features = build_batch_features(customers, score_date)
    scores = score_customers(features)
    write_scores(scores, score_date)
```

Do not blindly reuse the online endpoint code if batch scoring has different performance, freshness, idempotency, and output requirements.

---

## 19. Pattern: online inference slice

Online inference needs latency, availability, request validation, observability, and often feature freshness checks.

```text
slices/fraud_online_inference/
  api.py
  schemas.py
  features.py
  predict.py
  explain.py
  latency_budget.py
  monitoring.py
  tests/
```

Example:

```python
# src/slices/fraud_online_inference/latency_budget.py
from dataclasses import dataclass


@dataclass(frozen=True)
class LatencyBudget:
    feature_fetch_ms: int
    model_predict_ms: int
    postprocess_ms: int
    total_ms: int


BUDGET = LatencyBudget(
    feature_fetch_ms=20,
    model_predict_ms=30,
    postprocess_ms=10,
    total_ms=80,
)
```

Latency budgets are not generic infrastructure. They are part of the product contract for that slice.

---

## 20. Pattern: LLM fine-tuning slice

LLM and VLM projects benefit from VSA because each training objective has its own dataset format, prompt template, loss masking, evaluation, and decoding rules.

```text
slices/fine_tune_support_agent/
  dataset.py
  chat_template.py
  loss_masking.py
  train_lora.py
  evaluate.py
  decode.py
  model_card.py
  tests/
```

Example:

```python
# src/slices/fine_tune_support_agent/chat_template.py
SYSTEM_PROMPT = "You are a precise support assistant."


def format_chat_sample(sample: dict) -> list[dict]:
    return [
        {"role": "system", "content": SYSTEM_PROMPT},
        {"role": "user", "content": sample["question"]},
        {"role": "assistant", "content": sample["answer"]},
    ]
```

```python
# src/slices/fine_tune_support_agent/loss_masking.py
ASSISTANT_ROLE = "assistant"


def should_train_token(message_role: str) -> bool:
    return message_role == ASSISTANT_ROLE
```

This avoids mixing unrelated prompt formats across projects.

---

## 21. Pattern: VLM evaluation slice

For multimodal evaluation:

```text
slices/evaluate_vlm_answer_quality/
  dataset.py
  prompt_builder.py
  image_loader.py
  inference.py
  decode.py
  metrics.py
  report.py
  tests/
```

Example:

```python
# src/slices/evaluate_vlm_answer_quality/prompt_builder.py
from dataclasses import dataclass


@dataclass(frozen=True)
class VlmEvalPrompt:
    text: str
    image_paths: list[str]


def build_prompt(sample: dict) -> VlmEvalPrompt:
    return VlmEvalPrompt(
        text=f"Answer the question based on the image: {sample['question']}",
        image_paths=[sample["image_path"]],
    )
```

```python
# src/slices/evaluate_vlm_answer_quality/decode.py
def clean_model_output(raw_text: str) -> str:
    return (
        raw_text
        .replace("<|assistant|>", "")
        .replace("<|end|>", "")
        .strip()
    )
```

Decoding rules should be slice-local if they are tied to the model family or evaluation protocol.

---

## 22. Pattern: recommender system slices

Recommender systems often have multiple vertical slices:

```text
slices/
  candidate_generation/
  product_ranking/
  rerank_for_diversity/
  similar_items/
  personalized_homepage/
```

Each slice may have different models and serving constraints.

Example product ranking slice:

```text
slices/product_ranking/
  query_context.py
  candidate_loader.py
  features.py
  ranker.py
  evaluate.py
  api.py
  tests/
```

Candidate generation and ranking should not be forced into one global `recommender/` if they change independently.

---

## 23. Pattern: forecasting slices

Forecasting often tempts teams to create global `forecasting/` modules. Prefer business-specific slices when the data, horizon, seasonality, and evaluation differ.

```text
slices/
  forecast_store_demand/
  forecast_cashflow/
  forecast_support_tickets/
  forecast_gpu_capacity/
```

Example:

```text
slices/forecast_gpu_capacity/
  dataset.py
  calendar_features.py
  train.py
  evaluate.py
  predict.py
  capacity_policy.py
  tests/
```

Forecasting policies, horizons, and loss functions are often domain-specific.

---

## 24. Pattern: computer vision slices

```text
slices/
  classify_product_image/
  detect_defective_part/
  extract_invoice_fields/
  compare_image_similarity/
```

Example image classifier:

```text
slices/classify_product_image/
  augmentations.py
  dataset.py
  labels.py
  train.py
  evaluate.py
  predict.py
  api.py
  tests/
```

Keep augmentation policy local if it is specific to the visual task.

---

## 25. Pattern: NLP classification slices

```text
slices/classify_support_ticket/
  labels.py
  tokenizer.py
  dataset.py
  train.py
  evaluate.py
  predict.py
  api.py
  tests/
```

Label taxonomy belongs with the slice unless it is a company-wide stable taxonomy.

---

## 26. Pattern: anomaly detection slices

```text
slices/detect_payment_anomaly/
  dataset.py
  features.py
  baseline.py
  train.py
  score.py
  thresholds.py
  alert_policy.py
  tests/
```

Anomaly thresholds and alert policies should usually be slice-local because false-positive costs differ by domain.

---

## 27. Pattern: model monitoring inside slices

Monitoring has generic mechanisms and slice-specific meaning.

Generic platform:

```text
platform/observability/
  metrics_client.py
  tracing.py
  drift_detector.py
```

Slice-specific monitoring:

```text
slices/fraud_detection/monitoring.py
slices/churn_prediction/monitoring.py
```

Example:

```python
# src/slices/fraud_detection/monitoring.py
CRITICAL_FEATURES = [
    "transaction_amount",
    "merchant_risk_score",
    "user_velocity_1h",
]


def record_prediction(metrics_client, prediction, features, latency_ms: float) -> None:
    metrics_client.histogram("fraud.latency_ms", latency_ms)
    metrics_client.histogram("fraud.score", prediction.score)

    for name in CRITICAL_FEATURES:
        metrics_client.histogram(f"fraud.feature.{name}", float(features[name]))
```

The platform records metrics. The slice decides which metrics matter.

---

## 28. Recommended slice anatomy by maturity level

### Level 0: exploration

```text
slices/churn_prediction/
  notebooks/
  README.md
```

### Level 1: reproducible local training

```text
slices/churn_prediction/
  dataset.py
  features.py
  train.py
  evaluate.py
  tests/
```

### Level 2: model registry and batch inference

```text
slices/churn_prediction/
  dataset.py
  features.py
  train.py
  evaluate.py
  register.py
  batch_score.py
  tests/
```

### Level 3: online inference

```text
slices/churn_prediction/
  api.py
  schemas.py
  features.py
  predict.py
  model_loader.py
  monitoring.py
  tests/
```

### Level 4: continuous training and governance

```text
slices/churn_prediction/
  train_pipeline.py
  data_validation.py
  drift.py
  quality_gates.py
  promote.py
  rollback.py
  model_card.py
  monitoring.py
  tests/
```

---

## 29. Dependency rules

### Allowed dependencies

```text
slice -> platform
slice -> shared_kernel
slice -> shared_ml
app -> slice
pipeline runner -> slice
```

### Forbidden dependencies

```text
slice A -> slice B
platform -> slice
shared_ml -> slice
shared_kernel -> platform
```

### Example dependency direction

```text
app/main.py
  imports slices/churn_prediction/api.py

slices/churn_prediction/api.py
  imports slices/churn_prediction/predict.py
  imports platform/model_registry/client.py

platform/model_registry/client.py
  imports mlflow
  imports platform/config
```

Do not let `platform` import `churn_prediction`, because then platform becomes business-aware.

---

## 30. Testing strategy

ML vertical slices need more than unit tests.

### 30.1 Unit tests

Test pure local behavior:

```text
features.py
thresholds.py
data_validation.py
prompt_builder.py
decode.py
```

Example:

```python
# src/slices/churn_prediction/tests/test_thresholds.py
from slices.churn_prediction.thresholds import classify_risk


def test_classify_risk_high():
    assert classify_risk(0.82) == "high"
```

### 30.2 Contract tests

Test request/response schemas and model artifact contracts.

```python
# src/slices/fraud_detection/tests/test_model_contract.py
from slices.fraud_detection.model_loader import validate_loaded_model


def test_rejects_wrong_slice_model():
    metadata = {
        "contract": {
            "slice_name": "churn_prediction",
            "model_kind": "xgboost",
            "input_schema_version": "v3",
            "feature_spec_version": "v2",
            "output_schema_version": "v1",
            "threshold_policy_version": "v1",
        }
    }

    try:
        validate_loaded_model(metadata)
    except ValueError as error:
        assert "Wrong model" in str(error)
    else:
        raise AssertionError("Expected ValueError")
```

### 30.3 Data tests

Test schema, nulls, ranges, cardinality, distribution assumptions, and label quality.

```python
# src/slices/churn_prediction/data_validation.py
def validate_training_frame(frame):
    required = {"customer_id", "label", "days_since_last_login"}
    missing = required - set(frame.columns)
    if missing:
        raise ValueError(f"Missing columns: {sorted(missing)}")

    if frame["label"].isna().any():
        raise ValueError("label contains null values")
```

### 30.4 Training smoke tests

Run training on a tiny fixture dataset to validate the pipeline does not break.

```python
def test_training_pipeline_smoke(tmp_path):
    model_version = run_churn_training(
        dataset_uri="tests/fixtures/churn_small.parquet",
        run_id="test-run",
    )
    assert model_version
```

### 30.5 Evaluation gate tests

Test promotion thresholds.

```python
def test_quality_gate_rejects_bad_calibration():
    report = EvaluationReport(
        auc=0.90,
        brier_score=0.10,
        latency_p95_ms=20,
        calibration_error=0.20,
    )
    try:
        assert_candidate_is_promotable(report)
    except ValueError as error:
        assert "Calibration" in str(error)
    else:
        raise AssertionError("Expected rejection")
```

### 30.6 Inference golden tests

Use fixed inputs and expected response shapes or approximate scores.

```python
def test_predict_response_contract(predictor):
    response = predictor.predict(sample_request())
    assert 0.0 <= response.churn_probability <= 1.0
    assert response.risk in {"low", "medium", "high"}
```

### 30.7 Performance tests

For online inference:

```text
- p50 latency
- p95 latency
- p99 latency
- throughput
- memory usage
- cold start time
- model load time
```

Performance budgets are slice-specific.

---

## 31. CI/CD recommendations

A VSA ML repository should detect changed slices and run targeted validation.

Example GitHub Actions idea:

```yaml
name: ml-ci

on:
  pull_request:

jobs:
  detect-changed-slices:
    runs-on: ubuntu-latest
    outputs:
      slices: ${{ steps.detect.outputs.slices }}
    steps:
      - uses: actions/checkout@v4
      - id: detect
        run: python tools/detect_changed_slices.py

  test-slices:
    needs: detect-changed-slices
    runs-on: ubuntu-latest
    strategy:
      matrix:
        slice: ${{ fromJson(needs.detect-changed-slices.outputs.slices) }}
    steps:
      - uses: actions/checkout@v4
      - run: uv sync --all-extras
      - run: pytest src/slices/${{ matrix.slice }}/tests
```

For changed platform code, run all slice tests.

---

## 32. Pipeline orchestration

Vertical Slice Architecture does not replace Airflow, Kubeflow, Dagster, Prefect, TFX, SageMaker Pipelines, or Vertex AI Pipelines.

The orchestration layer should call slice entrypoints.

Good:

```python
# pipelines/churn_training_pipeline.py
from slices.churn_prediction.train_pipeline import run_churn_training


def pipeline():
    run_churn_training(dataset_uri="...", run_id="...")
```

Bad:

```python
# pipelines/churn_training_pipeline.py
# Contains all churn dataset, feature, training, evaluation, and promotion logic.
```

The pipeline file wires steps. The slice owns behavior.

---

## 33. Deployment recommendations

### 33.1 Online API deployment

```text
app/main.py imports API routers from slices.
Each slice exposes its router.
Deployment bundles only needed slices when possible.
```

Example:

```python
# src/app/main.py
from fastapi import FastAPI
from slices.churn_prediction.api import router as churn_router
from slices.fraud_detection.api import router as fraud_router

app = FastAPI()
app.include_router(churn_router)
app.include_router(fraud_router)
```

### 33.2 KServe or BentoML deployment

Keep KServe/BentoML service definitions close to slice deployment config when the serving behavior is slice-specific:

```text
slices/fraud_detection/deploy/
  kserve.yaml
  bentofile.yaml
  service.py
```

But keep shared Kubernetes templates under `infra/`.

### 33.3 Batch deployment

Batch jobs should call slice commands:

```bash
python -m slices.churn_batch_scoring.command --score-date 2026-05-28
```

---

## 34. Model serving boundaries

Separate these concerns:

```text
Serving infrastructure:
  - process management
  - autoscaling
  - GPU/CPU resources
  - networking
  - health checks
  - canary routing

Slice inference logic:
  - request schema
  - feature construction
  - model selection policy
  - thresholding
  - post-processing
  - explainability policy
  - prediction monitoring
```

KServe, BentoML, TorchServe, TensorFlow Serving, vLLM, and MLServer are serving infrastructure or serving frameworks. They should not own business-specific prediction behavior unless the model is a simple pure model artifact with no domain postprocessing.

---

## 35. Data ownership recommendations

Data is the hardest part of applying VSA to ML.

Use these rules:

1. Raw source connectors belong to platform or data platform modules.
2. Slice-specific dataset definitions belong inside the slice.
3. Stable shared entity definitions can belong to shared kernel.
4. Feature definitions belong inside slices unless they are intentionally reused and governed.
5. Do not create a global dumping ground called `data_utils.py`.

Example:

```text
platform/data_sources/postgres.py
platform/data_sources/bigquery.py
platform/data_sources/s3.py

slices/churn_prediction/dataset.py
slices/fraud_detection/dataset.py
```

---

## 36. Feature store recommendations

A feature store is platform infrastructure. Feature definitions may be shared or slice-owned.

Use slice-owned feature definitions when:

```text
- Feature is used by one model or capability
- Feature meaning depends on local label definition
- Feature freshness differs by capability
- Feature has domain-specific transformation rules
```

Use shared governed features when:

```text
- Feature is used by many models
- Feature definition is stable
- Feature has a clear owner
- Feature is useful as a cross-company semantic feature
```

Example:

```text
platform/feature_store/client.py
features/governed/customer_profile.py
slices/fraud_detection/local_features.py
slices/churn_prediction/local_features.py
```

Avoid premature centralization.

---

## 37. Versioning strategy

Every production ML slice should track versions of:

```text
- Code commit
- Dataset snapshot
- Feature spec
- Label policy
- Model artifact
- Evaluation report
- Input schema
- Output schema
- Threshold policy
- Runtime image
- Deployment config
```

Example metadata:

```json
{
  "slice": "fraud_detection",
  "code_commit": "abc123",
  "dataset_version": "transactions-2026-05-28",
  "feature_spec_version": "v4",
  "label_policy_version": "v2",
  "model_artifact_version": "fraud-xgb-42",
  "input_schema_version": "v3",
  "output_schema_version": "v1",
  "threshold_policy_version": "v5"
}
```

This is not optional for serious ML systems.

---

## 38. Slice-specific configuration

Use global config only for environment and infrastructure. Use slice config for business behavior.

```text
configs/production.yaml
slices/churn_prediction/config.yaml
slices/fraud_detection/config.yaml
```

Example:

```yaml
# slices/fraud_detection/config.yaml
model_name: fraud_detection
production_alias: production
thresholds:
  manual_review: 0.65
  block: 0.92
latency_budget_ms: 80
feature_freshness:
  user_velocity_1h: 10
  merchant_risk_score: 60
```

Do not hide domain rules in a giant root config file.

---

## 39. Anti-patterns

### 39.1 Fake slices

```text
slices/churn_prediction/api.py
slices/fraud_detection/api.py
shared/all_features.py
shared/all_training.py
shared/all_metrics.py
```

The folder names are vertical, but the logic is centralized.

### 39.2 Model-type slicing

```text
slices/xgboost/
slices/bert/
slices/resnet/
```

This is not VSA. It slices by algorithm, not capability.

Better:

```text
slices/fraud_detection/
slices/support_ticket_classification/
slices/product_image_quality/
```

### 39.3 Infrastructure knows business

```text
platform/model_registry/should_promote_fraud_model.py
```

Bad because platform imports business policy.

### 39.4 One global training pipeline for all models

```text
training/train_any_model.py
```

This often becomes full of conditionals:

```python
if model_name == "churn":
    ...
elif model_name == "fraud":
    ...
elif model_name == "ranking":
    ...
```

Replace with slice-owned commands and shared technical helpers.

### 39.5 Over-slicing too early

Do not split every tiny function into its own slice. A slice should represent a real capability or workflow.

### 39.6 Ignoring cross-cutting ML concerns

VSA does not remove the need for:

```text
- lineage
- reproducibility
- monitoring
- privacy
- governance
- cost control
- model security
- data validation
```

These concerns need platform mechanisms plus slice-specific policies.

---

## 40. Tradeoffs

### Benefits

```text
- Locality of change
- Better ownership
- Easier testing by capability
- Clearer model lifecycle per capability
- Less accidental coupling across models
- Better support for different metrics and deployment policies
- Easier onboarding to one ML product area
- Natural integration with product teams
```

### Costs

```text
- Some duplication across slices
- Harder to enforce global consistency
- Requires discipline around shared code extraction
- Can create many small folders
- May conflict with data science teams used to notebooks/layers
- Shared features require governance decisions
```

### Recommendation

Prefer local duplication until a concept is stable and reused by at least two or three slices. Extract only after repeated use proves the abstraction.

---

## 41. Decision matrix

| Question | Prefer slice-local | Prefer shared/platform |
|---|---|---|
| Does it encode business behavior? | Yes | No |
| Does it decide model promotion? | Yes | No |
| Is it a storage/client wrapper? | No | Yes |
| Is it a reusable math primitive? | Maybe | Yes |
| Is it a feature definition for one model? | Yes | No |
| Is it a governed feature used by many models? | No | Yes |
| Is it endpoint schema for one capability? | Yes | No |
| Is it a generic tracing client? | No | Yes |
| Is it a prompt template for one task? | Yes | No |
| Is it a shared tokenizer wrapper? | Maybe | Maybe |

---

## 42. Architecture review checklist

Use this checklist when reviewing an ML project for VSA quality.

### Slice boundaries

- Does each slice represent a business capability or ML workflow?
- Can a feature change be completed mostly inside one slice?
- Are unrelated slices independent?
- Are slices named by use case, not technology?

### Training

- Does the slice own its dataset definition?
- Does the slice own its feature logic?
- Does the slice own its training entrypoint?
- Does the slice own evaluation and quality gates?
- Is model registration metadata complete?

### Inference

- Does the slice own request and response schemas?
- Does the slice validate input contracts?
- Does the slice own post-processing and thresholds?
- Does the slice define latency and quality expectations?
- Are loaded model artifacts validated against the slice contract?

### Shared code

- Is platform code free of business-specific rules?
- Are shared ML utilities stable and generic?
- Are shared features governed and documented?
- Is there any `utils.py` that should be split?

### Testing

- Are there unit tests for slice-local logic?
- Are there data validation tests?
- Are there contract tests for schemas and model artifacts?
- Are there training smoke tests?
- Are there inference golden tests?
- Are promotion quality gates tested?

### Operations

- Are metrics and monitoring slice-specific where needed?
- Is drift detection configured per capability?
- Is retraining policy explicit?
- Is rollback possible?
- Are model cards or equivalent documentation generated?

---

## 43. Example: full tabular ML slice

```text
slices/credit_risk_scoring/
  README.md
  config.yaml
  schemas.py
  dataset.py
  feature_spec.py
  features.py
  labels.py
  train.py
  evaluate.py
  quality_gates.py
  register.py
  predict.py
  api.py
  monitoring.py
  model_card.py
  tests/
    test_features.py
    test_labels.py
    test_quality_gates.py
    test_predict_contract.py
```

### schemas.py

```python
from pydantic import BaseModel, Field


class CreditRiskRequest(BaseModel):
    customer_id: str
    requested_amount: float = Field(gt=0)
    term_months: int = Field(gt=0, le=120)


class CreditRiskResponse(BaseModel):
    customer_id: str
    default_probability: float
    decision: str
    model_version: str
```

### thresholds.py

```python
def decide(default_probability: float) -> str:
    if default_probability >= 0.35:
        return "reject"
    if default_probability >= 0.18:
        return "manual_review"
    return "approve"
```

### predict.py

```python
from .features import build_online_features
from .thresholds import decide
from .schemas import CreditRiskRequest, CreditRiskResponse


class CreditRiskPredictor:
    def __init__(self, model, model_version: str, feature_store):
        self.model = model
        self.model_version = model_version
        self.feature_store = feature_store

    def predict(self, request: CreditRiskRequest) -> CreditRiskResponse:
        x = build_online_features(request, self.feature_store)
        probability = float(self.model.predict_proba(x)[0, 1])
        return CreditRiskResponse(
            customer_id=request.customer_id,
            default_probability=probability,
            decision=decide(probability),
            model_version=self.model_version,
        )
```

---

## 44. Example: LLM training and inference slice

```text
slices/support_chatbot/
  README.md
  dataset.py
  conversation_schema.py
  chat_template.py
  loss_masking.py
  train_lora.py
  evaluate_generation.py
  decode.py
  serve.py
  safety_policy.py
  model_card.py
  tests/
```

### conversation_schema.py

```python
from typing import Literal
from pydantic import BaseModel


class Message(BaseModel):
    role: Literal["system", "user", "assistant", "tool"]
    content: str


class ConversationSample(BaseModel):
    sample_id: str
    messages: list[Message]
    source: str
```

### loss_masking.py

```python
def train_on_message(role: str, include_tool_results: bool = False) -> bool:
    if role == "assistant":
        return True
    if role == "tool":
        return include_tool_results
    return False
```

### serve.py

```python
from .chat_template import render_messages
from .decode import clean_assistant_output
from .safety_policy import validate_response


class SupportChatbot:
    def __init__(self, llm_client):
        self.llm_client = llm_client

    def answer(self, messages: list[dict]) -> str:
        prompt = render_messages(messages)
        raw = self.llm_client.generate(prompt)
        answer = clean_assistant_output(raw)
        validate_response(answer)
        return answer
```

The slice owns the chat template, loss masking, decoding, and safety policy because these are task-specific.

---

## 45. Example: image classification slice

```text
slices/product_image_classification/
  labels.py
  augmentations.py
  dataset.py
  model.py
  train.py
  evaluate.py
  predict.py
  api.py
  tests/
```

### labels.py

```python
LABELS = [
    "shoe",
    "shirt",
    "bag",
    "watch",
]

LABEL_TO_ID = {label: index for index, label in enumerate(LABELS)}
ID_TO_LABEL = {index: label for label, index in LABEL_TO_ID.items()}
```

### predict.py

```python
from .labels import ID_TO_LABEL


class ProductImageClassifier:
    def __init__(self, model, transform):
        self.model = model
        self.transform = transform

    def predict(self, image):
        tensor = self.transform(image)
        logits = self.model(tensor)
        predicted_id = int(logits.argmax(dim=-1).item())
        return ID_TO_LABEL[predicted_id]
```

Labels and transforms are local because they are part of this capability.

---

## 46. Example: forecasting slice

```text
slices/forecast_support_tickets/
  dataset.py
  calendar.py
  features.py
  train.py
  evaluate.py
  forecast.py
  staffing_policy.py
  tests/
```

### staffing_policy.py

```python
def required_agents(predicted_tickets: int) -> int:
    tickets_per_agent = 35
    minimum_agents = 2
    return max(minimum_agents, (predicted_tickets + tickets_per_agent - 1) // tickets_per_agent)
```

The model forecast is not the only business output. The slice owns the decision policy derived from the forecast.

---

## 47. Example: recommendation ranking slice

```text
slices/homepage_ranking/
  schemas.py
  candidate_source.py
  features.py
  rank.py
  diversify.py
  evaluate.py
  api.py
  tests/
```

### rank.py

```python
class HomepageRanker:
    def __init__(self, model):
        self.model = model

    def rank(self, user_context, candidates):
        rows = [candidate.to_feature_row(user_context) for candidate in candidates]
        scores = self.model.predict(rows)
        ranked = sorted(zip(candidates, scores), key=lambda pair: pair[1], reverse=True)
        return [candidate for candidate, _score in ranked]
```

### diversify.py

```python
def diversify_products(products, max_per_category: int = 3):
    counts = {}
    result = []
    for product in products:
        count = counts.get(product.category, 0)
        if count >= max_per_category:
            continue
        counts[product.category] = count + 1
        result.append(product)
    return result
```

The ranking model and diversity policy belong together in the slice because the output is a product experience, not just a score.

---

## 48. How to migrate from layered ML architecture to VSA

### Step 1: Identify capabilities

Look for API endpoints, batch jobs, model names, product features, or training workflows.

Example:

```text
churn prediction
fraud detection
invoice extraction
product recommendation
support chatbot
```

### Step 2: Pick one slice

Do not refactor everything at once. Choose one high-change capability.

### Step 3: Move local logic into the slice

Move:

```text
- schemas
- dataset query
- feature logic
- training entrypoint
- evaluation
- inference
- thresholds
- tests
```

### Step 4: Leave infrastructure shared

Keep:

```text
- S3 clients
- database clients
- model registry client
- metrics client
- config loader
```

### Step 5: Add contract tests

Before and after refactoring, verify the external contract remains the same.

### Step 6: Extract only proven duplication

After multiple slices stabilize, extract shared utilities carefully.

---

## 49. Naming conventions

Prefer capability names:

```text
fraud_detection
churn_prediction
invoice_ocr
support_chatbot
homepage_ranking
product_similarity
```

Avoid technology names:

```text
xgboost_model
bert_classifier
resnet_service
sklearn_training
```

File names should expose intent:

```text
dataset.py
features.py
train.py
evaluate.py
predict.py
quality_gates.py
model_contract.py
monitoring.py
```

For larger slices, use subfolders:

```text
commands/
queries/
training/
inference/
contracts/
tests/
```

---

## 50. Recommended implementation templates

### 50.1 Small slice template

```text
slices/<capability>/
  README.md
  schemas.py
  dataset.py
  features.py
  train.py
  evaluate.py
  predict.py
  tests/
```

### 50.2 Production slice template

```text
slices/<capability>/
  README.md
  config.yaml
  contracts/
    schemas.py
    model_contract.py
    feature_spec.py
  training/
    dataset.py
    features.py
    train.py
    evaluate.py
    quality_gates.py
    register.py
  inference/
    api.py
    features.py
    predict.py
    postprocess.py
    explain.py
  operations/
    monitoring.py
    drift.py
    rollback.py
    model_card.py
  tests/
```

### 50.3 LLM/VLM slice template

```text
slices/<llm_capability>/
  README.md
  dataset.py
  prompt_template.py
  chat_template.py
  tokenization.py
  loss_masking.py
  train.py
  evaluate.py
  decode.py
  serve.py
  safety.py
  tests/
```

---

## 51. Agent instructions for generating ML VSA code

When asked to generate code for an ML project using VSA:

1. Identify the capability first.
2. Name the slice by business behavior.
3. Put training and inference code near each other if they share contracts.
4. Keep infrastructure clients in `platform/`.
5. Keep business thresholds, feature sets, labels, prompt templates, and quality gates inside the slice.
6. Add tests next to the slice.
7. Define artifact metadata and schema versions.
8. Do not create global `utils.py` files.
9. Do not create model-type slices.
10. Do not centralize all features unless there is a governed feature store or shared feature domain.

---

## 52. Recommended answer pattern for architecture questions

When answering ML VSA architecture questions, structure the answer like this:

```text
1. Identify the slice boundary.
2. Explain what belongs inside the slice.
3. Explain what belongs in platform/shared modules.
4. Show a project tree.
5. Show one or two code examples.
6. Explain training/inference consistency.
7. List tests.
8. List tradeoffs and anti-patterns.
```

---

## 53. Edge cases

### 53.1 Shared model, multiple slices

Sometimes one foundation model powers many slices.

Example:

```text
platform/llm_gateway/
  client.py

slices/support_chatbot/
  prompt_template.py
  safety.py
  evaluate.py

slices/document_extraction/
  prompt_template.py
  schema.py
  evaluate.py
```

The foundation model client is shared. Prompts, schemas, decoding, and evaluation are slice-local.

### 53.2 One slice, multiple models

A slice can own multiple models if they jointly deliver one capability.

Example:

```text
slices/product_search/
  lexical_retriever.py
  embedding_retriever.py
  ranker.py
  reranker.py
  evaluate.py
```

The slice is `product_search`, not `embedding_model`.

### 53.3 Feature reused by many slices

Create a governed feature module:

```text
features/customer_lifetime_value.py
features/account_age.py
```

But document owner, definition, freshness, and compatibility.

### 53.4 Shared dataset, multiple slices

Raw dataset access can be shared. Dataset selection, labels, and filtering should usually be slice-local.

### 53.5 Multi-tenant model serving

If one service hosts many models, keep serving infrastructure shared and route to slice-specific handlers:

```text
platform/serving/router.py
slices/fraud_detection/predict.py
slices/churn_prediction/predict.py
```

### 53.6 Online learning

For online learning, keep update policy and safety gates slice-local:

```text
slices/ad_click_prediction/
  online_update.py
  drift.py
  rollback.py
  monitoring.py
```

### 53.7 Regulated ML

For finance, healthcare, hiring, insurance, or similar domains, each slice should include:

```text
model_card.py
explainability.py
fairness.py
audit_log.py
approval_workflow.py
```

Do not hide these in generic compliance modules only.

---

## 54. Practical recommendation for Rust + Python ML projects

For systems where Rust handles API/performance-critical serving and Python handles training/data science:

```text
repo/
  python/
    src/
      platform/
      shared_ml/
      slices/
        train_churn_model/
        evaluate_vlm_model/

  rust/
    crates/
      api/
      platform/
      slices/
        churn_prediction/
        fraud_detection/
```

Keep shared contracts explicit:

```text
contracts/
  churn_prediction.request.schema.json
  churn_prediction.response.schema.json
  fraud_detection.model_contract.json
```

Python training writes model metadata. Rust inference validates it before loading.

---

## 55. Practical recommendation for Kubernetes ML systems

```text
infra/k8s/base/
infra/k8s/overlays/production/

src/slices/fraud_detection/deploy/
  kserve.yaml
  batch-score-cronjob.yaml

src/slices/churn_prediction/deploy/
  kserve.yaml
  retrain-cronjob.yaml
```

Shared Kubernetes components such as namespaces, service accounts, secrets, ingress, and observability belong in `infra/`. Slice-specific deployments can live near the slice or be generated from slice metadata.

---

## 56. Practical recommendation for ML platform teams

If you are building an internal ML platform, your platform capabilities may themselves be vertical slices:

```text
slices/
  register_model/
  promote_model/
  run_evaluation_suite/
  create_training_job/
  deploy_inference_service/
```

These are not business ML models. They are platform product capabilities.

This is valid if your users are ML engineers or data scientists.

---

## 57. Summary rules

Use these as the shortest version of the skill:

```text
1. Slice by ML capability, not by model type.
2. Keep training, evaluation, inference, schemas, feature logic, and tests close to the capability.
3. Share infrastructure, not business behavior.
4. Treat model artifacts, datasets, feature specs, thresholds, and schemas as versioned contracts.
5. Use command/query separation when training and inference workflows become complex.
6. Keep promotion quality gates local to the slice.
7. Keep monitoring mechanisms shared but monitoring meaning slice-specific.
8. Avoid fake slices that delegate all real logic to shared modules.
9. Extract common code only after the duplication is proven stable.
10. Design every production slice for reproducibility, observability, rollback, and testing.
```

---

## 58. Bibliography and reference links

### Vertical Slice Architecture

- Jimmy Bogard, "Vertical Slice Architecture": https://www.jimmybogard.com/vertical-slice-architecture/

### MLOps and ML lifecycle

- Google Cloud, "MLOps: Continuous delivery and automation pipelines in machine learning": https://docs.cloud.google.com/architecture/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning
- Google Cloud, "Practitioners Guide to Machine Learning Operations": https://cloud.google.com/resources/mlops-whitepaper
- AWS, "What is MLOps?": https://aws.amazon.com/what-is/mlops/
- AWS SageMaker, "Implement MLOps": https://docs.aws.amazon.com/sagemaker/latest/dg/mlops.html
- Microsoft Team Data Science Process lifecycle: https://github.com/Azure/Microsoft-TDSP/blob/master/Docs/lifecycle-detail.md
- Kreuzberger, Kuehl, Hirschl, "Machine Learning Operations (MLOps): Overview, Definition, and Architecture": https://arxiv.org/abs/2205.02302
- MLOps Principles: https://ml-ops.org/content/mlops-principles

### Model lifecycle, registry, serving, and feature stores

- MLflow Model Registry: https://mlflow.org/docs/latest/ml/model-registry/
- MLflow Tracking: https://mlflow.org/docs/latest/ml/tracking/
- Feast documentation: https://docs.feast.dev/
- Feast use cases: https://docs.feast.dev/getting-started/use-cases
- KServe documentation: https://kserve.github.io/website/
- Kubeflow KServe introduction: https://www.kubeflow.org/docs/components/kserve/introduction/
- BentoML documentation: https://docs.bentoml.com/
- TensorFlow Extended ExampleGen: https://www.tensorflow.org/tfx/guide/examplegen
- Uber Michelangelo ML platform: https://www.uber.com/us/en/blog/michelangelo-machine-learning-platform/

---

## 59. Final agent rule

When uncertain where something belongs, ask:

```text
Will this change because this ML capability changes, or because the shared platform changes?
```

If it changes because the capability changes, put it in the slice.

If it changes because all ML capabilities need the same mechanism, put it in platform or shared infrastructure.
