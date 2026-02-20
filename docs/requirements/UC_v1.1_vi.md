# USE CASE SPECIFICATION (UCS)
## Hệ thống Kiểm soát Đầu tư Tài sản Cũ
```
Version : UC v1.1
Owner : Dang
Status : Baseline for Development & QA (Aligned with SR v1.1 + BR v1.1)
Date : 2026-02-19
Related Documents: 
- CR v1.0 (Client-Facing)
- SR v1.1 (Frozen Open Points)
- BR v1.1 (Audit Grade)
```
---

# 1. ACTOR LIST

| Actor | Description |
|--------|------------|
| Owner | Người ra quyết định tài chính cuối cùng |
| Reviewer | Người hỗ trợ kiểm tra |
| Rule Engine | Hệ thống tính toán & đề xuất decision_suggested |
| AI Reviewer | Hệ thống AI phân tích dữ liệu & hình ảnh |
| System | Google Sheets + Apps Script + Logging + Drive Storage |

---

# 2. USE CASE LIST (OVERVIEW)

| UC ID | Name | Primary Actor | Description |
|--------|------|--------------|------------|
| UC-01 | Create Asset | Owner | Tạo bản ghi tài sản |
| UC-02 | Update Asset Data | Owner | Cập nhật dữ liệu tài sản |
| UC-03 | Auto Calculate Financial | Rule Engine | Tính cost, ROI, scrap floor |
| UC-04 | Decision Suggestion | Rule Engine | Đề xuất BUY/NEGOTIATE/SKIP |
| UC-05 | Apply Risk Override | Rule Engine | Override khi risk cao |
| UC-06 | Trigger AI Review | Owner | Gọi AI đánh giá |
| UC-07 | AI Analysis & Advisory | AI Reviewer | Phân tích & advisory |
| UC-08 | Data Completeness & Confidence Gating | System | Gating theo thiếu data/confidence |
| UC-09 | Final Decision Approval | Owner | Chốt decision_final |
| UC-10 | Manual Override | Owner | Override có reason |
| UC-11 | Logging & Audit Trail (CRITICAL_ONLY) | System | Audit & governance logging |
| UC-12 | Change Configuration | Owner | Thay đổi threshold |
| UC-13 | Image Upload & Link | Owner | Upload ảnh & link vào asset |
| UC-14 | Image Validation & Folder Convention | System | Validate & lưu đúng cấu trúc |
| UC-15 | Image Access Control & Retention | System | Permission & retention 1 năm |
| UC-16 | Large Deal AI Enforcement | System | Deal ≥ 300k bắt buộc AI |
| UC-17 | ART Provenance Enforcement | System | ART ≥ 300k bắt buộc provenance |

---

# 3. DETAILED USE CASES

---

## UC-01: Create Asset

**Primary Actor:** Owner  
**Precondition:** System hoạt động

### Main Flow
1. Owner nhập thông tin asset.
2. System validate asset_id unique.
3. Lưu record vào ASSETS.
4. Trigger UC-03 và UC-04.

### Postcondition
- Asset được lưu bền vững.
- decision_suggested được tạo.

### Exception
- asset_id trùng → block lưu.

---

## UC-02: Update Asset Data

**Primary Actor:** Owner

### Main Flow
1. Owner sửa dữ liệu.
2. System validate (không âm, đúng type).
3. Recalculate (UC-03, UC-04).
4. Nếu thay đổi critical field → trigger UC-11.

### Critical Fields
- purchase_price
- evaluation_mode
- risk_score
- decision_final

---

## UC-03: Auto Calculate Financial

**Primary Actor:** Rule Engine

### Logic
- total_cost = purchase_price + dismantle_cost + transport_cost + hazardous_cost
- expected_profit = resale_est - total_cost
- roi = expected_profit / total_cost (null nếu total_cost = 0)
- SCRAP:
  - scrap_total_value = weight_est_kg × scrap_price_per_kg
  - net_scrap_floor = scrap_total_value - dismantle_cost - transport_cost - hazardous_cost

---

## UC-04: Decision Suggestion

### RESALE
- ROI ≥ roi_buy_threshold_resale → BUY
- ROI ≥ roi_negotiate_threshold_resale → NEGOTIATE
- Else → SKIP

### SCRAP
- purchase_price > net_scrap_floor → SKIP
- purchase_price ≤ scrap_buy_ratio × net_scrap_floor → BUY
- Else → NEGOTIATE

### ART
- ROI ≥ roi_buy_threshold_art AND provenance valid → BUY
- ROI ≥ roi_negotiate_threshold_resale → NEGOTIATE
- Else → SKIP

### Applied Rules
- DGR-01 / DGR-02 (thiếu data → không BUY)
- RR-01 (risk override)

---

## UC-05: Apply Risk Override

- If risk_score ≥ risk_skip_threshold
- decision_suggested = SKIP (override)

---

## UC-06: Trigger AI Review

### Preconditions
- Chọn đúng 1 asset
- ai_enabled = true

### Flow
1. Owner nhấn Evaluate.
2. Image Requirement Gate:
   - image_required_for_ai = true:
     - AT_LEAST_ONE → phải có ≥1 ảnh
     - BOTH → phải có đủ 2 ảnh
3. Build payload.
4. Call AI API.
5. Retry 1 lần nếu JSON invalid.
6. Lưu ai_confidence, ai_last_review_at, ai_summary.
7. Trigger UC-11 log AI event.

---

## UC-07: AI Analysis & Advisory

### AI Output Contract
- summary
- risks
- recommendation
- confidence (1–5)

### AI Tasks
- OCR nameplate
- Condition analysis
- Data completeness check
- Market advisory (no realtime crawling)
- Counter-offer suggestion

---

## UC-08: Data Completeness & Confidence Gating

### Mandatory Fields

| Mode | Required |
|------|----------|
| RESALE | purchase_price, resale_est |
| SCRAP | purchase_price, weight_est_kg, scrap_price_per_kg |
| ART | purchase_price, resale_est, provenance (if required) |

### Rules
- Missing mandatory → không BUY
- Nếu AI chạy → ai_confidence ≤ 2
- confidence ≤ 2 → không BUY

---

## UC-09: Final Decision Approval

### Gate 1 — Large Deal
- purchase_price ≥ large_deal_threshold_jpy
- ai_last_review_at must exist

### Gate 2 — ART Provenance
- ART AND purchase_price ≥ art_provenance_threshold_jpy
- provenance required
- thiếu → block BUY

### Flow
1. Owner chọn decision_final.
2. System validate gates.
3. Lưu decision_final + approver + timestamp.
4. Snapshot rule_version.
5. Trigger UC-11.

---

## UC-10: Manual Override

- decision_final ≠ decision_suggested
- decision_reason bắt buộc
- Nếu trống → block

---

## UC-11: Logging & Audit Trail (CRITICAL_ONLY)

### MUST Audit
- purchase_price
- evaluation_mode
- risk_score
- decision_final

### MUST Log
- AI Evaluate
- Threshold change
- rule_version snapshot

### Fail-safe
- Logging failure không làm mất data chính.

---

## UC-12: Change Configuration

- Owner chỉnh threshold.
- Áp dụng realtime cho decision mới.
- Log threshold change.

---

## UC-13: Image Upload & Link

- Upload nameplate / overall.
- Link đúng asset_id.
- Reload sheet vẫn tồn tại.

---

## UC-14: Image Validation & Folder Convention

### Validate
- File là ảnh hợp lệ
- Không rỗng
- Không vượt size limit

### Folder Structure

# 4. CHANGE LOG

| Version | Date       | Description of Change | Author |
|---------|------------|-----------------------|--------|
| v1.0 | 2026-02-19 | Initial draft | Dang |
| v1.1 | 2026-02-20 | Update for open points, image handle | Dang |