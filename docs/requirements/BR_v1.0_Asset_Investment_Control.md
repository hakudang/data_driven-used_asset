# BUSINESS RULE SPECIFICATION (BR)

## Hệ thống Kiểm soát Đầu tư Tài sản Cũ
```
Version: BR v1.0
Owner: Dang
Status: Baseline -- Governance Controlled
Created On: 2026-02-18
Related Docs: CR v1.0, SR v1.0 ENTERPRISE, RTM_v1.0
```
---

# 1. PURPOSE

Tài liệu này định nghĩa toàn bộ Business Rules điều phối:

-   Tính toán tài chính
-   Logic quyết định BUY / NEGOTIATE / SKIP
-   Risk control (override)
-   AI governance
-   Manual override control
-   Audit & version discipline
-   Configuration governance

Không được phép thay đổi rule mà không: 
1. Tăng rule_version 
2. Cập nhật SR 
3. Cập nhật RTM 
4. Regression test

---

# 2. RULE CLASSIFICATION

| Category | Prefix | Description |
|----------|--------|------------|
| Financial Rule | FRL | Quy tắc tính toán tài chính |
| Decision Rule | DR | Quy tắc quyết định theo mode |
| Risk Rule | RR | Quy tắc kiểm soát rủi ro |
| AI Governance Rule | AR | Quy tắc kiểm soát AI |
| Control & Audit Rule | CRL | Quy tắc override & audit |
| Configuration Rule | CFG | Quy tắc cấu hình hệ thống |

---

# 3. FINANCIAL RULES (FRL)

## FRL-01: Total Cost Calculation

total_cost = purchase_price + dismantle_cost + transport_cost + hazardous_cost
- Cost field trống → mặc định = 0
- Không được phép âm

## FRL-02: Expected Profit

expected_profit = resale_est - total_cost\
Áp dụng cho RESALE / ART mode

## FRL-03: ROI Formula

roi = expected_profit / total_cost\
Nếu total_cost = 0 → roi = null

## FRL-04: Scrap Floor Calculation

scrap_total_value = weight_est_kg × scrap_price_per_kg\
net_scrap_floor = scrap_total_value - dismantle_cost - transport_cost -
hazardous_cost

---

# 4. DECISION RULES (DR)

## DR-01: RESALE Decision Logic

ROI ≥ roi_buy_threshold_resale → BUY\
ROI ≥ roi_negotiate_threshold_resale → NEGOTIATE\
Else → SKIP

## DR-02: ART Decision Logic

ROI ≥ roi_buy_threshold_art AND provenance valid → BUY\
Else → theo ngưỡng NEGOTIATE hoặc SKIP

## DR-03: SCRAP Decision Logic

purchase_price \> net_scrap_floor → SKIP\
purchase_price ≤ scrap_buy_ratio × net_scrap_floor → BUY\
Else → NEGOTIATE

## DR-04: Mode Switching Rule

Đổi evaluation_mode → áp dụng logic mode mới, không giữ decision cũ nếu
không hợp lệ.

---

# 5. RISK RULES (RR)

## RR-01: Risk Override (Highest Priority)

IF risk_score ≥ risk_skip_threshold\
THEN decision_suggested = SKIP

## RR-02: Data Completeness Enforcement

Thiếu dữ liệu nghiêm trọng theo mode → không được BUY và hiển thị cảnh
báo.

---

# 6. AI GOVERNANCE RULES (AR)

## AR-01: AI Advisory Only

AI chỉ tư vấn, không thay thế decision_final.

## AR-02: Confidence Gating

confidence ≤ 2 → không được BUY

## AR-03: No Real-Time Crawling

AI không crawl marketplace realtime và luôn có disclaimer verify
manually.

## AR-04: AI Output Contract

AI bắt buộc trả về: - Summary - Risks - Recommendation - Confidence
(1--5) Sai format → retry 1 lần → vẫn lỗi → fail graceful.

---

# 7. CONTROL & AUDIT RULES (CRL)

## CRL-01: Manual Override Rule

decision_final ≠ decision_suggested → decision_reason bắt buộc.

## CRL-02: Rule Version Snapshot

Chốt decision_final → snapshot rule_version.

## CRL-03: Audit Logging Mandatory

Phải log thay đổi purchase_price, evaluation_mode, risk_score,
decision_final, threshold, AI evaluate.

## CRL-04: Logging Integrity

Logging failure không làm mất dữ liệu chính.

---

# 8. CONFIGURATION RULES (CFG)

## CFG-01: Threshold Externalized

Threshold phải nằm trong CONFIG, không hard-code.

## CFG-02: Real-Time Effect

Thay đổi threshold áp dụng cho decision mới, không thay đổi snapshot cũ.

---

# 9. RULE PRIORITY ORDER

1.  Financial Rules (FRL)
2.  Decision Rules (DR)
3.  Risk Override (RR-01)
4.  Data Completeness (RR-02)
5.  AI Confidence Gating (AR-02)
6.  Manual Override Validation (CRL-01)

---

# 11. RULE TABLE SUMMARY

|No.| ID | Category | Description |
| ---- |----|----------|-------------|
| 01 | FRL-01 | Financial Rule | Tính tổng chi phí đầu tư |
| 02 | FRL-02 | Financial Rule | Tính lợi nhuận kỳ vọng |
| 03 | FRL-03 | Financial Rule | Tính ROI |
| 04 | FRL-04 | Financial Rule | Tính giá trị Scrap Floor |
| 05 | DR-01 | Decision Rule | Quyết định RESALE |
| 06 | DR-02 | Decision Rule | Quyết định ART |
| 07 | DR-03 | Decision Rule | Quyết định SCRAP |
| 08 | DR-04 | Decision Rule | Quy tắc chuyển mode |
| 09 | RR-01 | Risk Rule | Override khi risk cao |
| 10 | RR-02 | Risk Rule | Kiểm soát thiếu dữ liệu |
| 11 | AR-01 | AI Governance Rule | AI chỉ tư vấn, không thay đổi decision |
| 12 | AR-02 | AI Governance Rule | Confidence gating - không được BUY nếu confidence thấp |
| 13 | AR-03 | AI Governance Rule | Không crawl realtime |
| 14 | AR-04 | AI Governance Rule | Hợp đồng đầu ra của AI |
| 15 | CRL-01 | Control & Audit Rule | Override phải có lý do |
| 16 | CRL-02 | Control & Audit Rule | Snapshot version khi chốt decision |
| 17 | CRL-03 | Control & Audit Rule | Logging bắt buộc |
| 18 | CRL-04 | Control & Audit Rule | Logging failure không mất dữ liệu chính |
| 19 | CFG-01 | Configuration Rule | Threshold phải externalized |
| 20 | CFG-02 | Configuration Rule | Thay đổi threshold có hiệu lực realtime |

---

# 12. GOVERNANCE GUARANTEE

Hệ thống đảm bảo: 
- Không BUY khi risk cao 
- Không BUY khi confidence thấp 
- Không override không lý do 
- Không thay đổi rule không version 
- Không mất audit trace

---

# END OF DOCUMENT
