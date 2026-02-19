# SYSTEM REQUIREMENT (SR)
## Hệ thống Kiểm soát Đầu tư Tài sản Cũ 

- FR/NFR + Acceptance chi tiết cho từng yêu cầu

```
Owner: Dang  
Version: v1.0  
Status: Product-Scale Baseline – Ready for Development & QA  
Region: Japan  
```
---

# 1. PHẠM VI TÀI LIỆU

Tài liệu này là đặc tả kỹ thuật cấp sản phẩm, bao gồm:
- Kiến trúc hệ thống
- Data model & contract
- Functional Requirements (FR) – có Acceptance chi tiết cho từng FR
- Non-Functional Requirements (NFR) – có Acceptance chi tiết cho từng NFR
- Logging/Audit, Error handling, Security, Scalability

---

# 2. KIẾN TRÚC

- Layer 1: Google Sheets (Rule Engine / tính toán)
- Layer 2: Apps Script (backend trigger, payload builder, logging)
- Layer 3: AI Reviewer API (Vision + Advisory)
- UI: Sidebar UX (hiển thị kết quả AI)
- Future: Web App/SaaS (migrate)

---

# 3. DATA MODEL (TỐI THIỂU)

## 3.1 ASSETS (bảng dữ liệu)
Các cột tối thiểu để hệ thống chạy đúng:

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

## 3.2 CONFIG (bảng cấu hình)
Các tham số cấu hình có thể thay đổi theo thời gian mà không cần sửa code:
| Tên tham số | Loại dữ liệu | Mô tả | Default | Bắt buộc |
|-------------|--------------|-------|---------|----------|
| roi_buy_threshold_resale | number (0–1) | Ngưỡng ROI để đề xuất BUY trong RESALE mode | 0.25 | Có |
| roi_negotiate_threshold_resale | number (0–1) | Ngưỡng ROI để đề xuất NEGOTIATE trong RESALE mode | 0.15 | Có |
| roi_buy_threshold_art | number (0–1) | Ngưỡng ROI để đề xuất BUY trong ART mode | 0.40 | Có |
| risk_skip_threshold | number (0–100) | Ngưỡng risk để override thành SKIP | 80 | Có |
| scrap_buy_ratio | number (0–1) | Tỷ lệ so với scrap floor để đề xuất BUY trong SCRAP mode | 0.80 | Có |
| rule_version | string | Phiên bản hiện tại của rule engine | "v1.0" | Có |
| ai_enabled | boolean | Bật/tắt tính năng AI Reviewer | true | Có |
| retry_on_invalid_json | number (0–5) | Số lần retry khi nhận JSON lỗi từ AI | 1 | Có |

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

## 5.1 Rule Engine

- Rule Engine tự động tính toán và đề xuất decision_suggested theo logic đã định nghĩa ở phần 4. 
- Các cột auto (total_cost, expected_profit, roi, net_scrap_floor, decision_suggested) được cập nhật tự động khi có thay đổi dữ liệu liên quan.

| ID | Mô tả ngắn | Mô tả chi tiết |
|----|------------|----------------|
| FR-01 | Create Asset | Tạo bản ghi tài sản với asset_id duy nhất và lưu bền vững trong ASSETS. |
| FR-02 | Update Asset | Cho phép cập nhật dữ liệu asset mà không phá công thức/format các cột auto. |
| FR-03 | Calculate Total Cost | Tự động tính total_cost theo công thức chuẩn và default 0 cho ô trống. |
| FR-04 | Calculate Expected Profit | Tự động tính expected_profit theo resale_est và total_cost. |
| FR-05 | Calculate ROI | Tự động tính roi; xử lý chia 0 an toàn. |
| FR-06 | Calculate Scrap Floor | Tính scrap_total_value và net_scrap_floor trong SCRAP mode. |
| FR-07 | Decision Engine Resale | Đề xuất BUY/NEGOTIATE/SKIP theo ngưỡng ROI (RESALE). |
| FR-08 | Decision Engine Scrap | Đề xuất BUY/NEGOTIATE/SKIP theo net_scrap_floor (SCRAP). |
| FR-09 | Decision Engine Art | Đề xuất theo ROI ngưỡng ART và rule provenance (nếu áp dụng). |
| FR-10 | Risk Override | Risk ≥ threshold → auto SKIP (ưu tiên cao nhất). |
| FR-11 | Mode Switching | Khi đổi evaluation_mode, hệ thống áp dụng đúng logic tương ứng. |
| FR-12 | Configurable Threshold | Ngưỡng ROI/Risk/Scrap ratio cấu hình qua CONFIG và áp dụng realtime. |

---

## 5.2 AI Reviewer – Model A (Selected Row)

AI Reviewer được trigger khi người dùng chọn đúng 1 dòng và nhấn Evaluate. 
- Hệ thống build payload gồm dữ liệu tài chính + mode + risk + image URLs, sau đó gọi API AI để nhận về phân tích hình ảnh, đánh giá độ đầy đủ dữ liệu, khuyến nghị thị trường tham khảo, counter-offer suggestion, và confidence score. 
- AI chỉ đóng vai trò hỗ trợ, không thay thế quyết định cuối cùng. 
- Confidence thấp sẽ bị gating không được khuyến nghị BUY. 
- Kết quả AI được hiển thị rõ ràng trong sidebar, kèm theo cảnh báo nếu có thiếu dữ liệu hoặc rủi ro cao. 
- Mỗi lần Evaluate đều được ghi log đầy đủ để truy vết.

| ID | Mô tả ngắn | Mô tả chi tiết |
|----|------------|----------------|
| FR-13 | Trigger Evaluate | AI chỉ chạy khi người dùng chọn đúng 1 dòng và nhấn Evaluate. |
| FR-14 | Payload Builder | Build payload gồm dữ liệu tài chính + mode + risk + image URLs. |
| FR-15 | OCR Nameplate | Nếu có nameplate, OCR trích xuất maker/model/serial/year (nếu thấy). |
| FR-16 | Visual Condition Analysis | Phân tích ảnh tổng thể để phát hiện hư hỏng/thiếu bộ phận/red flags. |
| FR-17 | Data Completeness Check | Đánh giá thiếu dữ liệu theo mode và yêu cầu bổ sung ảnh/spec. |
| FR-18 | Market Advisory | Trả estimated range (không crawl realtime) + liquidity + keywords. |
| FR-19 | Counter Offer Suggestion | Đề xuất counter-offer dựa trên ROI/risk/scrap floor. |
| FR-20 | AI Output Contract | Output theo contract (section bắt buộc + confidence + risks). |
| FR-21 | Confidence Score | Trả confidence (1–5) và lưu vào ai_confidence. |
| FR-22 | Confidence Gating | Confidence thấp không được khuyến nghị BUY; phải verify. |
| FR-23 | Sidebar Rendering | Sidebar hiển thị theo section rõ ràng, có cảnh báo. |
| FR-24 | AI Logging | Mỗi Evaluate ghi AI_LOG: timestamp, asset_id, confidence, summary. |

---

## 5.3 Governance & Audit

- Governance & Audit là yêu cầu bắt buộc để đảm bảo tính minh bạch, truy vết, và kiểm soát rủi ro trong toàn bộ quá trình ra quyết định đầu tư tài sản cũ. 
- Các yêu cầu này bao gồm việc ghi nhận decision_final cùng approver và timestamp, tạo audit trail cho các thay đổi quan trọng, tracking version của rule khi chốt decision, kiểm soát manual override bằng lý do bắt buộc, lưu tóm tắt AI để truy vết nhanh, và thiết kế sẵn cho batch processing trong tương lai.

| ID | Mô tả ngắn | Mô tả chi tiết |
|----|------------|----------------|
| FR-25 | Decision Final Record | Lưu decision_final, approver, timestamp khi chốt. |
| FR-26 | Audit Trail | Ghi nhận thay đổi quan trọng (giá, mode, risk, decision_final). |
| FR-27 | Rule Version Tracking | Gắn rule_version vào asset tại thời điểm decision_final. |
| FR-28 | Manual Override Control | Override decision_suggested bắt buộc ghi lý do. |
| FR-29 | AI Summary Storage | Lưu ai_summary để truy vết nhanh. |
| FR-30 | Batch Ready | Thiết kế sẵn cho batch (tương lai), không bật mặc định. |

---

# 6. ACCEPTANCE CRITERIA – CHI TIẾT THEO TỪNG FR

### FR-01: Create Asset
- Tạo asset_id duy nhất (hoặc validate unique nếu user nhập).
- Lưu bản ghi trong ASSETS.

**Acceptance:** Tạo 5 asset → không trùng asset_id; dữ liệu còn sau reload.

---

### FR-02: Update Asset
- Sửa cột input không làm mất công thức cột auto.

**Acceptance:** Sửa purchase_price → roi/decision_suggested vẫn tính đúng; công thức không mất.

---

### FR-03: Calculate Total Cost
- total_cost đúng; cost trống = 0.

**Acceptance:** Thay dismantle/transport/hazardous → total_cost cập nhật đúng tức thì; ô trống không lỗi.

---

### FR-04: Calculate Expected Profit
- expected_profit = resale_est - total_cost.

**Acceptance:** resale_est hoặc total_cost đổi → expected_profit đổi đúng.

---

### FR-05: Calculate ROI
- roi = expected_profit / total_cost; total_cost=0 → roi null/rỗng.

**Acceptance:** Không #DIV/0!; ROI test thủ công khớp.

---

### FR-06: Calculate Scrap Floor
- Tính scrap_total_value và net_scrap_floor.

**Acceptance:** weight=3500, price=65, dismantle=30000, transport=25000, hazardous=5000 → net=167,500.

---

### FR-07: Decision Engine Resale
- Theo ngưỡng ROI config.

**Acceptance:** ROI=0.30 → BUY; ROI=0.20 → NEGOTIATE; ROI=0.10 → SKIP.

---

### FR-08: Decision Engine Scrap
- Theo net_scrap_floor và scrap_buy_ratio.

**Acceptance:** net=167,500: price=180,000 → SKIP; 130,000 → BUY; 150,000 → NEGOTIATE.

---

### FR-09: Decision Engine Art
- Theo ngưỡng ART và provenance policy.

**Acceptance:** ROI=0.45 + provenance ok → BUY; provenance missing → không BUY.

---

### FR-10: Risk Override
- Risk ≥ threshold → SKIP.

**Acceptance:** risk=85, roi=0.5 → SKIP.

---

### FR-11: Mode Switching
- Đổi mode → logic đổi theo.

**Acceptance:** RESALE→SCRAP → decision theo scrap; SCRAP→RESALE → decision theo ROI.

---

### FR-12: Configurable Threshold
- Threshold thay đổi qua CONFIG có hiệu lực.

**Acceptance:** Đổi roi_buy_threshold 0.25→0.30, ROI=0.28 từ BUY thành NEGOTIATE.

---

### FR-13: Trigger Evaluate
- Chỉ chạy khi chọn đúng 1 dòng.

**Acceptance:** Chọn 0/2 dòng → không gọi AI, có cảnh báo.

---

### FR-14: Payload Builder
- Payload đủ field, không chứa secret.

**Acceptance:** Payload debug có asset_id/mode/financials/risk/images; không có API key.

---

### FR-15: OCR Nameplate
- OCR khi có nameplate.

**Acceptance:** Ảnh rõ → trích xuất maker+model; ảnh mờ → yêu cầu chụp lại.

---

### FR-16: Visual Condition Analysis
- Nêu red flags từ ảnh tổng thể.

**Acceptance:** Ảnh rỉ nặng/thiếu bộ phận → AI nêu đúng cảnh báo.

---

### FR-17: Data Completeness Check
- Thiếu data → yêu cầu bổ sung + giảm confidence.

**Acceptance:** SCRAP thiếu weight → checklist + confidence ≤ 2.

---

### FR-18: Market Advisory
- Trả range/liquidity/keywords + disclaimer.

**Acceptance:** Luôn có keywords + “verify manually”; không claim realtime.

---

### FR-19: Counter Offer Suggestion
- Đề xuất mức giá thương lượng phù hợp.

**Acceptance:** ROI thấp → counter-offer giảm giá để đạt ngưỡng hoặc an toàn theo scrap (nếu SCRAP).

---

### FR-20: AI Output Contract
- Đúng contract; sai → retry 1 lần.

**Acceptance:** Thiếu section → retry; vẫn sai → fail graceful, không crash.

---

### FR-21: Confidence Score
- Lưu ai_confidence và timestamp.

**Acceptance:** Sau Evaluate, ai_confidence cập nhật; ai_last_review_at có timestamp.

---

### FR-22: Confidence Gating
- confidence ≤ 2 → không BUY.

**Acceptance:** confidence=2 → decision_review không chứa BUY + “Verification Required”.

---

### FR-23: Sidebar Rendering
- Render theo section, rõ ràng.

**Acceptance:** Sidebar có heading đúng thứ tự; cảnh báo nổi bật khi thiếu data/risk cao.

---

### FR-24: AI Logging
- Mỗi Evaluate tạo 1 dòng AI_LOG.

**Acceptance:** 10 lần Evaluate → 10 dòng log đủ asset_id/timestamp/confidence.

---

### FR-25: Decision Final Record
- Lưu decision_final + approver + time.

**Acceptance:** Set decision_final → approver+timestamp được ghi.

---

### FR-26: Audit Trail
- Ghi thay đổi quan trọng.

**Acceptance:** Đổi purchase_price/mode/decision_final → audit có old/new/time/user.

---

### FR-27: Rule Version Tracking
- Snapshot rule_version tại thời điểm chốt.

**Acceptance:** Asset chốt trước giữ version cũ; asset chốt sau có version mới.

---

### FR-28: Manual Override Control
- Override phải có reason.

**Acceptance:** decision_final != suggested và reason trống → không cho lưu/hiển thị lỗi.

---

### FR-29: AI Summary Storage
- Lưu ai_summary.

**Acceptance:** Sau Evaluate, ai_summary có nội dung tóm tắt.

---

### FR-30: Batch Ready
- Sẵn sàng mở rộng, nhưng disabled.

**Acceptance:** Không có auto batch; feature-flag OFF mặc định.

---

# 7. NON-FUNCTIONAL REQUIREMENTS (NFR)

| ID | Mô tả ngắn | Mô tả chi tiết |
|----|------------|----------------|
| NFR-01 | Performance | Median ≤ 10s; p95 ≤ 20s (mục tiêu). |
| NFR-02 | Availability | Rule Engine chạy dù AI lỗi. |
| NFR-03 | Security | API key trong User Properties; không lưu sheet/repo. |
| NFR-04 | No Crawling | Không crawl marketplace/realtime. |
| NFR-05 | Logging Reliability | 100% Evaluate có log. |
| NFR-06 | Scalability | Sẵn sàng migrate Web App/SaaS. |
| NFR-07 | Config Management | Rule/threshold qua CONFIG, không hard-code. |
| NFR-08 | Data Integrity | Không mất dữ liệu khi Evaluate. |
| NFR-09 | Error Handling | JSON lỗi → retry 1 lần → fail graceful. |
| NFR-10 | Usability | Hiểu kết quả ≤ 30s. |
| NFR-11 | Maintainability | Tách logic/UI; dễ sửa rule. |
| NFR-12 | Audit Compliance | Truy vết được mọi quyết định. |
| NFR-13 | Extensibility | Thêm mode/category không phá core. |
| NFR-14 | Confidentiality | Không lưu ảnh ngoài phạm vi cho phép. |
| NFR-15 | Threshold Safety | Risk ≥ threshold không BUY. |
| NFR-16 | Recovery | Timeout/AI lỗi không crash. |
| NFR-17 | Provider Swap Ready | Đổi AI provider qua adapter. |
| NFR-18 | Data Completeness Enforcement | Thiếu dữ liệu nghiêm trọng không BUY. |
| NFR-19 | Versioning Discipline | Quản lý version + changelog. |
| NFR-20 | SaaS Ready | Contract độc lập layout; migrate dễ. |

---

# 8. ACCEPTANCE CRITERIA – CHI TIẾT THEO TỪNG NFR

### NFR-01: Performance
**Acceptance:** 10 lần Evaluate → median ≤ 10s; p95 ≤ 20s (mục tiêu).

### NFR-02: Availability
**Acceptance:** AI lỗi → ROI/Scrap/Decision Suggested vẫn hoạt động; UI báo lỗi rõ.

### NFR-03: Security
**Acceptance:** Không có API key trong sheet/code public; key chỉ ở User Properties.

### NFR-04: No Crawling
**Acceptance:** Không có request marketplace; output luôn kèm disclaimer verify.

### NFR-05: Logging Reliability
**Acceptance:** 20 Evaluate → 20 log; không thiếu.

### NFR-06: Scalability
**Acceptance:** Asset có asset_id + rule_version + audit fields để migrate DB.

### NFR-07: Config Management
**Acceptance:** Thay threshold trong CONFIG → decision đổi đúng, không sửa code.

### NFR-08: Data Integrity
**Acceptance:** Evaluate không ghi đè nhầm cột input; không mất dữ liệu.

### NFR-09: Error Handling
**Acceptance:** JSON lỗi → retry 1 lần; vẫn lỗi → fail graceful không crash.

### NFR-10: Usability
**Acceptance:** Người dùng ra quyết định trong ≤ 30s dựa trên sidebar.

### NFR-11: Maintainability
**Acceptance:** Dev thay đổi rule/threshold dễ, không ảnh hưởng UI.

### NFR-12: Audit Compliance
**Acceptance:** Truy vết decision_final + approver + time + reason + ai_summary.

### NFR-13: Extensibility
**Acceptance:** Thêm category/mode mới không làm lỗi core rule.

### NFR-14: Confidentiality
**Acceptance:** Hình ảnh tuân thủ policy lưu trữ/quyền; không leak ra ngoài.

### NFR-15: Threshold Safety
**Acceptance:** Không có trường hợp risk≥threshold mà suggested=BUY.

### NFR-16: Recovery
**Acceptance:** Timeout API → UI hiện lỗi + fallback; sheet không treo.

### NFR-17: Provider Swap Ready
**Acceptance:** Đổi provider chỉ sửa adapter, không sửa rule engine.

### NFR-18: Data Completeness Enforcement
**Acceptance:** Thiếu dữ liệu nghiêm trọng → confidence ≤ 2 và không BUY.

### NFR-19: Versioning Discipline
**Acceptance:** Mỗi release có version; SR/CR/Rule version đồng bộ.

### NFR-20: SaaS Ready
**Acceptance:** Payload/output/log độc lập layout; migrate web không rewrite toàn bộ.

---

# 9. OPEN POINTS

- Định nghĩa “deal lớn” (ngưỡng giá trị) để bắt buộc AI review.
- Policy provenance cho ART (bắt buộc theo giá trị hay luôn bắt buộc).
- Audit trail: full history hay chỉ critical fields.
- Policy lưu ảnh: Drive folder, permission, retention.

---

# END OF DOCUMENT
