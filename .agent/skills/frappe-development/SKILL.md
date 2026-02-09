---
name: frappe-development
description: Patterns and decision-making for Frappe Framework apps including DocTypes, controllers, client and server scripts, workflows, and ERPNext-style customisation.
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Frappe Development Patterns

> Opinionated guidance for building maintainable Frappe or ERPNext apps, with a focus on e-health systems.

---

## How to Use This Skill

Use this whenever:
- The project is built on Frappe Framework or ERPNext.
- The user mentions DocTypes, Bench, sites or apps, or ERPNext Healthcare.
- You are modelling business entities as DocTypes instead of ORM models.

Combine this with:
- python-patterns for general Python decisions.
- database-design for physical database concerns such as indexes and performance.
- healthcare-compliance when PHI or clinical workflows are involved.

---

## 1. Frappe Architecture in 60 Seconds

- Sites: contain databases and configuration for each tenant.
- Apps: Python packages that define DocTypes, pages, reports, and logic.
- DocTypes: metadata that defines database tables, forms, permissions, and controllers.
- Desk: the authenticated back-office UI, with List, Form, Report, Dashboard, and Workspace.
- Website: public pages, web forms, and portals, usually Jinja-based.
- Bench: CLI that manages sites, apps, migrations, and workers.

Design principle: DocType-first – model your domain as DocTypes, then attach logic via controllers and hooks.

---

## 2. DocType-Based Data Modelling

When modelling an entity such as patient, appointment, or encounter:

- Use Link fields for relations instead of manual foreign keys.
- Prefer child tables for repeating structures such as vitals, diagnoses, or procedures.
- Use Select with controlled vocabularies for status, priority, and triage levels.
- Avoid large free-text fields when structured data is needed for analytics.
- Mark sensitive DocTypes to track changes or views for auditability.

For healthcare:
- Keep core DocTypes small and focused, for example Patient versus Encounter versus MedicalRecord.
- Avoid mixing administrative and clinical data in the same DocType.

---

## 3. Controllers, Hooks, and Services

Each DocType normally has a Python controller at:
- doctype/<doctype>/<doctype>.py

Guidelines:
- Put validation and invariants in controller methods such as validate, before_save, and on_submit.
- Keep controllers thin by delegating to service modules, for example my_app/api/encounter.py.
- Register domain events in hooks.py using doc_events, scheduler_events, and whitelisted API methods.

Example decisions:
- Use DocType controller methods for lifecycle-related logic such as enforcing required links or status transitions.
- Use service modules for cross-DocType workflows such as encounter to orders to billing.
- Avoid business logic in random utility modules or client scripts.

---

## 4. Client Scripts versus Server Scripts

Client Scripts, written in JavaScript and executed in Desk:
- Use for small UX tweaks, dynamic field behaviour, and validation that improves user feedback.
- Never rely on them for security or permission enforcement.

Server Scripts, written in Python and executed on the backend:
- Use cautiously for on-the-fly customisation in production when editing the app is not possible.
- Prefer first-class app code such as controllers, services, and tests for long-term maintainability.

Rules:
- Any rule that protects data integrity or implements compliance must exist in server-side app code.
- Treat Client Scripts as progressive enhancement, not the source of truth.

---

## 5. Workflows, Permissions, and Healthcare Roles

Frappe Workflows:
- Use workflow states and transitions for long-running processes, for example the lab test lifecycle.
- Map each state to allowed actions for specific roles, for example technician can submit result, doctor can verify.

Permission system:
- Start with the DocType permissions matrix per role and per action.
- Refine with User Permissions and Sharing for record-level control.
- Avoid duplicating permission checks in every function; centralise where possible.

Healthcare-specific hints:
- Define clear roles such as Receptionist, Nurse, Physician, Lab Tech, Billing, Administrator.
- Ensure that views for PHI-heavy DocTypes respect least-privilege principles.
- Enable standard audit fields and change tracking on all clinical DocTypes.

---

## 6. Patterns for E-Health Apps

When building Frappe-based healthcare systems:
- Reuse ERPNext Healthcare DocTypes when licensing and scope allow; extend via Custom Fields and Custom Scripts.
- For greenfield apps, start from core DocTypes such as Patient, Encounter, Appointment, Prescription, LabTest, Invoice.
- Keep integration boundaries clean: design FHIR-facing resources that map one-to-one or many-to-one with DocTypes.
- Use background jobs for heavy tasks such as bulk data migration, document generation, and external synchronisation.
- Write automated tests for critical workflows including registration, encounter, prescription, and billing.

Always design with clinical safety, traceability, and operator ergonomics in mind, not only developer convenience.

---

## 7. ERPNext Healthcare Module Reference

When building on top of the existing ERPNext Healthcare module (rather than greenfield), these are the standard DocTypes you inherit and extend.

### Core Healthcare DocTypes (ERPNext Healthcare)

| DocType | Purpose | Key Fields |
|---------|---------|------------|
| Patient | Master patient record | patient_name, sex, dob, blood_group, mobile, email, uid (national ID), status, territory |
| Patient Appointment | Scheduled visit | patient, practitioner, appointment_type, appointment_date, appointment_time, department, status, duration |
| Patient Encounter | Clinical visit note | patient, practitioner, encounter_date, symptoms (child table), diagnosis (child table), drug_prescription (child table), lab_test_prescription (child table) |
| Vital Signs | Structured vitals | patient, signs_date, temperature, pulse, respiratory_rate, bp_systolic, bp_diastolic, bmi, height, weight, oxygen_saturation |
| Lab Test | Diagnostic test | patient, practitioner, lab_test_name, template, result_date, normal_test_items (child table), descriptive_test_items (child table), status |
| Lab Test Template | Test definition | lab_test_name, lab_test_group, lab_test_template_type, normal_test_templates (child table), descriptive_test_templates (child table) |
| Clinical Procedure | Procedures | patient, practitioner, procedure_template, start_date, status, consume_stock |
| Clinical Procedure Template | Procedure definition | template, medical_department, description, rate |
| Prescription Dosage | Dosage presets | dosage, dosage_strength (child table) |
| Healthcare Practitioner | Clinician record | practitioner_name, department, designation, op_consulting_charge, employee (link) |
| Healthcare Service Unit | Facility rooms/beds | service_unit_type, warehouse, company, is_group, parent_healthcare_service_unit |
| Medical Code Standard | Coding system | medical_code_standard (ICD-10, SNOMED, LOINC) |
| Medical Code | Individual code | medical_code, code, description |
| Therapy Type | Rehab/therapy | therapy_type, default_duration, rate, exercises (child table) |
| Therapy Plan | Treatment plan | patient, start_date, therapy_plan_details (child table) |
| Inpatient Record | Admission record | patient, admitted_datetime, discharge_datetime, primary_practitioner, admission_service_unit |

### Extending Healthcare DocTypes with Custom Fields

When you need to add fields to standard Healthcare DocTypes without forking the module:

- Use Custom Fields via Setup > Customize > Custom Field or programmatically in your app's fixtures.
- Use fixtures in hooks.py to export and version-control Custom Fields:

```python
# hooks.py
fixtures = [
    {
        "dt": "Custom Field",
        "filters": [["module", "=", "My EHealth"]]
    }
]
```

- Use Custom Scripts (Client Script DocType) for UI-side behaviour on standard forms.
- Use doc_events in hooks.py to attach server logic to standard Healthcare DocTypes without modifying their controllers:

```python
doc_events = {
    "Patient Encounter": {
        "validate": "my_ehealth.overrides.encounter.custom_validate",
        "on_submit": "my_ehealth.overrides.encounter.on_submit"
    }
}
```

### Naming Conventions in Healthcare

| DocType | Default Naming | Override |
|---------|---------------|----------|
| Patient | PAT-.#####  | Naming Series or field-based |
| Patient Appointment | HLC-APP-.YYYY.-.##### | Naming Series |
| Patient Encounter | HLC-ENC-.YYYY.-.##### | Naming Series |
| Lab Test | HLC-LAB-.YYYY.-.##### | Naming Series |
| Vital Signs | HLC-VIT-.YYYY.-.##### | Naming Series |

### Common Extension Patterns

- Add insurance fields to Patient via Custom Fields (insurance_provider, policy_number, expiry_date).
- Add triage_level Select field to Patient Encounter for emergency department workflows.
- Create custom child tables for structured data not covered by standard DocTypes (e.g., Allergy child table on Patient).
- Override print formats for prescriptions, lab reports, and discharge summaries using custom Jinja templates.
- Add Workflow states to Patient Encounter for multi-step approval (Draft → In Progress → Completed → Billed).

