# Test Case: Google Sheet Deployment

## SCRAP Mode Test Case
- Google Drive → New → File upload → chọn file .xlsx ở trên

- Mở bằng Google Sheets

- tab CONFIG 
  - nhập giá sắt/kg ở H2


- tab ASSETS

  - C phương thức định giá (evaluation_mode) = SCRAP

  - U (asking_price): nhập giá người bán đang yêu cầu (ví dụ 150000)

  - V (negotiation_margin): ví dụ 0.10 (kỳ vọng deal xuống 10%)
  → Khi đó W sẽ tự ra = 135,000

  - Nhập SCRAP inputs:
    - AF (hazardous_material_cost/phí xử lý dầu): 5000
    - AK tổng trọng lượng (estimated_weight_kg): 3500
    - AL giá mua phế liệu (scrap_price_per_kg): 65
    - AN phí tháo dỡ (dismantling_cost): 30000
    - AO phí vận chuyển (transportation_cost): 25000


- Lưu ý quan trọng về cột

  - Bạn đang nhầm tên cột:

    - AM giá trị phế liệu thô = scrap_total_value = 3500 × 65 = 227,500

    - AP giá sàn ròng = net_scrap_floor = 227,500 - 30,000 - 25,000 - 5,000 = 167,500

  - C phương thức định giá (evaluation_mode) để phân biệt giữa 3 mode: RESALE / SCRAP / ART. Từ đó sẽ có các rule và input khác nhau.
  - U (asking_price) là giá người bán đang yêu cầu, dùng để so sánh với giá trị phế liệu thô (AM) và giá sàn ròng (AP) để đưa ra gợi ý quyết định.

## NEXT STEPS
Nếu bạn muốn, mình có thể nâng bản sheet lên v3.1 theo kiểu “thực chiến” hơn: checklist tự chấm điểm risk_points (không cần chấm tay) và dashboard pivot mẫu sẵn.

## Test Case SCRAP hiện tại chưa có AI 
vì Google Sheet v1.0 bạn đang dùng là rule-based engine (công thức + checklist + decision rules). Nó giống “máy tính quyết định”, không phải AI.

AI chỉ xuất hiện khi bạn kết nối 1 trong 3 cách dưới đây.

### 1. Vì sao chưa thấy AI?

Vì v1.0 đang làm đúng mục tiêu “chạy thực tế ngay”:

Tính ROI / Scrap floor / Risk / Decision bằng công thức

Bảo vệ vốn (default SKIP nếu thiếu dữ liệu)

Không cần API, không tốn tiền gọi model

AI không tự bật lên nếu bạn chưa thiết kế “điểm gọi AI”.

### 2. 3 cách đưa AI vào Sheet (chọn 1)
#### Cách A — “AI thủ công” (nhanh nhất, không code)

Thêm 2 cột trong ASSETS:

- AI_PROMPT

- AI_OUTPUT

AI_PROMPT: công thức ghép prompt từ dữ liệu deal (mode, giá, kg, chi phí…).

Bạn copy prompt đó qua ChatGPT → dán kết quả vào AI_OUTPUT.

✅ Ưu: 5 phút là chạy, không cần lập trình

❌ Nhược: vẫn phải copy/paste

#### Cách B — Apps Script gọi API (AI tự chạy)

Bạn tạo hàm Google Sheets kiểu:

- =AI_EVAL(asset_id) hoặc =AI_SCRAP_REVIEW(...)

Apps Script sẽ:

- đọc dòng ASSETS

- build prompt theo mode

- gọi API (OpenAI hoặc model bạn chọn)

- trả output về ô

✅ Ưu: AI “hiện diện” đúng nghĩa, tự động

❌ Nhược: cần API key + quản lý chi phí + bảo mật

#### Cách C — Hybrid (khuyến nghị vận hành thật)

- Sheet vẫn là “decision engine” (AW)

- AI chỉ dùng để:

  - tạo checklist gợi ý

  - phát hiện thiếu dữ liệu

  - tăng “confidence score”

  - đề xuất giá thương lượng (counter-offer)

✅ Ưu: AI hỗ trợ tốt nhưng không phá cơ chế bảo vệ vốn

✅ Rất hợp mô hình của bạn

### 3. Nếu bạn muốn “AI xuất hiện ngay” trong SCRAP Test Case

Làm theo Cách A (không code) — mình đưa prompt chuẩn luôn:

Prompt AI cho SCRAP (copy nguyên)

Dán vào ChatGPT kèm dữ liệu của deal:

- Mode: SCRAP
- Asking price (U): ___
- Negotiation margin (V): ___
- Purchase target (W): ___
- Weight (AK): 3500 kg
- Scrap price/kg (AL): 65
- Dismantle (AN): 30000
- Scrap transport (AO): 25000
- Hazardous (AF): 5000
- Net scrap floor (AP): 167500

Prompt:
```
Bạn là chuyên gia mua bán tài sản cũ tại Nhật. Hãy đánh giá deal theo SCRAP mode (bảo vệ vốn).

1. Kiểm tra dữ liệu có thiếu không, rủi ro nằm ở đâu (weight, phân loại kim loại, tạp chất, dầu).

2. Tính lại scrap_total_value và net_scrap_floor.

3. Đưa ra khuyến nghị BUY/NEGOTIATE/SKIP dựa trên rule:

- W > AP => SKIP

- W <= 0.8*AP => BUY

- else => NEGOTIATE

4. Đề xuất “counter-offer price” cụ thể (mức giá nên chốt) và lý do.
Trả lời ngắn gọn theo format: [Data Check] [Calculation] [Decision] [Counter Offer] [Risks].
```

Bạn sẽ thấy AI “hiện diện” ngay qua output tư vấn + counter-offer.

### 4. Muốn AI kiểu nào?

- v1.0A (No-code AI Prompt Columns): thêm cột AI_PROMPT/Ai_OUTPUT + prompt auto theo mode
hoặc

- v1.1 (Apps Script AI Function): bấm là AI trả lời trong cell