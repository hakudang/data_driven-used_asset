# BÁO CÁO PHÂN TÍCH TÍNH NHẤT QUÁN HỆ THỐNG: 
## Các tài liệu đánh giá 
- CR V1.0 
- SR V1.1 
- BR V1.1

# 1. Phân tích Mục tiêu Chiến lược và Sự chuyển hóa Nghiệp vụ (CR → BR)

Quá trình chuyển hóa từ mục tiêu chiến lược (CR v1.0) sang quy tắc nghiệp vụ (BR v1.1) đảm bảo các yêu cầu định tính của khách hàng được thực thi bằng các thuật toán kiểm soát định lượng, loại bỏ các kẽ hở trong vận hành thủ công.

## Bảng đối chiếu mục tiêu chiến lược:

| Mục tiêu CR (v1.0) |	Nhóm quy tắc BR tương ứng (v1.1) |
|--------------------|-----------------------------------|
| Bảo vệ vốn	| FRL (Công thức tính giá trị sàn), RR-01 (Cơ chế Risk Override chặn deal rủi ro cao) |
| Giảm mua theo cảm tính	| DR (Áp đặt ngưỡng ROI cố định 25% và 40%) |
| Tăng chất lượng thương lượng	| AR (Khuyến nghị counter-offer dựa trên biên lợi nhuận thực tế) |
| Kiểm soát rủi ro	| RR (Tự động SKIP nếu risk_score ≥ 80), AR (Gating theo độ tin cậy AI) |
| Chuẩn hóa quyết định	| DGR (Dữ liệu bắt buộc theo mode), DR-04 (Duy trì tính nhất quán khi chuyển đổi mode đánh giá) |  

**Phân tích chuyển hóa:** 
- Mục tiêu "Bảo vệ vốn" được cụ thể hóa bằng việc kết hợp giữa quy tắc tính toán giá trị thu hồi tối thiểu (FRL) và cơ chế tự động loại bỏ (RR-01) các deal có risk_score vượt ngưỡng an toàn. 
- Yêu cầu "Chuẩn hóa quyết định" được đảm bảo thông qua DR-04, yêu cầu hệ thống tính toán lại toàn bộ logic khi có sự thay đổi về chế độ đánh giá (evaluation_mode), đảm bảo không có sai lệch dữ liệu giữa các phương pháp tiếp cận.

# 2. Phân tích Cấu trúc Logic và Công thức Tính toán (BR → SR)

Sự chuyển hóa từ BR sang SR tập trung vào việc hiện thực hóa các công thức tài chính và quy tắc quyết định thông qua các Engine chức năng (FR) và mô hình dữ liệu (Data Model).

Các công thức cốt lõi và thực thi kỹ thuật:

1. Total Cost (FRL-01): total_cost = purchase_price + dismantle_cost + transport_cost + hazardous_cost. Thực thi kỹ thuật qua FR-03.
2. Expected Profit (FRL-02): expected_profit = resale_est - total_cost. Thực thi kỹ thuật qua FR-04.
3. ROI (FRL-03): roi = expected_profit / total_cost. Thực thi kỹ thuật qua FR-05 (xử lý an toàn khi chia cho 0).
4. Scrap Floor (FRL-04): net_scrap_floor = (weight_est_kg × scrap_price_per_kg) - các chi phí liên quan. Thực thi kỹ thuật qua FR-06.
5. RESALE Engine (DR-01): Quyết định dựa trên ngưỡng ROI RESALE cấu hình. Thực thi kỹ thuật qua FR-07.
6. SCRAP Engine (DR-03): Quyết định dựa trên so sánh purchase_price và net_scrap_floor. Thực thi kỹ thuật qua FR-08.
7. ART Engine (DR-02): Quyết định dựa trên ROI ART và kiểm soát chứng thực nguồn gốc. Thực thi kỹ thuật qua FR-09.

Bảng ánh xạ dữ liệu đầu vào (Data Mapping):

| Trường dữ liệu (SR 3.1) |	Biến số công thức (BR) | Quy tắc logic kích hoạt |
|-------------------------|-------------------------|-----------------------|   
| purchase_price | purchase_price |FRL-01, DR-03 |
| resale_est | resale_est |FRL-02 (cho mode RESALE/ART) |
| weight_est_kg | weight_est_kg |FRL-04 (cho mode SCRAP) |
| scrap_price_per_kg | scrap_price_per_kg |FRL-04 (cho mode SCRAP) |
| risk_score | risk_score |RR-01 (Trigger auto-SKIP) | 
| dismantle_cost | dismantle_cost |FRL-01, FRL-04 |

# 3. Phân tích Chuyên sâu các Điểm Chốt (Frozen Points) v1.1

Phiên bản v1.1 thiết lập các ngưỡng kiểm soát cứng để đóng lại các Open Points từ phiên bản trước.

## 3.1. Ngưỡng Deal lớn và Xác thực ART

Hệ thống áp đặt cơ chế kiểm soát kép cho các giao dịch giá trị cao:

- Bắt buộc AI Review: Khi purchase_price ≥ 300,000 JPY, quy tắc FR-31 và NFR-21 yêu cầu bản ghi phải có kết quả đánh giá AI (ai_last_review_at) trước khi cho phép chốt quyết định cuối cùng.
- Chứng thực nguồn gốc (Provenance): Đối với chế độ ART, nếu purchase_price ≥ 300,000 JPY, quy tắc FR-32 và DR-02 yêu cầu chứng thực nguồn gốc phải hợp lệ. Nếu thiếu, hệ thống tự động chặn (Block) trạng thái BUY bất kể ROI cao.

## 3.2. Phạm vi Kiểm toán (Audit Scope)

Theo chế độ CRITICAL_ONLY (SR 9.3) để tối ưu hóa dữ liệu, quy tắc CRL-03 trong BR xác định 6 trường dữ liệu bắt buộc phải log khi có thay đổi:

1. purchase_price (Giá mua)
2. evaluation_mode (Chế độ đánh giá)
3. risk_score (Điểm rủi ro)
4. decision_final (Quyết định cuối cùng)
5. threshold change (Thay đổi ngưỡng cấu hình)
6. AI evaluate (Lịch sử đánh giá AI)

## 3.3. Chính sách Lưu trữ Ảnh

Tuân thủ NFR-24 và đặc tả tại SR 9.4, hệ thống thực thi quản trị hình ảnh như sau:

- Cấu trúc lưu trữ: Ảnh được tổ chức theo đường dẫn /Drive/Asset-Investment-Control/YYYY/asset_id/.
- Quản lý truy cập: Phân cấp quyền (Owner toàn quyền, Reviewer chỉ xem) qua FR-34.
- Thời gian lưu trữ: Ảnh có thời hạn lưu giữ là 1 năm. Quy tắc FR-35 thực thi việc archive hoặc xóa dữ liệu sau thời hạn này.

# 4. Cơ chế Quản trị và Kiểm soát Rủi ro (Governance & Risk)

Hệ thống thiết lập các tầng bảo vệ ngăn chặn việc phê duyệt sai lầm thông qua cơ chế ưu tiên nghiêm ngặt.

Thứ tự ưu tiên của các quy tắc (Rule Priority Order):

1. Financial Rules: Tính toán nền tảng.
2. Decision Rules: Đề xuất dựa trên ROI.
3. Risk Override (Ưu tiên cao nhất): Quy tắc RR-01 ghi đè mọi đề xuất thành SKIP nếu rủi ro vượt ngưỡng, không phụ thuộc vào ROI.
4. Data Governance: Kiểm tra tính toàn vẹn dữ liệu.
5. AI Confidence Gating: Chặn dựa trên độ tin cậy.
6. Manual Override Validation: Kiểm soát can thiệp con người.

Cơ chế "Gating" (Cổng chặn): Sự liên kết giữa dữ liệu và quyết định được thực hiện qua chuỗi:

- Thiếu dữ liệu (DGR-01): Ví dụ thiếu purchase_price hoặc resale_est trong mode RESALE.
- Xử lý thiếu dữ liệu (DGR-02): Kích hoạt trạng thái không được BUY.
- Confidence thấp (AR-03): Hệ thống tự động set ai_confidence ≤ 2.
- Chặn đề xuất (AR-02, FR-22): Mọi đề xuất BUY bị chặn, yêu cầu xác thực thủ công (Verification Required).

Kiểm soát ghi đè: Theo quy tắc CRL-01 và FR-28, khi người dùng thay đổi decision_final khác với đề xuất của hệ thống, trường decision_reason trở thành bắt buộc để phục vụ hậu kiểm.

# 5. Đánh giá Tính Nhất quán và Khả năng Truy vết (Traceability)

Hệ thống v1.1 đảm bảo khả năng đáp ứng các tiêu chí thành công cốt lõi từ CR: Tỷ lệ lỗ < 10% và ROI trung bình ≥ 25% thông qua việc thắt chặt các quy tắc tài chính và rủi ro.

## Bảng chứng minh tính truy vết:

| Thực thể (Asset/Decision) |	Quy tắc tương ứng |	Dữ liệu lưu trữ phục vụ truy vết |
|-------------------------|-------------------|----------------------------------|
| Chốt quyết định cuối cùng	| CRL-02, FR-25	|rule_version snapshot, timestamp, approver |
| Thay đổi thông số tài chính	| CRL-03, FR-26	|Giá trị cũ (old_val), giá trị mới (new_val), user_id |
| Thay đổi ngưỡng cấu hình	| FR-33, CFG-02	|Lịch sử thay đổi threshold tại bảng CONFIG |
| Đánh giá của AI	| FR-24, FR-29	|ai_summary, ai_confidence, AI_LOG timestamp |

**Kết luận:** Bộ tài liệu v1.1 thể hiện mức độ đồng bộ tuyệt đối. Mọi Open Points từ v1.0 đã được xử lý bằng các tham số kỹ thuật cụ thể (300,000 JPY, 1 năm). Hệ thống không chỉ đáp ứng mục tiêu nghiệp vụ mà còn thiết lập một khung quản trị rủi ro chặt chẽ, đảm bảo tính minh bạch và khả năng kiểm toán toàn diện.

# 6. Danh mục Quy tắc Tổng hợp (Reference Table)

| No. |	ID |	Category	| Tên   |	Mô tả |
|---- |----|----------------|-------|---------|
| 01 | FRL-01 | Financial Rule | Total Cost | Tính tổng chi phí đầu tư |
| 02 | FRL-02 | Financial Rule | Expected Profit | Tính lợi nhuận kỳ vọng |
| 03 | FRL-03 | Financial Rule | ROI | Tính Return on Investment |
| 04 | FRL-04 | Financial Rule | Scrap Floor | Tính ngưỡng giá trị tối thiểu khi bán phế liệu |
| 05 | DR-01 | Decision Rule | RESALE Logic | Quy tắc quyết định cho mode RESALE |
| 06 | DR-02 | Decision Rule | ART Logic | Quy tắc quyết định cho mode ART |
| 07 | DR-03 | Decision Rule | SCRAP Logic | Quy tắc quyết định cho mode SCRAP |
| 08 | DR-04 | Decision Rule | Mode Switching | Quy tắc khi chuyển đổi evaluation_mode |
| 09 | RR-01 | Risk Rule | Risk Override | Quy tắc override khi risk cao |
| 10 | DGR-01 | Data Governance Rule | Mandatory Fields by Mode | Quy tắc trường dữ liệu bắt buộc theo mode |
| 11 | DGR-02 | Data Governance Rule | Severe Missing Data Handling | Quy tắc xử lý khi thiếu dữ liệu nghiêm trọng |
| 12 | AR-01 | AI Governance Rule | AI Advisory Only | AI chỉ đóng vai trò tư vấn, không thay thế quyết định |
| 13 | AR-02 | AI Governance Rule | Confidence Gating | Quy tắc gating khi confidence thấp |
| 14 | AR-03 | AI Governance Rule | Confidence Linkage | Liên kết giữa thiếu dữ liệu nghiêm trọng và confidence thấp |
| 15 | AR-04 | AI Governance Rule | No Realtime Crawling | AI không crawl dữ liệu realtime |
| 16 | AR-05 | AI Governance Rule | AI Output Contract | Quy tắc về định dạng output của AI và chính sách retry |
| 17 | CRL-01 | Control & Audit Rule | Manual Override | Quy tắc khi decision_final khác decision_suggested |
| 18 | CRL-02 | Control & Audit Rule | Rule Version Snapshot | Quy tắc snapshot version khi chốt decision_final |
| 19 | CRL-03 | Control & Audit Rule | Audit Logging Mandatory | Quy tắc về các trường dữ liệu bắt buộc phải log |
| 20 | CRL-04 | Control & Audit Rule | Logging Integrity | Quy tắc đảm bảo logging không làm mất dữ liệu chính |
| 21 | CFG-01 | Configuration Rule | Threshold Externalized | Quy tắc về việc cấu hình threshold qua CONFIG |
| 22 | CFG-02 | Configuration Rule | Real-Time Effect | Quy tắc về hiệu lực của thay đổi threshold |
| 23 | CFG-03 | Configuration Rule | Retry Policy | Quy tắc về chính sách retry khi AI trả về JSON lỗi |    
