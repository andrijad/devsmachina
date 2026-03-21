# DEVSMACHINA — Tehnička dokumentacija (MVP)

**Verzija:** 1.0  
**Datum:** Mart 2026  
**Klasifikacija:** Interno / Poverljivo

---

## Sadržaj

1. [Executive Summary](#1-executive-summary)
2. [Arhitektura sistema](#2-arhitektura-sistema)
3. [Agent sistem — MVP funkcionalnosti](#3-agent-sistem--mvp-funkcionalnosti)
4. [Uporedni pregled agenata](#4-uporedni-pregled-agenata)
5. [Integracije i povezivanje](#5-integracije-i-povezivanje)
6. [Reporting sistem](#6-reporting-sistem)
7. [Data model](#7-data-model)
8. [Bezbednost i compliance](#8-bezbednost-i-compliance)
9. [Skalabilnost i infrastruktura](#9-skalabilnost-i-infrastruktura)
10. [Faze razvoja (Roadmap)](#10-faze-razvoja-roadmap)
11. [Otvorena pitanja i odluke](#11-otvorena-pitanja-i-odluke)
12. [Van opsega MVP-a](#12-van-opsega-mvp-a)
13. [CI/CD i DevOps](#13-cicd-i-devops)
14. [Rečnik pojmova](#14-rečnik-pojmova)

---

## 1. Executive Summary

DevsMachina je SaaS platforma za e-commerce intelligence koja koristi AI agente za automatizovano praćenje performansi, konkurencije i operativnog zdravlja online prodavnica. Platforma zamenjuje manuelne analitičke procese sa inteligentnim, automatizovanim agentima koji generišu konkretne, akcione preporuke.

Ciljni korisnici su vlasnici i timovi e-commerce prodavnica (prioritetno WooCommerce, u budućnosti Shopify) koji žele da donose odluke bazirane na podacima bez potrebe za in-house data analitičarima.

**Ključne vrednosti platforme:**

- **Automatizovani monitoring** — SEO, CRO, operacije, cene konkurencije, zalihe
- **Akcioni izveštaji** — ne samo brojke, već konkretne instrukcije šta i kako promeniti
- **Eisenhower Matrix prioritizacija** — svaka preporuka je kategorizovana po hitnosti i značaju
- **Modularni agent sistem** — korisnici aktiviraju samo agente koji su im potrebni
- **Multi-tenant arhitektura** — stroga izolacija podataka između korisnika

---

## 2. Arhitektura sistema

### 2.1 Arhitekturni principi

Sistem je dizajniran prema sledećim ključnim principima koji obezbeđuju skalabilnost, pouzdanost i održivost:

| Princip | Opis | Implementacija |
|---------|------|----------------|
| Multi-tenant izolacija | Stroga separacija podataka po korisniku; nikada data leakage | Row-level security u PostgreSQL, tenant_id na svakoj tabeli |
| Fazna skalabilnost | 10 → 100 → 500 → 1000+ korisnika bez prepisivanja | Stateless servisi, horizontalno skalabilni workeri |
| LLM provider agnosticizam | Arhitektura podržava više AI provajdera sa fallback logikom | Apstraktni LLM sloj sa adapter pattern-om |
| Graceful degradation | Pad eksternog servisa pauzira agenta, ne ruši sistem | Circuit breaker pattern, health check monitoring |
| Adapter pattern | Promena u 3rd party API = promena adaptera, ne core koda | Wrapper klase za sve eksterne integracije |
| Modularni agenti | Dodavanje novog agenta ne dira postojeću platformu | Plugin arhitektura sa standardizovanim interfejsom |

### 2.2 Tehnološki stek

| Sloj | Tehnologija | Obrazloženje |
|------|-------------|--------------|
| Frontend | Next.js (React) + TypeScript | SSR/SSG, type safety, ekosistem komponenti |
| Backend API | Node.js + Express/Fastify | Brz razvoj, deljenje TypeScript tipova sa frontendom |
| Baza podataka | PostgreSQL (Supabase/NeonDB) | ACID compliance, RLS, JSONB za fleksibilne šeme |
| ORM | Prisma | Type-safe upiti, automatske migracije, introspection |
| Job Queue | BullMQ (Redis) | Prioritetni redovi, retry logika, cron scheduling |
| Autentifikacija | Supabase Auth / NextAuth | OAuth 2.0, JWT, multi-provider podrška |
| LLM sloj | Provider-agnostic wrapper | OpenAI, Anthropic, Gemini sa fallback lancem |
| Scraping | Playwright / Puppeteer | Headless browser za dinamičke sajtove |
| Email/Notifikacije | Resend / SendGrid | Transakcioni emailovi, template engine |
| Hosting | Vercel (frontend) + Railway/Fly.io (backend) | Auto-scaling, edge deployment |
| CI/CD | GitHub Actions | Automatsko testiranje, staging, zero-downtime deploy |
| Monitoring | Sentry + Axiom/Betterstack | Error tracking, strukturirani logovi, uptime |

### 2.3 Sistemski dijagram — High Level

Sistem se sastoji od četiri glavna sloja koji komuniciraju kroz jasno definisane interfejse:

| Sloj | Komponente | Odgovornosti |
|------|------------|--------------|
| Presentation Layer | Next.js App, Dashboard UI | Korisnički interfejs, onboarding wizard, agent konfiguracija, pregled izveštaja |
| API Layer | REST/GraphQL API Gateway | Autentifikacija, autorizacija, rate limiting, request validation, routing |
| Business Logic Layer | Agent Orchestrator, Job Queue, LLM Service | Kreiranje i upravljanje agentima, scheduling, AI procesiranje, izveštavanje |
| Data & Integration Layer | PostgreSQL, Redis, External APIs | Perzistencija, keširanje, GA4/GSC/WooCommerce konektori, scraping engine |

Komunikacija između slojeva je isključivo jednosmerna odozgo nadole za zahteve (request flow), dok se odgovori (response flow) i notifikacije propagiraju prema gore. Agent workeri su stateless i komuniciraju sa Business Logic slojem isključivo kroz job queue.

---

## 3. Agent sistem — MVP funkcionalnosti

MVP verzija uključuje sedam AI agenata/alata. Svaki agent ima standardizovan interfejs, nezavisan životni ciklus i sopstvenu konfiguraciju.

### 3.1 Operations Report Agent

**Tip:** Redovni reporting + On-demand

Sintetizuje GA4, ad spend i revenue podatke u nedeljni/mesečni digest sa variance analizom i trend alertima. Report je strukturiran da bude odmah akcioni za marketing, web i sales timove.

**Ulazni podaci:**
- GA4 Analytics (sesije, konverzije, revenue, kanali, kampanje)
- Ad spend podaci (ako su dostupni u GA4)
- Istorijski podaci za poređenje (prethodni period, YoY)

**Izlaz:**
- Strukturiran HTML/email izveštaj sa ključnim KPI-jevima
- Akcione preporuke prioritizovane po Eisenhower Matrix logici
- Konkretne instrukcije: „Povećajte ad budžet za Proizvod X jer razlog Y", „Ponudite veći popust segmentu X"
- BI/Data-analyst mindset u odlučivanju šta je vredno za akciju

**Prototip:** Lokalni n8n workflow koji je čuvao/slao HTML report. Migracija na platformski agent sa istom logikom ali skalabilnom infrastrukturom.

### 3.2 SEO Monitoring Agent

**Tip:** Redovni reporting + On-demand

Prati ključne SEO pokazatelje na osnovu podataka iz GA4 (i eventualno GSC za MVP). Generiše report sa konkretnim instrukcijama šta i kako promeniti, prioritizovano po Eisenhower Matrix.

**Ulazni podaci:**
- GA4 organski saobraćaj, landing page performanse
- GSC podaci (opciono u MVP): impressions, clicks, pozicije, crawl errors
- Interna SEO baza znanja i direktive

**Izlaz:**
- Spisak konkretnih SEO aktivnosti: „META podaci na strani X su loši/fale", „Indexiraju se strane koje ne treba"
- Canonical tag provere, crawl budget optimizacija, indexation status
- Prioritizovane preporuke bazirane na proverenim SEO praksama

**Napomena:** SEO direktive se definišu na osnovu poznatih dobrih praksi sa pouzdanih izvora, pohranjenog AI SEO skill-a, kao i internih vodiča (SEO & CONTENT Guide, website launch/migration tech SEO to-do lista).

### 3.3 CRO Monitoring Agent

**Tip:** Redovni reporting + On-demand

Struktura i logika identična SEO agentu, ali fokusirana na Conversion Rate Optimization parametre i ponašanje korisnika na sajtu.

**Fokus oblasti:**
- CTR optimizacija po stranicama i elementima
- Engagement rate analiza (scroll depth, time on page, bounce rate)
- Checkout flow analiza — exit points, cart abandonment
- Konkretne instrukcije za unapređenje konverzija

**Prototip:** Nije kreiran lokalni prototip, ali se koristi ista infrastruktura kao SEO agent sa prilagođenim prompt-om i ulaznim podacima.

### 3.4 Pricing Intelligence Agent

**Tip:** Trigger-based reporting + On-demand

Analizira jedan ili više sajtova konkurencije i izveštava o promenama cena, dodavanju/uklanjanju proizvoda i aktivnim promocijama.

**Logika rada:**
1. U definisanom intervalu (npr. jednom dnevno) radi proveru stanja/promena
2. Ako je bilo promena → šalje strukturiran report
3. Ako nije bilo promena → čuva info da je poslednje stanje nepromenjeno
4. On-demand mod: izvuče listu proizvoda sa osnovnim info (bez istorijskih podataka za poređenje)

**Prototip:** n8n workflow sa URL adresom za scraping u konfiguraciji. Report o promenama slat na email i čuvan lokalno + Google Drive.

### 3.5 Inventory Signals Agent

**Tip:** Trigger-based reporting (samo redovni)

Prati stock level stanje i promene, potražnju i prodaju, i šalje alerte/izveštaje. Detektuje low stock rizike, sezonske promene potražnje i supplier lead time nepodudarnosti pre nego što postanu stockout-i.

**MVP integracije:**
- WP/WooCommerce — preko REST API ili Application Key
- Shopify — samo ako integracija nije kompleksna (opciono za MVP)

**Napomena:** ERP/CRM integracije (za naprednije inventory tracking) su planirane za post-MVP faze. U MVP se fokusiramo isključivo na podatke dostupne kroz WooCommerce API.

### 3.6 Landing Page In-Depth Auditor

**Tip:** On-demand only

Alat koji omogućava korisniku da uradi sveobuhvatan audit bilo koje strane: tech SEO, content SEO, Core Web Vitals i CRO analizu.

**Korisnički flow:**
1. Korisnik unosi URL za analizu
2. Bira koje audite želi (sve ili selektivno): Tech SEO, Content SEO, CWV/PSI, CRO
3. Sistem izvršava analizu i generiše report sa jasnim preporukama
4. Report sadrži konkretne instrukcije šta i kako popraviti na strani

**Prototip:** n8n workflow sa popup-om za unos URL-a. Nakon egzekucije prikazivao ekran sa rezultatima, čuvao HTML lokalno i nudio opciju za preuzimanje.

### 3.7 Localization Coverage Auditor

**Tip:** On-demand only

Proverava da li na multijezičnom sajtu (MVP: samo WordPress/WooCommerce) postoje prevodi svih strana na dodate jezike, i da li je sadržaj tih strana zaista preveden (a ne kopija default jezika).

**Šta proverava:**
- Da li sve strane imaju instance prevoda za svaki konfigurisani jezik
- Da li je sadržaj na prevedenim stranama zaista različit od default jezika
- Identifikuje gaps u pokrivenosti lokalizacije

**Prototip:** Python skripta koja se izvršavala on-demand i čuvala report kao HTML/JSON/CSV.

**Napomena:** Kontinuirani monitoring prevoda (real-time) nije u opsegu MVP-a jer korist ne opravdava kompleksnost u ovoj fazi.

---

## 4. Uporedni pregled agenata

| Agent | Tip | Podaci | Prototip |
|-------|-----|--------|----------|
| Operations Report | Redovni + On-demand | GA4, ad spend, revenue | n8n workflow (HTML) |
| SEO Monitoring | Redovni + On-demand | GA4, GSC (opciono) | n8n workflow (HTML) |
| CRO Monitoring | Redovni + On-demand | GA4, ponašanje korisnika | Nema (baziran na SEO) |
| Pricing Intelligence | Trigger + On-demand | Web scraping konkurencije | n8n workflow (email + GDrive) |
| Inventory Signals | Trigger-based | WooCommerce API | Nema |
| Landing Page Auditor | On-demand | URL scraping + analiza | n8n workflow (HTML popup) |
| Localization Coverage | On-demand | WP/Woo stranice | Python skripta (HTML/JSON/CSV) |

---

## 5. Integracije i povezivanje

### 5.1 Pregled integracija za MVP

Svaka eksterna integracija implementira se kroz adapter pattern — promena u API-ju eksternog servisa zahteva samo promenu adaptera, ne core platformskog koda.

| Integracija | Metod povezivanja | Agenti koji koriste | Napomene |
|-------------|-------------------|---------------------|----------|
| Google Analytics 4 | OAuth 2.0 token | Operations, SEO, CRO | Prioritetna integracija; onboarding wizard |
| Google Search Console | OAuth 2.0 token | SEO Monitoring | Opciono za MVP; deli OAuth sa GA4 |
| WooCommerce | REST API / App Key | Inventory Signals | Zahteva admin pristup korisnika |
| Web Scraping | Playwright/Puppeteer | Pricing Intel, LP Auditor, Localization | Proxy strategija za skaliranje |
| Shopify | OAuth 2.0 / API Key | Inventory Signals | Samo ako nije kompleksno za MVP |

### 5.2 OAuth Flow za GA4/GSC

Korisnik se povezuje sa GA4/GSC kroz standardni OAuth 2.0 flow integrisan u onboarding wizard. Sistem čuva refresh token i automatski obnavlja access token. Napomena za budućnost: pripremiti fallback na Google Service Account pristup kao redundantnost.

### 5.3 WooCommerce integracija

Povezivanje kroz WooCommerce REST API koristeći Application Password ili consumer key/secret. Korisnik generiše credentials u svom WP admin panelu i unosi ih u DevsMachina onboarding wizard.

### 5.4 Scraping infrastruktura

Za Pricing Intelligence, Landing Page Auditor i Localization Coverage koristi se headless browser (Playwright). MVP koristi direktan pristup, sa planiranim prelaskom na proxy rotaciju za skaliranje.

**Pravni aspekt:** Scraping legalnost u skali je otvoreno pitanje za post-MVP fazu. MVP se oslanja na javno dostupne podatke bez autentifikacije.

---

## 6. Reporting sistem

### 6.1 Kanali isporuke (MVP)

U MVP fazi, primarni kanal isporuke izveštaja je email. Korisnici mogu mapirati koji tip izveštaja ide na koju email adresu (npr. Inventory izveštaj ide inventory timu, Operations report menadžmentu).

**MVP opcije:**
- Email (default i primarni kanal)
- Signal/Telegram/Slack poruke (opciono, ako implementacija nije kompleksna)

**Buduće opcije (post-MVP):**
- Pregled izveštaja u samoj aplikaciji sa opcijama send, save, export, delete
- Automatsko rutiranje na osnovu tipa trigera/eventa/konteksta

### 6.2 Format izveštaja

Svi izveštaji se generišu kao strukturirani HTML dokumenti optimizovani za email rendering. Svaki izveštaj sadrži zaglavlje sa ključnim metrikama, telo sa detaljnom analizom, i zaključak sa prioritizovanim akcionim tačkama.

### 6.3 Istorijski podaci

- Korisnici mogu da vide prethodne izveštaje/audite
- Mogu ih preuzeti/eksportovati
- Dodatne opcije za upravljanje istorijom planirane za post-MVP

---

## 7. Data model

### 7.1 Ključne entitetske tabele

Sve tabele sadrže `tenant_id` kolonu sa row-level security policy-jem u PostgreSQL-u. Ovo garantuje da nijedan korisnik ne može pristupiti podacima drugog korisnika.

| Tabela | Opis | Ključne kolone |
|--------|------|----------------|
| `tenants` | Organizacije/kompanije | id, name, plan_tier, billing_status, created_at |
| `users` | Korisnici platforme | id, tenant_id, email, role (Owner/Admin/Viewer), auth_id |
| `integrations` | Povezani eksterni servisi | id, tenant_id, type (ga4/gsc/woo), credentials_enc, status |
| `agents` | Konfigurisani agenti po tenantu | id, tenant_id, agent_type, config_json, schedule, status |
| `agent_runs` | Izvršavanje agenata (audit trail) | id, agent_id, tenant_id, started_at, status, result_ref |
| `reports` | Generisani izveštaji | id, agent_run_id, tenant_id, type, content_html, created_at |
| `notification_rules` | Pravila za slanje izveštaja | id, tenant_id, agent_type, channel, destination |
| `competitor_snapshots` | Snimci konkurentskih sajtova | id, tenant_id, agent_id, url, data_json, captured_at |

### 7.2 Enkripcija i sigurnost podataka

Svi kredencijali eksternih servisa (OAuth tokeni, API ključevi) se čuvaju enkriptovani (AES-256) u bazi. Dekriptovanje se vrši samo u runtime-u prilikom pozivanja eksternog API-ja. Nema plain-text kredencijala u bazi ili logovima.

---

## 8. Bezbednost i compliance

### 8.1 Autentifikacija i autorizacija

- OAuth 2.0 / JWT bazirana autentifikacija
- Role-based access control (RBAC): Owner, Admin, Viewer
- Row-level security u PostgreSQL za multi-tenant izolaciju
- Rate limiting na API nivou

### 8.2 Zaštita podataka

- Enkripcija u mirovanju (AES-256 za kredencijale) i u tranzitu (TLS 1.3)
- GDPR compliance: pravo na brisanje podataka, data export, consent management
- Redovni security audit-i i dependency scanning
- Parameterizovani upiti (Prisma ORM) — zaštita od SQL injection

### 8.3 Monitoring i audit trail

- Svaka akcija agenta se loguje (agent audit trail)
- Strukturirani logovi sa correlation ID za traženje kroz sisteme
- Error tracking (Sentry) sa automatskim alertima
- Uptime monitoring sa SLA tracking-om

---

## 9. Skalabilnost i infrastruktura

### 9.1 Horizontalno skaliranje

Agent worker-i su dizajnirani kao stateless procesi koji čitaju zadatke iz BullMQ reda. Dodavanje novih instanci ne zahteva promenu konfiguracije — novi worker se automatski uključuje u pool.

### 9.2 Job Queue sistem

BullMQ (Redis-backed) omogućava prioritetne redove za različite tipove zadataka:

| Prioritet | Tip zadatka | Primer |
|-----------|-------------|--------|
| Kritičan (P0) | Alerti i notifikacije | Stock level pao ispod minimuma |
| Visok (P1) | Redovni agentski taskovi | Dnevna provera cena konkurencije |
| Normalan (P2) | Zakazani izveštaji | Nedeljni operations report |
| Nizak (P3) | On-demand zahtevi | Landing page audit |

### 9.3 Faze skaliranja

| Faza | Korisnici | Infrastruktura |
|------|-----------|----------------|
| MVP | 10–20 | Single instance backend, shared DB, 1–2 workera |
| Rast | 20–100 | Horizontal worker scaling, connection pooling, CDN |
| Skaliranje | 100–500 | Read replicas, dedicated job queues po tipu, caching sloj |
| Enterprise | 500–1000+ | Multi-region deploy, dedicated DB per shard, feature flags |

---

## 10. Faze razvoja (Roadmap)

### 10.1 Mesec 1–2: Core platforma

- Autentifikacija i autorizacija (signup, login, roles)
- Multi-tenant sistem sa data izolacijom
- Onboarding wizard za povezivanje store-a i eksternih servisa
- Dashboard sa pregledom aktivnih agenata i ključnih KPI-jeva
- Billing integracija (pretplata, fakture, promena metode plaćanja)
- WooCommerce konektor (adapter)
- GA4 OAuth konektor

### 10.2 Mesec 2–3: Prva dva agenta

- Operations Report agent (migracija iz n8n prototipa)
- SEO Monitoring agent (migracija iz n8n prototipa)
- Reporting engine (email delivery)
- Agent scheduling i job queue

### 10.3 Mesec 3–4: CRO + Pricing

- CRO Monitoring agent
- Pricing Intelligence agent sa scraping infrastrukturom
- Landing Page Auditor (on-demand)
- Localization Coverage Auditor

### 10.4 Mesec 4–5: Beta i stabilizacija

- Beta lansiranje sa 10–20 korisnika
- Inventory Signals agent
- Feedback loop i bug fixing
- Performance optimizacija
- Dokumentacija: API spec, agent specifikacije, runbook-ovi

### 10.5 Mesec 6+: Optimizacija i skaliranje

- Horizontal scaling implementacija
- Feature flags sistem
- Novi alati i agenti na osnovu korisničkog feedbacka
- Shopify integracija (ako validirana potražnja)
- META Reports (objedinjeni cross-agent izveštaji)

---

## 11. Otvorena pitanja i odluke

Sledeća pitanja zahtevaju dodatnu analizu i donošenje odluka pre ili tokom razvoja:

| Pitanje | Kontekst | Prioritet |
|---------|----------|-----------|
| Shopify podrška | Kada i kako uvesti — zavisi od validacije tržišta | Srednji |
| Finalni pricing model | Šta svaki tier uključuje, koliko agenata po planu | Visok |
| LLM provider selekcija | Koji provajderi, fallback strategija, cost optimizacija | Visok |
| White-label / partner opcija | Ponuda za agencije — zahteva branding customization | Nizak |
| Compliance (SOC 2, HIPAA) | Zavisno od ciljnih industrija (zdravstvo = HIPAA) | Nizak (MVP) |
| Scraping legalnost u skali | Proxy strategija, robots.txt poštovanje, rate limiting | Srednji |
| Marketplace za custom agente | Plugin ekosistem za third-party developere | Nizak (post-MVP) |
| Mobilna app vs responsive web | Nativna app ili PWA pristup | Nizak |
| Redundancy eksternih servisa | Fallback za OAuth, API promene (npr. Service Account) | Srednji |

---

## 12. Van opsega MVP-a

Sledeće funkcionalnosti su identifikovane kao vredne, ali nisu deo MVP deliverable-a:

- **META Reports** — objedinjavanje više izveštaja u jedan sa holističkim preporukama
- **ERP/CRM integracije** za napredni inventory tracking
- **Integracije sa marketing/sales alatima** (Semrush, Ahrefs, Hotjar, Klaviyo, Mailchimp, AfterShip)
- **Redundancy za eksterne servise** (fallback OAuth mehanizmi, alternativni API pristupi)
- **Kontinuirani localization monitoring** (real-time praćenje prevoda)
- **Pregled i menadžment izveštaja u aplikaciji** (MVP: samo slanje)
- **Automatsko rutiranje izveštaja** na osnovu konteksta

---

## 13. CI/CD i DevOps

### 13.1 Pipeline

1. Push na feature branch → automatski lint + unit testovi
2. Pull request → integration testovi + security scan
3. Merge u main → deploy na staging environment
4. Manual approval → zero-downtime deploy na produkciju

### 13.2 Testing strategija

- **Unit testovi:** biznis logika agenata, adapter-i, utility funkcije
- **Integration testovi:** API endpoint-i, database operacije, queue procesiranje
- **E2E testovi:** kritični korisnički flow-ovi (onboarding, agent kreiranje, report generisanje)

### 13.3 Backup i recovery

- Automatski dnevni backup PostgreSQL baze
- Point-in-time recovery mogućnost (zahvaljujući Supabase/NeonDB)
- Backup verifikacija na nedeljnom nivou

---

## 14. Rečnik pojmova

| Pojam | Definicija |
|-------|------------|
| **Tenant** | Organizacija/kompanija koja koristi platformu; svaki tenant ima izolovane podatke |
| **Agent** | AI modul koji izvršava specifičan zadatak (monitoring, reporting, auditing) |
| **Adapter** | Wrapper klasa koja apstrahuje komunikaciju sa eksternim API-jem |
| **Job Queue** | Sistem redova za asinhrono procesiranje zadataka (BullMQ/Redis) |
| **Worker** | Stateless proces koji preuzima i izvršava zadatke iz job queue-a |
| **Circuit Breaker** | Pattern koji zaustavlja pozive ka servisu koji je u kvaru |
| **RLS** | Row-Level Security — PostgreSQL mehanizam za izolaciju podataka po tenantu |
| **Eisenhower Matrix** | Framework za prioritizaciju: Hitno/Važno, Hitno/Nije važno, Nije hitno/Važno, Nije hitno/Nije važno |
| **Graceful Degradation** | Sposobnost sistema da nastavi rad sa smanjenom funkcionalnošću umesto potpunog pada |
| **MVP** | Minimum Viable Product — prva funkcionalna verzija sa minimalnim setom feature-a |

---

*— Kraj dokumenta —*
