# HỆ THỐNG KINH DOANH THIẾT BỊ CŨ DỰA TRÊN DỮ LIỆU
Phiên bản: v2.1
Chủ sở hữu: Dang
Khu vực: Nhật Bản
Mô hình: Mua – Tái chế – Bán lại thiết bị cũ
Triết lý: Bảo toàn vốn + Ra quyết định bằng dữ liệu + Kiểm soát rủi ro

---

# 1. TÓM TẮT ĐIỀU HÀNH

Tài liệu này định nghĩa một hệ thống chuẩn hóa để:

- Đánh giá thiết bị cũ trước khi mua
- Kiểm soát vốn
- Tính toán ROI
- Giảm thiểu rủi ro kỹ thuật và thanh khoản
- Chuẩn hóa quyết định: MUA / THƯƠNG LƯỢNG / BỎ QUA

Mục tiêu:
Không mua theo cảm tính.
Mỗi quyết định phải có số liệu bảo vệ.

---

# 2. NỀN TẢNG CHIẾN LƯỢC

## 2.1 Nhóm hàng mục tiêu

Ưu tiên:

1. Thiết bị IT (PC / Server / Laptop)
2. Máy nông nghiệp
3. Máy công nghiệp (CNC – chỉ khi có khách trước)
4. Đồ nghệ thuật (chỉ khi có chuyên gia xác thực)

---

## 2.2 Nguyên tắc bảo toàn vốn

Quy tắc 1:
Không một thương vụ nào vượt quá 20% tổng vốn lưu động.

Quy tắc 2:
ROI mục tiêu tối thiểu = 25%

Quy tắc 3:
Chu kỳ quay vòng tiền mặt < 60 ngày

Quy tắc 4:
Tồn kho tối đa không vượt quá 3 tháng doanh thu trung bình.

---

# 3. CẤU TRÚC DỮ LIỆU CHÍNH

## 3.1 Bảng đánh giá tài sản

| Trường | Mô tả |
|--------|------|
| asset_id | Mã nội bộ |
| category | IT / Nông nghiệp / CNC / Nghệ thuật |
| maker | Hãng |
| model | Model |
| serial_number | Số serial |
| manufacture_year | Năm SX |
| usage_hours | Số giờ hoạt động |
| condition_grade | A / B / C / D |
| cosmetic_score | Điểm ngoại hình (1–5) |
| mechanical_score | Điểm cơ khí (1–5) |
| data_security_risk | Rủi ro dữ liệu (IT) |
| asking_price | Giá chào bán |
| negotiation_margin | % kỳ vọng giảm giá |
| purchase_price_target | Giá mục tiêu mua |
| transport_cost | Chi phí vận chuyển |
| repair_cost | Chi phí sửa |
| cleaning_cost | Chi phí làm sạch |
| storage_cost_est | Chi phí lưu kho |
| compliance_cost | Chi phí pháp lý |
| total_cost | Tổng chi phí |
| domestic_resale_est | Giá bán nội địa |
| export_resale_est | Giá bán xuất khẩu |
| liquidity_score | Thanh khoản (1–5) |
| expected_profit | Lợi nhuận kỳ vọng |
| roi_percent | ROI (%) |
| risk_score | Điểm rủi ro (1–100) |
| decision | MUA / THƯƠNG LƯỢNG / BỎ QUA |
| reviewer | Người duyệt |
| evaluation_date | Ngày đánh giá |

---

# 4. MÔ HÌNH TÍNH CHI PHÍ

## 4.1 Công thức tổng chi phí

Tổng chi phí =
Giá mua mục tiêu
+ Vận chuyển
+ Sửa chữa
+ Làm sạch
+ Lưu kho
+ Pháp lý
+ Xóa dữ liệu (nếu là IT)

---

## 4.2 Chi phí lưu kho

Chi phí lưu kho =
Chi phí kho hàng tháng × Số tháng dự kiến bán

Thời gian bán dự kiến:

- IT: 30–45 ngày
- Nông nghiệp: 60–120 ngày
- CNC: 90–180 ngày

---

# 5. MÔ HÌNH LỢI NHUẬN & ROI

Lợi nhuận kỳ vọng =
Giá bán cao hơn (nội địa hoặc xuất khẩu) – Tổng chi phí

ROI (%) =
(Lợi nhuận / Tổng chi phí) × 100

---

# 6. HỆ THỐNG CHẤM ĐIỂM RỦI RO

## 6.1 Thành phần rủi ro

- Không chắc chắn kỹ thuật (0–20)
- Hư hỏng ngoại hình (0–10)
- Thanh khoản thị trường (0–20)
- Không chắc chắn sửa chữa (0–20)
- Rủi ro pháp lý (0–10)
- Tập trung vốn (0–20)

Tổng tối đa: 100 điểm

---

## 6.2 Diễn giải

0–30: Thấp  
31–60: Trung bình  
61–100: Cao  

Nếu Risk > 60 → hạ cấp quyết định 1 mức  
Nếu Risk > 80 → Tự động BỎ QUA  

---

# 7. CHECKLIST THEO NHÓM HÀNG

## 7.1 Thiết bị IT

- Kiểm tra BIOS lock
- Kiểm tra SMART HDD
- Kiểm tra số vòng khởi động
- Kiểm tra RAID
- Có cần giấy chứng nhận xóa dữ liệu?
- Test nguồn PSU
- Kiểm tra asset tag doanh nghiệp

Rủi ro ẩn:
Khóa bảo mật từ công ty cũ.

---

## 7.2 Máy nông nghiệp

- Test khởi động nguội
- Kiểm tra rò dầu
- Kiểm tra hệ thủy lực
- Kiểm tra hao mòn gầm
- Kiểm tra gian lận đồng hồ giờ
- Kiểm tra phụ tùng còn sản xuất không

Rủi ro ẩn:
Hỏng hộp số chi phí cực cao.

---

## 7.3 CNC

- Kiểm tra controller
- Độ chính xác trục
- Giờ spindle
- Chi phí tháo dỡ
- Pháp lý xử lý dầu

Quy tắc:
Không mua nếu chưa có người mua trước.

---

## 7.4 Nghệ thuật

- Xác thực nguồn gốc
- So sánh đấu giá
- Kiểm tra giả mạo
- Yêu cầu chuyên gia xác nhận

---

# 8. HỆ THỐNG PROMPT AI

## 8.1 Prompt đánh giá chuẩn

Bạn là chuyên gia phân tích thị trường thiết bị cũ tại Nhật.

Hãy cung cấp:

1. Giá bán nội địa ước tính
2. Giá xuất khẩu ước tính
3. Thanh khoản (1–5)
4. Rủi ro chi tiết
5. Rủi ro ẩn
6. Giá mua đề xuất
7. Kết luận: MUA / THƯƠNG LƯỢNG / BỎ QUA

Hãy đánh giá bảo thủ và ưu tiên an toàn vốn.

---

## 8.2 Prompt phân tích hình ảnh

Phân tích hình ảnh để:

- Xác định mức độ rỉ sét
- Rò rỉ dầu
- Hư hỏng kết cấu
- Thiếu linh kiện
- Dấu hiệu hao mòn nặng

Trả về điểm rủi ro cơ khí 1–5.

---

# 9. QUY TRÌNH VẬN HÀNH

1. Tìm nguồn hàng
2. Thu thập dữ liệu
3. Chụp ảnh
4. Nhập vào hệ thống
5. Chạy AI đánh giá
6. Tính ROI
7. Kiểm tra rủi ro
8. Phê duyệt
9. Thương lượng
10. Mua
11. Tái chế
12. Đăng bán
13. Theo dõi
14. Đánh giá sau bán

---

# 10. KPI THEO DÕI

- Tỷ suất lợi nhuận gộp (%)
- ROI trung bình
- Vòng quay tồn kho
- Số ngày tồn kho trung bình
- Tỷ lệ lỗ
- Tỷ lệ thành công thương lượng
- Tỷ lệ thu hồi vốn

---

# 11. TUÂN THỦ PHÁP LÝ

Bắt buộc:

- 古物商許可
- Lưu log giao dịch
- Xác minh người bán
- Lưu chứng nhận xóa dữ liệu
- Kiểm tra tuân thủ xuất khẩu

Kiểm tra định kỳ mỗi quý.

---

# 12. CHIẾN LƯỢC PHÂN BỔ VỐN

Giai đoạn 1:
80% vốn → IT
20% → thử nghiệm

Giai đoạn 2:
Mở rộng sang nông nghiệp

Giai đoạn 3:
CNC chỉ khi có khách

---

# 13. LỘ TRÌNH MỞ RỘNG

Level 1:
Google Sheet + AI thủ công

Level 2:
Tự động hóa bằng Apps Script

Level 3:
Dashboard riêng (React + Node)

Level 4:
Dự đoán nhu cầu thị trường bằng AI

---

# 14. CHIẾN LƯỢC THOÁT HÀNG

Nếu > 120 ngày chưa bán:
→ Giảm giá mạnh

Nếu 2 tháng liên tiếp lỗ:
→ Tạm dừng nhóm hàng đó

---

# 15. TẦM NHÌN DÀI HẠN

Xây dựng:

Cỗ máy đầu tư tài sản cũ dựa trên AI.

Định vị:
Không phải người bán lại.
Mà là nhà giao dịch tài sản có lợi thế dữ liệu.



## NEXT STEPS
- Xuất thêm bản inventory-template.csv

- Hoặc thiết kế luôn cấu trúc Google Sheet chuẩn để dùng ngay

- Hoặc build kiến trúc API + Apps Script tự động hóa