# BUSINESS RULE SPECIFICATION (BR)

## Hệ thống Kiểm soát Đầu tư Tài sản Cũ
```
Version: BR v1.1 (Audit Grade) 
Owner: Dang 
Region: Japan 
Status: Governance Controlled -- Audit Ready Create on: 2026-02-19
Related Documents: CR v1.0, SR v1.0 ENTERPRISE, RTM_v1.0
```

---

# 1. PURPOSE

BR v1.1 cập nhật nhằm: 
- Đồng bộ hoàn toàn với SR 
- Làm rõ ART negotiate threshold 
- Chuẩn hóa Data Completeness theo mode 
- Liên kết rõ Data Completeness ↔ Confidence Gating 
- Chuẩn hóa Error Handling Governance 
- Tăng mức kiểm soát audit

Mọi thay đổi rule phải: 
1. Tăng rule_version 
2. Update SR 
3. Update RTM
4. Regression test 
5. Update CHANGELOG

---

# 2. RULE CLASSIFICATION

  | Category            |  Prefix | Description |
  |---------------------|---------|-------------|
  | Financial Rule      | FRL     | Quy tắc tính toán tài chính |
  | Decision Rule       | DR      | Quy tắc quyết định theo mode |
  | Risk Rule           | RR      | Quy tắc kiểm soát rủi ro |
  | AI Governance Rule  | AR      | Quy tắc kiểm soát AI |
  | Control & Audit Rule| CRL     | Quy tắc override & audit ( ghi đè và kiểm toán )|
  | Configuration Rule  | CFG     | Quy tắc cấu hình hệ thống |
  | Data Governance Rule| DGR     | Quy tắc quản trị dữ liệu |

---

# 3. FINANCIAL RULES (FRL)

## FRL-01: Total Cost

total_cost = purchase_price + dismantle_cost + transport_cost +
hazardous_cost\
Cost trống → mặc định 0\
Không cho phép giá trị âm.

## FRL-02: Expected Profit

expected_profit = resale_est - total_cost\
Áp dụng cho RESALE và ART.

## FRL-03: ROI

roi = expected_profit / total_cost\
Nếu total_cost = 0 → roi = null.

## FRL-04: Scrap Floor

scrap_total_value = weight_est_kg × scrap_price_per_kg\
net_scrap_floor = scrap_total_value - dismantle_cost - transport_cost -
hazardous_cost

---

# 4. DECISION RULES (DR)

## DR-01: RESALE Logic

ROI ≥ roi_buy_threshold_resale → BUY\
ROI ≥ roi_negotiate_threshold_resale → NEGOTIATE\
Else → SKIP

## DR-02: ART Logic (Clarified)

ROI ≥ roi_buy_threshold_art AND provenance valid → BUY\
ROI ≥ roi_negotiate_threshold_resale → NEGOTIATE\
Else → SKIP

## DR-03: SCRAP Logic

purchase_price \> net_scrap_floor → SKIP\
purchase_price ≤ scrap_buy_ratio × net_scrap_floor → BUY\
Else → NEGOTIATE

## DR-04: Mode Switching

Đổi evaluation_mode → bắt buộc recalculation toàn bộ.

---

# 5. RISK RULES (RR)

## RR-01: Risk Override

IF risk_score ≥ risk_skip_threshold\
THEN decision_suggested = SKIP

---

# 6. DATA GOVERNANCE RULES (DGR)

## DGR-01: Mandatory Fields by Mode

  | Mode   |  Mandatory Fields | Description |
  |--------|------------------| -------------|
  | RESALE | purchase_price, resale_est | Thiếu → không BUY |
  | SCRAP  | purchase_price, weight_est_kg, scrap_price_per_kg | Thiếu → không BUY |
  | ART    | purchase_price, resale_est, provenance (nếu policy yêu cầu) | Thiếu → không BUY |
  ---

## DGR-02: Severe Missing Data Handling

Thiếu mandatory fields: 
- Không được BUY 
- Must set ai_confidence ≤ 2 (nếu AI chạy) 
- Hiển thị checklist bổ sung

---

# 7. AI GOVERNANCE RULES (AR)

## AR-01: AI Advisory Only

AI không thay thế decision_final.

## AR-02: Confidence Gating

confidence ≤ 2 → Không được BUY.

## AR-03: Confidence Linkage

DGR-02 bắt buộc dẫn đến ai_confidence ≤ 2.

## AR-04: No Realtime Crawling

AI advisory không crawl realtime.

## AR-05: AI Output Contract

AI output phải có Summary, Risks, Recommendation, Confidence (1--5).\
Sai format → retry 1 lần → vẫn sai → fail graceful.

---

# 8. CONTROL & AUDIT RULES (CRL)

## CRL-01: Manual Override

decision_final ≠ decision_suggested → decision_reason bắt buộc.

## CRL-02: Rule Version Snapshot

Chốt decision_final → snapshot rule_version.

Giải thích: Mỗi lần owner chốt decision_final, hệ thống sẽ lưu lại rule_version hiện tại để đảm bảo truy vết được quy tắc đã áp dụng tại thời điểm đó.

## CRL-03: Audit Logging Mandatory

Log bắt buộc: purchase_price, evaluation_mode, risk_score,
decision_final, threshold change, AI evaluate.

## CRL-04: Logging Integrity

Logging failure không làm mất dữ liệu chính.

---

# 9. CONFIGURATION RULES (CFG)

## CFG-01: Threshold Externalized

Threshold phải nằm trong CONFIG.

## CFG-02: Real-Time Effect

Threshold mới áp dụng cho decision mới.

## CFG-03: Retry Policy

retry_on_invalid_json cấu hình qua CONFIG.

---

# 10. RULE PRIORITY ORDER

1.  Financial Rules
2.  Decision Rules
3.  Risk Override
4.  Data Governance
5.  AI Confidence Gating
6.  Manual Override Validation

---

# 11. RULE TABLE SUMMARY

Bảng tổng hợp các rule đã nêu ở trên, phân loại theo FRL, DR, RR, AR, CRL, CFG để dễ dàng tra cứu và đối chiếu với SR cũng như RTM.
| No. | ID | Category | Tên | Mô tả |
| ---- |----|----------|-------|-------------|
| 01 | FRL-01 | Financial Rule | Total Cost | Tính tổng chi phí đầu tư |
| 02 | FRL-02 | Financial Rule | Expected Profit | Tính lợi nhuận kỳ vọng |
| 03 | FRL-03 | Financial Rule | ROI | Tính Return on Investment |
| 04 | FRL-04 | Financial Rule | Scrap Floor | Tính ngưỡng giá trị tối thiểu khi bán phế liệu |
| 05 | DR-01 | Decision Rule | RESALE Logic | Quy tắc quyết định cho mode RESALE |
| 06 | DR-02 | Decision Rule | ART Logic | Quy tắc quyết định cho mode ART |
| 07 | DR-03 | Decision Rule | SCRAP Logic | Quy tắc quyết định cho mode SCRAP |
| 08 | DR-04 | Decision Rule | Mode Switching | Quy tắc khi chuyển đổi evaluation_mode |
| 09 | RR-01 | Risk Rule | Risk Override | Quy tắc override khi risk cao |
| 10 | DGR-01 | Data Governance Rule | Mandatory Fields by Mode | Quy tắc trường dữ liệu bắt buộc theo mode |
| 11 | DGR-02 | Data Governance Rule | Severe Missing Data Handling | Quy tắc xử lý khi thiếu dữ liệu nghiêm trọng |
| 12 | AR-01 | AI Governance Rule | AI Advisory Only | AI chỉ đóng vai trò tư vấn, không thay thế quyết định |
| 13 | AR-02 | AI Governance Rule | Confidence Gating | Quy tắc gating khi confidence thấp |
| 14 | AR-03 | AI Governance Rule | Confidence Linkage | Liên kết giữa thiếu dữ liệu nghiêm trọng và confidence thấp |
| 15 | AR-04 | AI Governance Rule | No Realtime Crawling | AI không crawl dữ liệu realtime |
| 16 | AR-05 | AI Governance Rule | AI Output Contract | Quy tắc về định dạng output của AI và chính sách retry |
| 17 | CRL-01 | Control & Audit Rule | Manual Override | Quy tắc khi decision_final khác decision_suggested |
| 18 | CRL-02 | Control & Audit Rule | Rule Version Snapshot | Quy tắc snapshot version khi chốt decision_final |
| 19 | CRL-03 | Control & Audit Rule | Audit Logging Mandatory | Quy tắc về các trường dữ liệu bắt buộc phải log |
| 20 | CRL-04 | Control & Audit Rule | Logging Integrity | Quy tắc đảm bảo logging không làm mất dữ liệu chính |
| 21 | CFG-01 | Configuration Rule | Threshold Externalized | Quy tắc về việc cấu hình threshold qua CONFIG |
| 22 | CFG-02 | Configuration Rule | Real-Time Effect | Quy tắc về hiệu lực của thay đổi threshold | 
| 23 | CFG-03 | Configuration Rule | Retry Policy | Quy tắc về chính sách retry khi AI trả về JSON lỗi |

# 12. GOVERNANCE GUARANTEE

Hệ thống đảm bảo: 
- Không BUY khi risk cao 
- Không BUY khi confidence thấp 
- Không BUY khi thiếu dữ liệu nghiêm trọng 
- Không override không lý do 
- Không thay đổi rule không version 
- Không mất audit trace

---

# END OF DOCUMENT
