# Deployment & Production Guide
## AI-Powered Credit Risk Assessment System

**Owner**: ML Engineering + DevOps Team
**Last Updated**: October 2025
**Production Status**: Phase 3 - Scaled Rollout (50% traffic)

---

## 1. Deployment Phases

### Phase 1: Shadow Mode ✅ COMPLETED
**Duration**: Month 1-2 (Aug-Sept 2025)
**Status**: ✅ Success Criteria Met

**Approach**:
- Run AI model in parallel with existing manual process
- Do NOT use AI predictions for actual decisions
- Compare AI predictions vs. human decisions
- Collect user feedback on explanations

**Results**:
- Agreement rate: 96.2% (target: >95%) ✅
- User satisfaction: 4.1/5 (positive feedback)
- Zero production impact (shadow mode)

---

### Phase 2: Pilot ✅ COMPLETED
**Duration**: Month 3-4 (Oct-Nov 2025)
**Status**: ✅ Success Criteria Met

**Approach**:
- Route 10% of applications through AI system
- Loan officers can override any decision
- A/B test: AI group vs. control group (manual)

**Configuration**:
```yaml
traffic_split:
  ai_model: 10%
  manual_baseline: 90%

rollout_criteria:
  target_users: [senior_loan_officers]
  loan_amount_cap: $25,000
  confidence_threshold: 0.70
```

**Results**:
- Override rate: 8.2% (target: <10%) ✅
- NPS: +38 (target: positive) ✅
- Default rate: 4.2% vs. 5.1% control (A/B test winner) ✅
- Decision time: 42 seconds vs. 4.1 days ✅

---

### Phase 3: Scaled Rollout 🚧 IN PROGRESS
**Duration**: Month 5-6 (Dec 2025-Jan 2026)
**Status**: 🚧 50% traffic (on track)

**Approach**:
- Increase traffic to 50% of applications
- Enable GenAI explanations for all decisions
- Full monitoring and alerting infrastructure

**Current Configuration**:
```yaml
traffic_split:
  ai_model: 50%
  manual_baseline: 50%

features_enabled:
  - real_time_predictions
  - genai_explanations
  - adverse_action_letters
  - conversational_qa

monitoring:
  - model_performance_dashboard
  - fairness_metrics
  - latency_alerts
  - cost_tracking
```

**Metrics to Date** (Week 2 of Phase 3):
- Accuracy: 93.2% (stable)
- Latency p95: 18.4s (target: <30s) ✅
- User adoption: 78% (target: >60%) ✅

---

### Phase 4: Full Production 📅 PLANNED
**Start Date**: February 2026
**Duration**: Ongoing

**Approach**:
- 100% of applications routed through AI (with manual review option)
- Continuous model retraining (monthly)
- Expansion to other loan products

**Success Criteria**:
- Sustained accuracy >90% for 3 months
- User adoption >80%
- Default rate <5%
- Zero critical production incidents

---

## 2. Infrastructure Architecture

### 2.1 Cloud Platform: AWS

```
┌─────────────────────────────────────────────────────────────────┐
│                         Load Balancer (ALB)                      │
│                    api.creditrisk.company.com                    │
└────────────────────────┬────────────────────────────────────────┘
                         │
         ┌───────────────┴───────────────┐
         │                               │
         ▼                               ▼
┌─────────────────┐             ┌─────────────────┐
│  API Gateway    │             │  API Gateway    │
│  (us-east-1a)   │             │  (us-east-1b)   │
└────────┬────────┘             └────────┬────────┘
         │                               │
         ▼                               ▼
┌─────────────────────────────────────────────────┐
│         FastAPI Application (Auto-Scaling)      │
│  - Request validation                           │
│  - Authentication (OAuth 2.0)                   │
│  - Rate limiting                                │
│  - Logging                                      │
└────────┬────────────────────────────────────────┘
         │
         ├──────────────┬──────────────┬───────────────┐
         ▼              ▼              ▼               ▼
┌────────────────┐ ┌────────────┐ ┌──────────┐ ┌──────────────┐
│  SageMaker     │ │  Lambda    │ │  Redis   │ │  PostgreSQL  │
│  ML Inference  │ │  (GenAI)   │ │  Cache   │ │  RDS         │
│                │ │            │ │          │ │              │
│  - EasyEnsemble│ │  - GPT-4   │ │  - Pred  │ │  - App data  │
│  - BalancedRF  │ │  - Claude  │ │  - Feat  │ │  - User data │
└────────────────┘ └────────────┘ └──────────┘ └──────────────┘
         │              │              │               │
         └──────────────┴──────────────┴───────────────┘
                         │
                         ▼
              ┌──────────────────────┐
              │  CloudWatch Logs     │
              │  & Datadog Monitoring│
              └──────────────────────┘
```

---

### 2.2 Component Details

#### A. ML Model Serving (AWS SageMaker)

**Endpoint Configuration**:
```python
# sagemaker_config.yaml
endpoint_name: credit-risk-production-v2
instance_type: ml.m5.xlarge
initial_instance_count: 2
auto_scaling:
  min_instances: 2
  max_instances: 10
  target_metric: CPUUtilization
  target_value: 70

models:
  primary: EasyEnsembleClassifier-v2.3.1
  fallback: BalancedRandomForest-v1.8.0
```

**Deployment Process**:
1. Train model locally or in SageMaker Training Job
2. Save model to S3 bucket (`s3://ml-models/credit-risk/`)
3. Register in MLflow Model Registry
4. Create SageMaker endpoint with canary deployment
5. Gradually shift traffic (5% → 25% → 50% → 100%)

---

#### B. GenAI Explanation Engine (AWS Lambda)

**Function Configuration**:
```python
# lambda_config.yaml
function_name: genai-credit-explainer
runtime: python3.11
memory: 2048 MB
timeout: 15 seconds
environment:
  OPENAI_API_KEY: ${OPENAI_KEY}
  ANTHROPIC_API_KEY: ${ANTHROPIC_KEY}
  MODEL_PRIMARY: gpt-4-turbo
  MODEL_FALLBACK: claude-3-opus

concurrency:
  reserved: 50
  provisioned: 10
```

**API Call Flow**:
```python
def generate_explanation(prediction_data):
    # 1. Get prediction from SageMaker
    risk_score, confidence, features = prediction_data

    # 2. Build prompt
    prompt = build_structured_prompt(risk_score, features)

    # 3. Call LLM API
    try:
        explanation = openai.ChatCompletion.create(
            model="gpt-4-turbo",
            messages=[{"role": "user", "content": prompt}],
            temperature=0.3,  # Lower = more consistent
            max_tokens=200,
            timeout=10
        )
    except TimeoutError:
        # Fallback to template
        explanation = generate_template_explanation(features)

    # 4. Fact-check (verify explanation matches model features)
    if not fact_check_explanation(explanation, features):
        raise HallucinationError()

    return explanation
```

---

#### C. Caching Layer (Redis)

**Purpose**: Reduce latency and cost for similar predictions

**Cache Strategy**:
```python
# Pseudocode
def predict_with_cache(applicant_features):
    # Hash features to create cache key
    cache_key = hash_features(applicant_features)

    # Check cache
    if redis.exists(cache_key):
        return redis.get(cache_key)  # Cache hit

    # Cache miss - call model
    prediction = sagemaker.predict(applicant_features)

    # Store in cache (TTL: 24 hours)
    redis.setex(cache_key, 86400, prediction)

    return prediction
```

**Cache Performance**:
- Hit rate: 23% (similar applicants within 24 hours)
- Latency reduction: 18s → 0.2s (90x faster)
- Cost savings: $0.24 → $0.01 per cached prediction

---

#### D. Database (PostgreSQL RDS)

**Tables**:
```sql
-- Applications table
CREATE TABLE applications (
    id UUID PRIMARY KEY,
    applicant_id UUID NOT NULL,
    loan_amount DECIMAL(10,2),
    features JSONB,  -- All 93 features
    created_at TIMESTAMP DEFAULT NOW()
);

-- Predictions table
CREATE TABLE predictions (
    id UUID PRIMARY KEY,
    application_id UUID REFERENCES applications(id),
    model_version VARCHAR(50),
    risk_score VARCHAR(20),  -- 'high_risk' or 'low_risk'
    confidence DECIMAL(5,4),
    feature_importance JSONB,
    explanation TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Decisions table (human decisions + overrides)
CREATE TABLE decisions (
    id UUID PRIMARY KEY,
    prediction_id UUID REFERENCES predictions(id),
    decision VARCHAR(20),  -- 'approved', 'denied', 'manual_review'
    decided_by UUID,  -- user_id
    override BOOLEAN DEFAULT FALSE,
    override_reason TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Audit log (compliance)
CREATE TABLE audit_log (
    id UUID PRIMARY KEY,
    entity_type VARCHAR(50),
    entity_id UUID,
    action VARCHAR(50),
    user_id UUID,
    metadata JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);
```

---

## 3. CI/CD Pipeline

### 3.1 Pipeline Stages

```yaml
# .github/workflows/ml-pipeline.yml

name: ML Model CI/CD

on:
  push:
    branches: [main, staging]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run unit tests
        run: pytest tests/ --cov=src --cov-report=xml

      - name: Run fairness tests
        run: python tests/test_fairness.py

      - name: Lint code
        run: flake8 src/

  train:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Train model
        run: python src/model_training.py --config configs/production.yaml

      - name: Evaluate model
        run: python src/model_evaluation.py

      - name: Check accuracy threshold
        run: |
          accuracy=$(cat metrics/accuracy.txt)
          if [ $(echo "$accuracy < 0.90" | bc) -eq 1 ]; then
            echo "Model accuracy below threshold: $accuracy"
            exit 1
          fi

      - name: Upload model to S3
        run: aws s3 cp models/model.pkl s3://ml-models/credit-risk/$(git rev-parse HEAD)/

  deploy:
    needs: train
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to SageMaker (Canary)
        run: |
          python scripts/deploy_sagemaker.py \
            --model-uri s3://ml-models/credit-risk/$(git rev-parse HEAD)/ \
            --endpoint credit-risk-production-v2 \
            --traffic 5  # Start with 5% traffic

      - name: Monitor canary (30 minutes)
        run: python scripts/monitor_canary.py --duration 1800

      - name: Shift traffic (5% → 100%)
        run: python scripts/shift_traffic.py --endpoint credit-risk-production-v2
```

---

### 3.2 Model Retraining Pipeline

**Trigger**: Monthly (1st of each month) OR accuracy drops <88%

```python
# airflow_dag.py (simplified)
from airflow import DAG
from airflow.operators.python import PythonOperator

dag = DAG('credit_risk_retrain', schedule_interval='@monthly')

def fetch_new_data():
    # Get last month's applications + outcomes
    df = query_database(
        "SELECT * FROM applications WHERE created_at > NOW() - INTERVAL '30 days'"
    )
    df.to_parquet('s3://training-data/2025-10/')

def train_model():
    # Load data + retrain
    df = pd.read_parquet('s3://training-data/2025-10/')
    model = EasyEnsembleClassifier(n_estimators=100, random_state=42)
    model.fit(X_train, y_train)

    # Save model
    joblib.dump(model, 's3://ml-models/credit-risk/2025-10-01/')

def evaluate_model():
    # Run fairness tests
    audit_result = run_fairness_audit(model, test_data)
    assert audit_result['disparate_impact'] >= 0.80

    # Check accuracy
    accuracy = balanced_accuracy_score(y_test, model.predict(X_test))
    assert accuracy >= 0.90

def deploy_model():
    # Deploy via SageMaker
    deploy_to_sagemaker(model_uri='s3://ml-models/credit-risk/2025-10-01/')

# DAG structure
fetch_task = PythonOperator(task_id='fetch_data', python_callable=fetch_new_data)
train_task = PythonOperator(task_id='train_model', python_callable=train_model)
eval_task = PythonOperator(task_id='evaluate_model', python_callable=evaluate_model)
deploy_task = PythonOperator(task_id='deploy_model', python_callable=deploy_model)

fetch_task >> train_task >> eval_task >> deploy_task
```

---

## 4. Monitoring & Alerting

### 4.1 Metrics to Monitor

| Metric | Tool | Alert Threshold | Action |
|--------|------|-----------------|--------|
| **Model Accuracy** | CloudWatch | <88% | Retrain model |
| **API Latency p95** | Datadog | >30s | Scale up instances |
| **Error Rate** | CloudWatch | >1% | Page on-call engineer |
| **Feature Drift (PSI)** | Custom script | >0.25 | Investigate data quality |
| **Cost per Request** | AWS Cost Explorer | >$0.60 | Optimize resources |
| **Fairness (Disparate Impact)** | Monthly script | <0.80 | Halt deployments |

---

### 4.2 Alert Configuration

```yaml
# datadog_monitors.yaml
monitors:
  - name: "High ML Latency"
    type: metric_alert
    query: "avg(last_5m):avg:sagemaker.invocation.duration{env:production} > 30"
    message: "ML inference latency >30s. Check SageMaker instances."
    tags: ["team:ml", "severity:warning"]

  - name: "Model Accuracy Drop"
    type: metric_alert
    query: "avg(last_1h):avg:ml.accuracy{model:credit-risk} < 0.88"
    message: "@pagerduty Model accuracy <88%. Initiate model retrain."
    tags: ["team:ml", "severity:critical"]

  - name: "GenAI High Error Rate"
    type: metric_alert
    query: "avg(last_10m):sum:lambda.errors{function:genai-explainer} > 10"
    message: "GenAI explanation failures spiking. Check OpenAI API status."
    tags: ["team:ml", "severity:warning"]
```

---

## 5. Rollback Procedure

**Scenario**: New model deployed but causing issues (low accuracy, high latency, bias detected)

**Steps**:
1. **Immediate** (<5 minutes):
   ```bash
   # Rollback to previous model version
   python scripts/rollback_model.py \
     --endpoint credit-risk-production-v2 \
     --target-version v2.2.4
   ```

2. **Verify** (<10 minutes):
   - Check accuracy returns to baseline
   - Confirm latency normalizes
   - Review last 100 predictions manually

3. **Post-Mortem** (<24 hours):
   - Document what went wrong
   - Update tests to catch issue
   - Re-train/fix model offline

**Recent Rollbacks**: 1 (September 2025 - model drift due to data quality issue)

---

## 6. Security & Compliance

### 6.1 Authentication

**OAuth 2.0 + MFA**:
```python
# FastAPI authentication
from fastapi.security import OAuth2PasswordBearer

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

@app.post("/predict")
async def predict(token: str = Depends(oauth2_scheme)):
    # Verify JWT token
    user = verify_token(token)
    if not user.has_permission("predict"):
        raise HTTPException(403, "Forbidden")

    # Proceed with prediction
    ...
```

---

### 6.2 Data Encryption

- **At Rest**: AES-256 encryption for S3, RDS
- **In Transit**: TLS 1.3 for all API calls
- **Secrets**: AWS Secrets Manager (API keys, DB passwords)

---

### 6.3 Compliance Logs

**Audit Trail** (every action logged):
```json
{
  "event_id": "evt_abc123",
  "timestamp": "2025-10-30T14:32:18Z",
  "user_id": "usr_jane_doe",
  "action": "override_decision",
  "entity_type": "prediction",
  "entity_id": "pred_xyz789",
  "metadata": {
    "original_decision": "high_risk",
    "override_decision": "approved",
    "reason": "Compensating factors (large down payment)",
    "manager_approval": "mgr_john_smith"
  }
}
```

**Retention**: 7 years per FCRA

---

## 7. Disaster Recovery

**RTO** (Recovery Time Objective): 1 hour
**RPO** (Recovery Point Objective): 15 minutes

**Backup Strategy**:
- Database: Automated snapshots every 15 minutes
- Models: All versions stored in S3 with versioning enabled
- Code: Git repository (GitHub)

**DR Procedure**:
1. Restore database from latest snapshot
2. Deploy last known-good model version
3. Verify system functionality
4. Resume traffic

---

## 8. Cost Optimization

**Current Monthly Costs**:
- SageMaker: $2,850
- Lambda (GenAI): $4,120
- RDS: $780
- S3: $210
- CloudWatch: $240
- **Total**: $8,200/month

**Optimization Opportunities**:
- Use SageMaker Serverless Inference (save 30% on idle time)
- Cache frequent predictions in Redis (save 20% on LLM API calls)
- Reserved instances for RDS (save 40%)

**Projected Savings**: $2,100/month (26% reduction)

---

## 9. Production Checklist

Before deploying to production, ensure:

- [ ] Model accuracy >90% on holdout test set
- [ ] Fairness audit passed (disparate impact >0.80)
- [ ] Load testing completed (1000 req/sec for 1 hour)
- [ ] Security scan (no vulnerabilities)
- [ ] Backup/restore tested
- [ ] Rollback procedure documented
- [ ] On-call rotation scheduled
- [ ] Stakeholder sign-off (PM, Eng, Compliance, Legal)

---

**Next Deployment**: December 15, 2025 (Phase 3 → 100% traffic)
**On-Call Engineer**: See PagerDuty schedule
