---
title: "Enlighten Capital Platform — Technical Architecture & Overview"
cover: https://images.unsplash.com/photo-1486854561807-38fc02359493?auto=format&fit=crop&w=2400&q=82
---


**Enlighten Capital** is an institutional deal room, startup evaluation engine, and syndicate capital allocation platform built for venture funds, syndicates, and angel investors. It unites statutory compliance (India DPDPA 2023, MCA / Zauba Corp registry integration), AI-powered deal diligence, and multi-portal investor relations into a unified full-stack architecture.

---

### 1. High-Level Architecture & Tech Stack

Code

```
┌────────────────────────────────────────────────────────────────────────┐
│                        React 19 SPA (Vite)                             │
│  Tailwind CSS • Recharts • Lucide Icons • Motion • HTML2Canvas / jsPDF │
└────────────────────────────────────┬───────────────────────────────────┘
                                     │ JSON REST / Real-time WebChannel
┌────────────────────────────────────▼───────────────────────────────────┐
│               Express Full-Stack Server (`server.ts`)                  │
│   • Middleware (Vite dev middleware / static dist in prod)             │
│   • Multi-tier Auth & RBAC (Super Admin, Admin, Investor, Startup)    │
│   • Google Gemini Flash 2.5 Engine (Valuation, Memo & DD generation)   │
│   • Corporate Registry Scraper & Grounding (MCA / Zauba Corp / DIN)    │
│   • Resend & SMTP Email Engine (Contribution Agreements, Drawdowns)    │
└───────────────────┬─────────────────────────────────┬──────────────────┘
                    │                                 │
┌───────────────────▼─────────────┐   ┌───────────────▼──────────────────┐
│  Firebase Cloud Firestore       │   │  Local File-System Mirror (/data)│
│  • Startup Profiles & Accounts  │   │  • Fallback persistence & cache  │
│  • Investor Soft Commitments    │   │  • Chunked document storage      │
│  • Audit & DPDPA Consent Logs   │   │  • Low-latency offline boot      │
│  • Automated DD & Deal Memos    │   │                                  │
└─────────────────────────────────┘   └──────────────────────────────────┘
```

- **Frontend**: React 19 SPA running on Vite 6 with Tailwind CSS, Recharts for investment analytics, Motion for fluid animations, and custom print/PDF generation utilities (jspdf, html2pdf.js, @resvg/resvg-js).
    
- **Backend**: Node.js & TypeScript full-stack Express service (server.ts) hosting REST APIs, statutory verification endpoints, email dispatchers, and Vite middlewares.
    
- **Data Persistence**: Hybrid resilient architecture combining **Firebase Cloud Firestore** for multi-user synchronization and atomic transactions with a local JSON file-system mirror (/data) for instantaneous cold boots, offline resilience, and oversized document caching.
    
- **Identity & Compliance**: OTP email authentication, Didit Protocol identity verification, and statutory Digital Personal Data Protection Act (DPDPA 2023) consent tracking with immutable audit trails.
    

---

### 2. Core Portals & Capabilities

1. **Investor Portal & Soft Commitments**:
    
    - Streamlined accreditation, KYC submission (PAN, Aadhaar, Passport for NRI/FDI), and target deal commitment.
        
    - Digital Contribution Agreement (CA) generator with inline electronic signatures, draw-down notices, and instant banking instructions.
        
    - Investor Portfolio tracking capital deployed, ownership percentages, valuation markups, and liquidation horizons.
        
2. **Startup Registration & Founder Console**:
    
    - Multi-step company disclosures: MCA CIN / LLPIN validation, Founder Director Identification Numbers (DIN), cap table structure, runway, and traction.
        
    - Document repository: Pitch deck, audited FY financials (P&L / Balance Sheet), Cap Tables, and Technical Due Diligence uploads with client-side chunking and base64 storage.
        
    - Real-time funding progress dashboard with deal interest and investor commit counters.
        
3. **Super Admin & Admin Intelligence Console**:
    
    - Multi-role access control (Super Admin approval queue with bypass headers for sensitive mutations like deletion or status overrides).
        
    - Real-time KPI stat cards: Total Capital Committed, Total Capital Deployed, Fund Thesis screening hit-rate, and pipeline funnel.
        
    - Bulk actions: Multi-select startup / investor deletion, CSV exports, and automated incomplete profile reminders.
        
    - Currency engine supporting multi-currency conversion (USD, INR, EUR, GBP, SGD, AED) with dynamic live rate calculations.
        
4. **Public Demo Day Showcase**:
    
    - Public curated deal showcase (/?view=demoday) for featured ventures.
        
    - One-click investment interest routing, pre-filling investor credentials directly into the deal commit workflow.
        

---

### 3. Automated Due Diligence & AI Deal Engines

- **1st-Level Fund Thesis Screening**: Configurable parametric evaluation filtering startups against ticket sizes (e.g. ₹1–5 Cr), valuation bands, minimum ARR thresholds, and sector focus.
    
- **2nd-Level Automated Due Diligence Engine**: Structured evaluation across 5 operational dimensions (Corporate & Legal, Market Opportunity, Financial Health, Technology & IP, Founder Pedigree) generating numerical health scores and risk profiles.
    
- **MCA & Zauba Corporate Registry Verification**: Live statutory registry validation checking official incorporation dates, RoC jurisdiction, authorized/paid-up capital, active charges, and promoter director listings directly against Ministry of Corporate Affairs data.
    
- **AI Investment Memo Generator**: Powered by @google/genai (Gemini Flash), auto-compiling institutional investment memos, SWOT matrix, SFB readiness gaps, and target investor persona matching into print-ready dossiers.
    

---

### 4. Enterprise Security & Statutory Compliance

- **HTTP Security Headers**: Strict-Transport-Security (HSTS), Content-Security-Policy (CSP) tailored for AI Studio and Cloud Run iframe embedding, X-Frame-Options, X-Content-Type-Options: nosniff, and Permissions-Policy.
    
- **DPDPA 2023 Audit Log**: Immutable record of privacy consent timestamps, purpose disclosures, and data principal verification tokens.
    
- **Resilient Firestore Operations**: Dedicated serialized write queues, backoff retries, and document size boundaries preventing 1MB Firestore document overflow and 128MB single-query limits.