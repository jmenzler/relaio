<div align="center">

# Relaio

**After-sales email for insurance brokers, approved by the broker before anything is sent.**

![case study: code private](https://img.shields.io/badge/case%20study-code%20private-6e7781)
![Next.js](https://img.shields.io/badge/Next.js-000000?logo=nextdotjs&logoColor=white)
![Vercel](https://img.shields.io/badge/Vercel-000000?logo=vercel&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?logo=fastapi&logoColor=white)
![Supabase: RLS](https://img.shields.io/badge/Supabase-RLS-3FCF8E?logo=supabase&logoColor=white)
![Claude: Haiku](https://img.shields.io/badge/Claude-Haiku-D97757?logo=claude&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)

[The problem](#the-problem) · [Design decisions](#design-decisions) · [Architecture](#architecture) · [Features](#features) · [Stack](#stack)

</div>

After-sales email automation for German insurance brokers: birthday and holiday mails, service reminders, referral and review requests and multi-step sequences, with consent tracking built in and every email approved by the broker before it goes out.

Solo project: frontend, backend, database and deployment.

> [!NOTE]
> The implementation is private. This repository is a case study: design decisions, architecture and screenshots.

## The problem

Solo brokers and small firms look after hundreds of existing customers. Staying in touch, asking for referrals and collecting reviews keeps those customers, but by hand it takes hours every week, so most brokers skip it. In Germany every automated email also needs provable consent.

## Design decisions

| Decision | Why |
|---|---|
| **Tenant isolation in the database.** Supabase row-level security scopes every query by `tenant_id` from the JWT's `app_metadata`; the backend also verifies the JWT | A backend bug cannot leak one broker's customers to another |
| **AI kept deliberately narrow.** One Claude Haiku call per email rewrites the broker's template with customer context and tone (formal "Sie" or informal "Du"); if the call fails, the original template is sent | The model personalises text and decides nothing else |
| **Broker approval before dispatch.** Emails are generated about a week ahead and wait in a review queue (approve, reject, edit, bulk actions) | The broker stays responsible for what is sent in their name |
| **Compliance as design.** Double opt-in with an audit trail, one-click unsubscribe, blacklist enforcement, EU-region database | Consent is checked before every send, not bolted on |
| **Delivery state from webhooks.** Resend events verified with Svix update each email's status | Sent, delivered, opened, clicked and bounced come from the provider, not guesses |
| **Sequences as data.** Send, wait, condition, follow-up steps with per-customer enrollment and pause/resume | Brokers build flows without code |

<p align="center">
  <img src="docs/screenshots/dashboard-screenshot.png" alt="Relaio dashboard" width="700" />
</p>

## Architecture

```mermaid
flowchart LR
    web["Next.js frontend<br/>Vercel"] -- "REST + JWT" --> api["FastAPI backend<br/>Docker + Caddy"]
    api --> db[("Supabase Postgres<br/>row-level security")]
    api --> ai["Claude Haiku<br/>personalization"]
    api --> mail["Resend"]
    mail -- "Svix webhooks" --> api
```

## Features

- **Automation:** triggers for birthdays, holidays, welcome mails, service appointments, referral and review requests.
- **Contacts:** CSV/Excel import with encoding detection and column mapping, tags, notes, consent status.
- **Campaigns:** one-off sends targeted by tags, life events or consent status.
- **Public pages:** opt-in confirmation, one-click unsubscribe, referral form, life-event survey.

<details>
<summary>More screenshots</summary>
<br />
<img src="docs/screenshots/contacts-screenshot.png" alt="Contacts table with consent status, tags and filters" width="700" />
</details>

## Stack

| Layer | Technology | Why |
|---|---|---|
| Frontend | Next.js, React, Tailwind, shadcn/ui | Fast dashboard development |
| Backend | FastAPI, Pydantic v2 | Typed API and validation |
| Database | Supabase Postgres with RLS | Tenant isolation enforced below the application |
| Email | Resend, Svix | Delivery webhooks with signature verification |
| AI | Claude Haiku | One narrow rewrite per email, not an agent |
| Hosting | Vercel (frontend), Docker + Caddy (backend) | Managed Next.js hosting; backend container with automatic TLS |
| CI | GitHub Actions: ruff, pytest, eslint, tsc, build | Lint, types and tests on every change |

---

<p align="center">
  <sub>Built by <a href="https://github.com/jmenzler">Jannis Menzler</a></sub>
</p>
