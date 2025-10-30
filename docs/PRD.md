# Product Requirements Document (PRD)
## AI-Powered Credit Risk Assessment System

**Document Owner**: AI Product Manager
**Last Updated**: October 2025
**Status**: Phase 2 - Development
**Stakeholders**: Engineering, Credit Risk, Compliance, Business Development

---

## 1. Executive Summary

### Problem
Manual credit risk assessment is slow (3-7 days), expensive ($200/application), and inconsistent (35% false positive rate). Traditional credit scoring models (FICO) lack explainability, creating regulatory risk and user frustration.

### Solution
AI-powered credit risk system combining ensemble ML models (93% accuracy) with GenAI explanations, delivering instant decisions with human-readable rationale for regulatory compliance and user trust.

### Success Criteria
- **Speed**: <30 seconds per decision (vs. 3-7 days)
- **Accuracy**: >90% balanced accuracy
- **Cost**: <$10 per decision (vs. $200)
- **Adoption**: >80% of loan officers actively using within 6 months

---

## 2. Goals & Objectives (OKRs)

### Objective 1: Deliver Production-Ready Credit Risk ML Model
**Key Results:**
- KR1: Achieve >90% balanced accuracy on test set
- KR2: Recall for high-risk detection >85% (minimize false negatives)
- KR3: Process 1,000 applications/hour with <100ms latency
- KR4: Model drift monitoring alerting with <1% monthly degradation

### Objective 2: Build Explainable AI Layer (GenAI)
**Key Results:**
- KR1: 100% of decisions have auto-generated explanations
- KR2: Explanation clarity score >4.0/5 (user surveys)
- KR3: FCRA-compliant adverse action notices for all denials
- KR4: Conversational Q&A interface with <2s response time

### Objective 3: Drive User Adoption
**Key Results:**
- KR1: 50% of loan officers using system by Month 3
- KR2: 80% adoption by Month 6
- KR3: Net Promoter Score (NPS) >40
- KR4: Override rate <10% (indicates trust in model)

### Objective 4: Ensure Compliance & Ethics
**Key Results:**
- KR1: Pass external fairness audit with zero critical findings
- KR2: Disparate impact ratio >0.80 for all protected groups
- KR3: 100% of high-risk flags manually reviewed within 24 hours
- KR4: Zero regulatory violations (FCRA, ECOA, GDPR)

---

## 3. User Personas

### Primary: Sarah - Senior Loan Officer
**Demographics**: 42 years old, 12 years experience in lending
**Goals**: Approve good loans quickly, avoid bad debt, meet quarterly targets
**Pain Points**:
- Spends 4 hours/day manually reviewing credit reports
- Pressured to approve loans faster but scared of defaults
- Struggles to explain rejections to angry applicants
- Inconsistent decisions lead to compliance audits

**Success Metrics**: Decisions per day, default rate, customer satisfaction

**User Story**:
> "As a loan officer, I want instant risk scores with clear explanations so I can make confident decisions and explain them to applicants without fear of regulatory violations."

### Secondary: Marcus - Compliance Officer
**Demographics**: 51 years old, former attorney, risk-averse
**Goals**: Prevent regulatory fines, audit ML decisions, protect company reputation
**Pain Points**:
- Black-box ML models terrify him (can't audit them)
- Worried about AI discrimination lawsuits
- Board asks tough questions he can't answer
- Manual fairness testing takes weeks

**Success Metrics**: Zero regulatory violations, audit pass rate, documented explanations

**User Story**:
> "As a compliance officer, I want every ML decision to be explainable and auditable so I can prove to regulators that we're not discriminating and meet FCRA requirements."

### Tertiary: Jenny - Loan Applicant
**Demographics**: 29 years old, first-time borrower, anxious about credit
**Goals**: Get approved for $15K personal loan, understand credit score
**Pain Points**:
- Rejected by 2 lenders with vague "insufficient credit history"
- Doesn't know what to improve
- Feels discriminated against
- Comparing offers across lenders

**Success Metrics**: Approval rate, time to decision, understanding of rejection

**User Story**:
> "As an applicant, I want to know exactly why I was rejected and what I can do to improve so I can get approved next time and build my credit."

---

## 4. Functional Requirements

### 4.1 ML Prediction Engine (Must Have - P0)

#### REQ-ML-001: Real-Time Risk Scoring
**Description**: System must predict credit risk for new applications in <30 seconds
**Acceptance Criteria**:
- Input: 93 applicant features (financial, credit history, demographics)
- Output: Binary classification (high_risk / low_risk)
- Latency: p95 <30 seconds, p99 <60 seconds
- Availability: 99.9% uptime SLA
- Throughput: 100 concurrent requests

**Technical Details**:
- Model: Easy Ensemble AdaBoost (93.2% balanced accuracy)
- Fallback: Balanced Random Forest if primary fails
- Load balancing: Auto-scale on CPU >70%

---

#### REQ-ML-002: Confidence Scoring
**Description**: Every prediction must include confidence/probability score
**Acceptance Criteria**:
- Range: 0.0 - 1.0 (percentage)
- Calibration: Confidence within ±5% of actual accuracy
- Borderline flagging: Confidence 0.45-0.55 triggers manual review

---

#### REQ-ML-003: Feature Importance
**Description**: Identify top 10 risk factors for each application
**Acceptance Criteria**:
- Ranked list of features with importance scores
- Applicant-specific values shown alongside dataset averages
- Human-readable feature names (no "X_train_col_42")

---

### 4.2 GenAI Explanation Layer (Must Have - P0)

#### REQ-AI-001: Decision Summaries
**Description**: Auto-generate plain language explanation for every decision
**Acceptance Criteria**:
- Length: 3-5 sentences (100-150 words)
- Reading level: 8th grade (Flesch-Kincaid)
- Format: Markdown with bullet points
- Latency: <2 seconds (99th percentile)
- Languages: English (Phase 1), Spanish (Phase 2)

**Example Output**:
```
This application was flagged as HIGH RISK with 87% confidence.
Key risk factors include:
• High debt-to-income ratio (31.4% vs. 19.8% average)
• 2 delinquencies in the past 2 years
• 5 credit inquiries in the last 6 months

Recommendation: Request additional income verification or cosigner.
```

---

#### REQ-AI-002: Adverse Action Notices
**Description**: Generate FCRA-compliant rejection letters automatically
**Acceptance Criteria**:
- Cite specific reasons for denial (per FCRA requirements)
- Include applicant rights (free credit report, dispute process)
- Provide actionable improvement steps
- Legal review: All templates approved by counsel

**Regulations**:
- Fair Credit Reporting Act (FCRA) - 15 U.S.C. § 1681m
- Equal Credit Opportunity Act (ECOA) - 15 U.S.C. § 1691

---

#### REQ-AI-003: Conversational Q&A Interface
**Description**: Loan officers can ask questions about decisions in natural language
**Acceptance Criteria**:
- Input: Natural language question (text)
- Output: Contextual answer with data citations
- Latency: <3 seconds
- Accuracy: >90% (user thumbs up/down feedback)
- Safety: Content filtering for hallucinations, offensive language

**Example Interactions**:
- "Why was applicant #12345 flagged?"
- "What's the biggest predictor of default?"
- "Can I override this decision?"

**Technical Stack**:
- LLM: GPT-4 Turbo or Claude 3 Opus
- RAG: Company policies, regulatory docs, model training data
- Vector DB: Pinecone or Weaviate for similarity search

---

### 4.3 User Interface (Must Have - P0)

#### REQ-UI-001: Loan Officer Dashboard
**Description**: Web dashboard for reviewing applications and model decisions
**Acceptance Criteria**:
- Display: Risk score, confidence, explanation, top risk factors
- Actions: Approve, Deny, Request More Info, Override
- Search: By applicant name, application ID, date range
- Export: PDF reports for compliance

**Mockup**: [Link to Figma designs]

---

#### REQ-UI-002: Override Workflow
**Description**: Loan officers can override model decisions with justification
**Acceptance Criteria**:
- Required fields: Override reason (dropdown + text), manager approval
- Notifications: Alert manager for approval within 4 hours
- Audit trail: Log all overrides for compliance review
- Limits: Officers with >15% override rate triggering defaults require training

---

### 4.4 Monitoring & Ops (Must Have - P0)

#### REQ-OPS-001: Model Performance Monitoring
**Description**: Real-time dashboards tracking model accuracy, drift, fairness
**Acceptance Criteria**:
- Metrics: Accuracy, precision, recall, F1 (daily)
- Alerts: Email + Slack if accuracy drops >5%
- Feature drift: Statistical tests on input distributions
- Retrain trigger: Accuracy <88% for 3 consecutive days

---

#### REQ-OPS-002: Fairness Audits
**Description**: Monthly demographic parity analysis for bias detection
**Acceptance Criteria**:
- Disparate impact ratio: >0.80 (per 80% rule)
- Protected attributes: Race, gender, age (if available)
- Reporting: Board-level quarterly fairness report
- Third-party: Annual external audit by AI ethics firm

---

## 5. Non-Functional Requirements (NFRs)

### Performance
- **Latency**: p95 <30s (prediction), p99 <3s (explanation)
- **Throughput**: 1,000 applications/hour (peak load)
- **Availability**: 99.9% uptime (43 minutes downtime/month)

### Security
- **Authentication**: OAuth 2.0 + MFA for all users
- **Authorization**: Role-based access control (RBAC)
- **Encryption**: TLS 1.3 in transit, AES-256 at rest
- **PII Handling**: GDPR-compliant data retention (7 years)

### Scalability
- **Horizontal Scaling**: Auto-scale on CPU >70% or latency >50s
- **Database**: PostgreSQL with read replicas
- **Caching**: Redis for frequently accessed predictions

### Compliance
- **FCRA**: Adverse action notices within 30 days
- **ECOA**: No discrimination on protected characteristics
- **GDPR**: Right to explanation, right to be forgotten
- **SOC 2 Type II**: Security audit by end of Year 1

---

## 6. Out of Scope (v1.0)

The following features are explicitly **NOT** included in the initial release:

❌ Mobile app (loan officer or customer-facing)
❌ Integration with external credit bureaus (Experian, Equifax, TransUnion)
❌ Automated decisioning (all approvals require human review)
❌ Multi-language support (Spanish, Mandarin - deferred to v2.0)
❌ Deep learning models (LSTM, transformers)
❌ Real-time model retraining pipeline
❌ White-label SaaS for other lenders

---

## 7. Success Metrics (KPIs)

### North Star Metric: **Credit Decision Quality Score**
Composite metric combining accuracy, speed, and user satisfaction:
```
Quality Score = (Accuracy × 0.4) + (Speed × 0.3) + (NPS × 0.3)
Target: >80/100
```

### Product Metrics
| Metric | Baseline | Target (6mo) | Actual |
|--------|----------|--------------|--------|
| Decision accuracy | 72% | >90% | TBD |
| Time to decision | 3-7 days | <30 sec | TBD |
| Cost per decision | $200 | <$10 | TBD |
| User adoption | 0% | 80% | TBD |
| NPS | N/A | >40 | TBD |

### Business Metrics
| Metric | Baseline | Target (Year 1) | Actual |
|--------|----------|-----------------|--------|
| Default rate | 7.2% | <5% | TBD |
| Approval rate | 62% | >70% | TBD |
| Revenue from approved loans | $12M | $15M (+25%) | TBD |
| Regulatory fines | $0 | $0 | TBD |

---

## 8. Risks & Mitigation

### Risk 1: Model Bias / Discrimination Lawsuit
**Probability**: Medium
**Impact**: High (multi-million dollar fine)
**Mitigation**:
- Monthly fairness audits with third-party review
- Exclude protected attributes from training data
- Human review for all high-risk flags
- Explainable AI for all decisions
- $5M liability insurance

### Risk 2: Low User Adoption
**Probability**: High
**Impact**: Medium (project failure)
**Mitigation**:
- Co-design with 10 pilot loan officers
- Gradual rollout (10% → 50% → 100% over 6 months)
- Weekly user feedback sessions
- Gamification (leaderboards for fastest accurate decisions)
- Executive sponsorship and mandate

### Risk 3: Model Accuracy Degrades Over Time
**Probability**: Medium
**Impact**: High (business impact)
**Mitigation**:
- Automated drift detection and alerting
- Monthly model retraining with new data
- A/B testing for model updates
- Fallback to conservative baseline if accuracy drops

### Risk 4: GenAI Hallucinations
**Probability**: High
**Impact**: Medium (user confusion, compliance risk)
**Mitigation**:
- Structured prompts with strict output formats
- Fact-checking layer (verify explanations match model features)
- Human review of 10% of explanations (random sample)
- Confidence thresholds for auto-generation (>70%)

---

## 9. Dependencies

### Internal
- **Data Team**: Provide cleaned LendingClub dataset by Week 2
- **Legal**: Review adverse action letter templates by Week 4
- **Infrastructure**: Provision AWS SageMaker by Week 1
- **Design**: Deliver UI mockups by Week 3

### External
- **OpenAI / Anthropic**: API access for GPT-4 / Claude ($5K/month budget)
- **Cloud Provider**: AWS account with SageMaker, S3, RDS
- **Third-Party Audit**: Contract with AI ethics firm for fairness audits

---

## 10. Timeline & Milestones

### Phase 1: MVP (Months 1-3)
- **Month 1**: Data prep, model training, baseline established
- **Month 2**: GenAI integration, basic UI
- **Month 3**: Pilot with 10 loan officers, feedback iteration

**Success Criteria**: 90% accuracy, positive pilot feedback

### Phase 2: Scaled Rollout (Months 4-6)
- **Month 4**: 50% of applications routed through system
- **Month 5**: Full monitoring, alerting, compliance review
- **Month 6**: 100% rollout, NPS >40

**Success Criteria**: 80% user adoption, <5% default rate

### Phase 3: Optimization (Months 7-12)
- **Month 7-9**: A/B test model improvements, feature engineering
- **Month 10-12**: Expand to other loan products (auto, mortgage)

**Success Criteria**: 93%+ accuracy sustained, business metrics hit

---

## 11. Open Questions

1. **Should we integrate with credit bureaus in v1.0 or v2.0?**
   - Decision: v2.0 (scope reduction)
   - Rationale: Too complex for MVP, can use LendingClub data initially

2. **Which LLM for GenAI explanations - GPT-4 or Claude?**
   - Decision: A/B test both, then choose based on cost/quality
   - Criteria: Explanation clarity score, latency, cost per request

3. **What's the threshold for manual review (borderline cases)?**
   - Decision: Confidence 0.45-0.55 (10% of applications)
   - Data: Pilot will validate this threshold

4. **Can loan officers fully automate approvals (no human review)?**
   - Decision: NO for v1.0 (compliance requirement)
   - Future: Possible for low-risk + high-confidence cases only

---

## 12. Appendix

### A. Regulatory References
- **FCRA** (Fair Credit Reporting Act): 15 U.S.C. § 1681 et seq.
- **ECOA** (Equal Credit Opportunity Act): 15 U.S.C. § 1691 et seq.
- **GDPR** (General Data Protection Regulation): EU Regulation 2016/679

### B. Technical Stack
- **ML**: Python, scikit-learn, imbalanced-learn, XGBoost
- **GenAI**: OpenAI GPT-4 API, Anthropic Claude API
- **Backend**: FastAPI, PostgreSQL, Redis
- **Frontend**: React, TypeScript, Tailwind CSS
- **Infrastructure**: AWS (SageMaker, S3, RDS, CloudWatch)
- **MLOps**: MLflow, DVC, GitHub Actions

### C. Reference Documents
- [Technical Design Doc](./DEPLOYMENT.md)
- [Metrics Dashboard](./METRICS.md)
- [AI Ethics Framework](./ETHICS.md)
- [User Research Summary](./research/user_interviews.pdf)

---

**Document Sign-Off:**

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Product Manager | [Your Name] | _________ | 2025-10-30 |
| Engineering Lead | [Name] | _________ | _______ |
| Compliance Officer | [Name] | _________ | _______ |
| VP of Product | [Name] | _________ | _______ |

**Next Review Date**: December 1, 2025
