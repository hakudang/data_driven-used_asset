
# TÀI LIỆU YÊU CẦU KHÁCH HÀNG (CLIENT REQUIREMENT – CR)
## Hệ thống Đánh giá & Kiểm soát Đầu tư Tài sản Cũ dựa trên Dữ liệu

```
Phiên bản: 1.0 (Bản Tiếng Việt hoàn chỉnh)
Chủ sở hữu: Dang
Khu vực áp dụng: Nhật Bản
Loại tài liệu: Client Requirement (CR)
Trạng thái: Hoàn chỉnh – Sẵn sàng chuyển sang System Requirement (SR)
```
---

# 1. TÓM TẮT ĐIỀU HÀNH

Tài liệu này mô tả các yêu cầu nghiệp vụ của hệ thống Đánh giá tài sản cũ dựa trên dữ liệu.

Mục tiêu hệ thống:

- Chuẩn hóa quy trình quyết định đầu tư tài sản cũ
- Bảo vệ vốn thông qua quy tắc rõ ràng
- Giảm mua hàng theo cảm tính
- Định lượng ROI và rủi ro trước khi mua
- Có cơ chế bảo vệ vốn bằng giá trị sắt vụn (Scrap Floor)
- Hỗ trợ đánh giá có kiểm soát rủi ro xác thực đối với tài sản nghệ thuật

Giai đoạn đầu triển khai trên Google Sheet.
Thiết kế phải cho phép nâng cấp thành hệ thống Web trong tương lai.

---

# 2. MỤC TIÊU KINH DOANH

## 2.1 Mục tiêu chính

1. ROI trung bình tối thiểu ≥ 25%
2. Tỷ lệ lỗ < 10%
3. Chu kỳ quay vòng vốn ≤ 60 ngày
4. Không cho phép mua tài sản rủi ro cao thiếu dữ liệu
5. Luôn có cơ chế bảo vệ vốn thông qua Scrap Mode

## 2.2 Định vị hệ thống

Đây không phải là công cụ theo dõi bán hàng.
Đây là hệ thống kiểm soát quyết định đầu tư.

---

# 3. PHẠM VI DỰ ÁN (SCOPE)

## 3.1 Trong phạm vi (In Scope)

- Thiết bị IT (PC / Server / Laptop)
- Máy nông nghiệp
- Máy công nghiệp (CNC – có kiểm soát)
- Tranh / Gốm / Tài sản nghệ thuật
- Tính toán ROI
- Tính Scrap Floor
- Chấm điểm rủi ro
- Bộ máy quyết định BUY / NEGOTIATE / SKIP
- Theo dõi KPI
- Ghi nhận kết quả sau bán

## 3.2 Ngoài phạm vi (Giai đoạn 1)

- Tích hợp API marketplace tự động
- Crawl giá thời gian thực
- Dự đoán giá bằng Machine Learning
- Hệ thống kế toán / thuế
- AI xác thực tranh nâng cao

---

# 4. NGƯỜI DÙNG & VAI TRÒ

| Vai trò | Mô tả |
|----------|------|
| Owner | Người quyết định cuối cùng |
| Evaluator | Nhập dữ liệu & đánh giá ban đầu |
| Reviewer | Kiểm soát rủi ro & phê duyệt |

Giai đoạn đầu có thể một người đảm nhiệm tất cả.

---

# 5. CÁC CHẾ ĐỘ HOẠT ĐỘNG

Hệ thống phải hỗ trợ 3 chế độ:

---

## 5.1 RESALE MODE (Mua để bán lại)

Yêu cầu nhập:

- Giá chào bán
- Ước tính giá bán nội địa
- Ước tính giá xuất khẩu
- Các chi phí liên quan

Hệ thống phải:

- Tính Total Cost : giá mua + chi phí
- Tính Expected Profit : giá bán tốt nhất – chi phí
- Tính ROI : expected profit / total cost
- Tính Risk Score : cộng điểm rủi ro theo checklist
- Đưa ra gợi ý quyết định

Nếu thiếu dữ liệu resale → tự động chuyển sang SCRAP mode.
ý nghĩa của ROI - tỉ xuất lợi nhuận ( giúp đánh giá hiệu quả đầu tư.): 
- ROI ≥ 25% → BUY
- 15% ≤ ROI < 25% → NEGOTIATE
- ROI < 15% → SKIP
---

## 5.2 SCRAP MODE (Mua theo giá trị kim loại)

Áp dụng khi không có dữ liệu resale đáng tin cậy.

Yêu cầu nhập:

- Trọng lượng ước tính (kg)
- Giá sắt/kg
- Chi phí tháo dỡ
- Chi phí vận chuyển
- Chi phí xử lý dầu / chất thải

Hệ thống phải:

- Tính scrap_total_value : trọng lượng × giá sắt/kg
- Tính net_scrap_floor (giá mua sắt phế liệu): scrap_total_value - chi phí
- So sánh giá mua với scrap floor
- Đưa ra gợi ý quyết định

Nếu không tính được scrap floor → mặc định SKIP.

---

## 5.3 ART MODE (Đầu tư tranh có kiểm soát)

Yêu cầu nhập:

- Tên nghệ sĩ
- Tình trạng chữ ký
- Giấy tờ provenance
- Dữ liệu đấu giá gần nhất
- Tình trạng xác thực chuyên gia
- Các chi phí liên quan

Hệ thống phải:

- Tính ROI
- Tăng điểm rủi ro nếu thiếu dữ liệu
- Áp dụng ngưỡng ROI cao hơn (≥ 40%)
- Đưa ra gợi ý quyết định

---

# 6. YÊU CẦU CHỨC NĂNG (FUNCTIONAL REQUIREMENTS)

FR-01: Tạo bản ghi tài sản  
FR-02: Nhập đầy đủ các loại chi phí  
FR-03: Tự động tính Total Cost  
FR-04: Xác định giá bán tốt nhất  
FR-05: Tính Expected Profit  
FR-06: Tính ROI  
FR-07: Tính Scrap Floor (nếu có dữ liệu)  
FR-08: Tính Risk Score (có cộng điểm theo mode)  
FR-09: Decision Engine (BUY / NEGOTIATE / SKIP)  
FR-10: Ghi nhận kết quả sau bán  

---

# 7. QUY TẮC RA QUYẾT ĐỊNH

## 7.1 Resale Mode

ROI ≥ 25% → BUY  
15% ≤ ROI < 25% → NEGOTIATE  
ROI < 15% → SKIP  

## 7.2 Risk Rule

Risk ≥ 80 → AUTO SKIP  

- ý nghĩa của điểm rủi ro:
    - Điểm rủi ro càng cao → khả năng thua lỗ càng lớn
    - Điểm rủi ro ≥ 80 → hệ thống tự động đề xuất SKIP
    - Điểm rủi ro < 80 → hệ thống vẫn đưa ra gợi ý quyết định dựa trên ROI và các yếu tố khác, nhưng người dùng cần xem xét kỹ lại trước khi quyết định BUY hoặc NEGOTIATE
- Cách tính điểm rủi ro có thể bao gồm:
    - Thiếu dữ liệu resale → +20 điểm
    - Tình trạng tài sản kém → +15 điểm
    - Lịch sử giao dịch xấu → +10 điểm
    - Rủi ro xác thực (đối với tranh) → +30 điểm
    - Các yếu tố rủi ro khác → +5-10 điểm tùy trường hợp

## 7.3 Scrap Rule

Giá mua > net_scrap_floor → SKIP  
Giá mua ≤ 80% net_scrap_floor → BUY  

## 7.4 Art Rule

Không provenance + nghệ sĩ không rõ → SKIP  
Không có expert validation (trên ngưỡng giá trị) → SKIP  
ROI < 40% → NEGOTIATE / SKIP  

---

# 8. YÊU CẦU PHI CHỨC NĂNG (NON-FUNCTIONAL)

NFR-01: Ưu tiên an toàn vốn  
NFR-02: Minh bạch công thức  
NFR-03: Có thể mở rộng thành Web  
NFR-04: Triển khai được trên Google Sheet  
NFR-05: Dễ kiểm toán & truy vết quyết định  

---

# 9. QUY TRÌNH VẬN HÀNH

1. Tìm nguồn tài sản  
2. Nhập dữ liệu  
3. Chọn mode  
4. Hoàn tất checklist rủi ro  
5. Xem kết quả ROI & Risk  
6. Xem decision_suggested  
7. Reviewer xác nhận decision_final  
8. Thực hiện mua  
9. Theo dõi sau bán  

---

# 10. KPI HỆ THỐNG

- ROI trung bình ≥ 25%
- Tỷ lệ lỗ < 10%
- Thời gian quay vòng ≤ 60 ngày
- Tỷ lệ scrap fallback < 20%
- Không có deal Risk ≥ 80 được BUY

---

# 11. RỦI RO HỆ THỐNG

- Ước tính resale sai
- Ước tính trọng lượng scrap sai
- Chấm điểm rủi ro thiếu chính xác
- Rủi ro xác thực tranh
- Sai sót nhập liệu

---

# 12. LỘ TRÌNH PHÁT TRIỂN

Phase 1: Google Sheet  
Phase 2: Apps Script tự động hóa  
Phase 3: Web Dashboard  
Phase 4: AI Pricing Engine  
Phase 5: Demand Forecasting  

---

# 13. TIÊU CHÍ THÀNH CÔNG

Hệ thống được coi là thành công khi:

- Kiểm soát được lỗ vốn
- Giảm biến động ROI
- Chuẩn hóa quy trình quyết định
- Loại bỏ mua hàng cảm tính
- Có dữ liệu đủ để mở rộng quy mô

---

KẾT THÚC TÀI LIỆU
