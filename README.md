# Insurance Agent Onboarding Pipeline

An automated pipeline for onboarding new insurance agents — licensing verification, background checks, training assignment, system provisioning, and compliance certification — running fully automated with structured decision gates at every step.

---

## How It Works

One workflow covers the complete onboarding lifecycle:

```
ONBOARD-AGENT
  screen-application ──► verify-licensing ──► check-background ──► assign-training ──► provision-access ──► certify-compliance ──► track-onboarding-issue
         │                                           │                                                              │
    [rejected] ──► reject-application          [disqualified] ──► disqualify-applicant                    [needs-training] ──► assign-training (rework)
                                               [flagged] ──► screen-application (rework)                  [failed] ──► disqualify-applicant
```

Each decision phase outputs a structured verdict:
- **screen-application**: `approved` | `conditional` | `rejected`
- **check-background**: `clear` | `flagged` | `disqualified`
- **certify-compliance**: `certified` | `needs-training` | `failed`

---

## Agents

| Agent | Model | Role |
|---|---|---|
| **application-screener** | claude-haiku-4-5 | Reviews application completeness — NPN, E&O, license type, self-disclosures |
| **license-verifier** | claude-sonnet-4-6 | Verifies licensing via NIPR — active status, lines of authority, CE currency, regulatory actions |
| **background-checker** | claude-sonnet-4-6 | Reviews criminal history, credit, FINRA BrokerCheck, OFAC, prior terminations |
| **training-coordinator** | claude-haiku-4-5 | Assigns required training curriculum by role and lines of authority; tracks completion |
| **provisioner** | claude-haiku-4-5 | Provisions AMS, CRM, carrier portals, quoting tools, email, and e-signature access |
| **compliance-certifier** | claude-sonnet-4-6 | Issues compliance certification and appointment letter; tracks onboarding in GitHub |

---

## AO Features Demonstrated

- **Multi-agent pipeline** — six specialized agents each own a distinct compliance domain
- **Decision contracts** — three phases output structured verdicts driving workflow routing
- **Phase routing** — rejected applications and disqualified candidates are terminated early; flagged backgrounds loop back for re-screening; incomplete training routes back to training assignment
- **Command phases** — rejection and disqualification trigger bash scripts that generate formal notices
- **Output contracts** — structured JSON at every step (screening report, license verification, background assessment, training plan, provisioning record, compliance certificate)
- **Manual approval gate** — compliance-certifier produces the appointment letter for agency principal review before carrier filing
- **GitHub integration** — onboarding tracked as a GitHub issue with a completion checklist for principal oversight

---

## Quick Start

```bash
# 1. Set up environment variables
cp .env.example .env
# Edit .env with your GitHub token

# 2. Add an agent application
cp data/application.example.json data/application.json
# Edit data/application.json with the applicant's details

# 3. Start the AO daemon
ao daemon start --autonomous

# 4. Queue the onboarding workflow
ao queue enqueue \
  --title "jane-smith-onboarding" \
  --description "New P&C agent application — Jane Smith, NPN 12345678" \
  --workflow-ref onboard-agent

# 5. Watch it run
ao daemon stream --pretty
```

---

## Data Files

| File | Contents | Written By |
|---|---|---|
| `data/application.json` | Raw agent application (provided by applicant) | Human |
| `data/nipr-data.json` | NIPR license lookup data (optional — agent simulates if absent) | External or test fixture |
| `data/background-check-raw.json` | Raw background check results | Background check vendor |
| `data/screening-report.json` | Structured screening verdict | application-screener |
| `data/license-verification.json` | License verification results | license-verifier |
| `data/background-assessment.json` | Background assessment verdict | background-checker |
| `data/training-plan.json` | Assigned training modules and deadlines | training-coordinator |
| `data/training-completion.json` | Training completion records | training-coordinator / LMS |
| `data/provisioning-record.json` | System access provisioning log | provisioner |
| `documents/compliance-certificate.json` | Final compliance certification | compliance-certifier |
| `documents/appointment-letter.md` | Formal appointment letter | compliance-certifier |
| `logs/onboarding-issue.json` | GitHub issue tracking reference | compliance-certifier |

---

## Environment Variables

See `.env.example` for required variables.

| Variable | Purpose |
|---|---|
| `GITHUB_TOKEN` | GitHub Personal Access Token for the `gh-cli-mcp` server (onboarding issue tracking) |
| `GITHUB_REPO` | Repository for onboarding issue tracking (e.g., `your-org/agent-onboarding`) |

`GITHUB_TOKEN` needs `repo` scope to create issues.

---

## Regulatory Context

This pipeline enforces several insurance regulatory requirements:

- **18 U.S.C. §1033** — Prohibits persons convicted of a crime of dishonesty from engaging in the business of insurance. Background checker enforces this as a hard disqualification.
- **NIPR** — National Insurance Producer Registry. Verifies license status across all 50 states.
- **E&O insurance** — Errors & Omissions coverage is required for all producing agents.
- **State appointment requirements** — Agents must be appointed with each carrier in each state before writing business.
- **CE requirements** — Continuing education credits must be current for each state license.
