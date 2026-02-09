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

