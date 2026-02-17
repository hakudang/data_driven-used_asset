# SYSTEM REQUIREMENT (SR)
## Data-Driven Used Asset Investment Control System
### Based on CR v1.3 FINAL – Enterprise Ready

```text
Version: v1.1
Source: Client Requirement v1.3 FINAL
Owner: Dang
Document Type: System Requirement (SR)
Status: Technical Baseline
Region: Japan
```

---

# 1. PURPOSE

This document translates the approved Client Requirement (CR v1.3 FINAL) into technical system specifications.

This SR defines:

- System architecture
- Data structure
- Calculation logic
- AI interaction flow
- Validation rules
- Error handling
- Security constraints
- Logging & audit behavior

---

# 2. SYSTEM ARCHITECTURE

## 2.1 High-Level Architecture

Layer 1: Google Sheets Rule Engine  
Layer 2: Google Apps Script (Backend Logic)  
Layer 3: OpenAI API (AI Reviewer)  
UI Layer: Sidebar UX (Apps Script HTML Service)

System Flow:

User → Google Sheet → Apps Script → AI API → Sidebar Output → Sheet Update

---

# 3. DATA STRUCTURE

## 3.1 Sheet: ASSETS

Minimum required columns:

| Field | Type | Description |
|-------|------|-------------|
| asset_id | string | Unique identifier |
| category | enum | IT / AGRI / CNC / ART |
| evaluation_mode | enum | RESALE / SCRAP / ART |
| purchase_price | number |
| resale_est | number |
| export_est | number |
| weight_est_kg | number |
| scrap_price_per_kg | number |
| dismantle_cost | number |
| transport_cost | number |
| hazardous_cost | number |
| risk_score | number |
| decision_suggested | enum |
| decision_final | enum |
| image_nameplate_url | string |
| image_overall_url | string |
| ai_confidence | number |
| ai_last_review_date | datetime |

---

## 3.2 Sheet: CONFIG

| Field | Type |
|-------|------|
| default_scrap_price |
| roi_buy_threshold |
| roi_negotiate_threshold |
| art_roi_threshold |
| risk_auto_skip_threshold |

---

## 3.3 Sheet: AI_LOG

| Field | Description |
|-------|------------|
| log_id |
| asset_id |
| timestamp |
| ai_output_summary |
| confidence |
| reviewer |
| version |

---

# 4. RULE ENGINE LOGIC

## 4.1 Total Cost

total_cost = purchase_price + dismantle_cost + transport_cost + hazardous_cost

---

## 4.2 Expected Profit (Resale Mode)

expected_profit = resale_est - total_cost

ROI = expected_profit / total_cost

---

## 4.3 Scrap Mode

scrap_total_value = weight_est_kg × scrap_price_per_kg

net_scrap_floor = scrap_total_value - dismantle_cost - transport_cost - hazardous_cost

Decision Rules:

IF purchase_price > net_scrap_floor → SKIP  
IF purchase_price ≤ 0.8 × net_scrap_floor → BUY  
ELSE → NEGOTIATE

---

## 4.4 Risk Rule

IF risk_score ≥ risk_auto_skip_threshold → AUTO SKIP

---

# 5. AI REVIEWER (MODEL A)

## 5.1 Trigger Condition

- Active sheet = ASSETS
- Exactly one row selected
- User clicks "Evaluate"

---

## 5.2 AI Input Payload

The following data is sent to AI:

- Financial data
- Evaluation mode
- Risk score
- Scrap floor values
- Image URLs (if provided)

---

## 5.3 AI Output Format (Strict JSON Required)

{
  "data_completeness": "...",
  "calculation_check": "...",
  "vision_nameplate": "...",
  "vision_condition": "...",
  "market_advisory": "...",
  "decision_review": "...",
  "counter_offer": "...",
  "confidence": 1-5,
  "top_risks": ["...", "..."]
}

---

# 6. VISION MODULE

## 6.1 Nameplate OCR

System must:

- Extract maker/model/serial/year
- Compare with sheet values
- Flag mismatch

## 6.2 Overall Image Analysis

System must:

- Detect major visible damage
- Detect missing components
- Estimate risk severity
- Suggest required additional photos

---

# 7. DATA COMPLETENESS CONTROL

AI must evaluate:

Critical Missing Data:

- Missing purchase price
- Missing evaluation mode
- Missing weight in scrap mode
- Missing provenance (art mode)

If critical missing:

- confidence ≤ 2
- system must block BUY suggestion

---

# 8. CONFIDENCE GATING RULE

If confidence ≤ 2:

- decision_review cannot recommend BUY
- must state verification required

---

# 9. ERROR HANDLING

## 9.1 AI API Failure

- System must not block sheet usage
- Display: "AI unavailable"
- Log error in AI_LOG

## 9.2 Invalid JSON from AI

- Retry once
- If still invalid → fail gracefully

## 9.3 Missing Image Access

- Flag: "Image not accessible"

---

# 10. SECURITY REQUIREMENTS

- API key stored in User Properties
- No API key stored in sheet
- No external marketplace crawling
- No automatic execution

---

# 11. PERFORMANCE REQUIREMENTS

- AI response time target ≤ 10 seconds
- Sheet calculation must remain responsive
- No batch processing in v1.1

---

# 12. AUDIT & LOGGING

Each AI evaluation must:

- Write to AI_LOG
- Store confidence
- Store timestamp
- Store summarized output

---

# 13. DEPLOYMENT MODEL

Phase 1: Google Sheet + Apps Script  
Phase 2: Vision Enabled  
Phase 3: Logging Enabled  
Future: Web App migration

---

# 14. LEGAL & RESPONSIBILITY

AI output is advisory only.  
Final financial responsibility remains with Owner.

---

# 15. ACCEPTANCE CRITERIA

System is accepted when:

- All calculation rules function correctly
- AI returns structured output
- Confidence gating works
- BUY is blocked when risk ≥ threshold
- BUY is blocked when data incomplete
- AI log captures each evaluation

---

# END OF SYSTEM REQUIREMENT
