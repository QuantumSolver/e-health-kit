---
description: Frappe Bench operations. Site management, migrations, builds, cache, workers, and production setup. Use when working on Frappe or ERPNext projects.
---

# /bench - Frappe Bench Operations

$ARGUMENTS

---

## Sub-commands

```
/bench                  - Show available bench operations
/bench setup            - Initialize bench, create site, install app
/bench migrate          - Run migrations after DocType or schema changes
/bench build            - Rebuild JS/CSS assets
/bench restart          - Restart workers and web server
/bench clear            - Clear cache and restart
/bench test             - Run Frappe test suite
/bench console          - Open Frappe Python console for site
/bench deploy           - Full production deployment sequence
/bench new-doctype      - Scaffold a new DocType
/bench backup           - Backup site database and files
/bench restore          - Restore site from backup
```

---

## Common Operations

### Setup New Project

```bash
bench init frappe-bench --frappe-path https://github.com/frappe/frappe
cd frappe-bench
bench new-site mysite.localhost --db-type mariadb --admin-password admin
bench new-app my_ehealth
bench --site mysite.localhost install-app my_ehealth
bench start
```

### After DocType / Schema Changes

```bash
bench --site mysite.localhost migrate
bench build
bench --site mysite.localhost clear-cache
```

### After Python Code Changes

```bash
# Dev mode: auto-reloads, but if stuck:
bench restart
```

### After JS / CSS Changes

```bash
bench build --app my_ehealth
# Or for dev with live reload:
bench watch
```

### Cache Issues

```bash
bench --site mysite.localhost clear-cache
bench --site mysite.localhost clear-website-cache
```

### Run Tests

```bash
# All tests for an app
bench --site mysite.localhost run-tests --app my_ehealth

# Specific DocType
bench --site mysite.localhost run-tests --doctype "Patient"

# Specific test file
bench --site mysite.localhost run-tests --module my_ehealth.my_ehealth.doctype.patient.test_patient

# With verbose output
bench --site mysite.localhost run-tests --app my_ehealth -v
```

### Console / Debug

```bash
bench --site mysite.localhost console
# Inside console:
# frappe.get_doc("Patient", "PAT-00001")
# frappe.db.sql("SELECT name FROM tabPatient LIMIT 5")

bench --site mysite.localhost mariadb   # Direct DB shell
```

### Backup & Restore

```bash
bench --site mysite.localhost backup --with-files
bench --site mysite.localhost restore /path/to/backup.sql.gz
```

### Production Setup

```bash
# Setup Supervisor + Nginx (Linux only)
sudo bench setup production <user>
bench setup nginx
sudo supervisorctl restart all
sudo service nginx reload

# Or with Docker
# Use official frappe_docker repo
```

### Multi-Tenancy

```bash
bench config dns_multitenant on
bench new-site site2.localhost
bench --site site2.localhost install-app my_ehealth
bench setup nginx
sudo service nginx reload
```

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| Assets not loading | `bench build && bench clear-cache` |
| Migration errors | Check DocType JSON for conflicts, `bench migrate --skip-failing` |
| Workers not starting | `bench restart` or check Supervisor logs |
| Permission denied | Check file ownership: `bench setup sudoers` |
| Redis connection error | Ensure Redis is running: `redis-cli ping` |
| Site not found | `bench use mysite.localhost` or pass `--site` flag |
| Scheduler not running | `bench enable-scheduler` then `bench restart` |

---

## Healthcare-Specific Operations

```bash
# Install ERPNext + Healthcare (if using ERPNext base)
bench get-app erpnext
bench get-app healthcare
bench --site mysite.localhost install-app erpnext
bench --site mysite.localhost install-app healthcare

# After adding Custom Fields to Healthcare DocTypes
bench --site mysite.localhost migrate
bench --site mysite.localhost clear-cache
```

---

## Examples

```
/bench setup
/bench migrate
/bench test
/bench deploy
/bench clear
/bench backup
```

