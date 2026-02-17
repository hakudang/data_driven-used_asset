# HỆ THỐNG KINH DOANH THIẾT BỊ CŨ DỰA TRÊN DỮ LIỆU
```
Phiên bản: v1.0
Chủ sở hữu: Dang
Khu vực: Nhật Bản
Triết lý: Bảo toàn vốn – Ra quyết định bằng dữ liệu – Kiểm soát rủi ro – Tối ưu thanh khoản
Deployment : google sheet, app (tương lai)
```
✔ Scrap Valuation Engine

✔ Art Authentication & Risk Model

✔ Logic chuyển đổi giữa Resale Mode và Scrap Mode

✔ Cập nhật risk scoring & decision flow

---

# 1. TRIẾT LÝ HỆ THỐNG

Không đầu cơ.
Không mua theo cảm tính.
Mỗi tài sản phải có:

- Giá sàn bảo vệ vốn
- ROI định lượng
- Điểm rủi ro
- Kịch bản thoát hàng

---

# 2. 3 CHẾ ĐỘ ĐÁNH GIÁ TÀI SẢN

Hệ thống hoạt động theo 3 mode:

1. RESALE MODE → Bán lại kiếm lợi nhuận
2. SCRAP MODE → Mua theo giá trị sắt vụn
3. ART MODE → Đầu tư có kiểm soát rủi ro xác thực

---

# 3. MASTER DATA STRUCTURE (CẬP NHẬT)

## 3.1 Bảng Asset Evaluation

| Trường | Mô tả |
|--------|------|
| asset_id | Mã nội bộ |
| evaluation_mode | RESALE / SCRAP / ART |
| category | IT / Nông nghiệp / CNC / Art |
| maker | Hãng |
| model | Model |
| serial_number | Serial |
| manufacture_year | Năm SX |
| usage_hours | Số giờ |
| condition_grade | A/B/C/D |
| cosmetic_score | 1–5 |
| mechanical_score | 1–5 |
| asking_price | Giá chào |
| purchase_price_target | Giá mục tiêu |
| transport_cost | Vận chuyển |
| repair_cost | Sửa |
| cleaning_cost | Làm sạch |
| storage_cost_est | Lưu kho |
| compliance_cost | Pháp lý |
| total_cost | Tổng chi phí |
| domestic_resale_est | Giá bán JP |
| export_resale_est | Giá xuất khẩu |
| expected_profit | Lợi nhuận |
| roi_percent | ROI |
| scrap_floor_value | Giá trị sắt vụn ròng |
| risk_score | 1–100 |
| liquidity_score | 1–5 |
| decision | BUY / NEGOTIATE / SKIP |

---

# 4. MODULE 1: SCRAP VALUATION ENGINE

Áp dụng khi:
- Không có dữ liệu thị trường
- Máy hỏng nặng
- Không có buyer

## 4.1 Trường bổ sung

| Field | Mô tả |
|-------|------|
| estimated_weight_kg | Khối lượng ước tính |
| scrap_price_per_kg | Giá sắt/kg |
| scrap_total_value | Tổng giá trị sắt |
| dismantle_cost | Chi phí tháo dỡ |
| scrap_transport_cost | Vận chuyển |
| hazardous_material_cost | Xử lý dầu, chất thải |
| net_scrap_floor | Giá sàn ròng |

---

## 4.2 Công thức

scrap_total_value =
estimated_weight_kg × scrap_price_per_kg

net_scrap_floor =
scrap_total_value
- dismantle_cost
- scrap_transport_cost
- hazardous_material_cost

---

## 4.3 Quy tắc quyết định SCRAP MODE

- If Purchase Price > net_scrap_floor → SKIP

- If Purchase Price ≤ 80% net_scrap_floor → BUY (Low Risk)

- If 80–100% → NEGOTIATE

---

## 4.4 Risk Control

- Nếu không xác định được trọng lượng tương đối chính xác → Risk +20 điểm

- Nếu có dầu công nghiệp cần xử lý → Risk +15 điểm

---

# 5. MODULE 2: ART AUTHENTICATION & RISK MODEL

Áp dụng khi:
category = ART

## 5.1 Trường bổ sung

| Field | Mô tả |
|-------|------|
| artist_name | Tên nghệ sĩ |
| signature_present | Có chữ ký |
| signature_verified | Đã xác thực |
| provenance_docs | Có giấy tờ |
| auction_comparison_price | Giá đấu giá gần nhất |
| medium | Sơn dầu / Gốm / In |
| reproduction_risk | Low/Medium/High |
| expert_validation | Có chuyên gia |

---

## 5.2 Quy tắc bắt buộc

- If No provenance AND Unknown artist → SKIP

- If No expert validation AND giá trị > 500,000 JPY → SKIP

- If No auction data → Risk +25

---

## 5.3 ROI cho ART

expected_profit =
auction_comparison_price - total_cost

ROI ≥ 40% mới xem xét BUY

---

## 5.4 Rủi ro đặc thù

- Hàng in hàng loạt giả chữ ký
- Thị trường biến động mạnh
- Thanh khoản thấp

---

# 6. MÔ HÌNH TÍNH ROI CHUNG

total_cost =
purchase_price_target
+ transport
+ repair
+ cleaning
+ storage
+ compliance

expected_profit =
max(domestic_resale_est, export_resale_est) - total_cost

roi_percent =
(expected_profit / total_cost) × 100

---

# 7. HỆ THỐNG CHẤM ĐIỂM RỦI RO (CẬP NHẬT)

Risk components:

- Mechanical uncertainty (0–20)
- Market liquidity (0–20)
- Repair unpredictability (0–20)
- Legal exposure (0–10)
- Capital concentration (0–20)
- Mode-specific risk (0–20)

Total max: 110

Chuẩn hóa về 100 nếu cần.

If Risk > 80 → AUTO SKIP

---

# 8. PROMPT AI (CẬP NHẬT)

## 8.1 Prompt RESALE

Phân tích tài sản dưới góc nhìn bảo thủ.
Tính giá bán nội địa và xuất khẩu.
Đưa ra ROI.
Đánh giá thanh khoản.
Phân tích rủi ro.
Kết luận BUY / NEGOTIATE / SKIP.

---

## 8.2 Prompt SCRAP

Ước tính trọng lượng.
Đánh giá khả năng tháo dỡ.
Phân tích chi phí xử lý.
Xác định net scrap floor.
Khuyến nghị có nên mua theo giá sắt vụn không.

---

## 8.3 Prompt ART

Phân tích khả năng xác thực.
So sánh giá đấu giá.
Đánh giá rủi ro giả mạo.
Phân tích thanh khoản.
Khuyến nghị.

---

# 9. WORKFLOW CẬP NHẬT

1. Nhập dữ liệu
2. Chọn evaluation_mode
3. Chạy AI theo mode
4. Tính ROI
5. Tính Scrap Floor (nếu cần)
6. Tính Risk Score
7. Human Review
8. Final Decision

---

# 10. KPI NÂNG CAO

- ROI trung bình theo mode
- Tỷ lệ mua thành công
- Tỷ lệ sai lệch ước tính
- Tỷ lệ hàng phải bán lỗ
- Tỷ lệ scrap fallback

---

# 11. CHIẾN LƯỢC PHÒNG THỦ

Nếu resale thất bại:
→ Chuyển sang scrap mode

Nếu art không thanh khoản:
→ Giảm giá 30% sau 90 ngày

---

# 12. TẦM NHÌN v1.0

Xây dựng hệ thống:

"Multi-Strategy Used Asset Investment Engine"

Không chỉ bán lại. Mà có 3 lớp bảo vệ:

- Resale
- Scrap
- Authentication Control

Bảo vệ vốn trước. Tối ưu lợi nhuận sau.

---

## Bản v1.0 này:

✔ Có 3 chế độ rõ ràng
✔ Có Scrap Floor bảo vệ vốn
✔ Có Art Risk Model
✔ Có logic chuyển đổi giữa mode
✔ Có risk control nâng cao

## NEXT 

- thiết kế Google Sheet v3.0 chạy thực tế

- Hoặc viết schema SQL + ERD

- Hoặc thiết kế App structure React + Node