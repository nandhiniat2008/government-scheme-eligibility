# government-scheme-eligibility
## Project Requirements Document (PRD): Intelligent Government Scheme Eligibility Assessment Platform

### 1) Overview
**Goal**: Build a privacy-first platform that helps citizens **instantly determine eligibility** for government schemes/subsidies/welfare programs by evaluating complex criteria (income, residency, age, family size, occupation, category, disability, assets, etc.), while providing **real-time questioning**, **confidence/probability scoring**, and **enterprise-grade fraud/attack resistance**.

**Primary users**
- **Citizen**: checks eligibility, understands required documents, optionally proceeds to application handoff.
- **Case worker / Program officer**: reviews flagged cases, analytics, policy tuning (optional).
- **Policy admin**: manages schemes and rule updates safely (versioned governance).
- **Security/Compliance**: audit, monitoring, privacy controls.

---

## 2) Scope

### In Scope
- **50+ schemes** with **dynamic rules** (thresholds, residency, demographic/category constraints, benefit tiers).
- **Smart Questioning Engine** that asks minimal, contextual follow-ups and eliminates ineligible schemes early.
- **Real-time eligibility results** with **confidence scores** and **probability matching** per scheme.
- **Privacy-first architecture** using one or more: **ZK proofs**, **MPC/secure computation**, **federated learning** (as appropriate).
- **Fraud prevention**: synthetic identity detection, duplicates, manipulation/attack detection, tamper-evident logs.

### Out of Scope (initial MVP suggestions)
- Full end-to-end submission to every government system (instead: generate checklist + handoff package/API).
- Biometric verification (can integrate later).
- Final entitlement approval (platform provides eligibility assessment, not final adjudication).

---

## 3) Success Metrics
- **Coverage**: ≥ 50 schemes modeled; rule updates deployed without downtime.
- **User effort**: median questions asked ≤ target (e.g., 8–12) while preserving accuracy.
- **Latency**: initial shortlist < 1s; each follow-up answer updates results < 300ms (typical).
- **Accuracy**: high agreement with ground-truth determinations (measured with pilot datasets).
- **Privacy**: no raw sensitive attributes exposed outside trusted boundary; auditable encryption + access.
- **Fraud**: reduce duplicate/synthetic attempts; measurable precision/recall on fraud flags.

---

## 4) Functional Requirements

### 4.1 Multi-Scheme Rules & Program Modeling
- **FR-1**: System stores each scheme’s eligibility logic as **versioned rules** (effective dates, jurisdictions).
- **FR-2**: Support rule types:
  - **Hard constraints**: must satisfy (age ≥ X, resident of state Y).
  - **Thresholds**: income ≤ X; assets ≤ Y; family-size adjusted thresholds.
  - **Category-based**: occupation, caste/category, disability, veteran status, rural/urban, BPL card, etc.
  - **Tiered benefits**: benefit amount changes by bracket.
  - **Document requirements**: list & conditional documents.
- **FR-3**: Rule authoring via **admin console** with guardrails (linting, simulation, approval workflow).
- **FR-4**: **Explainability**: show user-friendly reasons: “Eligible because…”, “Not eligible because…”.

### 4.2 Smart Questioning Engine (Real-Time)
- **FR-5**: Ask the **minimum** questions needed to narrow to likely eligible schemes.
- **FR-6**: Support **structured Q&A** and optional **NLP** input (“I’m a 32-year-old farmer in…”).
- **FR-7**: Question selection optimized by:
  - **Information gain** across remaining schemes
  - User burden (sensitive/effortful questions later)
  - Evidence confidence (prefer verifiable attributes earlier)
- **FR-8**: Must eliminate ineligible schemes early and update results after every answer.

### 4.3 Eligibility Scoring & Probability Matching
- **FR-9**: Output per scheme:
  - **Status**: Eligible / Ineligible / Possibly eligible (needs verification) / Unknown
  - **Confidence score** (0–1) and **probability match** (calibrated)
  - **Missing fields** and **next-best questions**
- **FR-10**: Scores derived from:
  - Deterministic rule evaluation + uncertainty model (missing/uncertain inputs)
  - Optional ML model for noisy inputs (free text, OCR, partial records)

### 4.4 Privacy-First Architecture
- **FR-11**: Data minimization: collect only what’s needed; default to local processing where possible.
- **FR-12**: Support privacy-preserving verification options (configurable by jurisdiction):
  - **ZK proofs** for “income ≤ threshold”, “age ≥ 18”, “resident of state” without revealing raw value.
  - **MPC** for cross-agency eligibility checks without data pooling.
  - **Federated learning** for improving models without centralizing raw user data.
- **FR-13**: Strong encryption:
  - In transit (TLS 1.2+ / 1.3)
  - At rest (AES-256)
  - Field-level encryption for highly sensitive attributes
- **FR-14**: User consent + purpose limitation + retention controls.

### 4.5 Fraud Prevention & Attack Resistance
- **FR-15**: Detect and prevent:
  - **Synthetic identities** (behavioral + document signals)
  - **Duplicate applications** (graph linking, fuzzy matching, device fingerprints)
  - **Manipulation attempts** (income tampering, repeated probing, inconsistent answers)
  - **Automated attacks** (bots, enumeration)
- **FR-16**: Risk scoring pipeline with:
  - Real-time checks during Q&A
  - Post-assessment review queue for high-risk sessions
- **FR-17**: Tamper-evident audit logs (append-only, signed).

---

## 5) Non-Functional Requirements (NFRs)
- **Security**: OWASP ASVS-aligned, rate limiting, WAF, secrets management, secure SDLC.
- **Availability**: 99.9% target; multi-AZ deployment; graceful degradation.
- **Performance**: sub-second shortlist; low-latency incremental updates.
- **Scalability**: handle spikes during enrollment windows.
- **Compliance**: align with applicable privacy laws (e.g., DPDP/ GDPR-like), local data residency.
- **Accessibility**: WCAG 2.1 AA; multi-language; low-bandwidth mode.
- **Observability**: audit logs, metrics, tracing, anomaly alerts.
- **Maintainability**: rule tests, simulation, version rollbacks, feature flags.

---

## 6) Workflow (End-to-End)

### 6.1 Citizen Eligibility Check (Real-Time)
1. **Entry**: citizen selects language + region; consents to data use.
2. **Initial intake** (minimal): age band, district/state, household size, occupation category, rough income band.
3. **Scheme pre-filtering**: deterministic rules remove impossible schemes (jurisdiction, age, base categories).
4. **Smart questioning loop**:
   - engine picks next question (highest info gain / lowest burden)
   - user answers (structured or NLP)
   - system updates eligibility statuses + confidence in real time
5. **Results page**:
   - ranked list of schemes with probability/confidence
   - eligibility reasons + missing requirements
   - document checklist + next steps (apply link/handoff)
6. **Optional verification**:
   - user can provide proofs (ZK/MPC/attested claims) to increase confidence
7. **Fraud/risk handling**:
   - normal users proceed
   - high-risk sessions get limited actions + manual review path

### 6.2 Scheme & Rules Management
1. Admin creates/updates scheme rules in **Rules Studio**.
2. Automated **lint + simulation** against test personas and historical anonymized distributions.
3. Approval workflow (4-eyes) + versioned publish (effective date).
4. Monitoring detects abnormal eligibility shifts after rule changes.

### 6.3 Fraud Operations
1. Real-time signals captured (device, velocity, inconsistency, doc validation outcomes).
2. Risk score produced; thresholds route to:
   - allow
   - step-up verification (additional proof requests)
   - block or manual review
3. Investigator console: graph view of linked identities/devices; decision logging.

---

## 7) Data & Rule Model (Conceptual)

### Core Entities
- **CitizenSession**: session id, consent, region, timestamps, risk signals.
- **ProfileAttributes**: age, residency, family size, occupation, income band/value, category tags.
- **Scheme**: id, name, jurisdiction, benefit type, documents, owner dept.
- **RuleSet**: version, effective dates, rule expressions, required fields.
- **EligibilityResult**: scheme id, status, confidence, reasons, missing fields.
- **FraudSignals**: device fingerprint, IP reputation, behavioral metrics, duplication links.

### Rules Engine Requirements
- Expression language (e.g., JSONLogic / CEL / DSL) with:
  - numeric comparisons, ranges, set membership
  - derived values (per-capita income, household thresholds)
  - jurisdiction scoping
  - explainability hooks (“reason codes”)

---

## 8) Recommended Tech Stack (Reference Architecture)

### Frontend
- **Web**: React + Next.js (or Angular)  
- **Mobile** (optional): React Native / Flutter  
- **UI**: component library + i18n (multi-language), offline/low-bandwidth support

### Backend (Core Services)
- **API gateway**: Kong / AWS API Gateway / NGINX
- **App services**: Node.js (NestJS) or Python (FastAPI) or Java (Spring Boot)
- **Rules service**: dedicated microservice for rule evaluation + explanation
- **Questioning engine**: service using decision trees + bandit/info-gain optimization
- **Fraud service**: streaming + graph-based identity linking + risk scoring
- **AuthN/AuthZ**: OAuth2.1/OIDC (Keycloak/Cognito), RBAC/ABAC

### Data Stores
- **Primary DB**: PostgreSQL (schemes, rules, audit metadata)
- **Cache**: Redis (session state, rule compilation cache)
- **Search**: OpenSearch/Elasticsearch (scheme discovery, admin queries)
- **Graph** (fraud/duplication): Neo4j or Amazon Neptune
- **Data lake/warehouse** (analytics): S3/GCS + Athena/BigQuery (privacy-safe, aggregated)

### ML / NLP (as needed)
- **NLP**: lightweight intent/entity extraction (spaCy / transformers) with on-device option for privacy
- **Model serving**: TorchServe / BentoML / KFServing (KServe)
- **Federated learning**: Flower / TensorFlow Federated (when improving models from distributed data)

### Privacy & Cryptography
- **ZK proofs**: zkSNARK frameworks (Circom/snarkjs) or enterprise ZK stack (jurisdiction dependent)
- **MPC**: frameworks like MP-SPDZ / EMP-toolkit (for cross-agency computations)
- **Key management**: HashiCorp Vault / cloud KMS (AWS KMS/Azure Key Vault/GCP KMS)
- **Encryption**: field-level encryption libs; tokenization for identifiers

### Security / Observability / DevOps
- **Containers**: Docker + Kubernetes
- **Service mesh** (optional): Istio/Linkerd (mTLS, policy)
- **CI/CD**: GitHub Actions/GitLab CI/Azure DevOps
- **Monitoring**: Prometheus + Grafana
- **Tracing**: OpenTelemetry + Jaeger/Tempo
- **Logging**: ELK/OpenSearch + SIEM integration
- **WAF + bot protection**: Cloudflare/AWS WAF

---

## 9) Fraud Prevention Design (Minimum Controls)
- **Identity & duplication**:
  - deterministic match (government ID where allowed)
  - fuzzy match (name/address/phone), salted hashing
  - graph linking (shared device, address, phone, document reuse)
- **Behavioral risk**:
  - rapid retries, inconsistent answers, scripted navigation patterns
  - anomaly detection on input distributions per region
- **Document & claim verification**:
  - OCR + integrity checks (metadata, tamper detection)
  - attestation from issuers (verifiable credentials) where available
- **Adversarial defenses**:
  - rate limits, CAPTCHA/turnstile, device binding
  - signed client telemetry, replay protection, nonce-based forms
  - tamper-evident audit trail

---

## 10) Privacy & Compliance Requirements
- **Consent**: explicit consent, granular purposes, revocation path.
- **Retention**: short default retention for sessions; configurable per policy.
- **Access control**: least privilege, break-glass procedures, audit for admin actions.
- **Data residency**: region-based storage and processing.
- **Privacy-by-design**: prefer computed proofs/attestations over raw values; minimize PII.

---

## 11) Deliverables
- **Requirements & Architecture**: PRD (this), threat model, data flow diagrams, DPIA-style privacy assessment.
- **MVP**:
  - scheme registry + rules engine + 50 schemes loaded
  - questioning engine with real-time updates
  - scoring + explainability
  - baseline fraud checks + audit logs
- **Admin portal**: rule authoring, versioning, simulation, approvals.
- **Security baseline**: encryption, IAM, WAF, rate limiting, monitoring.

---

## 12) Implementation Phases (Suggested)
- **Phase 1 (MVP)**: deterministic rules + smart questioning (info gain), explainability, basic fraud/rate limiting.
- **Phase 2**: probability calibration, NLP intake, stronger fraud graph linking, rule simulation at scale.
- **Phase 3**: ZK proofs for key attributes + MPC pilots for cross-agency verification, federated learning for model improvement.

If you want, I can convert this into a **formal SRS** (IEEE-style), plus **system architecture diagram + DFD + threat model table** tailored to a specific country/state and its typical scheme attributes.
