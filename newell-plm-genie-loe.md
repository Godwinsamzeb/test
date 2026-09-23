# LOE: Newell Brands — PLM Genie Ready MVP

**Client:** Newell Brands
**Date:** 2026-09-22
**Prepared by:** zeb delivery
**Version:** 1.0
**Target price:** $30,100 (43 ECUs × $700/ECU)

## Engagement Summary

Newell Brands is standing up a Databricks Genie Ready MVP over product lifecycle management — engineering change, material and specification workflow, and launch execution — across three source systems already landing in Unity Catalog on a two-hour cadence: Windchill (ECR/ECN, parts, control records), Precision (Oracle-based material master and workflow), and Project Online with MS Planner. The existing import pipelines are unchanged. This engagement builds the silver and gold medallion above them under Unity Catalog across an estate scoped at 15 tables and 250GB, conformed on a shared PLM lifecycle model.

A pre-hydrated Genie space sits on the governed gold layer with persona-based access for 5–8 candidate personas, seeded with curated instructions, a business glossary, trusted example queries, and defined time-range defaults. Because PLM questions cross systems, the engagement delivers a part-to-material crosswalk and configures Genie to disclose, on every cross-system answer, the crosswalk coverage it relied on and the as-of time of the oldest contributing source. An AI/BI dashboard reads the same layer as a canonical visual reference, and user-based cost alerts give per-user and per-persona spend visibility from day one. The existing *Where's my Stuff* Power BI report is unaffected and runs in parallel.

Delivery is bounded to a four-week window with zeb engineers full-time and roughly ten hours per week of Newell Brands business and technical time. Every payment is gated by an Outcome Acceptance Event tied to one of three outcome milestones.

## Stories

### S-001: Bronze profiling & grain validation — Windchill, Precision, Project Online, Planner
- **Specialization:** Data Engineering
- **Complexity:** Medium
- **ECUs:** 5
- **Description:** Profile every in-scope bronze table for grain, keys, cardinality, and null rates; document actual column names; evidence a uniqueness test per table; review physical-to-logical mapping with Newell Brands; confirm existing import pipelines are unmodified.
- **Notes:** Rolls into Outcome 1. Assumes read access to Windchill, Precision, Project Online, and Planner bronze tables is granted at kickoff.

### S-002: Silver/gold PLM lifecycle conformance + part↔material crosswalk with published coverage
- **Specialization:** Data Engineering
- **Complexity:** High
- **ECUs:** 8
- **Description:** Build silver and gold layers under Unity Catalog conformed on the shared PLM lifecycle model — engineering change and affected objects, material and workflow status, launch tasks — with documented promotion logic. Build the part-to-material crosswalk carrying match method (exact, normalized, rule-based, manually curated) and confidence per row; stand up the business-maintained override table; publish coverage as a governed metric.
- **Dependencies:** S-001
- **Notes:** Rolls into Outcome 1. Single largest technical dependency in the engagement; complexity assumes an existing crosswalk or normalization rules can be sourced from Precision, SAP, or a maintained spreadsheet during discovery.

### S-003: Data-quality expectations & conformance test suite on the gold pipeline
- **Specialization:** Data Engineering
- **Complexity:** Low
- **ECUs:** 2
- **Description:** Author and wire conformance tests running as part of the gold pipeline — grain uniqueness, referential integrity between changes and affected objects, unresolved-identity rate, and crosswalk coverage — with results visible to Newell Brands rather than held by zeb.
- **Dependencies:** S-002
- **Notes:** Rolls into Outcome 1.

### S-004: Gold semantic views + metric views + PLM business glossary
- **Specialization:** Data Engineering
- **Complexity:** Medium
- **ECUs:** 5
- **Description:** In Newell Brands' Databricks workspace, create curated gold-zone views and Unity Catalog metric views for the PLM metrics that ground Genie (open change count, change aging, materials in workflow, late launch tasks), along with table and column comments, synonyms, and the PLM business glossary.
- **Dependencies:** S-002
- **Notes:** Rolls into Outcome 2. Metric definitions confirmed with the PLM analyst persona during discovery.

### S-005: Pre-hydrated Genie — instructions, synonyms, trusted queries, time-range defaults
- **Specialization:** AI / ML Engineering
- **Complexity:** Medium
- **ECUs:** 5
- **Description:** Configure the Genie space over the governed gold layer with curated instructions, a PLM business glossary, trusted example queries, and defined time-range defaults. Map the captured question set to the certified gold-zone tables and metric views; validate the SQL Genie generates for representative questions with Newell Brands' technical team.
- **Dependencies:** S-004
- **Notes:** Rolls into Outcome 2.

### S-006: Persona-based access control — 5–8 personas configured in Unity Catalog
- **Specialization:** Data Engineering
- **Complexity:** Low
- **ECUs:** 2
- **Description:** Configure persona-based access for 5–8 candidate personas (PLM/change analysts, launch managers, engineering managers, material specialists, supply chain, information delivery) using Unity Catalog role-based and attribute-based controls; inherit from Newell Brands' existing Entra ID identity policy.
- **Dependencies:** S-004
- **Notes:** Rolls into Outcome 2. No new security model is built by zeb; the space operates within the Client's existing row-level and login/authentication policy.

### S-007: Freshness & crosswalk-coverage disclosure configuration
- **Specialization:** AI / ML Engineering
- **Complexity:** Minimal
- **ECUs:** 1
- **Description:** Configure Genie to state the as-of timestamp of the oldest contributing source on every answer, and to disclose crosswalk coverage on any answer that crosses systems. Records whose brand or identity cannot be resolved surface explicitly rather than being dropped.
- **Dependencies:** S-005, S-002
- **Notes:** Rolls into Outcome 2.

### S-008: Question bank build + evaluation & accuracy report against verified answers
- **Specialization:** AI / ML Engineering
- **Complexity:** Medium
- **ECUs:** 5
- **Description:** Extrapolate seed questions into ~20–30 sample evaluation questions across personas; confirm with Newell Brands business users. Run the question bank against Genie, score against verified answers, produce the evaluation and accuracy report, triage misses, and configure per-answer feedback (thumbs up/down or comment) for the pilot.
- **Dependencies:** S-005, S-007
- **Notes:** Rolls into Outcome 3.

### S-009: AI/BI dashboard — pipeline health, change queue, process exceptions
- **Specialization:** Data Engineering
- **Complexity:** Medium
- **ECUs:** 5
- **Description:** Build the AI/BI dashboard on the governed gold layer surfacing pipeline health by lifecycle stage, the open change queue, and detected process exceptions. Reconciles with Genie answers on identical inputs.
- **Dependencies:** S-004
- **Notes:** Rolls into Outcome 3.

### S-010: User-based cost alerts & FinOps in Databricks
- **Specialization:** Platform / Infrastructure
- **Complexity:** Low
- **ECUs:** 2
- **Description:** Configure user-based cost alerts in Databricks with thresholds and notification channels agreed with Newell Brands, giving per-user and per-persona spend visibility from the first day of pilot use.
- **Notes:** Rolls into Outcome 3.

### S-011: Runbook, knowledge transfer & go-forward summary
- **Specialization:** Platform / Infrastructure
- **Complexity:** Low
- **ECUs:** 2
- **Description:** Produce the runbook covering Genie space maintenance, question bank updates, persona changes, crosswalk override maintenance, and cost alert tuning. Deliver 2–3 validation sessions with Newell Brands' PLM, engineering, and information delivery teams walking the flow end-to-end from bronze through gold, Genie, and the AI/BI dashboard. Hand over the go-forward summary of data-quality and access gaps to inform the subsequent full-build SOW.
- **Dependencies:** S-008, S-009
- **Notes:** Rolls into Outcome 3.

### S-012: PROD promotion via Client CI/CD — pipelines, Genie space, dashboard
- **Specialization:** Platform / Infrastructure
- **Complexity:** Minimal
- **ECUs:** 1
- **Description:** Promote gold pipelines, the Genie space, and the AI/BI dashboard to PROD through Newell Brands' existing CI/CD process; run against live imports.
- **Dependencies:** S-005, S-006, S-009
- **Notes:** Rolls into Outcome 3.

## Summary

| Specialization           | Story Count | Total ECUs |
|--------------------------|-------------|------------|
| Data Engineering         | 6           | 27         |
| AI / ML Engineering      | 3           | 11         |
| Platform / Infrastructure| 3           | 5          |
| **Total**                | **12**      | **43**     |

### Outcome milestone rollup

| Outcome | Title                                | Stories                             | ECUs | Payment |
|---------|--------------------------------------|-------------------------------------|------|---------|
| 1       | PLM Data Foundation                  | S-001, S-002, S-003                 | 15   | $10,500 |
| 2       | Genie Space & Persona Access         | S-004, S-005, S-006, S-007          | 13   | $9,100  |
| 3       | AI/BI Dashboard, Evaluation & Handover| S-008, S-009, S-010, S-011, S-012  | 15   | $10,500 |
| **Total**|                                     |                                     | **43** | **$30,100** |

## Assumptions & Exclusions

**Assumptions**
- Windchill, Precision, Project Online, and MS Planner already land in Unity Catalog on a two-hour cadence; existing import pipelines are unmodified.
- Estate is bounded to 15 tables and 250GB across the PLM lifecycle domain.
- 5–8 candidate personas provided by Newell Brands at kickoff; access inherits Entra ID identity policy.
- Approximately ten hours per week of Newell Brands business and technical availability across the four-week window.
- An existing crosswalk source or normalization rules are available in Precision, SAP, or a maintained spreadsheet for S-002.
- Newell Brands runs promotion to PROD through its existing CI/CD process.

**Exclusions**
- SAP ingestion, connectivity, or material-status reconciliation. Questions dependent on SAP material status return an explicit statement of that limitation.
- Migration, refactor, or decommissioning of the existing *Where's my Stuff* Power BI report; it runs in parallel throughout.
- Cycle-time prediction, launch-risk forecasting, and any predictive/ML capability.
- Front-end development, third-party integrations beyond agreed source systems, and extensive integration work.
- User training beyond initial handover; long-term operational management; code optimization or refactor of existing ETL beyond migration-related updates.
- Any domain outside PLM (engineering change from Windchill, material/spec workflow from Precision, launch execution from Project Online and MS Planner).
