# SYSTEM REQUIREMENT (SR)
## Hệ thống Kiểm soát Đầu tư Tài sản Cũ  
### SR v1.1 – Frozen Open Points + Image Handling Update (2026-02-19)

```text
Owner   : Dang
Version : v1.1
Status  : Product-Scale Baseline – Ready for Development & QA (Open Points Frozen + Image Handling Defined)
Region  : Japan
Date    : 2026-02-19
Updated : 2026-02-19 08:01:09
```

---

# 1. PHẠM VI TÀI LIỆU

Tài liệu này là đặc tả kỹ thuật cấp sản phẩm, bao gồm:

- Kiến trúc hệ thống
- Data model & contract
- Functional Requirements (FR) – có Acceptance chi tiết cho từng FR
- Non-Functional Requirements (NFR) – có Acceptance chi tiết cho từng NFR
- Logging/Audit, Error handling, Security, Scalability
- Các quyết định “Freeze” cho Open Points của SR v1.0
- Chuẩn hóa Image Handling (upload, validate, link) cho AI Review

---

# 2. KIẾN TRÚC

- Layer 1: Google Sheets (Rule Engine / tính toán)
- Layer 2: Apps Script (backend trigger, payload builder, logging, image handler)
- Layer 3: AI Reviewer API (Vision + Advisory)
- UI: Sidebar UX (hiển thị kết quả AI)
- Future: Web App/SaaS (migrate)

---

# 3. DATA MODEL (TỐI THIỂU)

## 3.1 ASSETS (bảng dữ liệu)

| Tên cột | Loại dữ liệu | Mô tả | Bắt buộc | Tự động |
|---------|--------------|-------|----------|---------|
| asset_id | string | ID duy nhất của tài sản | Có | Không |
| category | enum (IT/AGRI/CNC/ART/OTHER) | Phân loại tài sản | Có | Không |
| evaluation_mode | enum (RESALE/SCRAP/ART) | Chế độ đánh giá | Có | Không |
| purchase_price | number | Giá mua ban đầu | Có | Không |
| resale_est | number | Giá ước tính khi bán lại (RESALE/ART) | Tùy mode | Không |
| weight_est_kg | number | Trọng lượng ước tính (SCRAP) | Tùy mode | Không |
| scrap_price_per_kg | number | Giá phế liệu/kg (SCRAP) | Tùy mode | Không |
| dismantle_cost | number | Chi phí tháo dỡ | Không | Không |
| transport_cost | number | Chi phí vận chuyển | Không | Không |
| hazardous_cost | number | Chi phí xử lý chất thải nguy hại | Không | Không |
| risk_score | number (0–100) | Điểm rủi ro tổng hợp | Có | Không |
| decision_suggested | enum (BUY/NEGOTIATE/SKIP) | Đề xuất quyết định từ Rule Engine | Có | Có |
| decision_final | enum (BUY/NEGOTIATE/SKIP) | Quyết định cuối cùng sau review | Có | Không |
| decision_reason | string | Lý do khi override decision_suggested | Khi override | Không |
| image_nameplate_url | string | URL ảnh nameplate (nếu có) | Không | Không |
| image_overall_url | string | URL ảnh tổng thể (nếu có) | Không | Không |
| ai_confidence | number (1–5) | Điểm confidence từ AI Reviewer | Không | Có |
| ai_last_review_at | datetime | Timestamp lần đánh giá AI cuối cùng | Không | Có |
| ai_summary | string | Tóm tắt phân tích AI để truy vết nhanh | Không | Có |
| rule_version | string | Phiên bản rule engine tại thời điểm chốt decision | Có | Không |

---

## 3.2 CONFIG (bảng cấu hình)

| Tên tham số | Loại dữ liệu | Mô tả | Default | Bắt buộc |
|-------------|--------------|-------|---------|----------|
| roi_buy_threshold_resale | number (0–1) | Ngưỡng ROI để đề xuất BUY trong RESALE mode | 0.25 | Có |
| roi_negotiate_threshold_resale | number (0–1) | Ngưỡng ROI để đề xuất NEGOTIATE trong RESALE mode | 0.15 | Có |
| roi_buy_threshold_art | number (0–1) | Ngưỡng ROI để đề xuất BUY trong ART mode | 0.40 | Có |
| risk_skip_threshold | number (0–100) | Ngưỡng risk để override thành SKIP | 80 | Có |
| scrap_buy_ratio | number (0–1) | Tỷ lệ so với scrap floor để đề xuất BUY trong SCRAP mode | 0.80 | Có |
| rule_version | string | Phiên bản hiện tại của rule engine | v1.1 | Có |
| ai_enabled | boolean | Bật/tắt tính năng AI Reviewer | true | Có |
| retry_on_invalid_json | number (0–5) | Số lần retry khi nhận JSON lỗi từ AI | 1 | Có |
| large_deal_threshold_jpy | number | Ngưỡng deal lớn để bắt buộc AI review | 300000 | Có |
| art_provenance_threshold_jpy | number | Ngưỡng để bắt buộc provenance ở ART | 300000 | Có |
| image_retention_years | number | Thời gian retention ảnh (năm) | 1 | Có |
| audit_scope_mode | enum (CRITICAL_ONLY/FULL_HISTORY) | Chế độ audit trail | CRITICAL_ONLY | Có |
| image_folder_root | string | Drive folder root để lưu ảnh | /Asset-Investment-Control | Có |
| image_required_for_ai | boolean | Nếu true, bắt buộc có ít nhất 1 ảnh trước Evaluate | true | Có |
| image_min_required_set | enum (NAMEPLATE_ONLY/OVERALL_ONLY/BOTH/AT_LEAST_ONE) | Bộ ảnh tối thiểu khi Evaluate | AT_LEAST_ONE | Có |

---

# 4. CÔNG THỨC & LOGIC CORE (NORMATIVE)

## 4.1 Total Cost
total_cost = purchase_price + dismantle_cost + transport_cost + hazardous_cost  
- Nếu cost trống → coi như 0

## 4.2 Expected Profit (RESALE/ART)
expected_profit = resale_est - total_cost

## 4.3 ROI
roi = expected_profit / total_cost  
- Nếu total_cost = 0 → roi = null (không phát sinh #DIV/0!)

## 4.4 Scrap Mode
scrap_total_value = weight_est_kg × scrap_price_per_kg  
net_scrap_floor = scrap_total_value - dismantle_cost - transport_cost - hazardous_cost  

Decision (SCRAP):
- purchase_price > net_scrap_floor → SKIP
- purchase_price ≤ scrap_buy_ratio × net_scrap_floor → BUY
- else → NEGOTIATE

## 4.5 Risk Override (ưu tiên cao nhất)
Nếu risk_score ≥ risk_skip_threshold → decision_suggested = SKIP (override mọi mode)

---

# 5. FUNCTIONAL REQUIREMENTS (FR)

## 5.1 Rule Engine (FR-01 → FR-12)
Giữ nguyên như SR v1.0 / SR v1.1.

## 5.2 AI Reviewer – Model A (FR-13 → FR-24)
Giữ nguyên như SR v1.0 / SR v1.1.

## 5.3 Governance & Audit (FR-25 → FR-35)
Giữ nguyên như SR v1.1 (đã freeze open points).

---

## 5.4 IMAGE HANDLING – NEW FR (v1.1)

Mục tiêu: chuẩn hóa quy trình upload, validate, link ảnh để AI review và audit vận hành ổn định.

| ID | Mô tả ngắn | Mô tả chi tiết |
|----|------------|----------------|
| FR-36 | Image Upload UI Action | Trong Sidebar, cung cấp action Upload cho 2 loại ảnh: nameplate và overall. Upload xong → lưu file vào Drive và ghi URL vào ASSETS. |
| FR-37 | Image Validation and Linking | Validate file trước khi lưu: loại file ảnh hợp lệ, dung lượng tối đa, không rỗng. Lưu xong → set image_nameplate_url hoặc image_overall_url đúng record asset_id hiện tại. |
| FR-38 | Image Requirement Gate for Evaluate | Khi nhấn Evaluate, nếu image_required_for_ai = true thì kiểm tra bộ ảnh tối thiểu theo image_min_required_set. Không đạt → block gọi AI và hiển thị checklist cần bổ sung. |
| FR-39 | Image Folder Convention | Khi lưu ảnh, hệ thống tạo path chuẩn theo: image_folder_root / YYYY / asset_id /. Tên file phải chứa asset_id và loại ảnh để truy vết. |
| FR-40 | Image Access Control | File ảnh phải set permission theo policy: Owner full, Reviewer view, không public link mặc định. Nếu không set được permission → vẫn lưu URL nhưng hiển thị cảnh báo compliance và log event. |

---

# 6. ACCEPTANCE CRITERIA – CHI TIẾT THEO TỪNG FR

## FR-01 → FR-35
Giữ nguyên như SR v1.1.

---

## NEW FR – IMAGE HANDLING

### FR-36: Image Upload UI Action
Acceptance:
- Upload nameplate → image_nameplate_url được set đúng dòng asset_id.
- Upload overall → image_overall_url được set đúng dòng asset_id.
- Reload sheet → URL vẫn tồn tại và file vẫn truy cập được theo quyền.

### FR-37: Image Validation and Linking
Acceptance:
- Upload file không phải ảnh hoặc file rỗng → bị chặn và báo lỗi rõ.
- Upload ảnh vượt giới hạn dung lượng → bị chặn và báo lỗi rõ.
- Upload hợp lệ → URL ghi đúng cột, đúng asset_id, không ghi nhầm dòng.

### FR-38: Image Requirement Gate for Evaluate
Acceptance:
- image_required_for_ai = true và image_min_required_set = AT_LEAST_ONE
  - Không có cả 2 URL → block Evaluate, không gọi AI, checklist yêu cầu upload.
  - Có 1 trong 2 URL → cho phép Evaluate.
- image_min_required_set = BOTH
  - Thiếu 1 loại → block Evaluate và checklist rõ.

### FR-39: Image Folder Convention
Acceptance:
- Upload ảnh → file nằm đúng path: image_folder_root / YYYY / asset_id /
- Tên file chứa asset_id và tag nameplate hoặc overall.

### FR-40: Image Access Control
Acceptance:
- Người không có permission → không truy cập được file ảnh.
- Owner truy cập được.
- Nếu permission set thất bại → UI hiển thị cảnh báo compliance và có log.

---

# 7. NON-FUNCTIONAL REQUIREMENTS (NFR)

Giữ nguyên NFR theo SR v1.1.

---

# 8. OPEN POINTS – FROZEN OPTIONS (2026-02-19)

Giữ nguyên như SR v1.1.

---

# 9. CHANGE LOG

| Version | Date | Description of Change | Author |
|---------|------|------------------------|--------|
| v1.0 | 2025-12-01 | Initial SR baseline with open points. | Dang |
| v1.1 | 2026-02-19 | Freeze open points, add configs for thresholds, audit scope, retention. | Dang |
| v1.1 | 2026-02-19 | Add Image Handling FR-36 to FR-40 with acceptance. | Dang |

---

# END OF DOCUMENT
