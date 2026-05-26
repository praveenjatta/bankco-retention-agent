# Google Sheets Setup — BankCo Premium Customer Retention Agent

## Spreadsheet Name
`BankCo Premium Retention Data`

## Tab 1 — RMs

**Headers:** rm_id | rm_name | rm_email | rm_phone | region

**Sample data:**
| RM001 | Sarah Johnson | sarah.j@bankco.com | 555-0101 | West |
| RM002 | Michael Chen | michael.c@bankco.com | 555-0102 | East |

## Tab 2 — Premium_Customers

**Headers:** cust_id | rm_id | cust_name | tier | tenure_months | annual_fee_due_date | annual_fee_amount

**Sample data:**
| CUST001 | RM001 | James Wilson | Platinum | 36 | 2026-06-15 | 550 |
| CUST002 | RM001 | Emily Davis | Signature | 24 | 2026-07-02 | 695 |

## Tab 3 — Spend_Signals_90d

**Headers:** cust_id | total_spend_90d | lounge_visits_90d | redemptions_90d | travel_share_change

**Sample data:**
| CUST001 | 3200.00 | 0 | 0 | -25 |
| CUST002 | 8500.00 | 3 | 2 | 5 |

## Tab 4 — Experience_Signals_90d

**Headers:** cust_id | complaints_90d | disputes_90d | benefit_denials_90d | nps_last

**Sample data:**
| CUST001 | 0 | 1 | 0 | -5 |
| CUST002 | 2 | 0 | 0 | -25 |

## Tab 5 — Offer_Policy

**Headers:** offer_id | name | eligibility_rule_text | cost_band | playbook_copy

**Sample data:**
| OFF001 | Annual Fee Credit 50% | All Platinum+ customers | HIGH | We would like to offer you a 50% credit on your annual fee as a thank you for your loyalty... |
| OFF003 | Dining Boost 5% | All customers | LOW | Enjoy 5% back on dining for the next 3 months as our way of saying thank you... |
| OFF007 | Service Recovery Credit $50 | Customers with complaints >= 1 | LOW | We sincerely apologize for your recent experience and would like to offer you a $50 statement credit... |
| OFF008 | Retention Specialist Outreach | All customers | LOW | Our premium retention team would love to connect with you personally to ensure you're getting maximum value... |

## Tab 6 — Retention_Recommendations_Log (Output)

**Headers:** run_date | cust_id | rm_id | renewal_window_days | risk_reason_1 | risk_reason_2 | risk_reason_3 | recommended_offer_id | recommended_email_message | message_sent

Leave empty — agent writes here automatically.

## Zapier Tools to Add

| Tool | Purpose |
|------|---------|
| Google Sheets: Get Many Spreadsheet Rows (Advanced) | Read all 5 input tabs |
| Google Sheets: Create Spreadsheet Row | Write to Retention_Recommendations_Log |
| Gmail: Send Email | Send daily digest to each RM |
