
# LUỒNG QUYẾT ĐỊNH ĐẦU TƯ TÀI SẢN THÔNG MINH (DECISION FLOW)

## 1️⃣ Đầu vào & Quản trị dữ liệu (DGR – SR)

### Bước 1: Nhập liệu tài sản chuẩn hóa

- ID tài sản
- Phân loại (IT / Công nghiệp / ART…)
- Giá mua
- Chọn chế độ đánh giá:
    - RESALE
    - SCRAP
    - ART

### Bước 2: Kiểm tra độ đầy đủ dữ liệu

Nếu thiếu dữ liệu nghiêm trọng:

- Chặn BUY
- AI Confidence ≤ 2

👉 Đây là lớp kiểm soát đầu tiên trước khi tính toán.

## 2️⃣ Động cơ tài chính & Quy tắc (FRL & DR – BR)
### Tính toán tài chính tự động

- FRL-01: Tổng chi phí
- FRL-02: Lợi nhuận kỳ vọng
- FRL-03: ROI (tỉ xuất lợi nhuận)
- FRL-04: Net Scrap Floor (giá sàn phế liệu sau khi trừ chi phí)

### Logic đề xuất quyết định (Decision Engine)

Đề xuất BUY nếu:

- ROI ≥ 25% (RESALE)
- ROI ≥ 40% (ART)
- Giá mua ≤ 80% Net Scrap Floor (SCRAP)

Ngược lại → NEGOTIATE hoặc SKIP

## 3️⃣ Cổng kiểm soát rủi ro & AI (RR & AR)
### Cổng đỏ – Risk Override

- Risk ≥ 80 → Tự động SKIP
- Override toàn bộ logic tài chính

### Cổng vàng – Deal lớn

- Deal ≥ 300,000 JPY → Bắt buộc AI Review
- ART ≥ 300,000 JPY → Bắt buộc Provenance

## 4️⃣ Đánh giá AI Reviewer

AI trả về:

- Phân tích

- Cảnh báo

- Confidence Score (1–5)

### Confidence Gating ( )

- Confidence ≤ 2 → Không khuyến nghị BUY
- Chặn BUY (gating)

## 5️⃣ Chốt quyết định & Kiểm toán (CRL)
### Quyết định cuối cùng

- Thuộc về con người

- Nếu override → bắt buộc ghi lý do

### Snapshot phiên bản rule

- Ghi lại rule_version tại thời điểm chốt

### Nhật ký kiểm toán

Log các trường quan trọng:

- Giá mua
- Chế độ đánh giá
- Điểm rủi ro
- Người phê duyệt

## 🎯 Tóm tắt các ngưỡng kiểm soát v1.1

| Kiểm soát          | Ngưỡng                           | Hành động |
| ------------------ | -------------------------------- | --------- |
| Risk ≥ 80          | → SKIP                           |           |
| Deal ≥ 300,000 JPY | → Bắt buộc AI Review             |           |
| ROI ≥ 25% (Resale) | → BUY                            |           |
| ROI ≥ 40% (Art)    | → BUY (cần Provenance nếu ≥300k) |           |
| AI Confidence ≤ 2  | → Không cho BUY                  |           |


## 🔥 Bản chất của luồng này

Luồng được thiết kế theo nguyên tắc:

- Data →

- Financial logic →

- Risk override →

- AI validation →

- Human final decision →

- Audit snapshot

Nó kết nối chặt giữa:

- CR (Mục tiêu bảo vệ vốn)
- SR (Yêu cầu hệ thống)
- BR (Quy tắc nghiệp vụ)

## Sơ đồ
```mermaid
flowchart TD

  A[Start] --> B[Nhap lieu tai san chuan hoa DGR-01]

  B --> C{Chon Evaluation Mode}
  C --> C1[RESALE]
  C --> C2[SCRAP]
  C --> C3[ART]

  B --> D{Dữ liệu đầy đủ DGR-02}
  D -- No --> D1[Chan BUY va AI Confidence <= 2]
  D -- Yes --> E

  E[Rule Engine tính toán FRL] --> E1[Total Cost]
  E1 --> E2[Expected Profit]
  E2 --> E3[ROI]
  E3 --> E4[Net Scrap Floor]

  E4 --> F[Decision Engine de xuat DR]

  F --> F1{Đạt đk BUY}
  F1 -- ROI >= 25% RESALE --> G[Suggest BUY]
  F1 -- ROI >= 40% ART --> G
  F1 -- Gia mua <= 80% Net Scrap Floor SCRAP --> G
  F1 -- Không đạt --> H[Suggest NEGOTIATE hoặc SKIP]

  G --> R{Risk Score >= 80}
  H --> R

  R -- Yes --> R1[Override SKIP : Cổng Đỏ]
  R -- No --> L

  L{Deal >= 300000 JPY} -- Yes --> L1[phải AI Review : Cổng Vàng]
  L -- No --> M

  L1 --> P{Mode ART va Deal >= 300000 JPY}
  P -- Yes --> P1[Provenance Required]
  P -- No --> M

  M[AI Reviewer Evaluate AR] --> M1[Phân tích data, image]
  M1 --> M2[AI Confidence Score 1-5]

  M2 --> N{Confidence <= 2}
  N -- Yes --> N1[Không khuyến nghị BUY Gating]
  N -- No --> O[AI Result OK]

  R1 --> Z[Owner chốt decision_final CRL-01]
  N1 --> Z
  O --> Z

  Z --> Z1{Manual Override}
  Z1 -- Yes --> Z2[Nhap decision_reason bắt buộc]
  Z1 -- No --> Z3[không override]

  Z2 --> AA[Snapshot rule_version CRL-02]
  Z3 --> AA

  AA --> AB[Audit log critical fields CRL-03]
  AB --> AC[Image policy Drive folder permission retention 1 nam]

  AC --> END[End]

```