# USE CASE SPECIFICATION (UCS)
## Hệ thống Kiểm soát Đầu tư Tài sản Cũ
Version: UC v1.0  
Owner: Dang  
Status: Baseline for Development & QA  
Region: Japan  

---

# 1. ACTOR LIST

| Actor | Mô tả |
|-------|-------|
| Owner | Người ra quyết định tài chính cuối cùng |
| Reviewer | Người hỗ trợ kiểm tra (nếu có) |
| AI Reviewer | Hệ thống AI phân tích dữ liệu & hình ảnh |
| Rule Engine | Hệ thống tính toán & đề xuất quyết định |
| System | Apps Script + Sheet + Logging Layer |

---

# 2. USE CASE LIST (TỔNG QUAN)

| UC ID | Tên Use Case | Actor | Mô tả ngắn |
|-------|--------------|-------|------------|
| UC-01 | Create Asset | Owner | Tạo bản ghi tài sản |
| UC-02 | Update Asset Data | Owner | Cập nhật dữ liệu tài sản |
| UC-03 | Auto Calculate Financial | Rule Engine | Tính cost, profit, ROI |
| UC-04 | Decision Suggestion | Rule Engine | Đề xuất BUY/NEGOTIATE/SKIP |
| UC-05 | Apply Risk Override | Rule Engine | Override khi risk cao |
| UC-06 | Trigger AI Review | Owner | Gọi AI đánh giá |
| UC-07 | AI Analysis & Advisory | AI Reviewer | Phân tích & đề xuất |
| UC-08 | Confidence Gating | System | Chặn BUY nếu confidence thấp |
| UC-09 | Final Decision Approval | Owner | Chốt decision_final |
| UC-10 | Manual Override | Owner | Override có lý do |
| UC-11 | Logging & Audit Trail | System | Ghi log & truy vết |
| UC-12 | Change Configuration | Owner | Thay đổi threshold |

---

# 3. DETAILED USE CASE SPECIFICATIONS

---

## UC-01: Create Asset

**Primary Actor:** Owner  
**Precondition:** Hệ thống hoạt động  

### Main Flow
1. Owner nhập thông tin tài sản
2. Hệ thống validate asset_id unique
3. Lưu bản ghi vào ASSETS
4. Rule Engine kích hoạt tính toán

### Postcondition
- Asset được lưu bền vững
- Có decision_suggested

### Exception
- Nếu asset_id trùng → báo lỗi, không lưu

---

## UC-02: Update Asset Data
**Primary Actor:** Owner  
**Precondition:** Asset đã tồn tại

### Main Flow
1. Owner chỉnh sửa dữ liệu (cost, condition, etc.)
2. Hệ thống validate dữ liệu
3. Lưu thay đổi
4. Rule Engine kích hoạt tính toán lại
### Postcondition
- Dữ liệu được cập nhật
- Decision được cập nhật theo dữ liệu mới
### Exception
- Dữ liệu không hợp lệ → báo lỗi, không lưu

---

## UC-03: Auto Calculate Financial

**Primary Actor:** Rule Engine  

### Main Flow
1. Tính total_cost
2. Tính expected_profit
3. Tính ROI
4. Nếu mode SCRAP → tính net_scrap_floor

### Business Rules Applied
- Cost trống = 0
- total_cost = 0 → roi = null
- Công thức theo SR chuẩn

### Postcondition
- Các cột tính toán được cập nhật tự động

---

## UC-04: Decision Suggestion

**Primary Actor:** Rule Engine  

### Main Flow (RESALE)
1. So sánh ROI với threshold
2. ROI ≥ Buy threshold → BUY
3. ROI ≥ Negotiate threshold → NEGOTIATE
4. Còn lại → SKIP

### Main Flow (SCRAP)
1. So sánh purchase_price với net_scrap_floor
2. Áp dụng scrap_buy_ratio

### Postcondition
- decision_suggested được cập nhật

---

## UC-05: Apply Risk Override

**Primary Actor:** Rule Engine  

### Main Flow
1. Kiểm tra risk_score
2. Nếu risk ≥ threshold
3. decision_suggested = SKIP (override tất cả)

### Priority
Risk Override luôn cao nhất

---

## UC-06: Trigger AI Review

**Primary Actor:** Owner  

### Preconditions
- Chọn đúng 1 dòng asset

### Main Flow
1. Owner nhấn Evaluate
2. System build payload
3. Gửi request AI API
4. Nhận response

### Exception
- 0 hoặc >1 dòng → không gọi AI
- JSON lỗi → retry 1 lần

---

## UC-07: AI Analysis & Advisory

**Primary Actor:** AI Reviewer  

### Main Flow
1. OCR nameplate nếu có
2. Phân tích condition ảnh
3. Kiểm tra thiếu dữ liệu
4. Đưa market advisory
5. Đề xuất counter-offer
6. Trả confidence score

### Output Contract
- summary
- risks
- recommendation
- confidence (1–5)

---

## UC-08: Confidence Gating

**Primary Actor:** System  

### Rule
- confidence ≤ 2 → không khuyến nghị BUY
- Hiển thị "Verification Required"

---

## UC-09: Final Decision Approval

**Primary Actor:** Owner  

### Main Flow
1. Owner xem decision_suggested + AI summary
2. Chọn decision_final
3. Lưu approver + timestamp
4. Snapshot rule_version

### Postcondition
- Decision được audit-traceable

---

## UC-10: Manual Override

**Primary Actor:** Owner  

### Rule
- decision_final ≠ suggested → bắt buộc nhập reason

### Exception
- reason trống → không cho lưu

---

## UC-11: Logging & Audit Trail

**Primary Actor:** System  

### Logged Data
- asset_id
- timestamp
- ai_confidence
- decision change
- old/new values

### Postcondition
- Có thể truy vết toàn bộ lịch sử

---

## UC-12: Change Configuration

**Primary Actor:** Owner  

### Main Flow
1. Thay đổi threshold trong CONFIG
2. Rule Engine áp dụng realtime

### Postcondition
- Decision recalculated theo config mới

---

# 4. USE CASE DIAGRAM (LOGICAL)

Actors:
Owner → (Create Asset)
Owner → (Trigger AI Review)
Owner → (Final Decision)

Rule Engine → (Auto Calculate)
Rule Engine → (Decision Suggestion)

AI Reviewer → (AI Analysis)

System → (Logging & Audit)

---

# 5. BUSINESS PRIORITY CLASSIFICATION

| Priority | Use Cases |
|----------|----------|
| Critical | UC-03, UC-04, UC-05, UC-09 |
| High | UC-06, UC-07, UC-10 |
| Medium | UC-01, UC-02, UC-12 |
| Support | UC-08, UC-11 |

---

# 6. GOVERNANCE GUARANTEE

Hệ thống đảm bảo:

- Không BUY khi Risk cao
- Không BUY khi Confidence thấp
- Không Override không lý do
- Không quyết định thiếu truy vết

---

# END OF USE CASE DOCUMENT

## NEXT STEPS
- Tạo thêm Use Case Execution Matrix (UC ↔ FR ↔ BR ↔ VAL ↔ ERR) chuẩn BrSE

- Hoặc vẽ Use Case Diagram bằng Mermaid

- Hoặc tách riêng thành Business UC và System UC (chuẩn enterprise)
