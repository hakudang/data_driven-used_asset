# Data-Driven Used Equipment Investment Guide

```
version 1.0
```
Hướng dẫn sử dụng hệ thống đánh giá và ra quyết định đầu tư vào thiết bị đã qua sử dụng, được triển khai trên Google Sheets.
- RESALE : mua để bán lại máy móc 
- SCRAP : mua để bán sắt vụn
- ART : mua để bán lại tác phẩm nghệ thuật


## A. Tabs
- CONFIG: tham số (ROI, Risk, Scrap ratio, Holding days, storage default, giá sắt/kg)
- ASSETS: tab vận hành chính (nhập deal, chi phí, ước tính, auto ROI/Risk/Decision)
- IT_CHECKLIST / MACHINE_CHECKLIST / ART_CHECKLIST: checklist theo mode
- DASHBOARD: khung (tự tạo Pivot)

## B. Quy trình dùng hằng ngày
1. Vào CONFIG cập nhật:
   - ROI_MIN_BUY :  % ngưỡng tối thiểu để BUY
   - ROI_MIN_NEGOTIATE : % ngưỡng tối thiểu để NEGOTIATE( thương lượng )
   - RISK_AUTO_SKIP : nếu risk > ngưỡng này → gợi ý SKIP
   - SCRAP_BUY_RATIO : nếu scrap floor / asking_price > ratio này → BUY
   - ART_ROI_MIN_BUY : % ngưỡng ROI tối thiểu để BUY tác phẩm nghệ thuật
   - ART_VALUE_LIMIT_NEED_EXPERT : nếu giá đấu ước tính > ngưỡng này → cần expert validation
   - Giá sắt/kg (Steel) ở CONFIG!H2
2. Vào ASSETS:
   - Nhập asset_id, evaluation_mode, category, asking_price, chi phí chính
   - Điền ước tính resale (AH/AI) hoặc auction (AJ) hoặc scrap (AK/AL/AN/AO/AF)
3. Vào checklist đúng mode, nhập thông tin và chấm risk_points (0–80) theo quy ước nội bộ
4. Quay lại ASSETS:
   - Xem decision_suggested (AW)
   - Ghi decision_final (AX) + reviewer/date

## C Nguyên tắc “không có dữ liệu”
- Nếu không có resale data: chuyển sang SCRAP mode và bắt buộc có net_scrap_floor (AP).
- Nếu thiếu trọng lượng/giá sắt/tháo dỡ → net_scrap_floor trống → decision_suggested sẽ SKIP (bảo vệ vốn).
- Với ART: thiếu auction data hoặc thiếu expert_validation khi vượt ngưỡng → gợi ý SKIP.

## D Các cột quan trọng trong ASSETS
- W: purchase_price_target
- AG: total_cost
- AQ: expected_resale_best
- AS: roi_percent
- AV: risk_score_total
- AW: decision_suggested
