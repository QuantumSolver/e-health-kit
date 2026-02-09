---
name: testing-patterns
description: Testing patterns and principles. Unit, integration, mocking strategies.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# Testing Patterns

> Principles for reliable test suites.

---

## 1. Testing Pyramid

```
        /\          E2E (Few)
       /  \         Critical flows
      /----\
     /      \       Integration (Some)
    /--------\      API, DB queries
   /          \
  /------------\    Unit (Many)
                    Functions, classes
```

---

## 2. AAA Pattern

| Step | Purpose |
|------|---------|
| **Arrange** | Set up test data |
| **Act** | Execute code under test |
| **Assert** | Verify outcome |

---

## 3. Test Type Selection

### When to Use Each

| Type | Best For | Speed |
|------|----------|-------|
| **Unit** | Pure functions, logic | Fast (<50ms) |
| **Integration** | API, DB, services | Medium |
| **E2E** | Critical user flows | Slow |

---

## 4. Unit Test Principles

### Good Unit Tests

| Principle | Meaning |
|-----------|---------|
| Fast | < 100ms each |
| Isolated | No external deps |
| Repeatable | Same result always |
| Self-checking | No manual verification |
| Timely | Written with code |

### What to Unit Test

| Test | Don't Test |
|------|------------|
| Business logic | Framework code |
| Edge cases | Third-party libs |
| Error handling | Simple getters |

---

## 5. Integration Test Principles

### What to Test

| Area | Focus |
|------|-------|
| API endpoints | Request/response |
| Database | Queries, transactions |
| External services | Contracts |

### Setup/Teardown

| Phase | Action |
|-------|--------|
| Before All | Connect resources |
| Before Each | Reset state |
| After Each | Clean up |
| After All | Disconnect |

---

## 6. Mocking Principles

### When to Mock

| Mock | Don't Mock |
|------|------------|
| External APIs | The code under test |
| Database (unit) | Simple dependencies |
| Time/random | Pure functions |
| Network | In-memory stores |

### Mock Types

| Type | Use |
|------|-----|
| Stub | Return fixed values |
| Spy | Track calls |
| Mock | Set expectations |
| Fake | Simplified implementation |

---

## 7. Test Organization

### Naming

| Pattern | Example |
|---------|---------|
| Should behavior | "should return error when..." |
| When condition | "when user not found..." |
| Given-when-then | "given X, when Y, then Z" |

### Grouping

| Level | Use |
|-------|-----|
| describe | Group related tests |
| it/test | Individual case |
| beforeEach | Common setup |

---

## 8. Test Data

### Strategies

| Approach | Use |
|----------|-----|
| Factories | Generate test data |
| Fixtures | Predefined datasets |
| Builders | Fluent object creation |

### Principles

- Use realistic data
- Randomize non-essential values (faker)
- Share common fixtures
- Keep data minimal

---

## 9. Best Practices

| Practice | Why |
|----------|-----|
| One assert per test | Clear failure reason |
| Independent tests | No order dependency |
| Fast tests | Run frequently |
| Descriptive names | Self-documenting |
| Clean up | Avoid side effects |

---

## 10. Anti-Patterns

| ❌ Don't | ✅ Do |
|----------|-------|
| Test implementation | Test behavior |
| Duplicate test code | Use factories |
| Complex test setup | Simplify or split |
| Ignore flaky tests | Fix root cause |
| Skip cleanup | Reset state |

---

---

## 11. Frappe Test Patterns

### Test Runner

Frappe uses Python `unittest` with its own test runner via Bench CLI:

```bash
# All tests for an app
bench --site mysite.localhost run-tests --app my_ehealth

# Single DocType
bench --site mysite.localhost run-tests --doctype "Patient"

# Single module
bench --site mysite.localhost run-tests --module my_ehealth.my_ehealth.doctype.patient.test_patient

# Verbose
bench --site mysite.localhost run-tests --app my_ehealth -v
```

### Test File Location

Each DocType has a test file at:
```
my_ehealth/my_ehealth/doctype/<doctype_name>/test_<doctype_name>.py
```

### Frappe Test Utilities

| Utility | Purpose |
|---------|---------|
| `frappe.tests.utils.FrappeTestCase` | Base class — handles setup, teardown, DB rollback |
| `frappe.get_doc(...)` | Create/fetch documents in tests |
| `frappe.mock("method")` | Patch Frappe methods |
| `self.assertRaises(...)` | Assert validation errors |
| `frappe.set_user("user@example.com")` | Test as specific user/role |
| `frappe.flags.in_test = True` | Auto-set by runner; skip side effects |

### Example: DocType Unit Test

```python
import frappe
from frappe.tests.utils import FrappeTestCase

class TestPatient(FrappeTestCase):
    def setUp(self):
        self.patient = frappe.get_doc({
            "doctype": "Patient",
            "first_name": "Test",
            "last_name": "Patient",
            "sex": "Male",
            "dob": "1990-01-01"
        }).insert(ignore_permissions=True)

    def test_patient_creation(self):
        self.assertTrue(self.patient.name)
        self.assertEqual(self.patient.patient_name, "Test Patient")

    def test_duplicate_uid_rejected(self):
        self.patient.uid = "ID-12345"
        self.patient.save()
        duplicate = frappe.get_doc({
            "doctype": "Patient",
            "first_name": "Dup",
            "sex": "Female",
            "uid": "ID-12345"
        })
        self.assertRaises(frappe.DuplicateEntryError, duplicate.insert)

    def test_role_based_access(self):
        frappe.set_user("receptionist@example.com")
        # Receptionist can read Patient
        patient = frappe.get_doc("Patient", self.patient.name)
        self.assertTrue(patient.name)

    def tearDown(self):
        frappe.set_user("Administrator")
        frappe.delete_doc("Patient", self.patient.name, force=True)
```

### Testing Workflows and Submissions

```python
def test_encounter_submit_creates_medical_record(self):
    encounter = frappe.get_doc({
        "doctype": "Patient Encounter",
        "patient": self.patient.name,
        "practitioner": self.practitioner.name,
        "encounter_date": frappe.utils.today()
    }).insert()
    encounter.submit()
    # Verify downstream effect
    records = frappe.get_all("Medical Record", filters={"reference_name": encounter.name})
    self.assertTrue(len(records) > 0)
```

### Testing Whitelisted APIs

```python
def test_whitelist_api(self):
    from my_ehealth.api.encounter import get_patient_history
    frappe.set_user("doctor@example.com")
    result = get_patient_history(patient=self.patient.name)
    self.assertIsInstance(result, list)
```

### Frappe Test Best Practices

| Practice | Why |
|----------|-----|
| Inherit `FrappeTestCase` | Auto DB rollback per test |
| Use `ignore_permissions=True` in setUp | Isolate permission tests from data setup |
| Call `frappe.set_user("Administrator")` in tearDown | Reset user context |
| Never hardcode document names | Use variables from setUp |
| Test controller hooks (validate, on_submit) | These contain business rules |
| Test RBAC with `frappe.set_user()` | Verify role-based access |
| Avoid testing Frappe core | Only test your app logic |

> **Remember:** Tests are documentation. If someone can't understand what the code does from the tests, rewrite them.
