# DEVSMACHINA — Technical Documentation (MVP)

**Version:** 1.0  
**Date:** March 2026  
**Classification:** Internal / Confidential

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [System Architecture](#2-system-architecture)
3. [Agent System — MVP Features](#3-agent-system--mvp-features)
4. [Agent Comparison Overview](#4-agent-comparison-overview)
5. [Integrations & Connectivity](#5-integrations--connectivity)
6. [Reporting System](#6-reporting-system)
7. [Data Model](#7-data-model)
8. [Security & Compliance](#8-security--compliance)
9. [Scalability & Infrastructure](#9-scalability--infrastructure)
10. [Development Phases (Roadmap)](#10-development-phases-roadmap)
11. [Open Questions & Decisions](#11-open-questions--decisions)
12. [Out of Scope for MVP](#12-out-of-scope-for-mvp)
13. [CI/CD & DevOps](#13-cicd--devops)
14. [Glossary](#14-glossary)

---

## 1. Executive Summary

DevsMachina is a SaaS e-commerce intelligence platform that uses AI agents to automatically monitor performance, competition, and the operational health of online stores. The platform replaces manual analytics processes with intelligent, automated agents that generate concrete, actionable recommendations.

Target users are owners and teams of e-commerce stores (primarily WooCommerce, with Shopify planned for the future) who want to make data-driven decisions without the need for in-house data analysts.

**Key platform values:**

- **Automated monitoring** — SEO, CRO, operations, competitor pricing, inventory
- **Actionable reports** — not just numbers, but specific instructions on what to change and how
- **Eisenhower Matrix prioritization** — every recommendation is categorized by urgency and importance
- **Modular agent system** — users activate only the agents they need
- **Multi-tenant architecture** — strict data isolation between users

---

## 2. System Architecture

### 2.1 Architectural Principles

The system is designed according to the following key principles that ensure scalability, reliability, and maintainability:

| Principle | Description | Implementation |
|-----------|-------------|----------------|
| Multi-tenant isolation | Strict data separation per user; never data leakage | Row-level security in PostgreSQL, tenant_id on every table |
| Phased scalability | 10 → 100 → 500 → 1000+ users without rewriting | Stateless services, horizontally scalable workers |
| LLM provider agnosticism | Architecture supports multiple AI providers with fallback logic | Abstract LLM layer with adapter pattern |
| Graceful degradation | External service failure pauses the agent, doesn't crash the system | Circuit breaker pattern, health check monitoring |
| Adapter pattern | Change in 3rd party API = change in adapter, not core code | Wrapper classes for all external integrations |
| Modular agents | Adding a new agent doesn't touch the existing platform | Plugin architecture with a standardized interface |

### 2.2 Technology Stack

| Layer | Technology | Rationale |
|-------|------------|-----------|
| Frontend | Next.js (React) + TypeScript | SSR/SSG, type safety, component ecosystem |
| Backend API | Node.js + Express/Fastify | Rapid development, shared TypeScript types with frontend |
| Database | PostgreSQL (Supabase/NeonDB) | ACID compliance, RLS, JSONB for flexible schemas |
| ORM | Prisma | Type-safe queries, automatic migrations, introspection |
| Job Queue | BullMQ (Redis) | Priority queues, retry logic, cron scheduling |
| Authentication | Supabase Auth / NextAuth | OAuth 2.0, JWT, multi-provider support |
| LLM Layer | Provider-agnostic wrapper | OpenAI, Anthropic, Gemini with fallback chain |
| Scraping | Playwright / Puppeteer | Headless browser for dynamic sites |
| Email/Notifications | Resend / SendGrid | Transactional emails, template engine |
| Hosting | Vercel (frontend) + Railway/Fly.io (backend) | Auto-scaling, edge deployment |
| CI/CD | GitHub Actions | Automated testing, staging, zero-downtime deploy |
| Monitoring | Sentry + Axiom/Betterstack | Error tracking, structured logs, uptime |

### 2.3 System Diagram — High Level

The system consists of four main layers communicating through clearly defined interfaces:

| Layer | Components | Responsibilities |
|-------|------------|------------------|
| Presentation Layer | Next.js App, Dashboard UI | User interface, onboarding wizard, agent configuration, report viewing |
| API Layer | REST/GraphQL API Gateway | Authentication, authorization, rate limiting, request validation, routing |
| Business Logic Layer | Agent Orchestrator, Job Queue, LLM Service | Agent creation and management, scheduling, AI processing, reporting |
| Data & Integration Layer | PostgreSQL, Redis, External APIs | Persistence, caching, GA4/GSC/WooCommerce connectors, scraping engine |

Communication between layers is exclusively top-down for requests (request flow), while responses (response flow) and notifications propagate upward. Agent workers are stateless and communicate with the Business Logic layer exclusively through the job queue.

---

## 3. Agent System — MVP Features

The MVP version includes seven AI agents/tools. Each agent has a standardized interface, an independent lifecycle, and its own configuration.

### 3.1 Operations Report Agent

**Type:** Scheduled reporting + On-demand

Synthesizes GA4, ad spend, and revenue data into a weekly/monthly digest with variance analysis and trend alerts. The report is structured to be immediately actionable for marketing, web, and sales teams.

**Input data:**
- GA4 Analytics (sessions, conversions, revenue, channels, campaigns)
- Ad spend data (if available in GA4)
- Historical data for comparison (previous period, YoY)

**Output:**
- Structured HTML/email report with key KPIs
- Actionable recommendations prioritized by Eisenhower Matrix logic
- Specific instructions: "Increase ad budget for Product X because of reason Y", "Offer a larger discount to segment X"
- BI/Data-analyst mindset in deciding what is worth acting on

**Prototype:** Local n8n workflow that saved/sent HTML reports. Migration to a platform agent with the same logic but scalable infrastructure.

### 3.2 SEO Monitoring Agent

**Type:** Scheduled reporting + On-demand

Tracks key SEO indicators based on GA4 data (and optionally GSC for MVP). Generates a report with specific instructions on what to change and how, prioritized by Eisenhower Matrix.

**Input data:**
- GA4 organic traffic, landing page performance
- GSC data (optional in MVP): impressions, clicks, positions, crawl errors
- Internal SEO knowledge base and directives

**Output:**
- List of specific SEO actions: "META data on page X is poor/missing", "Pages that shouldn't be indexed are being indexed"
- Canonical tag checks, crawl budget optimization, indexation status
- Prioritized recommendations based on proven SEO practices

**Note:** SEO directives are defined based on known best practices from reliable sources, stored AI SEO skills, and internal guides (SEO & CONTENT Guide, website launch/migration tech SEO to-do list).

### 3.3 CRO Monitoring Agent

**Type:** Scheduled reporting + On-demand

Structure and logic identical to the SEO agent, but focused on Conversion Rate Optimization parameters and user behavior on the site.

**Focus areas:**
- CTR optimization by pages and elements
- Engagement rate analysis (scroll depth, time on page, bounce rate)
- Checkout flow analysis — exit points, cart abandonment
- Specific instructions for improving conversions

**Prototype:** No local prototype was created, but the same infrastructure as the SEO agent is used with an adapted prompt and input data.

### 3.4 Pricing Intelligence Agent

**Type:** Trigger-based reporting + On-demand

Analyzes one or more competitor websites and reports on price changes, product additions/removals, and active promotions.

**Operating logic:**
1. At a defined interval (e.g., once daily) checks for changes
2. If changes occurred → sends a structured report
3. If no changes → saves info that the last state is unchanged
4. On-demand mode: extracts a product list with basic info (no historical data for comparison)

**Prototype:** n8n workflow with a scraping URL in the configuration. Change reports sent via email and saved locally + Google Drive.

### 3.5 Inventory Signals Agent

**Type:** Trigger-based reporting (scheduled only)

Monitors stock level status and changes, demand and sales, and sends alerts/reports. Detects low stock risks, seasonal demand shifts, and supplier lead time mismatches before they become stockouts.

**MVP integrations:**
- WP/WooCommerce — via REST API or Application Key
- Shopify — only if integration is not complex (optional for MVP)

**Note:** ERP/CRM integrations (for advanced inventory tracking) are planned for post-MVP phases. In MVP, the focus is exclusively on data available through the WooCommerce API.

### 3.6 Landing Page In-Depth Auditor

**Type:** On-demand only

A tool that enables the user to perform a comprehensive audit of any page: tech SEO, content SEO, Core Web Vitals, and CRO analysis.

**User flow:**
1. User enters the URL for analysis
2. Selects which audits they want (all or selectively): Tech SEO, Content SEO, CWV/PSI, CRO
3. System performs the analysis and generates a report with clear recommendations
4. Report contains specific instructions on what to fix and how

**Prototype:** n8n workflow with a popup for URL input. After execution, it displayed a results screen, saved HTML locally, and offered a download option.

### 3.7 Localization Coverage Auditor

**Type:** On-demand only

Checks whether a multilingual site (MVP: WordPress/WooCommerce only) has translations for all pages in all configured languages, and whether the content on those pages is actually translated (not a copy of the default language).

**What it checks:**
- Whether all pages have translation instances for each configured language
- Whether the content on translated pages is actually different from the default language
- Identifies gaps in localization coverage

**Prototype:** Python script that ran on-demand and saved reports as HTML/JSON/CSV.

**Note:** Continuous translation monitoring (real-time) is not in scope for MVP as the benefit doesn't justify the complexity at this stage.

---

## 4. Agent Comparison Overview

| Agent | Type | Data Sources | Prototype |
|-------|------|--------------|-----------|
| Operations Report | Scheduled + On-demand | GA4, ad spend, revenue | n8n workflow (HTML) |
| SEO Monitoring | Scheduled + On-demand | GA4, GSC (optional) | n8n workflow (HTML) |
| CRO Monitoring | Scheduled + On-demand | GA4, user behavior | None (based on SEO) |
| Pricing Intelligence | Trigger + On-demand | Web scraping competitors | n8n workflow (email + GDrive) |
| Inventory Signals | Trigger-based | WooCommerce API | None |
| Landing Page Auditor | On-demand | URL scraping + analysis | n8n workflow (HTML popup) |
| Localization Coverage | On-demand | WP/Woo pages | Python script (HTML/JSON/CSV) |

---

## 5. Integrations & Connectivity

### 5.1 MVP Integration Overview

Every external integration is implemented through the adapter pattern — a change in the external service's API requires only a change in the adapter, not the core platform code.

| Integration | Connection Method | Agents Using It | Notes |
|-------------|-------------------|-----------------|-------|
| Google Analytics 4 | OAuth 2.0 token | Operations, SEO, CRO | Priority integration; onboarding wizard |
| Google Search Console | OAuth 2.0 token | SEO Monitoring | Optional for MVP; shares OAuth with GA4 |
| WooCommerce | REST API / App Key | Inventory Signals | Requires admin access from user |
| Web Scraping | Playwright/Puppeteer | Pricing Intel, LP Auditor, Localization | Proxy strategy for scaling |
| Shopify | OAuth 2.0 / API Key | Inventory Signals | Only if not complex for MVP |

### 5.2 OAuth Flow for GA4/GSC

The user connects to GA4/GSC through a standard OAuth 2.0 flow integrated into the onboarding wizard. The system stores a refresh token and automatically renews the access token. Note for the future: prepare a fallback to Google Service Account access as redundancy.

### 5.3 WooCommerce Integration

Connection through the WooCommerce REST API using Application Password or consumer key/secret. The user generates credentials in their WP admin panel and enters them in the DevsMachina onboarding wizard.

### 5.4 Scraping Infrastructure

For Pricing Intelligence, Landing Page Auditor, and Localization Coverage, a headless browser (Playwright) is used. MVP uses direct access, with a planned transition to proxy rotation for scaling.

**Legal aspect:** Scraping legality at scale is an open question for the post-MVP phase. MVP relies on publicly available data without authentication.

---

## 6. Reporting System

### 6.1 Delivery Channels (MVP)

In the MVP phase, the primary report delivery channel is email. Users can map which report type goes to which email address (e.g., Inventory report goes to the inventory team, Operations report to management).

**MVP options:**
- Email (default and primary channel)
- Signal/Telegram/Slack messages (optional, if implementation is not complex)

**Future options (post-MVP):**
- Report viewing within the application with send, save, export, delete options
- Automatic routing based on trigger type/event/context

### 6.2 Report Format

All reports are generated as structured HTML documents optimized for email rendering. Each report contains a header with key metrics, a body with detailed analysis, and a conclusion with prioritized action items.

### 6.3 Historical Data

- Users can view previous reports/audits
- They can download/export them
- Additional history management options planned for post-MVP

---

## 7. Data Model

### 7.1 Key Entity Tables

All tables contain a `tenant_id` column with a row-level security policy in PostgreSQL. This guarantees that no user can access another user's data.

| Table | Description | Key Columns |
|-------|-------------|-------------|
| `tenants` | Organizations/companies | id, name, plan_tier, billing_status, created_at |
| `users` | Platform users | id, tenant_id, email, role (Owner/Admin/Viewer), auth_id |
| `integrations` | Connected external services | id, tenant_id, type (ga4/gsc/woo), credentials_enc, status |
| `agents` | Configured agents per tenant | id, tenant_id, agent_type, config_json, schedule, status |
| `agent_runs` | Agent executions (audit trail) | id, agent_id, tenant_id, started_at, status, result_ref |
| `reports` | Generated reports | id, agent_run_id, tenant_id, type, content_html, created_at |
| `notification_rules` | Report delivery rules | id, tenant_id, agent_type, channel, destination |
| `competitor_snapshots` | Competitor site snapshots | id, tenant_id, agent_id, url, data_json, captured_at |

### 7.2 Encryption & Data Security

All external service credentials (OAuth tokens, API keys) are stored encrypted (AES-256) in the database. Decryption occurs only at runtime when calling the external API. No plain-text credentials in the database or logs.

---

## 8. Security & Compliance

### 8.1 Authentication & Authorization

- OAuth 2.0 / JWT-based authentication
- Role-based access control (RBAC): Owner, Admin, Viewer
- Row-level security in PostgreSQL for multi-tenant isolation
- Rate limiting at the API level

### 8.2 Data Protection

- Encryption at rest (AES-256 for credentials) and in transit (TLS 1.3)
- GDPR compliance: right to data deletion, data export, consent management
- Regular security audits and dependency scanning
- Parameterized queries (Prisma ORM) — protection against SQL injection

### 8.3 Monitoring & Audit Trail

- Every agent action is logged (agent audit trail)
- Structured logs with correlation ID for cross-system tracing
- Error tracking (Sentry) with automatic alerts
- Uptime monitoring with SLA tracking

---

## 9. Scalability & Infrastructure

### 9.1 Horizontal Scaling

Agent workers are designed as stateless processes that read tasks from the BullMQ queue. Adding new instances requires no configuration changes — a new worker automatically joins the pool.

### 9.2 Job Queue System

BullMQ (Redis-backed) enables priority queues for different task types:

| Priority | Task Type | Example |
|----------|-----------|---------|
| Critical (P0) | Alerts and notifications | Stock level dropped below minimum |
| High (P1) | Scheduled agent tasks | Daily competitor price check |
| Normal (P2) | Scheduled reports | Weekly operations report |
| Low (P3) | On-demand requests | Landing page audit |

### 9.3 Scaling Phases

| Phase | Users | Infrastructure |
|-------|-------|----------------|
| MVP | 10–20 | Single instance backend, shared DB, 1–2 workers |
| Growth | 20–100 | Horizontal worker scaling, connection pooling, CDN |
| Scale | 100–500 | Read replicas, dedicated job queues per type, caching layer |
| Enterprise | 500–1000+ | Multi-region deploy, dedicated DB per shard, feature flags |

---

## 10. Development Phases (Roadmap)

### 10.1 Month 1–2: Core Platform

- Authentication and authorization (signup, login, roles)
- Multi-tenant system with data isolation
- Onboarding wizard for connecting stores and external services
- Dashboard with overview of active agents and key KPIs
- Billing integration (subscription, invoices, payment method changes)
- WooCommerce connector (adapter)
- GA4 OAuth connector

### 10.2 Month 2–3: First Two Agents

- Operations Report agent (migration from n8n prototype)
- SEO Monitoring agent (migration from n8n prototype)
- Reporting engine (email delivery)
- Agent scheduling and job queue

### 10.3 Month 3–4: CRO + Pricing

- CRO Monitoring agent
- Pricing Intelligence agent with scraping infrastructure
- Landing Page Auditor (on-demand)
- Localization Coverage Auditor

### 10.4 Month 4–5: Beta & Stabilization

- Beta launch with 10–20 users
- Inventory Signals agent
- Feedback loop and bug fixing
- Performance optimization
- Documentation: API spec, agent specifications, runbooks

### 10.5 Month 6+: Optimization & Scaling

- Horizontal scaling implementation
- Feature flags system
- New tools and agents based on user feedback
- Shopify integration (if demand is validated)
- META Reports (combined cross-agent reports)

---

## 11. Open Questions & Decisions

The following questions require further analysis and decision-making before or during development:

| Question | Context | Priority |
|----------|---------|----------|
| Shopify support | When and how to introduce — depends on market validation | Medium |
| Final pricing model | What each tier includes, how many agents per plan | High |
| LLM provider selection | Which providers, fallback strategy, cost optimization | High |
| White-label / partner option | Offering for agencies — requires branding customization | Low |
| Compliance (SOC 2, HIPAA) | Depends on target industries (healthcare = HIPAA) | Low (MVP) |
| Scraping legality at scale | Proxy strategy, robots.txt compliance, rate limiting | Medium |
| Marketplace for custom agents | Plugin ecosystem for third-party developers | Low (post-MVP) |
| Mobile app vs responsive web | Native app or PWA approach | Low |
| External service redundancy | Fallback for OAuth, API changes (e.g., Service Account) | Medium |

---

## 12. Out of Scope for MVP

The following features have been identified as valuable but are not part of the MVP deliverable:

- **META Reports** — combining multiple reports into one with holistic recommendations
- **ERP/CRM integrations** for advanced inventory tracking
- **Marketing/sales tool integrations** (Semrush, Ahrefs, Hotjar, Klaviyo, Mailchimp, AfterShip)
- **External service redundancy** (fallback OAuth mechanisms, alternative API access)
- **Continuous localization monitoring** (real-time translation tracking)
- **In-app report viewing and management** (MVP: sending only)
- **Automatic report routing** based on context

---

## 13. CI/CD & DevOps

### 13.1 Pipeline

1. Push to feature branch → automatic lint + unit tests
2. Pull request → integration tests + security scan
3. Merge to main → deploy to staging environment
4. Manual approval → zero-downtime deploy to production

### 13.2 Testing Strategy

- **Unit tests:** agent business logic, adapters, utility functions
- **Integration tests:** API endpoints, database operations, queue processing
- **E2E tests:** critical user flows (onboarding, agent creation, report generation)

### 13.3 Backup & Recovery

- Automatic daily PostgreSQL database backup
- Point-in-time recovery capability (thanks to Supabase/NeonDB)
- Backup verification on a weekly basis

---

## 14. Glossary

| Term | Definition |
|------|------------|
| **Tenant** | Organization/company using the platform; each tenant has isolated data |
| **Agent** | AI module that performs a specific task (monitoring, reporting, auditing) |
| **Adapter** | Wrapper class that abstracts communication with an external API |
| **Job Queue** | Asynchronous task processing queue system (BullMQ/Redis) |
| **Worker** | Stateless process that picks up and executes tasks from the job queue |
| **Circuit Breaker** | Pattern that stops calls to a service that is down |
| **RLS** | Row-Level Security — PostgreSQL mechanism for data isolation per tenant |
| **Eisenhower Matrix** | Prioritization framework: Urgent/Important, Urgent/Not important, Not urgent/Important, Not urgent/Not important |
| **Graceful Degradation** | System's ability to continue operating with reduced functionality instead of a complete crash |
| **MVP** | Minimum Viable Product — first functional version with a minimal feature set |

---

*— End of Document —*
