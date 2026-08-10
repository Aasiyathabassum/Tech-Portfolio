# AI Assurance Workbench

A modular framework and toolkit for testing, scoring, and documenting the trustworthiness of deployed AI and ML models, built to close the gap between AI governance standards, which define principles, and AI governance practice, which requires repeatable, automated testing.

*Note: this repository describes the framework's design and methodology at a conceptual level. Specific tooling choices, granular standards mappings, and implementation details are intentionally omitted from this public description.*

## Where this fits in the AI lifecycle

Assurance is not a one-off gate before release. It applies at initial testing and evaluation, at deployment sign-off, and continuously during monitoring and retraining, since a model's behavior can drift after deployment.

```mermaid
%%{init: {
  "theme": "base",
  "flowchart": { "htmlLabels": false },
  "themeVariables": {
    "primaryColor": "#EAF2F3",
    "primaryTextColor": "#0B3C4A",
    "primaryBorderColor": "#0F4C5C",
    "lineColor": "#5C6B73",
    "secondaryColor": "#1B6B75",
    "tertiaryColor": "#EAF2F3",
    "fontSize": "16px",
    "edgeLabelBackground": "#ffffff"
  }
}}%%
flowchart LR
    A["Business Case"] --> B["Data<br/>Preparation"]
    B --> C["Modelling"]
    C --> D["Testing and<br/>Evaluation"]
    D --> E["Deployment"]
    E --> F["Monitoring and<br/>Retraining"]
    F -.->|retrain loop| C

    W(["AI Assurance<br/>Workbench"])
    W -.->|assures| D
    W -.->|assures| E
    W -.->|continuous checks| F

    classDef workbench fill:#0F4C5C,color:#ffffff,stroke:#0B3C4A,stroke-width:2px;
    class W workbench;
```

## Why this exists

Vision and ML models are increasingly deployed in safety and quality critical industrial settings, but proving a model is safe, robust, fair, and explainable is still largely a manual, ad-hoc process. Many open-source assurance techniques exist (explainability methods, adversarial robustness testing, fairness and drift checking, model documentation generation), but each is typically wired into a specific model's inputs and outputs by hand, on a case-by-case basis, and rarely traced back to a named regulatory or standards requirement.

This project standardizes that process: define a common model-submission schema, map recognized AI governance standards onto a consistent, standards-traceable set of testable dimensions, and automatically generate and run the right combination of tests for a given model, producing a scored, human-readable assurance report.

## Architecture

```mermaid
%%{init: {
  "theme": "base",
  "flowchart": { "htmlLabels": false },
  "themeVariables": {
    "primaryColor": "#0F4C5C",
    "primaryTextColor": "#ffffff",
    "primaryBorderColor": "#0B3C4A",
    "lineColor": "#5C6B73",
    "secondaryColor": "#1B6B75",
    "tertiaryColor": "#EAF2F3",
    "fontSize": "16px",
    "edgeLabelBackground": "#ffffff"
  }
}}%%
flowchart LR
    A["Model Owner"] --> B["Submit Model<br/>+ Manifest"]
    A --> B2["Submit Training /<br/>Test Data"]

    B --> AD["Framework<br/>Adapter Layer"]
    AD --> DET["Canonical Model<br/>Output Schema"]
    B2 --> DQ["Data Assurance<br/>Check"]

    DET --> PT["Automated Pillar<br/>Test Suite"]
    DQ --> PT
    HO["Human Oversight and<br/>Contestability Process<br/>(not automated)"]

    PT --> H["Assurance<br/>Report"]
    HO -.-> H

    H --> I["Governance<br/>Review"]
    I -.->|feedback| A

    classDef pillar fill:#1B6B75,color:#ffffff,stroke:#0B3C4A,stroke-width:2px;
    classDef process fill:#C97A2B,color:#ffffff,stroke:#0B3C4A,stroke-width:2px;
    class AD,DET,DQ,PT pillar;
    class HO process;
```

A model owner submits model weights plus a manifest describing the model's task, framework, and intended use, and separately points to the training/test data used. A framework adapter layer normalizes whatever architecture the model is built on into one canonical output schema, so every downstream test only ever needs to understand one data shape rather than each framework's native output. The workbench runs the relevant automated tests against that normalized output and against the submitted data, aggregates results into an assurance report alongside evidence from the (non-automated) human oversight and contestability process, and a governance reviewer signs off or sends feedback back to the model owner. Continuous drift monitoring runs independently of this pipeline against live production data, feeding the retrain loop shown in the lifecycle diagram above.

## Project journey

**Phase 1, completed:** applied existing assurance techniques end to end to validate a real-world computer vision system deployed on edge hardware for manufacturing assembly verification. This involved:

- Reviewing an existing deployed model, its documentation, and its test dataset to plan a testing approach.
- Running an explainability analysis to confirm the model's decisions were based on the correct visual features rather than spurious correlations.
- Running an adversarial robustness assessment across multiple attack types and perturbation levels, covering both natural degradation and deliberate manipulation.
- Running a fairness and data-quality assessment across varying environmental conditions such as lighting, sharpness, exposure, and contrast.
- Producing standardized model documentation covering capabilities, intended use, and known limitations.
- Consolidating all assessments into an overall trustworthiness scorecard, mapped against a simplified set of trust pillars, with risk-based recommendations ahead of production deployment.

**Phase 2, current:** generalizing that manual, case-by-case process into a reusable framework and workbench, so any submitted model can be tested automatically against a standard set of trust dimensions rather than assembled by hand each time. This phase also re-derives the dimension set directly from primary AI governance sources, rather than reusing Phase 1's simplified pillar set, so that assurance evidence can be traced to a named regulatory or standards area rather than an internal label alone.

*(All results and details above are generalized. No client, project, tooling, or quantitative specifics are included.)*

## Framework: dimensions of trustworthy AI

Model assurance is structured around a consistent set of dimensions, each independently testable and scoreable:

| Dimension | What it evaluates |
|---|---|
| Transparency and explainability | Can predictions be understood and attributed to input features? |
| Safety, security, and robustness | How does the model behave under noise, adversarial conditions, or distribution shift? |
| Fairness and non-discrimination | Does performance hold consistently across sub-populations or environmental conditions? |
| Data quality and representativeness | Is training/test data relevant, complete, accurate, and representative of the real deployment population? |
| Accountability and governance | Is the model's design, training data, and intended use properly documented? |
| Human oversight, contestability, and redress | Does a human retain final decision authority, and can affected stakeholders challenge or query a model's decision? |
| Privacy and data governance | Is data access, storage, and handling properly governed? |
| Societal and environmental wellbeing | Broader societal/environmental impact of the deployment |

Each automatable dimension produces a score and a status flag; dimensions that are inherently procedural (human oversight/contestability, privacy/data governance) produce documented evidence rather than a score. All results roll up into an overall assurance scorecard for the model, stating for each governing framework whether it is currently met, partially met, or not met, with recommendations for closing any gap.

## Reference frameworks

The dimension framework above is designed to map onto, not replace, existing AI governance and safety standards, including:

- **EU AI Act** — binding EU regulation; risk-tiered, with high-risk systems subject to technical documentation, risk management, conformity assessment, and human oversight obligations.
- **NIST AI Risk Management Framework (AI RMF)** — voluntary US framework, structured around four functions (Govern, Map, Measure, Manage) and a defined set of trustworthiness characteristics.
- **ISO/IEC 42001** — certifiable AI management system standard, covering organizational governance of AI development and deployment.
- **The broader ISO/IEC AI standards catalog** — covering topics including AI trustworthiness, bias, robustness testing, data quality, and human oversight.
- **Traditional functional safety standards** (e.g. IEC 61508) — predate AI but continue to apply to safety-critical industrial deployments.
- **Emerging ML-specific safety assurance methodologies** (e.g. AMLAS) — for building a formal safety case from ML test evidence, most relevant where model output feeds a safety-relevant decision.
- **Ethics-by-design process standards** (e.g. the IEEE 7000 series).
- **Sector-specific traceability and cybersecurity guidance** relevant to manufacturing and industrial AI deployments.

## Architectural principles

- **Schema-driven, not tool-driven.** The model submission schema decouples what is being tested from how it is tested, so underlying test tooling can be swapped or extended independently.
- **Framework-agnostic by design.** A single adapter layer normalizes model output from any supported architecture into one canonical schema, so pillar tests are written once and reused across every model, regardless of the framework it was built on.
- **Dimension-based modularity.** Each trust dimension is an independent, pluggable module, allowing partial assurance runs and incremental adoption.
- **Standards-traced by design.** Every dimension is mapped to relevant governance standards, so results can be reported as framework-level compliance status, not just an internal score.
- **Human-in-the-loop reporting.** Automated testing feeds a report designed for review and sign-off by governance stakeholders, not a fully autonomous pass or fail gate. Dimensions that are inherently procedural are documented as workflows, not simulated as automated tests.
- **Extensible beyond a single model type.** Phase 1 targeted a vision model. The orchestration and reporting layers are designed to generalize to other modalities, such as language or tabular models, as new test modules are added.

## Roadmap

- [x] Map governance standards onto a simplified, testable pillar set (Phase 1)
- [x] Validate framework against a real-world industrial vision model
- [x] Modular audit pipeline validated across explainability, robustness, fairness, and documentation checks
- [ ] Re-derive assurance dimensions directly from primary AI governance standards (supersedes the Phase 1 pillar set)
- [ ] Model-submission schema decoupled from any specific tool or model architecture
- [ ] Framework-agnostic adapter layer for normalizing model output across architectures
- [ ] Automated test-plan generation from a submitted model schema
- [ ] Data assurance module for training/test data quality and representativeness
- [ ] Documented human oversight and contestability workflow
- [ ] Live results dashboard
- [ ] Auto-generated assurance reports with governance-framework compliance status
- [ ] Generalize audit pipeline beyond vision models (for example, language, tabular, or time-series models)

## Tech stack

Built on a standard Python-based computer vision and MLOps stack, combining open-source explainability, robustness-testing, and data-quality tooling with a custom orchestration and reporting layer.

## Status

Active development. The core audit pipeline is functional and has been validated against a production use case. The unified workbench (schema-driven, framework-agnostic adapters, automated test orchestration, dashboard) is in progress.
