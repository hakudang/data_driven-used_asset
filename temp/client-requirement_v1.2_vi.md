# TÀI LIỆU YÊU CẦU KHÁCH HÀNG (CLIENT REQUIREMENT – CR)
## Hệ thống Đánh giá & Kiểm soát Đầu tư Tài sản Cũ  
### Rule Engine + AI Reviewer (Vision + Market Advisory – Option A)

```text
Phiên bản: 1.2 (Market Intelligence – Phương án A: Estimate, không crawl)
Chủ sở hữu: Dang
Khu vực áp dụng: Nhật Bản
Loại tài liệu: Client Requirement (CR)
Trạng thái: Hoàn chỉnh – Sẵn sàng chuyển System Requirement (SR)
```

---

# 1. MỤC ĐÍCH TÀI LIỆU

Tài liệu này mô tả yêu cầu nghiệp vụ cho hệ thống:

- Kiểm soát quyết định đầu tư tài sản cũ
- Bảo vệ vốn bằng Rule Engine
- Bổ sung AI Reviewer (Model A – Evaluate Selected Row)
- Phân tích hình ảnh (Vision OCR)
- Đưa Market Advisory Estimate (không crawl marketplace)

Đây là bản CR chính thức sau khi thống nhất:

- Không sử dụng tab MARKET_DATA
- Không crawl Yahoo/Mercari
- AI chỉ đưa estimate + keyword gợi ý

---

# 2. TỔNG QUAN HỆ THỐNG

Hệ thống gồm 2 Layer chính:

## Layer 1 – Rule Engine (Google Sheets)

Chịu trách nhiệm:

- Total Cost
- Expected Profit
- ROI
- Scrap Floor
- Risk Score
- Decision Suggested (BUY / NEGOTIATE / SKIP)

Là lõi quyết định đầu tư.

## Layer 2 – AI Reviewer (Model A)

Chịu trách nhiệm:

- Phân tích dữ liệu dòng đang chọn
- Phân tích ảnh (nếu có)
- Đưa Market Advisory Estimate
- Đề xuất Counter Offer
- Cảnh báo rủi ro

AI không override Rule Engine.

---

# 3. MỤC TIÊU KINH DOANH

1. ROI trung bình ≥ 25% (Resale)
2. ROI ≥ 40% (Art)
3. Tỷ lệ lỗ < 10%
4. Chu kỳ quay vòng ≤ 60 ngày (mục tiêu)
5. Không BUY nếu Risk ≥ 80
6. Luôn có Scrap Floor bảo vệ vốn
7. Deal giá trị lớn bắt buộc có AI review

---

# 4. PHẠM VI DỰ ÁN

## 4.1 In Scope

- Thiết bị IT
- Máy nông nghiệp
- Máy CNC
- Tranh / Gốm
- ROI calculation
- Scrap Floor
- Risk Scoring
- Decision Engine
- AI Reviewer (Evaluate Selected Row)
- Vision Analysis (OCR Info Panel)
- Market Advisory Estimate
- Counter Offer Strategy
- Sidebar UX

## 4.2 Out of Scope

- Crawl marketplace tự động
- Realtime AI trigger
- Batch evaluation
- Machine learning tự học
- Hệ thống kế toán
- Web scraping service
- AI xác thực tranh cấp chuyên gia

---

# 5. CHẾ ĐỘ HOẠT ĐỘNG

## 5.1 RESALE MODE

Rule:

- ROI ≥ 25% → BUY
- 15% ≤ ROI < 25% → NEGOTIATE
- ROI < 15% → SKIP

AI bổ sung:

- Kiểm tra resale_est có hợp lý
- So sánh với Market Advisory Estimate
- Cảnh báo nếu resale_est quá lạc quan
- Đề xuất counter-offer

---

## 5.2 SCRAP MODE

Tính:

scrap_total_value = weight × scrap_price_per_kg  
net_scrap_floor = scrap_total_value - dismantle - transport - hazardous  

Rule:

- purchase_price > net_scrap_floor → SKIP
- purchase_price ≤ 80% net_scrap_floor → BUY
- else → NEGOTIATE

AI bổ sung:

- Ước tính weight hợp lý theo model
- Cảnh báo sai lệch trọng lượng
- Kiểm tra logic bảo vệ vốn

---

## 5.3 ART MODE

Rule:

- ROI ≥ 40%
- Thiếu provenance → SKIP

AI bổ sung:

- Phân tích rủi ro xác thực
- Ước tính độ hiếm nghệ sĩ
- Confidence thấp nếu thiếu dữ liệu

---

# 6. VISION ANALYSIS (Layer 2.1)

Nếu có image_url:

AI phải:

- OCR bảng info panel
- Trích xuất:
  - Maker
  - Model
  - Serial
  - Voltage
  - Year
- So sánh với dữ liệu người nhập
- Cảnh báo mismatch
- Phát hiện dấu hiệu hư hỏng cơ bản

Nếu ảnh mờ:

- Yêu cầu chụp lại
- Confidence giảm

---

# 7. MARKET ADVISORY ESTIMATE (Layer 2.2 – Option A)

AI không được truy cập internet.

AI phải:

1. Dựa trên maker + model + category
2. Sử dụng kiến thức thị trường phổ biến
3. Đưa ra:

   - Estimated Market Range (min–max)
   - Estimated Median
   - Thanh khoản (High / Medium / Low)
   - Độ phổ biến model

4. Tạo Suggested Search Keyword:

   Ví dụ:
   - "Kubota L1-22 tractor sold"
   - "Fanuc A02B-0098 used price Japan"

5. Đánh dấu rõ:

   "Market estimate only – user must verify manually."

---

# 8. AI OUTPUT FORMAT (BẮT BUỘC)

AI phải trả về cấu trúc sau:

[Data Check]  
[Calculation Check]  
[Vision Summary]  
[Market Advisory]  
[Decision Review]  
[Counter Offer]  
[Confidence 1–5]  
[Top Risks]  

---

# 9. GIỚI HẠN HỆ THỐNG

- AI không crawl Yahoo/Mercari
- Không đảm bảo giá realtime
- Không cam kết tính chính xác thị trường
- Rule Engine là quyết định chính
- Owner là người chốt cuối cùng

---

# 10. YÊU CẦU PHI CHỨC NĂNG

- API key lưu User Properties
- Không lưu API key trong sheet
- Không auto trigger AI
- Không batch
- Không làm lag sheet
- Hệ thống vẫn chạy khi AI lỗi
- Có thể nâng cấp thành Web App

---

# 11. KPI

- ROI trung bình ≥ 25%
- Tỷ lệ lỗ < 10%
- 100% deal lớn có AI review
- Counter-offer cải thiện ROI ≥ 5% (mục tiêu)
- Không có Risk ≥ 80 được BUY

---

# 12. RỦI RO HỆ THỐNG

- OCR sai
- AI estimate sai
- Người dùng phụ thuộc quá mức vào AI
- Nhập dữ liệu sai
- API quota

---

# 13. LỘ TRÌNH PHÁT TRIỂN

Phase 1: Rule Engine v3.0  
Phase 2: AI Reviewer Model A  
Phase 3: Vision Module  
Phase 4: Tối ưu Market Advisory  
Phase 5: Web Application  

---

# 14. TIÊU CHÍ THÀNH CÔNG

Hệ thống được coi là thành công khi:

- Giảm mua hàng cảm tính
- Chuẩn hóa quyết định
- Bảo vệ vốn hiệu quả
- AI giúp thương lượng tốt hơn
- Có nền tảng đủ mạnh để scale

---

# KẾT THÚC TÀI LIỆU
