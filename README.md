# Phase 2: Apps Script tự động hóa  
Google Sheet bản thân nó không UX tốt cho “ứng dụng”, nhưng bạn có thể biến nó thành mini web app ngay trong Sheet bằng Apps Script.

Bạn sẽ có:

- Sidebar UI (Form nhập deal đẹp, không nhập trực tiếp vào cell)
- Nút “AI Evaluate”
- Popup kết quả
- Log AI riêng
- Không phải copy/paste prompt
- Không đụng vào cột công thức

Đây là hướng chuẩn nếu bạn muốn nâng lên hệ thống thật.

## 🔥 Cách làm đúng: Google Sheets + Apps Script UI

Có 3 cấp UX bạn có thể chọn:

## 🟢 Cấp 1 – Sidebar Form (Khuyến nghị)

Bạn tạo:

- Menu: Used Asset AI

- Sidebar form gồm:

    - Mode (dropdown)
    - Asking price
    - Kg
    - Scrap price
    - Chi phí
    - Button: “Run AI”

Apps Script sẽ:

1. Đọc dữ liệu dòng hiện tại
2. Build prompt
3. Gọi OpenAI API
4. Trả kết quả vào:
- Cột AI_OUTPUT
- Hoặc popup sidebar

👉 Đây là cách chuyên nghiệp nhất cho Google Sheet.

## 🟡 Cấp 2 – Nút bấm trong Sheet

Thêm button:

- “AI Evaluate”
- “Recalculate Risk”
- “Suggest Counter Offer”

Click là chạy script.

UX ổn, nhưng không đẹp bằng sidebar.

## 🔴 Cấp 3 – Custom Web App (Advanced)

Apps Script có thể publish thành Web App.

Sheet = Database
Web App = Giao diện

Đây là hướng đi nếu bạn muốn scale.

## 🎯 Gợi ý kiến trúc phù hợp với bạn

Vì bạn đang xây:

Data-Driven Used Asset Investment Engine

Tôi khuyên:

- Sheet = Data + Rule Engine
- Apps Script = AI Layer
- Sidebar = UX Layer

## 🧠 AI sẽ làm gì?

Trong SCRAP mode:

AI không thay rule.

AI sẽ:

- Kiểm tra thiếu dữ liệu
- Phát hiện risk ẩn (ví dụ: 3500kg có thể không phải toàn thép)
- Đề xuất counter-offer cụ thể
- Tạo confidence score
- Ghi chú negotiation strategy

## 🚀 Tôi đề xuất UX như sau

Menu trên cùng:
```
Used Asset AI
├── Evaluate Current Row
├── AI Scrap Review
├── AI Resale Review
├── Show AI Panel
```
Sidebar hiển thị:

🟦 Deal Summary
🟨 Calculated Scrap Floor
🟥 Risk Analysis
🟩 AI Recommendation
💰 Counter Offer Suggestion
📊 Confidence Score

## 🛠 NEXT STEPS

Tôi có thể:

1. Viết sẵn toàn bộ Apps Script code
2. Viết HTML sidebar UI
3. Viết hàm gọi OpenAI API
4. Tích hợp với cấu trúc sheet v1.0 của bạn

Bạn chỉ cần:

- Dán code vào Extensions → Apps Script
- Nhập API key
- Deploy

## ❓Câu hỏi quan trọng

muốn:

A) AI chạy cho 1 dòng đang chọn

B) AI chạy hàng loạt (batch evaluation)

C) AI chạy realtime khi sửa cell

Chọn 1 hướng -> thiết kế đúng kiến trúc.
```
Hệ thống chọn theo A) sẽ đơn giản nhất, dễ triển khai nhất.
```

# 🎯 Yêu cầu mới của khách hàng

Layer 2 – AI Reviewer cần:

Đọc hình ảnh gắn với dòng (ví dụ photo_link_1)

Phân tích:

Tên máy

Model

Serial

Thông số kỹ thuật trên info panel

Đối chiếu:

Yahoo Auction

Mercari

Thị trường nội địa Nhật

Tư vấn:

Ước tính kg (nếu là máy)

Giá thị trường tham khảo

Khuyến nghị thương lượng

## 🚨 Cần làm rõ một điểm quan trọng

AI trong Apps Script không thể tự động crawl Yahoo/Mercari nếu:

Không có API chính thức

Không có backend riêng

Không có scraping service

Google Apps Script cũng có giới hạn runtime.

Vì vậy phải thiết kế đúng kiến trúc.

🏗 Kiến trúc Layer 2 sau khi nâng cấp
Layer 2.1 – Vision AI

Đọc ảnh từ photo_link_1

OCR info panel

Trích xuất:

Maker

Model

Serial

Voltage

Year

Capacity

Layer 2.2 – Market Intelligence (Không crawl trực tiếp)

Thay vì crawl realtime, có 3 phương án:

Phương án A (khuyến nghị thực tế)

AI:

Đọc model

Đưa ra từ khóa tìm kiếm gợi ý

Gợi ý khoảng giá thị trường dựa trên kiến thức huấn luyện

Không cam kết giá chính xác

→ Không vi phạm TOS
→ Không cần backend

Phương án B (Bán chuyên nghiệp)

Bạn tự xây:

Google Sheet tab MARKET_DATA

Lưu giá Yahoo sold comps

AI chỉ đọc tab này và phân tích

→ Dữ liệu do bạn nhập
→ AI chỉ phân tích, không crawl

Phương án C (Enterprise)

Backend riêng

Scraper hợp pháp

Database

AI phân tích

→ Không thuộc scope hiện tại

## 🧠 AI Vision có thể làm được gì?

Nếu gửi ảnh cho AI:

Nhận diện bảng info

Đọc text (OCR)

Suy luận model

Ước tính trọng lượng dựa trên loại máy

Phát hiện:

Rỉ sét nặng

Thiếu bộ phận

Hư hỏng bề ngoài

## 📐 Cập nhật cần thêm vào SR

Bạn phải thêm:

FR-AI-19: Vision Input

Hệ thống phải cho phép:

Gửi tối đa 3 hình ảnh từ photo_link_1, photo_link_2

AI phân tích ảnh và trả:

Extracted model info

Visual condition assessment

Suspected missing parts

FR-AI-20: Market Advisory

AI phải:

Tạo từ khóa tìm kiếm chuẩn Yahoo/Mercari

Đưa ra range giá tham khảo (estimate only)

Đánh dấu confidence level

FR-AI-21: Scrap Weight Advisory

Nếu là máy:

AI ước tính trọng lượng dựa trên model

So sánh với estimated_weight_kg

Cảnh báo nếu chênh lệch lớn

⚠️ Giới hạn phải ghi rõ trong SR

AI không truy cập trực tiếp Yahoo/Mercari

AI không đảm bảo giá thị trường realtime

Người dùng phải xác minh bằng tay

## 🧩 Cập nhật kiến trúc

### Layer 2 – AI Reviewer sẽ gồm:

- Text Analysis

- Vision Analysis

- Market Advisory (Estimate)

- Negotiation Strategy Generator

💡 UX cần thay đổi

Sidebar nên có thêm:

- Preview ảnh

- Nút “Analyze Images”

- Nút “Market Insight”

Hiển thị:

- Extracted model

- Suggested search keyword

- Estimated market range

- Estimated weight

- Visual risk flags

## 🎯 Câu hỏi quan trọng

Bạn muốn:

A) AI chỉ phân tích ảnh và đưa ra estimate (không market tab riêng)
B) AI phân tích ảnh + đọc MARKET_DATA tab bạn tự nhập
C) Thiết kế kiến trúc chuẩn cho tương lai Web App luôn

```
Chọn A) sẽ đơn giản nhất, dễ triển khai nhất.
```
