---
name: deployment-procedures
description: Production deployment principles and decision-making. Safe deployment workflows, rollback strategies, and verification. Teaches thinking, not scripts.
allowed-tools: Read, Glob, Grep, Bash
---

# Deployment Procedures

> Deployment principles and decision-making for safe production releases.
> **Learn to THINK, not memorize scripts.**

---

## ⚠️ How to Use This Skill

This skill teaches **deployment principles**, not bash scripts to copy.

- Every deployment is unique
- Understand the WHY behind each step
- Adapt procedures to your platform

---

## 1. Platform Selection

### Decision Tree

```
What are you deploying?
│
├── Static site / JAMstack
│   └── Vercel, Netlify, Cloudflare Pages
│
├── Simple web app
│   ├── Managed → Railway, Render, Fly.io
│   └── Control → VPS + PM2/Docker
│
├── Microservices
│   └── Container orchestration
│
└── Serverless
    └── Edge functions, Lambda
```

### Each Platform Has Different Procedures

| Platform | Deployment Method |
|----------|------------------|
| **Vercel/Netlify** | Git push, auto-deploy |
| **Railway/Render** | Git push or CLI |
| **VPS + PM2** | SSH + manual steps |
| **Docker** | Image push + orchestration |
| **Kubernetes** | kubectl apply |

---

## 2. Pre-Deployment Principles

### The 4 Verification Categories

| Category | What to Check |
|----------|--------------|
| **Code Quality** | Tests passing, linting clean, reviewed |
| **Build** | Production build works, no warnings |
| **Environment** | Env vars set, secrets current |
| **Safety** | Backup done, rollback plan ready |

### Pre-Deployment Checklist

- [ ] All tests passing
- [ ] Code reviewed and approved
- [ ] Production build successful
- [ ] Environment variables verified
- [ ] Database migrations ready (if any)
- [ ] Rollback plan documented
- [ ] Team notified
- [ ] Monitoring ready

---

## 3. Deployment Workflow Principles

### The 5-Phase Process

```
1. PREPARE
   └── Verify code, build, env vars

2. BACKUP
   └── Save current state before changing

3. DEPLOY
   └── Execute with monitoring open

4. VERIFY
   └── Health check, logs, key flows

5. CONFIRM or ROLLBACK
   └── All good? Confirm. Issues? Rollback.
```

### Phase Principles

| Phase | Principle |
|-------|-----------|
| **Prepare** | Never deploy untested code |
| **Backup** | Can't rollback without backup |
| **Deploy** | Watch it happen, don't walk away |
| **Verify** | Trust but verify |
| **Confirm** | Have rollback trigger ready |

---

## 4. Post-Deployment Verification

### What to Verify

| Check | Why |
|-------|-----|
| **Health endpoint** | Service is running |
| **Error logs** | No new errors |
| **Key user flows** | Critical features work |
| **Performance** | Response times acceptable |

### Verification Window

- **First 5 minutes**: Active monitoring
- **15 minutes**: Confirm stable
- **1 hour**: Final verification
- **Next day**: Review metrics

---

## 5. Rollback Principles

### When to Rollback

| Symptom | Action |
|---------|--------|
| Service down | Rollback immediately |
| Critical errors | Rollback |
| Performance >50% degraded | Consider rollback |
| Minor issues | Fix forward if quick |

### Rollback Strategy by Platform

| Platform | Rollback Method |
|----------|----------------|
| **Vercel/Netlify** | Redeploy previous commit |
| **Railway/Render** | Rollback in dashboard |
| **VPS + PM2** | Restore backup, restart |
| **Docker** | Previous image tag |
| **K8s** | kubectl rollout undo |

### Rollback Principles

1. **Speed over perfection**: Rollback first, debug later
2. **Don't compound errors**: One rollback, not multiple changes
3. **Communicate**: Tell team what happened
4. **Post-mortem**: Understand why after stable

---

## 6. Zero-Downtime Deployment

### Strategies

| Strategy | How It Works |
|----------|--------------|
| **Rolling** | Replace instances one by one |
| **Blue-Green** | Switch traffic between environments |
| **Canary** | Gradual traffic shift |

### Selection Principles

| Scenario | Strategy |
|----------|----------|
| Standard release | Rolling |
| High-risk change | Blue-green (easy rollback) |
| Need validation | Canary (test with real traffic) |

---

## 7. Emergency Procedures

### Service Down Priority

1. **Assess**: What's the symptom?
2. **Quick fix**: Restart if unclear
3. **Rollback**: If restart doesn't help
4. **Investigate**: After stable

### Investigation Order

| Check | Common Issues |
|-------|--------------|
| **Logs** | Errors, exceptions |
| **Resources** | Disk full, memory |
| **Network** | DNS, firewall |
| **Dependencies** | Database, APIs |

---

## 8. Anti-Patterns

| ❌ Don't | ✅ Do |
|----------|-------|
| Deploy on Friday | Deploy early in week |
| Rush deployment | Follow the process |
| Skip staging | Always test first |
| Deploy without backup | Backup before deploy |
| Walk away after deploy | Monitor for 15+ min |
| Multiple changes at once | One change at a time |

---

## 9. Decision Checklist

Before deploying:

- [ ] **Platform-appropriate procedure?**
- [ ] **Backup strategy ready?**
- [ ] **Rollback plan documented?**
- [ ] **Monitoring configured?**
- [ ] **Team notified?**
- [ ] **Time to monitor after?**

---

## 10. Best Practices

1. **Small, frequent deploys** over big releases
2. **Feature flags** for risky changes
3. **Automate** repetitive steps
4. **Document** every deployment
5. **Review** what went wrong after issues
6. **Test rollback** before you need it

---

---

## 11. Frappe / Bench Deployment

### Platform Selection for Frappe

| Scenario | Recommendation |
|----------|---------------|
| Development | `bench start` (Werkzeug + Redis + Scheduler) |
| Single-server production | Bench + Supervisor + Nginx |
| Containerised | Official frappe_docker (Docker Compose) |
| Multi-tenant | DNS multitenant with Nginx routing |
| Cloud managed | Frappe Cloud (managed hosting) |

### Production Setup (Single Server)

```bash
# 1. Setup production config (creates Supervisor + Nginx configs)
sudo bench setup production <linux_user>

# 2. Generate and enable Nginx config
bench setup nginx
sudo ln -s /etc/nginx/conf.d/frappe-bench.conf /etc/nginx/sites-enabled/
sudo nginx -t && sudo service nginx reload

# 3. Enable scheduler
bench enable-scheduler

# 4. Restart everything
sudo supervisorctl restart all
```

### Deployment Workflow (Code Update)

```bash
# 1. Backup before deploy
bench --site mysite.localhost backup --with-files

# 2. Pull latest code
bench update --pull --reset

# 3. Or for specific app only
cd apps/my_ehealth && git pull origin main && cd ../..

# 4. Run migrations
bench --site mysite.localhost migrate

# 5. Build assets
bench build --app my_ehealth

# 6. Clear cache
bench --site mysite.localhost clear-cache

# 7. Restart
sudo supervisorctl restart all
```

### Docker Deployment (frappe_docker)

```bash
# Clone official Docker setup
git clone https://github.com/frappe/frappe_docker.git
cd frappe_docker

# Production with custom app
export CUSTOM_APP_REPO=https://github.com/org/my_ehealth.git
docker compose -f compose.yaml up -d
```

### Pre-Deployment Checklist (Frappe)

- [ ] `bench --site mysite.localhost run-tests --app my_ehealth` passes
- [ ] `bench build` succeeds without errors
- [ ] Database backup taken (`bench backup --with-files`)
- [ ] DocType JSON changes committed (no uncommitted schema changes)
- [ ] `hooks.py` changes reviewed (scheduler_events, doc_events)
- [ ] Custom Fields exported to fixtures
- [ ] Environment config (site_config.json) has no secrets in Git
- [ ] Redis and MariaDB/PostgreSQL services healthy

### Post-Deployment Verification (Frappe)

| Check | Command / Action |
|-------|-----------------|
| Site loads | `curl -I https://mysite.com` |
| Scheduler running | `bench doctor` |
| Workers alive | `sudo supervisorctl status` |
| No migration errors | Check `bench migrate` output |
| Error log clean | Frappe Desk → Error Log |
| Background jobs | Frappe Desk → Background Jobs |

### Rollback (Frappe)

```bash
# 1. Restore code
cd apps/my_ehealth && git checkout <previous_tag> && cd ../..

# 2. Restore database
bench --site mysite.localhost restore /path/to/backup.sql.gz

# 3. Migrate and rebuild
bench --site mysite.localhost migrate
bench build
sudo supervisorctl restart all
```

### Multi-Tenant Deployment

```bash
bench config dns_multitenant on
bench new-site tenant2.example.com
bench --site tenant2.example.com install-app my_ehealth
bench setup nginx
sudo service nginx reload
```

### Healthcare-Specific Deployment Notes

- Ensure TLS 1.2+ is configured in Nginx for all sites handling PHI.
- Set `session_expiry` in site_config.json (e.g., "04:00:00" for 4 hours, or shorter for clinical workstations).
- Enable `deny_multiple_sessions` if required by compliance policy.
- Verify audit trail (Version DocType) is enabled for all healthcare DocTypes before go-live.
- Test backup restore procedure with PHI data to confirm encryption and integrity.

> **Remember:** Every deployment is a risk. Minimize risk through preparation, not speed.
