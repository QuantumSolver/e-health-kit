# Frappe E-Health Integration Guide

> How to use the adapted Antigravity Kit for building healthcare systems on Frappe Framework.

---

## Quick Start

1. **Scaffold** — use the `/plan` workflow. Mention "Frappe" or "healthcare" and the `project-planner` auto-routes to the healthcare pipeline.
2. **Template** — when the `app-builder` skill fires, pick **frappe-app** from the template list. It generates the full Bench app skeleton with healthcare DocTypes.
3. **Develop** — the backend, database, and frontend specialists all carry Frappe-specific sections that load automatically.
4. **Secure** — the `security-auditor` now includes PHI/HIPAA checks; the `healthcare-compliance` skill provides the full checklist.

---

## Agent → Skill Mapping for Healthcare

| Agent | Loaded Skills | What It Does for Healthcare |
|-------|---------------|-----------------------------|
| `project-planner` | frappe-development, healthcare-compliance | Detects healthcare trigger words, routes P1-H priority, sets agent order |
| `backend-specialist` | frappe-development, healthcare-compliance | Frappe controllers, hooks, `@frappe.whitelist()` APIs, background jobs, compliance |
| `database-architect` | frappe-development | DocType modelling, Link fields, Child Tables, naming rules, healthcare schema |
| `frontend-specialist` | frappe-development | Frappe Desk views, Client Scripts, Frappe UI (Vue 3), healthcare dashboards |
| `security-auditor` | healthcare-compliance | PHI encryption, audit trail, RBAC, TLS, HIPAA table |
| `product-manager` | frappe-development, healthcare-compliance | Healthcare user stories, regulatory context |

---

## Typical Workflow

```
User: "Build a patient management app on Frappe"

1. project-planner activates
   ├── Detects "patient" + "Frappe" → HEALTHCARE project type
   ├── Sets priority: P0 database-architect → P1-H backend + security → P2 frontend
   └── Outputs structured plan

2. database-architect designs DocTypes
   ├── Loads frappe-development skill § DocType Modelling
   ├── Designs Patient, Encounter, Appointment, Prescription, Lab Test
   └── Defines Link fields, Child Tables, naming rules

3. backend-specialist implements logic
   ├── Loads frappe-development § Controllers & Hooks
   ├── Loads healthcare-compliance § Access Control, Audit, PHI
   └── Writes controllers, whitelist APIs, scheduler events

4. security-auditor reviews
   ├── Loads healthcare-compliance § Full checklist
   ├── Checks PHI in logs, RBAC, encryption, session timeout
   └── Outputs compliance report

5. frontend-specialist builds UI
   ├── Loads frappe-development § Desk customisation
   ├── Loads frontend-design § Section 11 (Healthcare UI)
   └── Configures Workspaces, Dashboards, Client Scripts
```

---

## Common Healthcare Tasks

### 1. Patient Registration
- **Template**: `frappe-app` → `Patient` DocType already scaffolded
- **Skills**: frappe-development (DocType fields, naming), healthcare-compliance (PHI classification)
- **Key checks**: Encrypt PII fields, set role permissions (Receptionist can create, Physician can read)

### 2. Clinical Encounter / EMR
- **DocTypes**: Encounter (parent), Vital Signs (child), Diagnosis (child)
- **Skills**: frappe-development (controller lifecycle: validate → before_save → on_submit)
- **UI**: frontend-design § 11 — Clinical Encounter Form pattern (Chief Complaint → Assessment → Plan)

### 3. Appointment Scheduling
- **DocTypes**: Appointment, Practitioner Schedule
- **Skills**: frappe-development (scheduler_events for reminders), frontend-design § 11 — Appointment Calendar
- **Key**: Color-coded statuses, practitioner-centric weekly view

### 4. Prescriptions & Lab Orders
- **DocTypes**: Prescription, Lab Test, Lab Test Template
- **Skills**: healthcare-compliance (drug interaction warnings, FHIR MedicationRequest mapping)
- **UI**: Drug autocomplete, dosage presets, print-friendly output

### 5. FHIR / HL7 Integration
- **Skill**: healthcare-compliance § 6 — HL7 and FHIR Integration Patterns
- **Approach**: Translation layer mapping DocTypes ↔ FHIR Resources (Patient, Encounter, Observation, MedicationRequest)
- **Key**: Keep raw messages for audit, authenticate all endpoints, rate-limit

---

## Compliance Checklist (Gate Before Launch)

From `healthcare-compliance` skill § 7:

- [ ] PHI locations documented
- [ ] RBAC enforced on all health DocTypes
- [ ] Audit logging on PHI access and modifications
- [ ] Transport (TLS 1.2+) and at-rest encryption configured
- [ ] Backup, restore, and retention policies tested
- [ ] HL7/FHIR endpoints authenticated and rate-limited
- [ ] No PHI in logs, analytics, or URLs
- [ ] Compliance stakeholder sign-off

---

## Skills Reference

| Skill | Path | When to Read |
|-------|------|--------------|
| frappe-development | `.agent/skills/frappe-development/SKILL.md` | Any Frappe project |
| healthcare-compliance | `.agent/skills/healthcare-compliance/SKILL.md` | Any PHI-handling feature |
| app-builder (frappe-app template) | `.agent/skills/app-builder/templates/frappe-app/TEMPLATE.md` | Scaffolding a new Frappe healthcare app |
| frontend-design § 11 | `.agent/skills/frontend-design/SKILL.md` (Section 11) | Healthcare UI patterns & accessibility |

---

## Files Changed in This Adaptation

**New (3):** `frappe-development/SKILL.md`, `healthcare-compliance/SKILL.md`, `frappe-app/TEMPLATE.md`
**Agents updated (6):** backend-specialist, database-architect, frontend-specialist, security-auditor, product-manager, project-planner
**Skills updated (1):** frontend-design (Section 11 added)
**Architecture updated (1):** ARCHITECTURE.md (counts, mappings, quick ref)

