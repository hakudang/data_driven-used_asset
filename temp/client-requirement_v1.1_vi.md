# TÀI LIỆU YÊU CẦU KHÁCH HÀNG (CLIENT REQUIREMENT – CR)
## Hệ thống Đánh giá & Kiểm soát Đầu tư Tài sản Cũ dựa trên Dữ liệu + AI Reviewer (Google Sheets)

```text
Phiên bản: 1.1 (Cập nhật theo Model A – AI chạy theo dòng đang chọn)
Chủ sở hữu: Dang
Khu vực áp dụng: Nhật Bản
Loại tài liệu: Client Requirement (CR)
Trạng thái: Hoàn chỉnh – Sẵn sàng chuyển sang System Requirement (SR)
```

---

# 1. TÓM TẮT ĐIỀU HÀNH

Tài liệu này mô tả các yêu cầu nghiệp vụ của hệ thống đánh giá tài sản cũ dựa trên dữ liệu, triển khai theo 2 lớp:

1) **Rule Engine** (Google Sheets) – tính toán ROI/Risk/Scrap Floor và đề xuất quyết định  
2) **AI Reviewer** (Apps Script gọi API, UI bằng Sidebar) – chạy theo yêu cầu cho **1 dòng đang chọn**

Mục tiêu hệ thống:

- Chuẩn hóa quy trình quyết định đầu tư tài sản cũ
- Bảo vệ vốn thông qua quy tắc rõ ràng (Decision Engine)
- Giảm mua hàng theo cảm tính
- Định lượng ROI và rủi ro trước khi mua
- Có cơ chế bảo vệ vốn bằng giá trị sắt vụn (Scrap Floor)
- Hỗ trợ đánh giá có kiểm soát rủi ro xác thực đối với tài sản nghệ thuật
- Có AI tư vấn bổ sung (không thay thế rule engine): phát hiện thiếu dữ liệu, rủi ro ẩn, đề xuất counter-offer

Giai đoạn đầu triển khai trên Google Sheet.  
Thiết kế cho phép nâng cấp thành Web App trong tương lai.

---

# 2. MỤC TIÊU KINH DOANH

## 2.1 Mục tiêu chính

1. ROI trung bình tối thiểu ≥ 25% (đối với RESALE)
2. Tỷ lệ lỗ < 10%
3. Chu kỳ quay vòng vốn ≤ 60 ngày (mục tiêu)
4. Không cho phép mua tài sản rủi ro cao thiếu dữ liệu (Risk Gate)
5. Luôn có cơ chế bảo vệ vốn thông qua SCRAP mode (Scrap Floor)

## 2.2 Định vị hệ thống

Đây không phải là công cụ theo dõi bán hàng.  
Đây là hệ thống **kiểm soát quyết định đầu tư** (Investment Decision Control)  
+ **AI Reviewer** hỗ trợ đánh giá và thương lượng.

---

# 3. KIẾN TRÚC HỆ THỐNG

## 3.1 Layer 1 – Rule Engine (Google Sheet)

- Lưu dữ liệu deal (ASSETS)
- Tính Total Cost (giá vốn)
- Tính Expected Profit (lợi nhuận kỳ vọng) 
- Tính ROI (tỷ suất lợi nhuận)
- Tính Scrap Floor (giá sàn phế liệu) (SCRAP mode)
- Tính Risk Score (điểm rủi ro)
- Decision Suggested: BUY / NEGOTIATE / SKIP

## 3.2 Layer 2 – AI Reviewer (Apps Script + API)

- **Chạy on-demand** khi người dùng bấm Evaluate (Model A)
- AI đọc dữ liệu **1 dòng** đang chọn
- Gọi OpenAI API
- Trả kết quả vào các cột AI:
  - ai_output (phân tích)
  - ai_counter_offer (giá đề xuất)
  - ai_confidence (mức tự tin)
  - ai_status, ai_updated_at

**Nguyên tắc bắt buộc:** AI không được sửa công thức hoặc quyết định trong sheet. AI chỉ ghi output tư vấn.

## 3.3 UX Layer (Sidebar trong Google Sheet)

- Có menu + Sidebar Panel
- Người dùng không cần copy/paste prompt
- Không auto chạy khi sửa cell
- Tránh lag sheet, kiểm soát chi phí API

---

# 4. PHẠM VI DỰ ÁN (SCOPE)

## 4.1 Trong phạm vi (In Scope)

- Thiết bị IT (PC / Server / Laptop)
- Máy nông nghiệp
- Máy công nghiệp (CNC – có kiểm soát)
- Tranh / Gốm / Tài sản nghệ thuật
- Tính toán ROI
- Tính Scrap Floor (Scrap Mode)
- Chấm điểm rủi ro
- Bộ máy quyết định BUY / NEGOTIATE / SKIP (Rule Engine)
- Theo dõi KPI
- Ghi nhận kết quả sau bán
- **AI Reviewer chạy theo dòng đang chọn (Model A)**
- **Sidebar UX trong Google Sheet**
- Bảo mật API key bằng User Properties (không lưu trong sheet)

## 4.2 Ngoài phạm vi (Giai đoạn hiện tại)

- AI chạy realtime khi sửa cell
- AI chạy hàng loạt (batch)
- Crawl marketplace tự động / API giá realtime
- Dự đoán giá bằng Machine Learning
- Hệ thống kế toán / thuế
- AI vision xác thực tranh nâng cao
- Tự động sửa dữ liệu / tự động chốt decision_final

---

# 5. NGƯỜI DÙNG & VAI TRÒ

| Vai trò | Mô tả |
|---|---|
| Owner | Người quyết định cuối cùng, chốt decision_final |
| Evaluator | Nhập dữ liệu, ước tính resale/scrap, kiểm tra checklist |
| Reviewer | Kiểm soát rủi ro, chạy AI Reviewer, xác nhận quyết định |

Giai đoạn đầu có thể một người đảm nhiệm tất cả.

---

# 6. CÁC CHẾ ĐỘ HOẠT ĐỘNG

Hệ thống hỗ trợ 3 chế độ:

## 6.1 RESALE MODE (Mua để bán lại)

### Yêu cầu nhập
- Giá chào bán (asking_price)
- Ước tính giá bán nội địa (domestic_resale_est)
- Ước tính giá bán xuất khẩu (export_resale_est)
- Các chi phí liên quan

### Rule Engine phải
- Tính Total Cost = giá mua mục tiêu + chi phí
- Tính Expected Profit = giá bán tốt nhất – total_cost
- Tính ROI = expected_profit / total_cost
- Tính Risk Score = base risk + mode risk (nếu có)
- Đưa ra decision_suggested

### Quy tắc ROI (Resale)
- ROI ≥ 25% → BUY
- 15% ≤ ROI < 25% → NEGOTIATE
- ROI < 15% → SKIP

### AI Reviewer bổ sung
- Kiểm tra tính hợp lý của resale estimate (có evidence hay không)
- Phát hiện thiếu cost (shipping/repair/storage/compliance)
- Đề xuất counter-offer và chiến lược thương lượng
- Đưa confidence score 1–5

> Lưu ý: Việc “tự động chuyển sang SCRAP mode” khi thiếu resale data là **khuyến nghị vận hành**.  
> Trong v3.0 sheet có thể thực hiện bằng rule/validation; AI cũng sẽ cảnh báo thiếu dữ liệu.

---

## 6.2 SCRAP MODE (Mua theo giá trị kim loại)

Áp dụng khi không có dữ liệu resale đáng tin cậy hoặc mục tiêu là bảo vệ vốn.

### Yêu cầu nhập
- Trọng lượng ước tính (kg)
- Giá sắt/kg (**giá dealer thu mua thực nhận**, không phải giá bán ra)
- Chi phí tháo dỡ
- Chi phí vận chuyển
- Chi phí xử lý dầu / chất thải (nếu có)

### Rule Engine phải
- Tính `scrap_total_value` = trọng lượng × giá sắt/kg
- Tính `net_scrap_floor` = scrap_total_value - (tháo dỡ + vận chuyển + xử lý)
- So sánh **giá mua mục tiêu** với net_scrap_floor
- Đưa ra decision_suggested
- Nếu không tính được net_scrap_floor → mặc định SKIP

### Quy tắc SCRAP
- Giá mua > net_scrap_floor → SKIP
- Giá mua ≤ 80% net_scrap_floor → BUY
- Còn lại → NEGOTIATE

### AI Reviewer bổ sung
- Cảnh báo rủi ro: tạp chất, phân loại kim loại, cân sai số, dầu/chất thải phát sinh
- Kiểm tra logic tính scrap floor có “lạc quan” không
- Đề xuất counter-offer cụ thể để đạt vùng BUY/NEGOTIATE an toàn

---

## 6.3 ART MODE (Đầu tư tranh có kiểm soát)

### Yêu cầu nhập
- Tên nghệ sĩ
- Tình trạng chữ ký
- Giấy tờ provenance
- Dữ liệu đấu giá gần nhất (auction comps)
- Tình trạng xác thực chuyên gia (expert validation)
- Các chi phí liên quan

### Rule Engine phải
- Tính ROI
- Tăng điểm rủi ro nếu thiếu dữ liệu
- Áp dụng ngưỡng ROI cao hơn (≥ 40%)
- Đưa ra decision_suggested

### Quy tắc ART
- Không provenance + nghệ sĩ không rõ → SKIP
- Không có expert validation (trên ngưỡng giá trị) → SKIP
- ROI < 40% → NEGOTIATE / SKIP

### AI Reviewer bổ sung
- Đánh giá rủi ro giả mạo, thiếu giấy tờ, thiếu auction comps
- Đưa đề xuất hành động: cần xác thực gì trước khi mua
- Đưa confidence score 1–5

---

# 7. YÊU CẦU CHỨC NĂNG (FUNCTIONAL REQUIREMENTS)

## 7.1 Core Rule Engine (Google Sheet)

FR-01: Tạo bản ghi tài sản (1 dòng = 1 deal)  
FR-02: Nhập dữ liệu cơ bản, chi phí, resale/scrap/art inputs  
FR-03: Tự động tính Total Cost  
FR-04: Xác định giá bán tốt nhất (resale best / auction / scrap floor)  
FR-05: Tính Expected Profit  
FR-06: Tính ROI  
FR-07: Tính Scrap Floor (SCRAP mode)  
FR-08: Tính Risk Score (base + mode)  
FR-09: Decision Engine (BUY / NEGOTIATE / SKIP)  
FR-10: Ghi nhận kết quả sau bán (post_sale_result, final_profit_actual)

## 7.2 AI Reviewer (Model A – Selected Row)

FR-11: Có menu “Used Asset AI” trên Google Sheets  
FR-12: Có Sidebar UX (AI Panel) hiển thị preview dòng đang chọn  
FR-13: Chạy AI Evaluate cho **1 dòng đang chọn**  
FR-14: Build prompt theo mode (RESALE/SCRAP/ART)  
FR-15: Gọi OpenAI API và nhận kết quả  
FR-16: Ghi kết quả AI vào các cột ai_* (ai_output, ai_counter_offer, ai_confidence, ai_status, ai_updated_at)  
FR-17: Lưu API key ở User Properties (không lưu trong sheet)  
FR-18: Xử lý lỗi cơ bản (thiếu key, lỗi API, lỗi sheet name, row < 2)

**Ràng buộc:** AI không được tự sửa công thức, không được tự set decision_final.

---

# 8. QUY TẮC RA QUYẾT ĐỊNH (DECISION GOVERNANCE)

## 8.1 Rule Engine là nguồn quyết định chính
- decision_suggested do Rule Engine tạo ra
- decision_final do Owner/Reviewer chốt

## 8.2 Risk Rule (Gate)
- Risk ≥ 80 → AUTO SKIP (decision_suggested = SKIP)

## 8.3 Vai trò của AI
AI chỉ cung cấp:
- Data Check: thiếu/không chắc dữ liệu nào
- Calculation Check: kiểm tra tính hợp lý của input/cost
- Decision Review: đồng ý/không đồng ý với decision_suggested (nêu lý do)
- Counter-offer: giá đề xuất
- Confidence: 1–5
- Risks: top 3 rủi ro cần xác minh

AI không override quyết định.

---

# 9. YÊU CẦU PHI CHỨC NĂNG (NON-FUNCTIONAL REQUIREMENTS)

NFR-01: Ưu tiên an toàn vốn (capital protection)  
NFR-02: Minh bạch công thức (audit được)  
NFR-03: Triển khai trên Google Sheet trước (MVP)  
NFR-04: Dễ mở rộng lên Web trong tương lai  
NFR-05: Dễ kiểm toán & truy vết quyết định (log ai_updated_at, reviewer, decision_final)  
NFR-06: Bảo mật API key (User Properties, không lộ trong sheet)  
NFR-07: AI chỉ chạy on-demand (Model A) để kiểm soát chi phí và tránh lag  
NFR-08: Hệ thống vẫn chạy được nếu AI tạm thời không hoạt động (fallback rule engine)

---

# 10. QUY TRÌNH VẬN HÀNH (SOP)

1. Tìm nguồn tài sản (listing / dealer / giới thiệu)  
2. Nhập dữ liệu vào ASSETS (đúng mode)  
3. Điền checklist rủi ro (IT/MACHINE/ART) và set base_risk_score  
4. Xem ROI & Risk & decision_suggested (Rule Engine)  
5. Chọn dòng → mở AI Panel (Sidebar)  
6. Bấm Evaluate Selected Row  
7. Đọc ai_output + counter-offer + confidence  
8. Reviewer/Owner chốt decision_final  
9. Thực hiện mua hoặc bỏ qua  
10. Sau khi bán: cập nhật post_sale_result + final_profit_actual để học lại hệ thống

---

# 11. KPI HỆ THỐNG

- ROI trung bình ≥ 25% (RESALE)  
- Tỷ lệ lỗ < 10%  
- Thời gian quay vòng ≤ 60 ngày (mục tiêu)  
- Tỷ lệ fallback SCRAP < 20% (mục tiêu)  
- Không có deal Risk ≥ 80 được BUY  
- 100% deal “giá trị lớn” được AI review (quy định nội bộ)  
- AI không làm thay đổi rule logic (không sửa công thức)

---

# 12. RỦI RO HỆ THỐNG

- Ước tính resale sai (thiếu evidence)
- Ước tính trọng lượng scrap sai (tạp chất, cân thiếu, phân loại kim loại)
- Chấm điểm rủi ro thiếu chính xác (checklist chưa chuẩn)
- Rủi ro xác thực tranh (provenance/auction/expert)
- Sai sót nhập liệu (gõ nhầm số, format)
- Công thức bị ghi đè (phải protect ranges)
- API limit / quota / chi phí tăng nếu lạm dụng AI
- AI trả lời sai (AI là tư vấn, không phải quyết định)

---

# 13. LỘ TRÌNH PHÁT TRIỂN (ROADMAP – ĐIỀU CHỈNH)

Phase 1: Google Sheet Rule Engine (v3.0)  
Phase 2: Apps Script + Sidebar UX + AI Reviewer (Model A)  
Phase 3: AI Log & Audit Trail (tab AI_LOG, lưu history)  
Phase 4: Batch evaluation (nếu quy mô tăng, cân nhắc)  
Phase 5: Web App (Sheet làm DB hoặc migrate DB)

---

# 14. TIÊU CHÍ THÀNH CÔNG (SUCCESS CRITERIA)

Hệ thống được coi là thành công khi:

- Kiểm soát được lỗ vốn và giảm deal “mua theo cảm tính”
- Quyết định đầu tư được chuẩn hóa bằng rule + checklist
- AI hỗ trợ tạo ra counter-offer thực tế, giúp nâng ROI và giảm risk
- Có dữ liệu đủ để phân tích và nâng cấp (scale)
- Có thể chuyển đổi lên Web App khi cần

---

# KẾT THÚC TÀI LIỆU
