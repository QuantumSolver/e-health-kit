# FHIR R4 ↔ Frappe Healthcare DocType Mappings

> Field-level mapping reference for building FHIR-compliant integrations on Frappe/ERPNext Healthcare.

---

## 1. Patient → FHIR Patient

| Frappe Field | FHIR Path | Type | Notes |
|-------------|-----------|------|-------|
| name (ID) | Patient.id | id | Frappe document name (e.g., PAT-00001) |
| patient_name | Patient.name[0].text | HumanName | Split into given/family if structured |
| first_name | Patient.name[0].given[0] | string | |
| last_name | Patient.name[0].family | string | |
| sex | Patient.gender | code | Map: Male→male, Female→female, Other→other |
| dob | Patient.birthDate | date | YYYY-MM-DD |
| mobile | Patient.telecom[] | ContactPoint | system=phone, use=mobile |
| email | Patient.telecom[] | ContactPoint | system=email |
| uid | Patient.identifier[] | Identifier | system=national-id |
| blood_group | Patient.extension[] | Extension | url=blood-group |
| status | Patient.active | boolean | Active→true, Disabled→false |
| territory | Patient.address[0].state | string | |
| image | Patient.photo[] | Attachment | |

---

## 2. Patient Encounter → FHIR Encounter

| Frappe Field | FHIR Path | Type | Notes |
|-------------|-----------|------|-------|
| name (ID) | Encounter.id | id | |
| patient | Encounter.subject | Reference(Patient) | |
| practitioner | Encounter.participant[0].individual | Reference(Practitioner) | |
| encounter_date | Encounter.period.start | dateTime | |
| status | Encounter.status | code | Map: Open→in-progress, Closed→finished, Cancelled→cancelled |
| company | Encounter.serviceProvider | Reference(Organization) | |
| symptoms (child) | Encounter.reasonCode[] | CodeableConcept | Each symptom row → one reasonCode |
| diagnosis (child) | Encounter.diagnosis[] | BackboneElement | .condition = Reference(Condition) |

---

## 3. Patient Appointment → FHIR Appointment

| Frappe Field | FHIR Path | Type | Notes |
|-------------|-----------|------|-------|
| name (ID) | Appointment.id | id | |
| patient | Appointment.participant[].actor | Reference(Patient) | type=patient |
| practitioner | Appointment.participant[].actor | Reference(Practitioner) | type=practitioner |
| appointment_date | Appointment.start | dateTime | Combine with appointment_time |
| appointment_time | Appointment.start | dateTime | |
| duration | Appointment.minutesDuration | positiveInt | |
| status | Appointment.status | code | Map: Open→booked, Closed→fulfilled, Cancelled→cancelled |
| appointment_type | Appointment.appointmentType | CodeableConcept | |
| department | Appointment.specialty | CodeableConcept | |
| notes | Appointment.comment | string | |

---

## 4. Vital Signs → FHIR Observation (category=vital-signs)

| Frappe Field | FHIR Path | LOINC Code | Unit |
|-------------|-----------|------------|------|
| temperature | Observation.valueQuantity | 8310-5 | °C or °F |
| pulse | Observation.valueQuantity | 8867-4 | /min |
| respiratory_rate | Observation.valueQuantity | 9279-1 | /min |
| bp_systolic | Observation.component[0].valueQuantity | 8480-6 | mmHg |
| bp_diastolic | Observation.component[1].valueQuantity | 8462-4 | mmHg |
| oxygen_saturation | Observation.valueQuantity | 2708-6 | % |
| height | Observation.valueQuantity | 8302-2 | cm |
| weight | Observation.valueQuantity | 29463-7 | kg |
| bmi | Observation.valueQuantity | 39156-5 | kg/m² |
| signs_date | Observation.effectiveDateTime | — | YYYY-MM-DD |
| patient | Observation.subject | — | Reference(Patient) |

> Each vital sign should be a separate FHIR Observation resource with the appropriate LOINC code. Group them using Observation.hasMember if needed.

---

## 5. Lab Test → FHIR DiagnosticReport + Observation

| Frappe Field | FHIR Path | Notes |
|-------------|-----------|-------|
| name (ID) | DiagnosticReport.id | |
| patient | DiagnosticReport.subject | Reference(Patient) |
| practitioner | DiagnosticReport.resultsInterpreter | Reference(Practitioner) |
| lab_test_name | DiagnosticReport.code | CodeableConcept (LOINC) |
| result_date | DiagnosticReport.effectiveDateTime | |
| status | DiagnosticReport.status | Map: Draft→registered, Completed→final, Cancelled→cancelled |
| normal_test_items (child) | DiagnosticReport.result[] | Each row → Observation resource |
| — child.lab_test_event | Observation.code | LOINC code for the analyte |
| — child.result_value | Observation.valueQuantity | |
| — child.lab_test_uom | Observation.valueQuantity.unit | |
| — child.min_value / max_value | Observation.referenceRange | low/high |

---

## 6. Drug Prescription → FHIR MedicationRequest

| Frappe Field | FHIR Path | Notes |
|-------------|-----------|-------|
| drug_code | MedicationRequest.medicationCodeableConcept | RxNorm or local code |
| drug_name | MedicationRequest.medicationCodeableConcept.text | |
| dosage | MedicationRequest.dosageInstruction[0].doseAndRate | |
| period | MedicationRequest.dosageInstruction[0].timing.repeat.period | |
| dosage_form | MedicationRequest.dosageInstruction[0].route | |
| patient (from parent Encounter) | MedicationRequest.subject | Reference(Patient) |
| practitioner (from parent Encounter) | MedicationRequest.requester | Reference(Practitioner) |
| encounter (parent) | MedicationRequest.encounter | Reference(Encounter) |

---

## 7. Healthcare Practitioner → FHIR Practitioner

| Frappe Field | FHIR Path | Notes |
|-------------|-----------|-------|
| practitioner_name | Practitioner.name[0].text | |
| department | Practitioner.qualification[0].code | |
| designation | Practitioner.qualification[0].code.text | |
| mobile_phone | Practitioner.telecom[] | system=phone |
| image | Practitioner.photo[] | |

---

## Implementation Guidelines

1. **Translation layer**: Build a `my_ehealth/fhir/` module with one file per resource (patient.py, encounter.py, etc.) containing `to_fhir()` and `from_fhir()` functions.
2. **Validation**: Validate FHIR output against the R4 schema before sending. Use `fhir.resources` Python library or JSON Schema.
3. **Versioning**: Always include `meta.versionId` (use Frappe's `modified` timestamp) and `meta.lastUpdated`.
4. **Partial mappings**: Not every Frappe field maps to FHIR. Document unmapped fields explicitly and store FHIR extensions for fields that don't fit standard resources.
5. **Coding systems**: Use standard code systems — LOINC for observations, ICD-10 for diagnoses, RxNorm or local formulary for medications, SNOMED CT for procedures.
6. **Bundle operations**: For bulk sync, use FHIR Bundle (type=transaction) to send multiple resources atomically.
7. **Audit**: Log every inbound and outbound FHIR transaction with request ID, timestamp, resource type, and outcome.

