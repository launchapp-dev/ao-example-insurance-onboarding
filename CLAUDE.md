# Insurance Agent Onboarding — Agent Context

## What This Project Is

An automated insurance agent onboarding pipeline. When a new agent applies to join the agency, this pipeline handles the full onboarding sequence: screening their application, verifying their licenses via NIPR, running background checks, assigning and tracking training, provisioning system access, and issuing the compliance certification and appointment letter.

The pipeline enforces hard regulatory requirements at every step (18 U.S.C. §1033, state licensing law, E&O requirements) with automatic rejection and disqualification paths.

## Data Model — What's in Each File

| File | What It Contains | Who Reads It | Who Writes It |
|---|---|---|---|
| `data/application.json` | Raw agent application: NPN, legal name, DOB, SSN-4, address, license types, lines of authority, E&O info, self-disclosures | application-screener | Human (provided by applicant) |
| `data/nipr-data.json` | NIPR lookup results for the applicant's NPN (optional) | license-verifier | External NIPR API or test fixture |
| `data/background-check-raw.json` | Raw background check from vendor: criminal, credit, FINRA, OFAC, prior terminations | background-checker | Background check vendor |
| `data/screening-report.json` | Structured screening verdict: approved/conditional/rejected, missing items, NPN, self-disclosures | license-verifier, background-checker, training-coordinator, provisioner | application-screener |
| `data/license-verification.json` | License verification: NIPR name match, per-state license status, lines of authority, CE status, regulatory actions | background-checker, training-coordinator, provisioner, compliance-certifier | license-verifier |
| `data/background-assessment.json` | Background verdict: clear/flagged/disqualified, criminal history, credit concerns, FINRA disclosures, prior terminations | compliance-certifier | background-checker |
| `data/training-plan.json` | Training curriculum: module IDs, hours, deadlines, carrier certifications | training-coordinator (completion check), compliance-certifier | training-coordinator |
| `data/training-completion.json` | Training completion records from LMS | compliance-certifier | training-coordinator / LMS integration |
| `data/provisioning-record.json` | System access log: AMS, CRM, carrier portals, quoting tools, pending admin actions | compliance-certifier | provisioner |
| `documents/compliance-certificate.json` | Final certification: verdict, prerequisites met, carriers to appoint, appointment filing deadline | Agency principals | compliance-certifier |
| `documents/appointment-letter.md` | Formal appointment letter in agency letterhead format | Agency principals, carriers | compliance-certifier |
| `documents/rejection-notice.json` | Formal rejection record (if application rejected) | Agency principals | reject-application phase |
| `documents/disqualification-notice.json` | Formal disqualification record (if background fails) | Agency principals | disqualify-applicant phase |
| `logs/onboarding-issue.json` | Reference to GitHub issue tracking onboarding completion | Agency principals | compliance-certifier / track-onboarding-issue phase |

## Domain Terminology

**NPN (National Producer Number)** — Unique identifier assigned by NIPR to every insurance producer. This is the primary key for all license lookups.

**NIPR (National Insurance Producer Registry)** — The authoritative database for insurance producer licensing data across all 50 states. All license verifications use NIPR.

**Lines of Authority** — The specific insurance lines an agent is licensed to sell: Property, Casualty, Life, Health, Variable Life, etc. A license may have multiple lines, but the agent can only sell lines they are authorized for.

**E&O (Errors & Omissions)** — Professional liability insurance protecting against agent mistakes. Agencies require agents to maintain active E&O coverage before being appointed.

**Appointment** — The formal relationship between an agent and a carrier allowing the agent to sell that carrier's products. Appointment must be filed with the state DOI for each carrier-agent combination.

**CE (Continuing Education)** — State-required ongoing education hours to maintain a license. Failure to complete CE = license lapses.

**18 U.S.C. §1033** — Federal law prohibiting persons convicted of crimes of dishonesty from participating in the business of insurance without written consent from the state insurance commissioner. Any felony conviction = automatic disqualification.

**FINRA BrokerCheck** — FINRA's public database of broker-dealer and investment adviser registration information, including disclosure events (complaints, regulatory actions, terminations).

**OFAC** — Office of Foreign Assets Control. Agents cannot be on the OFAC Specially Designated Nationals list.

## Key Invariants Agents Must Respect

1. **NPN is the primary key** — Every data file must include the agent's NPN for cross-referencing. Never proceed without a valid NPN.

2. **Hard disqualifiers are non-negotiable** — Felony convictions (per 18 U.S.C. §1033), OFAC hits, and multiple regulatory actions result in immediate disqualification. No exceptions, no human override in the automated pipeline.

3. **Document everything** — Every verdict must be written to disk with a timestamp. Compliance audits require a complete paper trail.

4. **License status must match lines claimed** — An agent cannot be appointed for lines they are not licensed for. The provisioner must not provision carrier portals for lines where the agent lacks authorization.

5. **Training must precede certification** — The compliance-certifier must verify training completion before issuing a certificate. Do not certify an agent with incomplete mandatory training.

6. **Appointment deadlines are regulatory** — State appointment filing deadlines (typically 30 days from hire date) are regulatory requirements, not suggestions. The compliance certificate must include the filing deadline clearly.
