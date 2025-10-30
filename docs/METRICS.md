# Metrics & KPIs Dashboard
## AI-Powered Credit Risk Assessment System

**Owner**: AI Product Manager + Data Analytics Team
**Update Frequency**: Daily (automated), Weekly (reviewed), Monthly (business review)
**Dashboard Link**: [Datadog Dashboard] | [Looker Reports]

---

## North Star Metric

### Credit Decision Quality Score
**Formula**: `(Accuracy × 0.4) + (Speed × 0.3) + (NPS × 0.3) × 100`

| Period | Accuracy | Speed (normalized) | NPS (normalized) | Quality Score | Target | Status |
|--------|----------|-------------------|------------------|---------------|--------|--------|
| Oct 2025 | 93.2% | 95% (<30s avg) | 72% (NPS 44) | 87.4 | >80 | ✅ PASSING |
| Sept 2025 | 91.8% | 88% | 68% | 83.2 | >80 | ✅ PASSING |
| Aug 2025 | 89.5% | 82% | 65% | 79.1 | >80 | ⚠️ AT RISK |

**Trend**: ↗️ Improving (+4.2 points month-over-month)

---

## 1. Model Performance Metrics

### 1.1 Accuracy Metrics (Daily)

| Metric | Current | Target | Trend | Alert Threshold |
|--------|---------|--------|-------|-----------------|
| **Balanced Accuracy** | 93.2% | >90% | ↗️ +0.8% | <88% |
| **Precision (High-Risk)** | 9.1% | >8% | → Stable | <5% |
| **Recall (High-Risk)** | 92.1% | >85% | ↗️ +1.2% | <80% |
| **F1-Score (High-Risk)** | 0.16 | >0.15 | → Stable | <0.12 |
| **AUC-ROC** | 0.947 | >0.95 | ↘️ -0.003 | <0.90 |

**Interpretation**:
- ✅ Balanced accuracy exceeds target (93.2% vs. 90%)
- ✅ Recall is excellent - catching 92% of high-risk applicants
- ⚠️ Precision is low (9%) but expected for imbalanced data (intentional tradeoff)
- 📊 AUC-ROC slightly below target - monitor for model drift

**Actions**:
- Continue monthly model retraining
- Monitor precision-recall curve for degradation
- A/B test new features to improve precision without hurting recall

---

### 1.2 Confusion Matrix (Last 30 Days)

|  | Predicted Low Risk | Predicted High Risk | Total |
|--|-------------------|---------------------|-------|
| **Actual Low Risk** | 16,130 (94.3%) | 974 (5.7%) | 17,104 |
| **Actual High Risk** | 8 (7.9%) | 93 (92.1%) | 101 |
| **Total** | 16,138 | 1,067 | 17,205 |

**Business Impact**:
- **True Negatives (16,130)**: Correctly approved low-risk loans → $15.2M revenue
- **False Positives (974)**: Low-risk flagged for review → $48.7K manual review cost
- **False Negatives (8)**: High-risk approved → $120K potential losses
- **True Positives (93)**: High-risk correctly flagged → $1.4M losses prevented

**Net Impact**: +$16.5M revenue, -$169K costs = **$16.3M net benefit**

---

### 1.3 Model Drift Detection

**Feature Drift (Population Stability Index - PSI)**

| Feature | PSI | Status | Interpretation |
|---------|-----|--------|----------------|
| dti | 0.04 | ✅ Stable | No significant shift |
| int_rate | 0.12 | ⚠️ Monitor | Slight drift (interest rate changes) |
| delinq_2yrs | 0.02 | ✅ Stable | No drift |
| total_rec_prncp | 0.18 | 🔴 Alert | Significant drift - investigate |

**PSI Interpretation**:
- PSI < 0.10: No significant change
- PSI 0.10-0.25: Moderate drift - monitor closely
- PSI > 0.25: Severe drift - retrain model immediately

**Action Items**:
- 🔍 Investigate `total_rec_prncp` drift (economic conditions?)
- 📊 Schedule model retrain for next week (PSI >0.10 on key feature)
- 📧 Alert engineering team about drift

---

### 1.4 Prediction Confidence Distribution

| Confidence Range | Count | % of Total | Avg Accuracy | Recommendation |
|------------------|-------|------------|--------------|----------------|
| 0.00 - 0.45 | 342 | 2.0% | 78.2% | 🔴 Manual review |
| 0.45 - 0.55 | 1,721 | 10.0% | 82.5% | ⚠️ Manual review |
| 0.55 - 0.70 | 4,302 | 25.0% | 89.1% | ✅ Auto-process |
| 0.70 - 0.85 | 6,881 | 40.0% | 93.7% | ✅ Auto-process |
| 0.85 - 1.00 | 3,959 | 23.0% | 96.4% | ✅ Auto-process |

**Insights**:
- 88% of predictions have >0.55 confidence (safe for automation)
- 12% flagged for manual review (borderline cases)
- Calibration is good: 85%+ confidence → 96% actual accuracy

---

## 2. Business Metrics

### 2.1 Operational Efficiency

| Metric | Baseline (Pre-AI) | Current (With AI) | Improvement | Target |
|--------|------------------|-------------------|-------------|---------|
| **Avg Time to Decision** | 4.2 days | 28 seconds | **99.9%** faster ✅ | <30s |
| **Cost per Decision** | $187 | $8.50 | **95.5%** reduction ✅ | <$10 |
| **Applications Processed/Day** | 120 | 1,850 | **1,442%** increase ✅ | >1,500 |
| **Manual Review Rate** | 100% | 5.7% | **94.3%** reduction ✅ | <10% |

**ROI Calculation**:
```
Cost Savings = (Applications × Cost Reduction) = (17,205 × $178.50) = $3.07M annually
Revenue Increase = (Faster approvals × Conversion lift) = $2.1M annually
Total ROI = $5.17M / $800K investment = 647% ROI
```

---

### 2.2 Credit Portfolio Health

| Metric | Baseline | Target | Current | Status |
|--------|----------|--------|---------|--------|
| **Default Rate** | 7.2% | <5% | 4.8% | ✅ BEAT TARGET |
| **Approval Rate** | 62.3% | >70% | 71.2% | ✅ BEAT TARGET |
| **Avg Loan Amount (Approved)** | $14,250 | Maintain | $15,100 | ✅ +$850 |
| **Revenue from Approved Loans** | $12.1M | $15M | $16.3M | ✅ +35% |

**Portfolio Quality Trend**:
- 📈 Default rate DOWN 33% (7.2% → 4.8%)
- 📈 Approval rate UP 14% (62.3% → 71.2%)
- 📈 Revenue UP 35% ($12.1M → $16.3M)

**Interpretation**: Model correctly identifies low-risk applicants being rejected by baseline, increasing approvals WITHOUT increasing defaults.

---

### 2.3 User Adoption & Satisfaction

| Metric | Month 3 | Month 6 | Current | Target | Status |
|--------|---------|---------|---------|--------|--------|
| **Active Users (Loan Officers)** | 42% | 78% | 86% | >80% | ✅ |
| **Daily Active Usage** | 31% | 62% | 74% | >60% | ✅ |
| **Net Promoter Score (NPS)** | +18 | +38 | +44 | >40 | ✅ |
| **Override Rate** | 18.2% | 9.7% | 6.4% | <10% | ✅ |

**User Feedback (Qualitative)**:
- 😊 92% say system saves them time
- 😊 88% trust the model's decisions
- 😊 95% like the GenAI explanations
- 😕 12% want mobile app (roadmap item)

---

## 3. GenAI Explanation Metrics

### 3.1 Explanation Quality

| Metric | Target | Current | Trend |
|--------|--------|---------|-------|
| **Clarity Score (User Survey 1-5)** | >4.0 | 4.3 | ↗️ +0.2 |
| **Explanation Generation Rate** | 100% | 99.7% | ↘️ -0.1% |
| **Avg Explanation Length (words)** | 100-150 | 137 | → Stable |
| **Reading Level (Flesch-Kincaid)** | 8th grade | 7.8 | ✅ |
| **Latency p95 (seconds)** | <3s | 1.8s | ✅ |

**Failure Analysis (0.3% explanation failures)**:
- 45% = LLM API timeout (>10s)
- 30% = Rate limit exceeded (too many concurrent requests)
- 25% = Hallucination detected by fact-checker (rejected)

**Actions**:
- Increase LLM API timeout from 10s → 15s
- Add caching for similar feature combinations
- Improve hallucination detection prompts

---

### 3.2 Conversational Q&A Performance

| Metric | Target | Current | Status |
|--------|--------|---------|--------|
| **Query Response Time p95** | <3s | 2.4s | ✅ |
| **Answer Accuracy (Thumbs Up)** | >85% | 91.2% | ✅ |
| **Daily Query Volume** | N/A | 487 | 📊 Tracking |
| **Top Query Types** | N/A | See below | 📊 |

**Top 5 Query Categories** (by volume):
1. "Why was [applicant] flagged?" (32%)
2. "Can I override this decision?" (18%)
3. "What's the biggest risk factor?" (15%)
4. "How accurate is the model?" (12%)
5. "Explain [feature name]" (9%)

**Insights**: Users primarily want explanations for specific decisions (50% of queries). Consider proactive explanations in UI.

---

## 4. Compliance & Ethics Metrics

### 4.1 Fairness Metrics (Monthly Audit)

**Disparate Impact Ratio (80% Rule)**

| Demographic | Approval Rate | Ratio vs. Baseline | Status | Threshold |
|-------------|--------------|-------------------|--------|-----------|
| Group A (Baseline) | 71.2% | 1.00 | ✅ Baseline | - |
| Group B | 68.9% | 0.97 | ✅ Pass | >0.80 |
| Group C | 73.1% | 1.03 | ✅ Pass | >0.80 |
| Group D | 65.2% | 0.92 | ✅ Pass | >0.80 |

**Demographic Parity Difference**

| Metric | Value | Target | Status |
|--------|-------|--------|--------|
| Max Difference | 7.9% | <10% | ✅ Pass |
| Std Deviation | 3.2% | <5% | ✅ Pass |

**Interpretation**: No evidence of discriminatory bias. All groups within acceptable variance.

---

### 4.2 Regulatory Compliance

| Requirement | Status | Evidence | Last Audit |
|-------------|--------|----------|------------|
| **FCRA Adverse Action Notices** | ✅ 100% | Auto-generated for all denials | Oct 2025 |
| **ECOA Non-Discrimination** | ✅ Pass | Fairness audit (disparate impact <0.10) | Oct 2025 |
| **GDPR Right to Explanation** | ✅ 99.7% | GenAI explanations provided | Oct 2025 |
| **Model Explainability** | ✅ 100% | Feature importance + natural language | Oct 2025 |
| **Human Oversight** | ✅ 5.7% | Borderline cases reviewed | Oct 2025 |

**Regulatory Violations**: 0 (Year to Date)

---

### 4.3 AI Safety Metrics

| Metric | Target | Current | Status |
|--------|--------|---------|--------|
| **Hallucination Rate (GenAI)** | <1% | 0.8% | ✅ |
| **Factually Incorrect Explanations** | <0.5% | 0.3% | ✅ |
| **Offensive Language Detected** | 0% | 0% | ✅ |
| **Human Review of Explanations** | 10% sample | 11.2% | ✅ |

---

## 5. Infrastructure & Ops Metrics

### 5.1 System Performance

| Metric | Target | Current | Status |
|--------|--------|---------|--------|
| **API Uptime** | 99.9% | 99.94% | ✅ |
| **p50 Latency (Prediction)** | <5s | 2.1s | ✅ |
| **p95 Latency (Prediction)** | <30s | 18.4s | ✅ |
| **p99 Latency (Prediction)** | <60s | 42.7s | ✅ |
| **Throughput (req/sec)** | >10 | 14.2 | ✅ |

**Outages (Last 30 Days)**: 1 incident (43 min, planned maintenance)

---

### 5.2 Cost Metrics

| Component | Monthly Cost | Per-Application | Budget | Status |
|-----------|-------------|----------------|--------|--------|
| **AWS SageMaker (ML)** | $2,850 | $0.17 | $3,500 | ✅ Under budget |
| **OpenAI API (GenAI)** | $4,120 | $0.24 | $5,000 | ✅ Under budget |
| **Infrastructure (RDS, S3)** | $1,230 | $0.07 | $1,500 | ✅ Under budget |
| **Monitoring (Datadog)** | $450 | $0.03 | $500 | ✅ Under budget |
| **Total** | **$8,650** | **$0.51** | $10,500 | ✅ **18% under budget** |

**Cost per Decision**: $0.51 (ML) + $8.00 (labor) = **$8.51 total** vs. $187 baseline

---

## 6. Alerts & Action Items

### Active Alerts (as of October 30, 2025)

| Alert | Severity | Metric | Threshold | Current | Action |
|-------|----------|--------|-----------|---------|--------|
| Feature Drift | ⚠️ Warning | total_rec_prncp PSI | <0.25 | 0.18 | Schedule retrain |
| AUC-ROC Decline | ⚠️ Warning | AUC-ROC | >0.95 | 0.947 | Monitor for 7 days |

### Completed This Week

| Item | Status | Owner | Completion Date |
|------|--------|-------|-----------------|
| Monthly fairness audit | ✅ Pass | Compliance | Oct 28, 2025 |
| Model retrain (Sept data) | ✅ Deployed | ML Eng | Oct 25, 2025 |
| Q4 NPS survey | ✅ NPS +44 | PM | Oct 22, 2025 |

---

## 7. Dashboard Links

- **Real-Time Performance**: [Datadog Dashboard](https://app.datadoghq.com/dashboard/credit-risk)
- **Business Metrics**: [Looker Dashboard](https://looker.com/dashboards/credit-portfolio)
- **Fairness Audit**: [Internal Compliance Portal](https://compliance.internal.com/fairness)
- **User Feedback**: [Qualtrics Survey Results](https://qualtrics.com/results/nps-oct-2025)

---

**Next Metrics Review**: November 7, 2025 (Weekly Standup)
**Next Business Review**: November 15, 2025 (Monthly with Exec Team)
**Next External Audit**: January 2026 (Third-Party AI Fairness Firm)
