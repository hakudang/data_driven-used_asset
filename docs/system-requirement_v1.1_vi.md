# TÀI LIỆU YÊU CẦU HỆ THỐNG (SYSTEM REQUIREMENT – SR)
## Hệ thống Kiểm soát Đầu tư Tài sản Cũ dựa trên Dữ liệu
### Dựa trên CR v1.3 FINAL – Enterprise Ready

```text
Phiên bản: v1.1 (Tiếng Việt)
Nguồn: Client Requirement v1.3 FINAL
Chủ sở hữu: Dang
Loại tài liệu: System Requirement (SR)
Trạng thái: Technical Baseline
Khu vực: Nhật Bản
```
Checklist : 
- Chuẩn hóa cấu trúc kỹ thuật
- Giữ nguyên logic từ CR v1.3 FINAL
- Đầy đủ kiến trúc, data schema, AI flow
- Có validation, error handling, security, logging
- Sẵn sàng dùng làm tài liệu triển khai

---

# 1. MỤC ĐÍCH

Tài liệu này chuyển đổi yêu cầu nghiệp vụ (CR v1.3 FINAL) thành đặc tả kỹ thuật hệ thống.

SR này xác định:

- Kiến trúc hệ thống
- Cấu trúc dữ liệu
- Logic tính toán
- Luồng tương tác AI
- Quy tắc xác thực dữ liệu
- Xử lý lỗi
- Bảo mật
- Ghi log & kiểm toán

---

# 2. KIẾN TRÚC HỆ THỐNG

## 2.1 Kiến trúc tổng thể

Layer 1: Google Sheets – Rule Engine  
Layer 2: Google Apps Script – Backend Logic  
Layer 3: OpenAI API – AI Reviewer  
UI Layer: Sidebar UX (HTML Service của Apps Script)

Luồng hoạt động:

Người dùng → Google Sheet → Apps Script → AI API → Sidebar → Cập nhật Sheet

---

# 3. CẤU TRÚC DỮ LIỆU

## 3.1 Sheet: ASSETS
Bảng ASSETS chứa thông tin tài sản cũ, bao gồm các trường bắt buộc và tùy chọn như đã mô tả trong CR v1.3 FINAL.

Các cột tối thiểu bắt buộc:

| Trường | Kiểu | Mô tả |
|--------|------|-------|
| asset_id | string | Mã tài sản |
| category | enum | IT / AGRI / CNC / ART |
| evaluation_mode | enum | RESALE / SCRAP / ART |
| purchase_price | number | Giá mua |
| resale_est | number | Giá bán dự kiến nội địa |
| export_est | number | Giá xuất khẩu dự kiến |
| weight_est_kg | number | Trọng lượng ước tính |
| scrap_price_per_kg | number | Giá sắt/kg |
| dismantle_cost | number | Chi phí tháo dỡ |
| transport_cost | number | Chi phí vận chuyển |
| hazardous_cost | number | Chi phí xử lý chất thải |
| risk_score | number | Điểm rủi ro |
| decision_suggested | enum | BUY / NEGOTIATE / SKIP |
| decision_final | enum | Quyết định cuối |
| image_nameplate_url | string | Link ảnh info panel |
| image_overall_url | string | Link ảnh tổng thể |
| ai_confidence | number | Điểm tin cậy AI |
| ai_last_review_date | datetime | Ngày đánh giá gần nhất |

---

## 3.2 Sheet: CONFIG

Bảng CONFIG chứa các tham số cấu hình hệ thống:

| Trường | Mô tả |
|--------|------|
| default_scrap_price | Giá sắt mặc định |
| roi_buy_threshold | Ngưỡng BUY |
| roi_negotiate_threshold | Ngưỡng NEGOTIATE |
| art_roi_threshold | Ngưỡng ART |
| risk_auto_skip_threshold | Ngưỡng Risk auto SKIP |

---

## 3.3 Sheet: AI_LOG

Bảng AI_LOG ghi lại lịch sử đánh giá của AI:

| Trường | Mô tả |
|--------|------|
| log_id | ID log |
| asset_id | Mã tài sản |
| timestamp | Thời điểm |
| ai_output_summary | Tóm tắt AI |
| confidence | Điểm tin cậy |
| reviewer | Người thực hiện |
| version | Phiên bản hệ thống |

---

# 4. LOGIC RULE ENGINE

Logic rule engine được triển khai trong Google Sheets sử dụng công thức và Apps Script để tính toán các chỉ số tài chính, đánh giá rủi ro, và đưa ra gợi ý quyết định dựa trên dữ liệu nhập vào.

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

Quy tắc:

- purchase_price > net_scrap_floor → SKIP  
- purchase_price ≤ 0.8 × net_scrap_floor → BUY  
- Khác → NEGOTIATE  

---

## 4.4 Risk Rule

Nếu risk_score ≥ risk_auto_skip_threshold → AUTO SKIP

---

# 5. AI REVIEWER (MODEL A)

AI Reviewer được triển khai bằng OpenAI API, nhận dữ liệu từ Apps Script và trả về phân tích chi tiết để hỗ trợ quyết định đầu tư.

## 5.1 Điều kiện kích hoạt

- Sheet đang mở là ASSETS  
- Chỉ chọn 1 dòng  
- Người dùng nhấn "Evaluate"

---

## 5.2 Payload gửi AI

- Dữ liệu tài chính
- Evaluation mode
- Risk score
- Giá trị scrap floor
- Link ảnh (nếu có)

---

## 5.3 Định dạng Output AI (JSON bắt buộc)

Output AI (json) là hợp đồng kỹ thuật giữa AI và hệ thống, dùng để đảm bảo tính nhất quán và dễ dàng xử lý:
- hệ thống đọc được AI một cách tự động
  - Khi AI trả về JSON hợp lệ → hệ thống tự động ghi confidence, decision_review, và kích hoạt rule gating.
  - Apps Script có thể:
    - Parse JSON
    - Ghi confidence vào cột ai_confidence
    - Ghi decision_review vào sidebar
    - Lưu log vào AI_LOG
    - Kích hoạt confidence gating
    - Block BUY nếu confidence ≤ 2

    👉 Nếu AI trả text tự do → không parse được → không tự động hóa được.

- kích hoạt Rule Gating ( dùng confidence để kiểm soát BUY/NEGOTIATE )
- ghi log & kiểm toán ( Timestamp, Output tóm tắt, Confidence, Version. Không có format cố định → không lưu audit được)
- scale lên Web App ( Frontend → Backend → AI → JSON → DB, có json không cần tái cấu trúc lại dữ liệu )
- kiểm soát AI hallucination ( buộc AI không nói lan man, không thêm text thừa, không phá format )
- đo KPI AI (Average confidence, % confidence ≤ 2, % deal AI disagree với Rule Engine )

```json
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
```

---

# 6. MODULE VISION

Module Vision sử dụng AI để phân tích ảnh tài sản, hỗ trợ đánh giá tình trạng và phát hiện rủi ro ẩn.

## 6.1 Nameplate OCR

Hệ thống phải:

- Trích xuất maker/model/serial/year
- So sánh với dữ liệu Sheet
- Đánh dấu mismatch

## 6.2 Phân tích ảnh tổng thể

Hệ thống phải:

- Phát hiện hư hỏng lớn
- Phát hiện thiếu bộ phận
- Ước tính mức độ rủi ro
- Đề xuất ảnh bổ sung

---

# 7. KIỂM SOÁT ĐỘ ĐẦY ĐỦ DỮ LIỆU

Các dữ liệu quan trọng:

- purchase_price
- evaluation_mode
- weight_est_kg (SCRAP)
- provenance (ART)

Nếu thiếu:

- confidence ≤ 2
- Không được đề xuất BUY

---

# 8. CONFIDENCE GATING

Nếu confidence ≤ 2:

- decision_review không được đề xuất BUY
- Phải yêu cầu xác minh

---

# 9. XỬ LÝ LỖI

## 9.1 Lỗi API AI

- Không làm gián đoạn Sheet
- Hiển thị: “AI unavailable”
- Ghi log

## 9.2 JSON không hợp lệ

- Retry 1 lần
- Nếu vẫn lỗi → Fail graceful

## 9.3 Ảnh không truy cập được

- Hiển thị cảnh báo

---

# 10. BẢO MẬT

- API key lưu trong User Properties
- Không lưu API key trong Sheet
- Không crawl marketplace
- Không auto chạy

---

# 11. HIỆU NĂNG

- AI phản hồi ≤ 10 giây
- Sheet vẫn hoạt động mượt
- Không batch trong v1.1

---

# 12. GHI LOG & KIỂM TOÁN

Mỗi lần Evaluate:

- Ghi vào AI_LOG
- Lưu confidence
- Lưu timestamp
- Lưu tóm tắt output

---

# 13. TRIỂN KHAI

Phase 1: Rule Engine  
Phase 2: Apps Script + AI  
Phase 3: Vision  
Phase 4: Logging  
Tương lai: Web App

---

# 14. PHÁP LÝ & TRÁCH NHIỆM

AI chỉ mang tính tư vấn.  
Quyết định tài chính thuộc về Owner.

---

# 15. TIÊU CHÍ NGHIỆM THU

Hệ thống được nghiệm thu khi:

- Logic tính toán đúng
- AI trả JSON hợp lệ
- Confidence gating hoạt động
- Không BUY khi risk vượt ngưỡng
- Không BUY khi thiếu dữ liệu
- AI_LOG ghi đầy đủ

---

# KẾT THÚC TÀI LIỆU
