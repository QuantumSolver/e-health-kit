---
name: frappe-app
description: Frappe Framework healthcare/EHR app template. DocTypes, hooks, permissions, and best practices.
---

# Frappe Healthcare App Template

## Tech Stack

| Component | Technology | Notes |
|-----------|------------|-------|
| Framework | Frappe Framework 15+ | Meta-data driven, DocType-based full-stack |
| Language | Python 3.11+ | Server plus background jobs |
| DB | MariaDB or PostgreSQL | Managed by Frappe migrations |
| Frontend | Frappe Desk plus Frappe UI (Vue 3) | Desk forms, reports, dashboards |
| Templates | Jinja | Web pages, print formats, emails |
| Tools | Bench CLI | bench for sites, apps, patches |
| Optional | ERPNext plus Healthcare module | Reuse standard DocTypes if allowed |

---

## Bench and App Setup (reference)

Use Bench for project scaffolding (run in a terminal, not from the agent):

```bash
bench init frappe-bench --frappe-path https://github.com/frappe/frappe
cd frappe-bench
bench new-site ehr.localhost
bench new-app my_ehealth
bench install-app my_ehealth --site ehr.localhost
```

For ERPNext-based builds, install ERPNext and enable the Healthcare module instead of re-inventing core DocTypes.

---

## Directory Structure (app-level)

```text
frappe-bench/
├── sites/
│   └── ehr.localhost/
└── apps/
    └── my_ehealth/
        └── my_ehealth/
            ├── my_ehealth/__init__.py
            ├── hooks.py              # Doc events, scheduled jobs, overrides
            ├── config/
            │   ├── desktop.py        # Desk module and workspace entries
            │   └── docs.py           # User docs links
            ├── public/               # JS and CSS assets
            ├── www/                  # Website routes (Jinja templates)
            └── my_ehealth/doctype/
                ├── patient/
                ├── patient_appointment/
                ├── patient_encounter/
                ├── medical_record/
                └── clinical_procedure/
```

---

## Core Healthcare DocTypes

| DocType | Purpose | Key Fields and Notes |
|---------|---------|----------------------|
| Patient | Master record for a person | Demographics, identifiers (MRN), contact, consent flags |
| Patient Appointment | Scheduled visits | Patient, practitioner, department, appointment type, status |
| Patient Encounter | Clinical encounter or visit note | Links to Patient, Appointment, diagnoses, complaints, vitals, orders |
| Medical Record | Longitudinal record entry | Encounter reference, narrative note, attachments, coding (ICD or LOINC) |
| Prescription or Medication Order | Medication management | Drug, dosage, route, schedule, duration, allergies check |
| Lab Test or Investigation | Diagnostic tests | Test template, specimen, status, results, reference ranges |
| Vital Signs | Structured vitals | BP, HR, RR, Temp, BMI, measurement time, device |
| Healthcare Practitioner | Clinicians and staff | Role, specialty, department, license, calendars |

Model these as DocTypes rather than free-form tables; use Link fields to relate records instead of raw identifiers.

---

## Hooks and Server Logic

Use hooks.py to connect domain rules to the DocType lifecycle:

```python
doc_events = {
    'Patient Encounter': {
        'validate': 'my_ehealth.api.encounter.validate_encounter',
        'on_submit': 'my_ehealth.api.encounter.on_submit'
    },
    'Lab Test': {
        'on_submit': 'my_ehealth.api.lab.propagate_results'
    }
}
```

Guidelines:
- Keep business rules in Python controllers (doctype specific modules) or service modules, for example my_ehealth/api.
- Keep DocTypes lean; avoid duplicating logic in Client Scripts and Server Scripts unless customisation is unavoidable.
- For integrations such as FHIR or HL7, expose whitelisted methods in a thin API layer that maps to DocTypes.

---

## Permissions, Roles, and Audit

- Use role-based permissions at DocType level (read, write, create, submit, cancel) instead of ad-hoc checks.
- Define roles like Receptionist, Nurse, Physician, Lab Technician, Billing User, Administrator.
- Use User Permissions to restrict Patient access by facility, organisation, or practitioner where needed.
- Enable audit trail or versioning for all DocTypes that store PHI (patients, encounters, medical records).
- Never expose PHI in error messages, logs, or client-side script comments.

---

## Desk UI and Patient Workflows

- Build Workspaces for key personas (Front Desk, Doctor, Nurse, Billing) with shortcuts to relevant DocTypes and reports.
- Use standard Desk List plus Form patterns for Patient, Appointment, and Encounter instead of custom UIs when possible.
- For dashboards such as clinician or patient overview, use Frappe Dashboards and Reports and, for rich widgets, Frappe UI based on Vue.
- Optimise for keyboard-first data entry, large click targets, and high-contrast themes suitable for clinical environments.
- Keep forms simple: group fields by clinical meaning (triage, diagnosis, orders, billing) and avoid overloading a single screen.

---

## Minimum Best Practices for E-Health Apps

- Treat all healthcare data as PHI; combine this template with the healthcare-compliance skill.
- Enforce unique patient identifiers and avoid duplicating patient records.
- Use background jobs for heavy tasks such as bulk imports, large document generation, and external system synchronisation.
- Prefer integrations using FHIR resources mapped to DocTypes such as Patient, Encounter, Observation, and MedicationRequest.
- Document every key workflow (registration, triage, consultation, discharge, billing) alongside the DocTypes involved.

