# Tech Stack Selection (2026)

> Default and alternative technology choices for web applications.

## Healthcare/E-Health Stack (Frappe Framework)

```yaml
Runtime:
  framework: Frappe Framework 15+
  language: Python 3.11+
  cli: Bench CLI
Database:
  primary: MariaDB (default) or PostgreSQL
  modeling: DocTypes (metadata-driven; migrations managed by Frappe)
Frontend:
  ui: Frappe Desk (forms, lists, reports, dashboards)
  components: Frappe UI (Vue 3) for richer widgets
Domain:
  modules: ERPNext Healthcare or custom Frappe healthcare app
  patterns: Patient, Encounter, Appointment, Lab Test, Prescription DocTypes
Security:
  compliance: Use healthcare-compliance skill (HIPAA-style guardrails)
  authz: Role-based DocType permissions + User Permissions
```

## Default Stack (General Web App - 2026)

```yaml
Frontend:
  framework: Next.js 16 (Stable)
  language: TypeScript 5.7+
  styling: Tailwind CSS v4
  state: React 19 Actions / Server Components
  bundler: Turbopack (Stable for Dev)

Backend:
  runtime: Node.js 23
  framework: Next.js API Routes / Hono (for Edge)
  validation: Zod / TypeBox

Database:
  primary: PostgreSQL
  orm: Prisma / Drizzle
  hosting: Supabase / Neon

Auth:
  provider: Auth.js (v5) / Clerk

Monorepo:
  tool: Turborepo 2.0
```

## Alternative Options

| Need | Default | Alternative |
|------|---------|-------------|
| Healthcare/EHR app | Frappe Framework + Bench | Next.js general web app stack |
| Real-time | - | Supabase Realtime, Socket.io |
| File storage | - | Cloudinary, S3 |
| Payment | Stripe | LemonSqueezy, Paddle |
| Email | - | Resend, SendGrid |
| Search | - | Algolia, Typesense |
