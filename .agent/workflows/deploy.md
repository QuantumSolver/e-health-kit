---
description: Deployment command for production releases. Pre-flight checks and deployment execution.
---

# /deploy - Production Deployment

$ARGUMENTS

---

## Purpose

This command handles production deployment with pre-flight checks, deployment execution, and verification.

---

## Sub-commands

```
/deploy            - Interactive deployment wizard
/deploy check      - Run pre-deployment checks only
/deploy preview    - Deploy to preview/staging
/deploy production - Deploy to production
/deploy rollback   - Rollback to previous version
```

---

## Pre-Deployment Checklist

Before any deployment:

```markdown
## 🚀 Pre-Deploy Checklist

### Code Quality
- [ ] No TypeScript errors (`npx tsc --noEmit`)
- [ ] ESLint passing (`npx eslint .`)
- [ ] All tests passing (`npm test`)

### Security
- [ ] No hardcoded secrets
- [ ] Environment variables documented
- [ ] Dependencies audited (`npm audit`)

### Performance
- [ ] Bundle size acceptable
- [ ] No console.log statements
- [ ] Images optimized

### Documentation
- [ ] README updated
- [ ] CHANGELOG updated
- [ ] API docs current

### Ready to deploy? (y/n)
```

---

## Deployment Flow

```
┌─────────────────┐
│  /deploy        │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Pre-flight     │
│  checks         │
└────────┬────────┘
         │
    Pass? ──No──► Fix issues
         │
        Yes
         │
         ▼
┌─────────────────┐
│  Build          │
│  application    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Deploy to      │
│  platform       │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Health check   │
│  & verify       │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  ✅ Complete    │
└─────────────────┘
```

---

## Output Format

### Successful Deploy

```markdown
## 🚀 Deployment Complete

### Summary
- **Version:** v1.2.3
- **Environment:** production
- **Duration:** 47 seconds
- **Platform:** Vercel

### URLs
- 🌐 Production: https://app.example.com
- 📊 Dashboard: https://vercel.com/project

### What Changed
- Added user profile feature
- Fixed login bug
- Updated dependencies

### Health Check
✅ API responding (200 OK)
✅ Database connected
✅ All services healthy
```

### Failed Deploy

```markdown
## ❌ Deployment Failed

### Error
Build failed at step: TypeScript compilation

### Details
```
error TS2345: Argument of type 'string' is not assignable...
```

### Resolution
1. Fix TypeScript error in `src/services/user.ts:45`
2. Run `npm run build` locally to verify
3. Try `/deploy` again

### Rollback Available
Previous version (v1.2.2) is still active.
Run `/deploy rollback` if needed.
```

---

## Platform Support

| Platform | Command | Notes |
|----------|---------|-------|
| Vercel | `vercel --prod` | Auto-detected for Next.js |
| Railway | `railway up` | Needs Railway CLI |
| Fly.io | `fly deploy` | Needs flyctl |
| Docker | `docker compose up -d` | For self-hosted |
| Frappe/Bench | `bench update --pull --reset` | See Frappe section below |
| frappe_docker | `docker compose up -d` | Official Frappe Docker |

---

## Frappe / Bench Deployment

### Pre-Deploy (Frappe)

```markdown
## 🩺 Frappe Pre-Deploy Checklist

### Code Quality
- [ ] Tests passing (`bench run-tests --app my_ehealth`)
- [ ] `bench build` succeeds
- [ ] DocType JSON changes committed

### Data Safety
- [ ] Backup taken (`bench backup --with-files`)
- [ ] Rollback plan documented

### Healthcare Compliance
- [ ] No PHI in logs or error messages
- [ ] TLS configured for production site
- [ ] Audit trail enabled on clinical DocTypes

### Ready to deploy? (y/n)
```

### Deploy Flow (Frappe)

```bash
# 1. Backup
bench --site mysite.localhost backup --with-files

# 2. Pull + migrate
bench update --pull --reset
# Or: cd apps/my_ehealth && git pull && cd ../..
bench --site mysite.localhost migrate

# 3. Build + clear
bench build --app my_ehealth
bench --site mysite.localhost clear-cache

# 4. Restart
sudo supervisorctl restart all

# 5. Verify
bench doctor
curl -I https://mysite.com
```

### Rollback (Frappe)

```bash
cd apps/my_ehealth && git checkout <previous_tag> && cd ../..
bench --site mysite.localhost restore /path/to/backup.sql.gz
bench --site mysite.localhost migrate
bench build
sudo supervisorctl restart all
```

---

## Examples

```
/deploy
/deploy check
/deploy preview
/deploy production --skip-tests
/deploy rollback
```
