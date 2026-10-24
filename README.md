# 🧪 SQA Engineer Technical Assessment — Thaura AI

<div align="center">

![QA Assessment](https://img.shields.io/badge/Assessment-SQA%20Engineer-0ea5e9?style=for-the-badge&logo=testing-library&logoColor=white)
![Target](https://img.shields.io/badge/Target-Thaura.ai-8b5cf6?style=for-the-badge&logo=openai&logoColor=white)
![JMeter](https://img.shields.io/badge/Performance-Apache%20JMeter-d97706?style=for-the-badge&logo=apachejmeter&logoColor=white)
![Lighthouse](https://img.shields.io/badge/Audit-Google%20Lighthouse-10b981?style=for-the-badge&logo=lighthouse&logoColor=white)
![Bug Report](https://img.shields.io/badge/Bug%20Report-Excel%20%2F%20Markdown-107c41?style=for-the-badge&logo=microsoftexcel&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge)

<p align="center">
  <b>Comprehensive Technical Quality Assurance Assessment for Qtec Solution Limited</b><br>
  <i>In-depth functional validation, API contract verification, security boundary checks, data-integrity auditing, and JMeter load testing of <a href="https://thaura.ai/">Thaura AI</a>.</i>
</p>

[📋 Executive Summary](#-executive-summary) •
[🔍 Assessment Tasks](#-assessment-tasks--scope) •
[🐛 Key Findings](#-key-defect-highlights) •
[📊 Performance & Audits](#-performance-testing--audits) •
[📂 Repository Structure](#-repository-structure) •
[🚀 Reproduction Guide](#-reproduction--execution-guide) •
[💡 Task 3 AI Reflection](#-task-03-ai--testing-reflections)

---

</div>

## 📌 Methodology & Testing Ethics

> [!IMPORTANT]
> **Independent Testing & Ethical Declaration**
> - **Core Engineering & Discovery:** All functional exploratory testing, API contract evaluation, boundary probes, and JMeter load-testing plans were designed and conducted independently.
> - **Role of AI Tools:** AI (Claude Code) was utilized selectively for code-level investigation, syntax formatting, and linguistic polishing of Task 3 responses without altering the core findings, logic, or technical evaluation.
> - **Responsible Security & Data Ethics:** Testing followed non-intrusive security practices. No denial-of-service was attempted against production endpoints, and no passwords, live API tokens, or customer data are committed in this repository.

---

## 📋 Executive Summary

This repository contains the complete deliverables for the **SQA Engineer Hiring Assessment** at **Qtec Solution Limited**. The target system under test is **Thaura AI** (`https://thaura.ai/`), an ethical, privacy-focused AI companion and API platform.

### Quick Stats & Assessment Dashboard

| Metric | Result | Details |
| :--- | :--- | :--- |
| **Target Application** | [thaura.ai](https://thaura.ai/) | Web App (Chat Product), Public Marketing Site, Developer API |
| **Total Issues Documented** | **13+ Defect Entries** | Categorized across P1 (High) to P3 (Low) with reproduction steps |
| **Top Critical Finding** | **Mathematical Pricing Defect** | Annual plan advertises "Save 20%", but $12/mo × 12 = $144 (0% discount) |
| **Contract Discrepancy** | **Quota Limit Divergence** | Brief specifies `5 msgs / 2 hrs`; Live app enforces `5 msgs / 5 hrs` |
| **Lighthouse Audit** | **Perf: 98 \| A11y: 100 \| SEO: 100** | Full report captured with Core Web Vitals and resource metrics |
| **Performance Testing** | **Apache JMeter 5.x** | Configured JMX test plan, JTL logs, and generated HTML dashboard |

---

### 1️⃣ Task 01: Comprehensive Technical Testing of Thaura Web App
*Deep functional, data-integrity, security, and edge-case testing of the conversational AI product.*
- **Authentication & Sessions:** Sign-up/login flows, session persistence, token lifecycle, and concurrent session boundaries.
- **Quota & Rate-Limiting:** Verified enforcement of Free-tier limits (blocked precisely at message #6), fixed vs. rolling window mechanics, and quota exhaustion modal states.
- **Data Integrity & File Ingestion:** Validated upload parser behavior using synthetic fixtures (CSV, SVG, PDF) with embedded isolation tokens (`ISOLATION_MARKER_20260916`); confirmed robust rejection of corrupted files, empty files, and disallowed `.exe` extensions.
- **Memory & Incognito Mode:** Confirmed cross-session preference recall in standard chats; verified UI transitions and isolation parameters under Incognito Mode.
- **Developer API Boundaries:** Assessed bearer authentication, token limits (`32,000` cap), schema contract adherence, and 401 unauthorized handling.

### 2️⃣ Task 02: Thaura.ai Website Technical Testing
*Evaluating logic correctness, cross-page consistency, non-intrusive security, and system performance.*
- **Logic & Pricing Analysis:** Uncovered mathematical contradiction in annual billing discount calculations and pricing discrepancy between the Pricing page ($12/$144) and FAQ documentation ($15).
- **SEO & Canonical Routing:** Audited canonical tag divergence between `/` and `/home` without 301 redirects, risking duplicate indexation.
- **Client & Network Reliability:** Tracked unhandled 401 console errors on unauthenticated public routes (`/api/auth/me`).
- **Security Posture Review:** Inspected HTTPS enforcement, HSTS, X-Frame-Options (`DENY`), and Content Security Policy (`CSP`) directives (flagged permissive `unsafe-inline` and localhost bindings).
- **Performance & Load:** Executed desktop Lighthouse audits and JMeter performance scenarios.

### 3️⃣ Task 03: AI & Testing — Strategic Reflection
*Personal evaluation and technical perspectives on the evolving role of AI in SQA.*
- Real-world application of AI in test case design, Playwright automation maintenance, boundary discovery, and test data generation.
- Detailed responses provided in [`Task 3.docx`](Task%203.docx) and summarized below.

---

## 🐛 Key Defect Highlights

Below is a curated sample of defects discovered during the assessment. For the complete list, refer to [`Thaura AI_Bug_Report.xlsx`](Thaura%20AI_Bug_Report.xlsx) and [`test_report.md`](test_report.md).

| Bug ID | Severity | Priority | Component | Issue Description | Impact |
| :--- | :---: | :---: | :--- | :--- | :--- |
| **BUG-001** | `High` | `P1` | **Pricing Logic** | **"Save 20%" annual pricing claim is mathematically false**.<br>Toggle shows `$12/mo` and `Billed $144/yr`. ($12 × 12 = $144, representing exactly 0% savings). | Misleading commercial claim; legal & customer trust risk. |
| **BUG-002** | `High` | `P1` | **Pricing vs. FAQ** | **Pro plan price contradicts between pages**.<br>Pricing page lists Pro at `$12/mo`, while the FAQ section states `$15/mo`. | Confuses potential paying customers during checkout decisions. |
| **BUG-006** | `Medium` | `P2` | **Quota Contract** | **Free tier reset duration mismatch**.<br>Assessment specification requires `5 msgs / 2 hrs`, but live production enforces `5 msgs / 5 hrs`. | Contract divergence between requirements and deployment. |
| **BUG-009** | `Medium` | `P2` | **Rate Limiting** | **Sixth message hard blocked with 5h timer**.<br>Messages 1–5 succeed; message 6 returns `allowed: false` with UI lockout. | Verifies rate-limit enforcement, confirms 5h window. |
| **BUG-011** | `Medium` | `P2` | **Frontend Reliability** | **Expected unauthenticated check triggers 401 in console**.<br>`/api/auth/me` outputs an uncaught 401 on public landing pages. | Clutters browser console; breaks clean frontend hygiene. |
| **BUG-015** | `Low` | `P3` | **Form Validation** | **Contact Subject allows 10,000+ characters without limit**.<br>Input lacks client-side `maxlength` enforcement. | Potential payload bloat; unnecessary server-side validation load. |
| **BUG-003** | `Low` | `P3` | **SEO & Routing** | **Canonical tag points to `/home` while serving `/`**.<br>No canonical 301 redirect between root and `/home`. | SEO link-equity dilution and split page indexing. |

---

## 📊 Performance Testing & Audits

### 1. Google Lighthouse Audit Summary
Captured from automated audit runs against the production environment (detailed in [`thaura.ai_2026-09-16_23-28-04.html`](thaura.ai_2026-09-16_23-28-04.html)):

```
┌─────────────────────────────────────────────────────────────┐
│  Performance:  98/100   ██████████████████████████████░░    │
│  Accessibility: 100/100 ████████████████████████████████    │
│  Best Practices: 96/100 ███████████████████████████████░    │
│  SEO:          100/100  ████████████████████████████████    │
└─────────────────────────────────────────────────────────────┘
```
- **First Contentful Paint (FCP):** `0.3 s`
- **Largest Contentful Paint (LCP):** `1.0 s`
- **Total Blocking Time (TBT):** `50 ms`
- **Cumulative Layout Shift (CLS):** `0.00`
- **Speed Index:** `1.2 s`
- *Optimization Opportunities:* 173 KiB unused JavaScript cleanup, 120 ms render-blocking resource optimization.

### 2. Apache JMeter Performance Suite
The performance package under [`Performance Testing using JMeter/`](Performance%20Testing%20using%20JMeter/) contains the test plan, raw metrics, and the generated HTML dashboard.

```
Performance Testing using JMeter/
├── Qtech.jmx                             # Apache JMeter Test Plan
├── Screenshot_24-10-2026_201157_.jpeg     # Visual execution screenshot
└── qtech.html/                           # Interactive HTML Performance Dashboard
    ├── index.html                        # Dashboard Overview & APDEX ratings
    └── content/                          # Charts, response time distributions & graphs
```

---

## 📂 Repository Structure

```text
├── 📄 README.md                            # Executive overview, deliverables & documentation (You are here)
├── 📝 test_report.md                       # Comprehensive SQA report with findings, matrices & evidence
├── 📊 Thaura AI_Bug_Report.xlsx            # Standardized QA bug tracking spreadsheet (13+ defects)
├── 🌐 thaura.ai_2026-09-16_23-28-04.html    # Standalone Google Lighthouse audit report
├── 📑 Task 3.docx                          # Written responses for AI & Testing reflection questions
└── 📁 Performance Testing using JMeter/    # JMeter load testing test plan and reporting assets
    ├── Qtech.jmx                           # Executable JMeter test script
    ├── Screenshot_24-10-2026_201157_.jpeg  # Test run confirmation screenshot
    └── qtech.html/                         # Full interactive JMeter HTML performance dashboard
        └── index.html                      # Entry point for JMeter graphs and statistics
```

---

## 🚀 Reproduction & Execution Guide

### Reviewing Artifacts Locally

1. **Bug Report Spreadsheet:** Open [`Thaura AI_Bug_Report.xlsx`](Thaura%20AI_Bug_Report.xlsx) in Microsoft Excel, LibreOffice Calc, or Google Sheets for the prioritized bug log.
2. **Lighthouse Audit:** Double-click [`thaura.ai_2026-09-16_23-28-04.html`](thaura.ai_2026-09-16_23-28-04.html) to open the interactive performance and audit breakdown in any web browser.
3. **JMeter Performance Dashboard:** Open `Performance Testing using JMeter/qtech.html/index.html` in your browser to view response time over time, throughput, APDEX score, and error rates.

### Re-running the JMeter Performance Test

To execute the test plan in non-GUI mode and regenerate the HTML report:

```bash
# 1. Navigate to the JMeter folder
cd "Performance Testing using JMeter"

# 2. Execute test plan and output results to a JTL file
jmeter -n -t Qtech.jmx -l report/Qtech.jtl

# 3. Generate the interactive HTML Dashboard
jmeter -g report/Qtech.jtl -o report/qtech_dashboard
```

---

## 💡 Task 03: AI & Testing Reflections

*(Summary of personal responses authored in [`Task 3.docx`](Task%203.docx))*

> **1. How do you think AI will impact software testing over the next 2–3 years?**  
> AI will make software testing substantially faster and more efficient. It will automate repetitive overhead — such as dynamic test data generation, edge-case ideation, and initial script scaffolding. Crucially, AI-driven locators and self-healing automation will reduce regression maintenance. This shifts human testers from manual execution towards deeper critical thinking, exploratory testing, security boundaries, and architectural quality.
>
> **2. Do you personally use AI? Does AI help you in testing?**  
> Yes, daily. It assists in expanding boundary and negative test scenarios, accelerating API payload creation, standardizing bug report documentation, and updating Playwright automation locators whenever frontend UI components shift.
>
> **3. Which AI tools do you currently use?**  
> - **Claude (Anthropic) & Claude Code:** Primary tool for deep technical reasoning, code analysis, complex test logic, and script maintenance.
> - **ChatGPT (OpenAI):** General query exploration, documentation formulation, and rapid problem-solving.
> - **GitHub Copilot / Cursor:** IDE-integrated assistance for test automation scripting and regex formulation.
>
> **4. Which AI tool do you find most helpful for testing specifically?**  
> **Claude Code.** When paired with modern automation frameworks like Playwright, it rapidly detects UI structure changes, recommends robust selectors, and scaffolds end-to-end tests while preserving strict human oversight and verification of generated code.

---

## 📖 Recommended Review Order for Evaluators

```
  [1] test_report.md
      └── Complete findings, test matrices, and evidence breakdown
            │
  [2] Thaura AI_Bug_Report.xlsx
      └── Tabular defect tracking with severity, steps, and priority
            │
  [3] Performance Testing using JMeter/qtech.html/index.html
      └── Load test performance dashboard, response times, and APDEX
            │
  [4] thaura.ai_2026-09-16_23-28-04.html
      └── Lighthouse automated quality, accessibility, and Core Web Vitals
            │
  [5] Task 3.docx
      └── Candidate reflection on AI in SQA workflows
```

---

<div align="center">

**Author:** SQA Candidate  
**Submission For:** Qtec Solution Limited — SQA Engineer Assessment  
*All tests conducted with adherence to system safety, ethical guidelines, and non-intrusive QA standards.*

</div>
