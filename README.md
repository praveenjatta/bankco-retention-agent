# 🏦 BankCo Premium Credit Card Customer Retention Agent

A proactive AI agent that identifies at-risk premium credit card customers 30-90 days before their annual renewal date, evaluates 8 behavioral risk signals, matches each customer to the best eligible retention offer, drafts personalized Relationship Manager (RM) emails, logs every decision to an audit trail, and sends daily digest alerts to RMs — all automatically.

---

## 🎯 Problem Statement

Premium credit card customers generate disproportionate revenue, yet many cancel at renewal time due to perceived low value or unresolved service issues. Relationship Managers currently intervene reactively — often too late. This agent solves that by proactively identifying at-risk customers and equipping each RM with a personalized, data-driven retention recommendation 30-90 days before the renewal date.

---

## ⚡ How It Works

```
Daily Trigger → Read 5 Data Sources → Filter Renewal Window → Score Risk Signals → Match Offer → Draft Email → Log Results → Notify RM
```

### Risk Signal Detection

**2-of-N Rule** — 2 or more medium signals = MEDIUM risk:

| Signal | Column | Threshold |
|--------|--------|-----------|
| Low spending | `total_spend_90d` | < $4,000 |
| Travel spend dropped | `travel_share_change` | ≤ -20% |
| Not using lounge | `lounge_visits_90d` | = 0 |
| Not redeeming rewards | `redemptions_90d` | = 0 |
| Disputes filed | `disputes_90d` | ≥ 1 |

**Severe Signal Rule** — 1 severe signal alone = HIGH risk:

| Signal | Column | Severe Threshold |
|--------|--------|-----------------|
| Complaints | `complaints_90d` | ≥ 2 |
| Benefit denials | `benefit_denials_90d` | ≥ 2 |
| Low NPS | `nps_last` | ≤ -20 |

### Risk Priority Levels

| Priority | Criteria | RM Action |
|----------|----------|-----------|
| HIGH | 1+ severe signal | Contact within 48 hours |
| MEDIUM | 2+ medium signals | Contact within 7 days |
| LOW | 1 signal only | Monitor only |
| SAFE | No signals | No action needed |

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Google Sheets (5 Input Tabs)             │
│  ┌────┐ ┌──────────────────┐ ┌────────────────┐            │
│  │ RMs│ │Premium_Customers │ │Spend_Signals_90d│           │
│  └────┘ └──────────────────┘ └────────────────┘            │
│  ┌────────────────────────┐ ┌──────────────┐               │
│  │ Experience_Signals_90d │ │ Offer_Policy │               │
│  └────────────────────────┘ └──────────────┘               │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│              BankCo Retention AI Agent                      │
│                 (Zapier Automation)                         │
│                                                             │
│  Step 1: Read all data from 5 tabs                          │
│  Step 2: Filter customers with renewal in 30-90 days        │
│  Step 3: Apply severe signal rule (complaints/NPS/denials)  │
│  Step 4: Apply 2-of-N rule for medium signals               │
│  Step 5: Match best eligible offer (prefer LOW cost band)   │
│  Step 6: Draft personalized RM email using playbook_copy    │
│  Step 7: Write results to Retention_Recommendations_Log     │
│  Step 8: Send daily digest email to each RM                 │
└─────────────────────────────────────────────────────────────┘
                    │                    │
                    ▼                    ▼
┌──────────────────────────┐  ┌──────────────────────────┐
│  Retention_              │  │  RM Email Notification   │
│  Recommendations_Log     │  │  (Gmail — daily digest)  │
│  (Google Sheets Audit)   │  │                          │
└──────────────────────────┘  └──────────────────────────┘
```

---

## 📊 Google Sheets Structure

**Spreadsheet Name:** `BankCo Premium Retention Data`

| Tab # | Tab Name | Purpose |
|-------|----------|---------|
| 1 | `RMs` | RM contact info |
| 2 | `Premium_Customers` | Customer profile + renewal dates |
| 3 | `Spend_Signals_90d` | Spend and engagement metrics |
| 4 | `Experience_Signals_90d` | Complaints, disputes, NPS |
| 5 | `Offer_Policy` | Retention offers + eligibility rules |
| 6 | `Retention_Recommendations_Log` | Agent output — one row per customer |

### Key Schemas

**Premium_Customers:**
```
cust_id | rm_id | tier | tenure_months | annual_fee_due_date
```

**Spend_Signals_90d:**
```
cust_id | total_spend_90d | lounge_visits_90d | redemptions_90d | travel_share_change
```

**Experience_Signals_90d:**
```
cust_id | complaints_90d | disputes_90d | benefit_denials_90d | nps_last
```

**Offer_Policy:**
```
offer_id | name | eligibility_rule_text | cost_band | playbook_copy
```

**Retention_Recommendations_Log (Output):**
```
run_date | cust_id | rm_id | renewal_window_days | risk_reason_1 | risk_reason_2 | risk_reason_3 | recommended_offer_id | recommended_email_message | message_sent
```

---

## 🎁 Retention Offers Catalog

| Offer ID | Offer Name | Cost Band |
|----------|-----------|-----------|
| OFF001 | Annual Fee Credit (50%) | HIGH |
| OFF002 | Bonus Points (10,000) | MEDIUM |
| OFF003 | Dining Boost (5% for 3 months) | LOW |
| OFF004 | Travel Statement Credit ($150) | MEDIUM |
| OFF005 | Lounge Guest Pass Bundle | LOW |
| OFF006 | Rewards Accelerator (2x points) | MEDIUM |
| OFF007 | Service Recovery Credit ($50) | LOW |
| OFF008 | Retention Specialist Outreach | LOW |

> Agent always prefers LOW cost_band offers over MEDIUM or HIGH when multiple offers are eligible.

---

## 🔧 Zapier Tools Used

| Tool | Purpose |
|------|---------|
| `Google Sheets: Get Many Spreadsheet Rows (Advanced)` | Read all 5 input tabs |
| `Google Sheets: Create Spreadsheet Row` | Write to Retention Log |
| `Gmail: Send Email` | Send RM daily digest notification |

---

## 🔒 Governance & Guardrails

- Agent **NEVER** contacts customers directly — all outreach through RM only
- Agent **NEVER** modifies customer data — read-only on all input tabs
- Every recommendation logged with `run_date`, `cust_id`, and reason codes
- RM reviews and approves every outreach — agent output is advisory only
- Offer eligibility rules enforce guardrails (e.g. no fee credit if disputes > 0)

---

## 🚀 How to Run

1. Set up Google Sheet with 6 tabs and data from Excel file
2. Configure Zapier agent with 3 tools (Google Sheets read, Google Sheets write, Gmail)
3. Add all 6 tabs as Knowledge Sources in Zapier
4. Run agent: type `Run the retention analysis for today`
5. Verify:
   - Retention_Recommendations_Log has new rows
   - RM received daily digest email with at-risk customers
   - Risk signals, offers, and email drafts are correct

---

## 🔄 How to Rerun

1. Clear all rows below Row 1 (headers) and Row 2 (sample row) in Retention_Recommendations_Log
2. Update customer data if needed in the 3 input tabs
3. Run agent again in Zapier Agent Preview

---

## 🛠️ Tech Stack

![Zapier](https://img.shields.io/badge/Zapier-FF4A00?style=flat&logo=zapier&logoColor=white)
![Google Sheets](https://img.shields.io/badge/Google_Sheets-34A853?style=flat&logo=google-sheets&logoColor=white)
![Gmail](https://img.shields.io/badge/Gmail-D14836?style=flat&logo=gmail&logoColor=white)

---

## 👤 Author

**Praveen Kumar Jatta** — AI Automation Consultant & Technical Program Manager

- 🌐 [jattaai.com](https://jattaai.com)
- 💼 [linkedin.com/in/praveenjatta](https://linkedin.com/in/praveenjatta)
- 📧 jattaaihq@gmail.com

---

*Built as part of the EdgeUp for TPMs — Applied Agentic AI program by Interview Kickstart*
