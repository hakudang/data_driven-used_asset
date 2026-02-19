# 🧠 Asset Investment Control System

[![Version](https://img.shields.io/badge/version-v1.0-blue.svg)]()
[![Status](https://img.shields.io/badge/status-product--baseline-green.svg)]()
[![Architecture](https://img.shields.io/badge/architecture-Sheets%20%2B%20AppsScript-orange.svg)]()
[![Governance](https://img.shields.io/badge/governance-enterprise%20ready-purple.svg)]()
[![Region](https://img.shields.io/badge/region-Japan-red.svg)]()

---

## 📌 Overview

**Asset Investment Control System** là hệ thống hỗ trợ ra quyết định đầu tư tài sản cũ, tập trung vào:

- Chuẩn hóa logic tài chính
- Kiểm soát rủi ro nghiêm ngặt
- Giảm mua theo cảm tính
- AI advisory có kiểm soát
- Audit & traceability đầy đủ

> ⚠️ Hệ thống chỉ hỗ trợ phân tích. Quyết định cuối cùng thuộc về Owner.

---

## 🎯 Business Objectives

- ROI trung bình ≥ 25%
- Tỷ lệ lỗ < 10%
- Không BUY khi risk cao
- Không override thiếu lý do
- 100% quyết định truy vết được

---

## 🏗 Architecture
```
┌─────────────────────────────┐
│ Google Sheets (Rule Engine) │
│ - Financial Calculation │
│ - ROI / Scrap Floor │
│ - Decision Suggestion │
└──────────────┬──────────────┘
│
┌──────────────▼──────────────┐
│ Apps Script Backend │
│ - AI Trigger │
│ - Payload Builder │
│ - Logging / Audit │
│ - Error Handling │
└──────────────┬──────────────┘
│
┌──────────────▼──────────────┐
│ AI Reviewer API │
│ - OCR │
│ - Condition Analysis │
│ - Market Advisory │
│ - Confidence Score │
└─────────────────────────────┘
```
---

## 📂 Project Structure
```
data_driven-used_asset/
│
├── README.md
├── docs/
│ ├── CR_v1.0.md
│ ├── SR_v1.2_ENTERPRISE.md
│ ├── BR_v1.0_Asset_Investment_Control.md
│ ├── RTM_v1.0.md
│ ├── UC_v1.0.md
│ ├── UC_Execution_Matrix_v1.0.md
│ └── CHANGELOG.md
│
├── sheets/
│ ├── ASSETS_template.xlsx
│ └── CONFIG_template.xlsx
│
├── apps_script/
│ ├── Code.gs
│ ├── ai_adapter.js
│ ├── logging.js
│ └── sidebar.html
│
├── tests/
│ ├── test_case_spec.md
│ └── regression_checklist.md
│
└── LICENSE
```


---

## 🧮 Core Features

### 1️⃣ Financial Engine

- Total Cost Calculation
- Expected Profit
- ROI
- Scrap Net Floor

### 2️⃣ Decision Engine

Modes:
- RESALE
- SCRAP
- ART

Outputs:
- BUY
- NEGOTIATE
- SKIP

Priority:
1. Financial Rules
2. Mode Decision
3. Risk Override
4. Data Completeness
5. AI Confidence Gating

---

### 3️⃣ Risk Control

IF risk_score ≥ threshold
THEN decision_suggested = SKIP

Risk override luôn có ưu tiên cao nhất.

---

### 4️⃣ AI Advisory (Governed)

- Advisory only
- Confidence gating
- Không crawl realtime marketplace
- Output contract-based
- Retry 1 lần nếu JSON lỗi

---

### 5️⃣ Governance & Audit

- Manual override bắt buộc reason
- Rule version snapshot khi chốt decision
- Logging mọi thay đổi quan trọng
- Threshold configurable (không hard-code)

---

## 📊 Decision Logic Summary

### RESALE

| Condition | Decision |
|-----------|----------|
| ROI ≥ buy_threshold | BUY |
| ROI ≥ negotiate_threshold | NEGOTIATE |
| Else | SKIP |

### SCRAP

| Condition | Decision |
|-----------|----------|
| price > net_scrap_floor | SKIP |
| price ≤ scrap_ratio × net_scrap_floor | BUY |
| Else | NEGOTIATE |

### Global Override

- Risk ≥ threshold → SKIP
- Confidence ≤ 2 → Không BUY

---

## 🔐 Security

- API key lưu trong User Properties
- Không lưu secret trong Sheet hoặc repo
- Không crawl marketplace realtime
- Audit trail bắt buộc

---

## ⚡ Performance Target

- Median Evaluate ≤ 10s
- p95 ≤ 20s

---

## 🔄 Change Management

Khi thay đổi Business Rule:

1. Tăng rule_version
2. Update BR Spec
3. Update SR
4. Update RTM
5. Regression test
6. Update CHANGELOG

---

## 🚀 Roadmap

- Web App migration
- SaaS multi-tenant
- Batch AI review
- Advanced risk scoring
- ART provenance policy engine

---

## 🧾 License

Internal Governance Product  
Not open for public financial reliance.

---

## 👤 Maintainer

Mr.Dang    

---

## 🛡 Governance Guarantee

Hệ thống đảm bảo:

- Không BUY khi risk cao
- Không BUY khi confidence thấp
- Không override không lý do
- Không thay đổi rule không version
- Không mất audit trace

---

# End of README
