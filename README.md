# M-Vision BI

**An on-premise business intelligence platform for manufacturing companies — designed, built and deployed end-to-end by a single developer, working forward-deployed with the customers who use it.**

> This repository is a portfolio overview. The product is proprietary and used commercially; source code, customer information and internal details are intentionally not published here.

---

## Forward-deployed engineering

M-Vision BI was not built from a spec handed down to an engineering team. It was built **in the field**: sitting with plant managers, quality engineers and purchasing teams, learning how their operations really run, and turning that into software running on their own servers.

- **Embedded with customers** — requirements came from the shop floor and from customers' own quality-management procedures and forms, not from a product backlog. A report was "done" when the team using it trusted the numbers.
- **Working on live data, in constrained environments** — customer ERP databases, on-site Windows servers, VPN-only access, machines with no internet connection, differing Python and SQL Server setups. Every deployment had its own constraints, and the product had to adapt to them.
- **Shipping to multiple manufacturing companies from one codebase** — customer-specific behaviour (which reports, which criteria, which thresholds) is driven by configuration, so each site gets what it needs without forking the product.
- **Owning the whole loop** — discovery → data modelling → build → packaging → on-site installation → training → support → next iteration, with fast turnaround on customer requests and field issues.
- **Field-hardened delivery** — offline installation packages, versioned per-customer update packages, Windows-service deployment, and a troubleshooting playbook built from real issues seen on customer servers.

---

## What it is

M-Vision BI turns live ERP data into interactive dashboards for production, quality, supply chain and management teams. It was built as a self-hosted alternative to commercial BI tools: no per-user cloud licensing, data never leaves the customer's network, and every report is tailored to how manufacturing teams actually work on the shop floor.

- **30+ dashboards** across production (capacity, machine status / andon, downtime, OEE, scrap), quality (delivery conformity, returns, non-conformance rates, OTIF), supply chain (supplier performance, on-time delivery, stock) and analytics (quotations, customer-based production, workforce skill matrix, sales-order traceability)
- **Supplier evaluation engine** — configurable weighted criteria, scoring categories, period comparison, approval workflow and scheduled e-mail reports with Excel output
- **Multi-company / multi-tenant** data isolation, configurable per customer without code changes
- **Role-based access control** down to dashboard *and* individual field level, JWT authentication
- **Live updates** over WebSocket for shop-floor screens (designed for large wall-mounted displays)
- **Notification service** running as a Windows service with scheduled jobs and SMTP delivery
- **Offline installation packages** for customer servers without internet access, plus versioned update packages

## Architecture

```
  ERP database (MSSQL, read-only)      Auth / settings DB (MSSQL)
              │                                   │
              └──────────────┬────────────────────┘
                             │  SQLAlchemy / pyodbc
                 ┌───────────▼────────────┐
                 │  FastAPI backend       │  per-report query + service layers,
                 │  (Python, uvicorn)     │  RBAC guards, caching, WebSocket
                 └───────────┬────────────┘
                             │  REST + WebSocket
                 ┌───────────▼────────────┐
                 │  Web frontend          │  vanilla JS, Tailwind, Chart.js,
                 │  (static, no build)    │  Excel export
                 └────────────────────────┘
   Deployed on customer Windows servers as services (NSSM)
```

Each report follows the same layered pattern (query layer → data service → API route → dashboard), which keeps 30+ reports consistent and lets new ones be added quickly.

## Tech stack

| Area | Technologies |
|---|---|
| Backend | Python, FastAPI, uvicorn, SQLAlchemy, pyodbc, pandas, openpyxl, cachetools |
| Data | Microsoft SQL Server (ERP + application databases) |
| Security | JWT (python-jose), bcrypt (passlib), field-level RBAC |
| Frontend | HTML, vanilla JavaScript, Tailwind CSS, Chart.js (+ zoom / time adapters), SheetJS |
| Deployment | Windows Server, NSSM services, PowerShell packaging scripts, offline Python wheels |

## AI-assisted engineering workflow

The project is developed with a structured, multi-agent AI workflow (Cursor and Claude Code) that I designed:

- **Specialised agents** — an architect that plans work, domain "squads" that implement (production, quality, supply chain, analytics, auth, …), a code reviewer, a process self-improver and a memory-sync agent
- **Risk-based flow** — small changes run a light path (one squad, one review); risky changes (shared infrastructure, auth, schema, external writes) run the full path and require owner approval
- **Tiered project memory** — a structured knowledge base the agents read selectively, keeping per-task context small and costs low
- **Recurring-mistakes register** — lessons from past incidents are captured once and checked by the reviewer on every change
- **Code knowledge graph** — call-graph based impact analysis before changes
- **Guardrails** — pre-commit checks for secrets, file encoding and tool-configuration drift; the same rules and agents are kept in sync across both AI tools

## My role

I built M-Vision BI **from scratch and on my own, end to end**, as the single engineer between the customers and the product:

- **Customer-facing discovery** — worked directly with manufacturing customers on site, mapped their processes and quality procedures, and translated them into reports, KPIs and scoring rules
- **Data and backend** — designed the data model on top of the ERP schema, wrote the SQL behind every report, built the API, authentication / authorization and notification services
- **Frontend** — built every dashboard, optimised for both desk use and large shop-floor displays
- **Deployment and operations** — designed the on-premise deployment model, packaged releases (online and offline), installed and upgraded the product at customer sites, and handled field troubleshooting
- **Product evolution** — took the product from version 1.0 to 1.11 driven by customer feedback, adding new modules such as supplier evaluation, OEE and sales-order traceability
- **Engineering system** — designed the AI-assisted development workflow described above, so the product can keep growing beyond one person
