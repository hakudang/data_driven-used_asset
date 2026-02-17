# YÊU CẦU (CLIENT REQUIREMENT – CR)
## Hệ thống Đánh giá & Kiểm soát Đầu tư Tài sản Cũ  
### Rule Engine + AI Reviewer (Vision + Market Advisory + Data Completeness)

```text
Version: v1.3 FINAL (Enterprise Ready)
Owner: Dang
Document Type: Client Requirement (CR)
Status: Approved Baseline – Ready for SR
Region: Japan
```

---

# 0. ĐỊNH NGHĨA THUẬT NGỮ

- ROI (Return on Investment): tỉ suất lợi nhuận =  Expected Profit / Total Cost  
- Scrap Floor: giá sàn phế liệu = Giá trị bảo vệ vốn tối thiểu dựa trên giá trị kim loại sau chi phí  
- Decision Suggested: Quyết định đề xuất từ Rule Engine  
- Decision Final: Quyết định cuối cùng do Owner phê duyệt  
- Confidence (1–5): Mức độ tin cậy của AI đối với phân tích  
- Market Advisory: Ước tính thị trường tham khảo (không realtime, không crawl)

---

# 1. MỤC ĐÍCH TÀI LIỆU

Tài liệu này mô tả yêu cầu nghiệp vụ cho hệ thống kiểm soát đầu tư tài sản cũ nhằm:

- Chuẩn hóa quyết định đầu tư
- Bảo vệ vốn
- Giảm mua theo cảm tính
- Tăng chất lượng thương lượng
- Kiểm soát rủi ro bằng dữ liệu và AI reviewer

Đây là tài liệu nền tảng để chuyển sang System Requirement (SR).

---

# 2. KIẾN TRÚC HỆ THỐNG

## Layer 1 – Rule Engine (Google Sheets)

Chức năng:

- Tính Total Cost
- Tính Expected Profit
- Tính ROI
- Tính Scrap Floor
- Tính Risk Score
- Đưa Decision Suggested (BUY / NEGOTIATE / SKIP)

Rule Engine là nguồn quyết định chính.

---

## Layer 2 – AI Reviewer (Model A – Selected Row)

Kích hoạt khi:

- Người dùng chọn 1 dòng trong ASSETS
- Nhấn Evaluate
- Không auto-trigger

AI có thể:

- Phân tích tài chính
- OCR Info Panel
- Phân tích ảnh tổng thể thiết bị
- Đánh giá tình trạng
- Ước tính trọng lượng tham khảo
- Đưa Market Advisory Estimatev : không crawl, dựa trên kiến thức phổ biến. 
- Đề xuất Counter Offer
- Đánh giá Data Completeness
- Trả Confidence Score

AI không được:

- Crawl marketplace
- Truy cập internet realtime
- Override Decision Suggested
- Tự động chốt Decision Final

---

# 3. VAI TRÒ HỆ THỐNG

- Owner: Người phê duyệt cuối cùng
- Evaluator: Người nhập dữ liệu
- Reviewer: Người kiểm tra và xác nhận quyết định

---

# 4. MỤC TIÊU KINH DOANH

1. ROI trung bình ≥ 25% (Resale)
2. ROI ≥ 40% (Art)
3. Tỷ lệ lỗ < 10%
4. Không BUY nếu Risk ≥ 80
5. 100% deal lớn có AI review
6. Không deal thiếu dữ liệu nghiêm trọng được BUY

---

# 5. CHẾ ĐỘ HOẠT ĐỘNG

## 5.1 RESALE MODE

Rule:

- ROI ≥ 25% → BUY
- 15–25% → NEGOTIATE
- < 15% → SKIP

AI hỗ trợ:

- Kiểm tra resale_est
- So sánh Market Advisory
- Đề xuất Counter Offer

---

## 5.2 SCRAP MODE

scrap_total_value = weight × scrap_price  
net_scrap_floor = scrap_total_value – chi phí  

Rule:

- Giá mua > scrap_floor → SKIP
- ≤ 80% scrap_floor → BUY

---

## 5.3 ART MODE

Rule:

- ROI ≥ 40%
- Thiếu provenance → SKIP

---

# 6. VISION ANALYSIS

## 6.1 Nameplate OCR

- Trích xuất maker/model/serial/year
- So sánh dữ liệu nhập
- Cảnh báo mismatch

## 6.2 Ảnh Tổng Thể

- Đánh giá tình trạng
- Phát hiện hư hỏng
- Đề xuất ảnh bổ sung

---

# 7. DATA COMPLETENESS GOVERNANCE

Nếu thiếu dữ liệu quan trọng:

- Confidence ≤ 2
- Không đề xuất BUY
- Yêu cầu bổ sung ảnh hoặc thông số

---

# 8. MARKET ADVISORY (OPTION A)

- Không crawl internet
- Dựa trên kiến thức phổ biến
- Trả Estimated Range + Liquidity
- Sinh Suggested Search Keywords
- Ghi rõ: “Estimate only – verify manually.”

---

# 9. AI OUTPUT FORMAT

[Data Completeness Check]  
[Data Check]  
[Calculation Check]  
[Vision Analysis]  
[Market Advisory]  
[Decision Review]  
[Counter Offer]  
[Confidence 1–5]  
[Top Risks]  

---

# 10. QUẢN TRỊ QUYẾT ĐỊNH

- Rule Engine là nguồn quyết định chính
- AI chỉ đóng vai trò advisory
- Owner chịu trách nhiệm cuối cùng

---

# 11. YÊU CẦU CHỨC NĂNG (Functional Requirements)

Bảng tổng hợp yêu cầu chức năng:


# 12. PHI CHỨC NĂNG

- Không auto AI
- Không batch
- Không crawl
- API key bảo mật
- Hệ thống hoạt động nếu AI lỗi
- Có thể nâng cấp thành Web App

---

# 13. RỦI RO

- OCR sai
- Estimate sai
- Dữ liệu nhập sai
- Người dùng phụ thuộc AI

---

# 14. ROADMAP

Phase 1: Rule Engine v3.0  
Phase 2: Apps Script + Sidebar UX + AI Reviewer  
Phase 3: Vision + Data Completeness  
Phase 4: AI Log & Audit Trail  
Phase 5: Batch (optional)  
Phase 6: Web App  

Giải thích : 
- AI Log : Ghi lại mỗi lần Evaluate với timestamp, confidence, summary để kiểm toán sau này.
- Audit Trail : Lưu quyết định cuối cùng, người phê duyệt, và dữ liệu liên quan để truy vết.
- Batch : Nếu số lượng lớn, có thể thêm tính năng chạy AI hàng loạt, nhưng ưu tiên chất lượng hơn số lượng.
- Web App : Phiên bản cuối cùng có thể là Web App để mở rộng quy mô, nhưng bắt đầu với Google Sheets để nhanh chóng triển khai và kiểm thử.
---

# 15. SUCCESS CRITERIA

Hệ thống thành công khi:

- Kiểm soát lỗ vốn
- Giảm mua theo cảm tính
- Chuẩn hóa quyết định
- Counter-offer giúp tăng ROI ≥ 5%
- Không deal Risk ≥ 80 được BUY
- Sẵn sàng scale lên Web App

---

# 16. LEGAL & RESPONSIBILITY BOUNDARY

AI output chỉ mang tính advisory.  
Quyết định đầu tư và trách nhiệm tài chính thuộc về Owner.

---

# KẾT LUẬN

Hệ thống là Capital Protection Investment System kết hợp Rule Engine và AI Reviewer, đảm bảo kiểm soát rủi ro, minh bạch và khả năng mở rộng.
