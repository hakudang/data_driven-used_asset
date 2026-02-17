# TÀI LIỆU YÊU CẦU KHÁCH HÀNG (CLIENT REQUIREMENT – CR)
## Hệ thống Đánh giá & Kiểm soát Đầu tư Tài sản Cũ  
### Rule Engine + AI Reviewer (Vision + Market Advisory + Data Completeness)

```text
Version: v1.3 FINAL (Enterprise Ready – Updated with FR & NFR)
Owner: Dang
Document Type: Client Requirement (CR)
Status: Approved Baseline – Ready for SR
Region: Japan
```

---

# 0. ĐỊNH NGHĨA THUẬT NGỮ

- ROI (Return on Investment): tỉ suất lợi nhuận = Expected Profit / Total Cost  
- Scrap Floor: giá sàn phế liệu = Giá trị bảo vệ vốn tối thiểu dựa trên giá trị kim loại sau chi phí  
- Decision Suggested: Quyết định đề xuất từ Rule Engine  
- Decision Final: Quyết định cuối cùng do Owner phê duyệt  
- Confidence (1–5): Mức độ tin cậy của AI đối với phân tích  
- Market Advisory: Ước tính thị trường tham khảo (không realtime, không crawl)

---

# 1. MỤC ĐÍCH TÀI LIỆU

Chuẩn hóa quyết định đầu tư, bảo vệ vốn, giảm mua theo cảm tính, nâng cao chất lượng thương lượng và kiểm soát rủi ro bằng Rule Engine + AI Reviewer.

---

# 2. KIẾN TRÚC HỆ THỐNG

## Layer 1 – Rule Engine (Google Sheets)

- Tính Total Cost
- Tính Expected Profit
- Tính ROI
- Tính Scrap Floor
- Tính Risk Score
- Đưa Decision Suggested (BUY / NEGOTIATE / SKIP)

## Layer 2 – AI Reviewer (Model A – Selected Row)

Kích hoạt khi:
- Chọn đúng 1 dòng trong ASSETS
- Nhấn Evaluate
- Không auto-trigger

AI có thể:
- Phân tích tài chính
- OCR Info Panel
- Phân tích ảnh tổng thể
- Đưa Market Advisory (không crawl)
- Đề xuất Counter Offer
- Kiểm soát Data Completeness
- Trả Confidence Score

AI không được:
- Crawl marketplace
- Truy cập internet realtime
- Override Decision Suggested
- Tự động chốt Decision Final

---

# 3. VAI TRÒ HỆ THỐNG

- Owner
- Evaluator
- Reviewer

---

# 4. MỤC TIÊU KINH DOANH

1. ROI trung bình ≥ 25% (Resale)
2. ROI ≥ 40% (Art)
3. Tỷ lệ lỗ < 10%
4. Không BUY nếu Risk ≥ 80
5. 100% deal lớn có AI review
6. Không deal thiếu dữ liệu nghiêm trọng được BUY

---

# 5. FUNCTIONAL REQUIREMENTS (FR)

## 5.1 Rule Engine

| ID | Mô tả ngắn | Mô tả chi tiết |
|----|------------|----------------|
| FR-01 | Quản lý tài sản | Tạo, chỉnh sửa, lưu bản ghi trong ASSETS |
| FR-02 | Tính Total Cost | purchase_price + dismantle + transport + hazardous |
| FR-03 | Tính ROI | ROI = Expected Profit / Total Cost |
| FR-04 | Tính Scrap Floor | weight × scrap_price – chi phí |
| FR-05 | Decision Suggested | Tự động đề xuất BUY/NEGOTIATE/SKIP |
| FR-06 | Risk Auto Skip | Risk ≥ threshold → SKIP |

### FR-03: Tính ROI
- Tự động cập nhật khi thay đổi dữ liệu.
- Ảnh hưởng trực tiếp đến decision_suggested.

**Acceptance**: ROI cập nhật tức thì và decision thay đổi đúng rule.

---

## 5.2 AI Reviewer

| ID | Mô tả ngắn | Mô tả chi tiết |
|----|------------|----------------|
| FR-07 | Trigger AI | Chỉ chạy khi chọn 1 dòng + nhấn Evaluate |
| FR-08 | JSON Output | AI phải trả JSON hợp lệ |
| FR-09 | Parse & Display | Parse JSON và hiển thị Sidebar |
| FR-10 | Save Confidence | Ghi ai_confidence vào sheet |
| FR-11 | Confidence Gating | confidence ≤ 2 → không BUY |

### FR-07: Trigger AI
- Không auto run.
- Không chạy nếu chọn nhiều dòng.

**Acceptance**: Chọn sai điều kiện → AI không chạy.

---

## 5.3 Vision Module

| ID | Mô tả ngắn | Mô tả chi tiết |
|----|------------|----------------|
| FR-12 | OCR Nameplate | Trích xuất maker/model/serial/year |
| FR-13 | Compare OCR | So sánh với dữ liệu nhập |
| FR-14 | Analyze Overall Image | Phát hiện hư hỏng |
| FR-15 | Request More Photos | Yêu cầu ảnh bổ sung nếu thiếu |

### FR-12: OCR Nameplate
- Trích xuất thông tin chính.
- Cảnh báo mismatch.

**Acceptance**: Nếu model khác → hiển thị cảnh báo.

---

## 5.4 Data Completeness Governance

| ID | Mô tả ngắn | Mô tả chi tiết |
|----|------------|----------------|
| FR-16 | Check Mandatory Data | Kiểm tra dữ liệu bắt buộc theo mode |
| FR-17 | Confidence Reduction | Thiếu dữ liệu nghiêm trọng → confidence ≤ 2 |
| FR-18 | Block BUY | Không cho BUY khi thiếu dữ liệu |

### FR-18: Block BUY
- Thiếu weight (SCRAP) hoặc provenance (ART) → không BUY.

**Acceptance**: Thiếu dữ liệu nghiêm trọng → decision_review không thể là BUY.

---

# 6. NON-FUNCTIONAL REQUIREMENTS (NFR)

## 6.1 Performance

| ID | Mô tả ngắn | Mô tả chi tiết |
|----|------------|----------------|
| NFR-01 | AI Response Time | ≤ 10 giây |
| NFR-02 | Sheet Responsiveness | Không làm treo Sheet |

### NFR-01: AI Response Time
**Acceptance**: Trung bình ≤ 10 giây qua 10 lần test.

---

## 6.2 Security

| ID | Mô tả ngắn | Mô tả chi tiết |
|----|------------|----------------|
| NFR-03 | API Key Protection | Lưu trong User Properties |
| NFR-04 | No Crawling | Không crawl marketplace |

**Acceptance**: Không tìm thấy API key trong Sheet/code public.

---

# 7. ROADMAP

Phase 1: Rule Engine  
Phase 2: Apps Script + Sidebar + AI  
Phase 3: Vision + Data Completeness  
Phase 4: AI Log  
Phase 5: Batch (optional)  
Phase 6: Web App  

---

# 8. SUCCESS CRITERIA

- Giảm lỗ vốn
- Chuẩn hóa quyết định
- Counter-offer cải thiện ROI ≥ 5%
- Không BUY khi Risk ≥ 80
- Không BUY khi confidence ≤ 2
- Sẵn sàng scale Web App

---

# 9. LEGAL & RESPONSIBILITY

AI chỉ mang tính advisory.  
Quyết định tài chính thuộc về Owner.

---

# KẾT LUẬN

CR v1.3 FINAL đã bổ sung đầy đủ FR và NFR, sẵn sàng làm nền tảng cho SR, RTM và Use Case.
