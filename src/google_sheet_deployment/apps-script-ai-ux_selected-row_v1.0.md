# Apps Script + UX (Sidebar) – AI chạy cho 1 dòng đang chọn (Model A)
Version: 1.0  
Owner: Dang  
Scope: Google Sheets v3.0 (tab **ASSETS**) + Apps Script + Sidebar UI  
Goal: Chọn 1 dòng trong ASSETS → bấm nút → gọi OpenAI API → ghi kết quả AI vào các cột AI.

---

## 0) Tổng quan kiến trúc

- **Google Sheet (ASSETS)**: nơi lưu dữ liệu deal + công thức ROI/Risk/Decision (rule-based engine).
- **Apps Script**: lớp “AI Layer” (build prompt, gọi API, parse kết quả).
- **Sidebar UI (HTML)**: lớp “UX Layer” (không cần nhập trực tiếp vào cell, thao tác dễ).

Mô hình này giúp:
- AI chạy **on-demand** cho đúng 1 dòng đang chọn → tiết kiệm chi phí.
- Không phá công thức của sheet.
- Có audit trail thông qua các cột AI.

---

## 1) Chuẩn bị tab ASSETS (bắt buộc)

Trong tab **ASSETS**, thêm các cột sau ở hàng header (row 1). Có thể thêm ở cuối:

- `ai_status`
- `ai_output`
- `ai_confidence`
- `ai_counter_offer`
- `ai_updated_at`

> Script tìm theo **tên header**, không phụ thuộc chữ cột A/B/C.

---

## 2) Tạo Apps Script project

Google Sheets → **Extensions → Apps Script**

Tạo 2 file:

- `Code.gs`
- `Sidebar.html`

---

## 3) Code.gs (copy/paste nguyên)

```javascript
/***************
 * Used Asset AI - Google Sheets UX (Model A: selected row)
 * - Sidebar UI
 * - On-demand evaluation for the currently selected row
 * - Writes result to ASSETS columns: ai_status, ai_output, ai_confidence, ai_counter_offer, ai_updated_at
 ***************/

const OPENAI_ENDPOINT = "https://api.openai.com/v1/responses";
const DEFAULT_MODEL = "gpt-4o-mini"; // đổi model nếu cần

function onOpen() {
  SpreadsheetApp.getUi()
    .createMenu("Used Asset AI")
    .addItem("Open AI Panel", "showSidebar")
    .addSeparator()
    .addItem("Set OpenAI API Key", "setOpenAIKey")
    .addItem("Evaluate Selected Row", "evaluateSelectedRow")
    .addToUi();
}

function showSidebar() {
  const html = HtmlService.createHtmlOutputFromFile("Sidebar")
    .setTitle("Used Asset AI (v3.0)");
  SpreadsheetApp.getUi().showSidebar(html);
}

/**
 * Store API key in UserProperties (per-user). Do NOT store in cells.
 */
function setOpenAIKey() {
  const ui = SpreadsheetApp.getUi();
  const resp = ui.prompt(
    "Set OpenAI API Key",
    "Paste your OpenAI API key (will be stored in your User Properties).",
    ui.ButtonSet.OK_CANCEL
  );

  if (resp.getSelectedButton() !== ui.Button.OK) return;

  const key = (resp.getResponseText() || "").trim();
  if (!key) {
    ui.alert("API key is empty.");
    return;
  }
  PropertiesService.getUserProperties().setProperty("OPENAI_API_KEY", key);
  ui.alert("Saved. You can now run Evaluate Selected Row.");
}

function getOpenAIKey_() {
  return PropertiesService.getUserProperties().getProperty("OPENAI_API_KEY");
}

/**
 * Sidebar helper: returns the currently selected row info (ASSETS only).
 */
function getSelectedRowPreview() {
  const ss = SpreadsheetApp.getActive();
  const sheet = ss.getActiveSheet();
  if (sheet.getName() !== "ASSETS") {
    return { ok: false, message: 'Please select a row in sheet "ASSETS".' };
  }

  const row = sheet.getActiveRange().getRow();
  if (row < 2) return { ok: false, message: "Please select a data row (row >= 2)." };

  const map = getHeaderMap_(sheet);
  const values = getRowObject_(sheet, row, map);

  return {
    ok: true,
    sheet: sheet.getName(),
    row,
    asset_id: values.asset_id || "",
    evaluation_mode: values.evaluation_mode || "",
    category: values.category || "",
    asking_price: values.asking_price ?? "",
    purchase_price_target: values.purchase_price_target ?? "",
    net_scrap_floor: values.net_scrap_floor ?? "",
    roi_percent: values.roi_percent ?? "",
    risk_score_total: values.risk_score_total ?? "",
  };
}

/**
 * Main action: evaluate selected row and write back AI results.
 */
function evaluateSelectedRow() {
  const ss = SpreadsheetApp.getActive();
  const sheet = ss.getActiveSheet();

  if (sheet.getName() !== "ASSETS") {
    SpreadsheetApp.getUi().alert('Please run this on sheet "ASSETS".');
    return;
  }

  const apiKey = getOpenAIKey_();
  if (!apiKey) {
    SpreadsheetApp.getUi().alert('Missing API key. Run: Used Asset AI → "Set OpenAI API Key".');
    return;
  }

  const row = sheet.getActiveRange().getRow();
  if (row < 2) {
    SpreadsheetApp.getUi().alert("Please select a data row (row >= 2).");
    return;
  }

  const map = getHeaderMap_(sheet);
  const data = getRowObject_(sheet, row, map);

  // write status
  writeCellsByHeader_(sheet, row, map, {
    ai_status: "RUNNING",
    ai_updated_at: new Date()
  });

  const prompt = buildPrompt_(data);

  const aiText = callOpenAI_(apiKey, prompt);

  // Try parse structured sections (confidence / counter offer) from AI output
  const parsed = parseAIOutput_(aiText);

  writeCellsByHeader_(sheet, row, map, {
    ai_status: "DONE",
    ai_output: aiText,
    ai_confidence: parsed.confidence ?? "",
    ai_counter_offer: parsed.counterOffer ?? "",
    ai_updated_at: new Date()
  });

  SpreadsheetApp.getUi().toast("AI evaluation completed.", "Used Asset AI", 3);
}

/**
 * Build a mode-aware prompt from a row object.
 */
function buildPrompt_(d) {
  const mode = String(d.evaluation_mode || "").toUpperCase().trim();
  const category = String(d.category || "").toUpperCase().trim();

  // Core numbers (safe coercion)
  const asking = num_(d.asking_price);
  const neg = num_(d.negotiation_margin);
  const target = num_(d.purchase_price_target);
  const totalCost = num_(d.total_cost);
  const roi = d.roi_percent;
  const risk = num_(d.risk_score_total);

  // Scrap fields
  const weight = num_(d.estimated_weight_kg);
  const scrapKg = num_(d.scrap_price_per_kg);
  const scrapTotal = num_(d.scrap_total_value);
  const dismantle = num_(d.dismantle_cost);
  const scrapTrans = num_(d.scrap_transport_cost);
  const hazardous = num_(d.hazardous_material_cost);
  const scrapFloor = num_(d.net_scrap_floor);

  // Resale fields
  const domestic = num_(d.domestic_resale_est);
  const exportEst = num_(d.export_resale_est);

  // Art fields
  const auction = num_(d.auction_comparison_price);

  const assetId = d.asset_id || "";
  const notes = (d.notes_raw || "").toString().slice(0, 1200);

  // Decision context (from sheet)
  const decisionSuggested = d.decision_suggested || "";

  const base = `
Bạn là chuyên gia mua bán tài sản cũ tại Nhật.
Mục tiêu: tư vấn quyết định mua theo nguyên tắc "bảo vệ vốn", ưu tiên dữ liệu và rủi ro thực tế.
Hãy trả lời NGẮN GỌN, có số cụ thể, theo format:
[Data Check] ...
[Calculation Check] ...
[Decision] BUY/NEGOTIATE/SKIP + lý do
[Counter Offer] giá đề xuất (JPY) + chiến lược nói chuyện
[Confidence] 1-5 + giải thích 1 câu
[Risks] top 3 rủi ro cần xác minh

Thông tin deal:
asset_id: ${assetId}
mode: ${mode}
category: ${category}

Giá:
asking_price (U): ${asking}
negotiation_margin (V): ${neg}
purchase_price_target (W): ${target}
total_cost: ${totalCost}
roi_percent: ${roi}
risk_score_total: ${risk}
decision_suggested (sheet): ${decisionSuggested}

SCRAP:
weight_kg (AK): ${weight}
scrap_price_per_kg (AL): ${scrapKg}  (là giá dealer THU MUA trả cho mình)
scrap_total_value (AM): ${scrapTotal}
dismantle_cost (AN): ${dismantle}
scrap_transport_cost (AO): ${scrapTrans}
hazardous_cost (AF): ${hazardous}
net_scrap_floor (AP): ${scrapFloor}

RESALE:
domestic_resale_est (AH): ${domestic}
export_resale_est (AI): ${exportEst}

ART:
auction_comparison_price (AJ): ${auction}

Notes:
${notes}
`.trim();

  // Mode-specific hints
  const modeHint = (() => {
    if (mode === "SCRAP") {
      return `
SCRAP mode rules:
- Nếu purchase_price_target > net_scrap_floor → SKIP
- Nếu purchase_price_target <= 0.8 * net_scrap_floor → BUY
- Còn lại → NEGOTIATE
Hãy kiểm tra lại net_scrap_floor có hợp lý không (tạp chất, phân loại kim loại, cân thiếu, dầu/chất thải).
`.trim();
    }
    if (mode === "ART") {
      return `
ART mode:
- Ưu tiên xác thực: provenance, signature, auction comps.
- Nếu thiếu dữ liệu xác thực mà giá trị cao → đề xuất SKIP hoặc cần chuyên gia.
`.trim();
    }
    return `
RESALE mode:
- Ưu tiên evidence giá bán (sold comps), tốc độ thanh khoản, chi phí phát sinh (shipping/repair).
- Nếu ROI đẹp nhưng risk cao → vẫn nên NEGOTIATE/SKIP.
`.trim();
  })();

  return `${base}\n\n${modeHint}`;
}

/**
 * Call OpenAI Responses API.
 */
function callOpenAI_(apiKey, promptText) {
  const payload = {
    model: DEFAULT_MODEL,
    input: [
      {
        role: "user",
        content: [
          { type: "input_text", text: promptText }
        ]
      }
    ],
    max_output_tokens: 450
  };

  const options = {
    method: "post",
    contentType: "application/json",
    payload: JSON.stringify(payload),
    headers: { Authorization: "Bearer " + apiKey },
    muteHttpExceptions: true
  };

  const res = UrlFetchApp.fetch(OPENAI_ENDPOINT, options);
  const code = res.getResponseCode();
  const body = res.getContentText();

  if (code < 200 || code >= 300) {
    throw new Error(`OpenAI API error ${code}: ${body}`);
  }

  const json = JSON.parse(body);
  const text = extractResponseText_(json);
  return (text || "").trim();
}

/**
 * Extract plain text from Responses API response object.
 */
function extractResponseText_(json) {
  if (!json) return "";
  const out = json.output || [];
  let parts = [];

  for (const item of out) {
    const content = item.content || [];
    for (const c of content) {
      if (c.type === "output_text" && c.text) parts.push(c.text);
      if ((c.type === "text" || c.type === "output_text") && typeof c.text === "string") {
        // already handled above
      }
    }
  }

  if (!parts.length && typeof json.output_text === "string") return json.output_text;

  return parts.join("\n").trim();
}

/**
 * Parse [Confidence] and [Counter Offer] sections from AI text (best-effort).
 */
function parseAIOutput_(txt) {
  const res = { confidence: "", counterOffer: "" };
  if (!txt) return res;

  const confMatch = txt.match(/\[Confidence\]\s*([^\n]+)/i);
  if (confMatch) res.confidence = confMatch[1].trim();

  const coMatch = txt.match(/\[Counter Offer\]\s*([^\n]+)/i);
  if (coMatch) res.counterOffer = coMatch[1].trim();

  return res;
}

/**
 * Helpers: header mapping and row object
 */
function getHeaderMap_(sheet) {
  const lastCol = sheet.getLastColumn();
  const headers = sheet.getRange(1, 1, 1, lastCol).getValues()[0];
  const map = {};
  headers.forEach((h, i) => {
    const key = String(h || "").trim();
    if (key) map[key] = i + 1;
  });
  return map;
}

function getRowObject_(sheet, row, map) {
  const lastCol = sheet.getLastColumn();
  const headers = sheet.getRange(1, 1, 1, lastCol).getValues()[0];
  const values = sheet.getRange(row, 1, 1, lastCol).getValues()[0];

  const obj = {};
  for (let i = 0; i < headers.length; i++) {
    const k = String(headers[i] || "").trim();
    if (!k) continue;
    obj[k] = values[i];
  }
  return obj;
}

function writeCellsByHeader_(sheet, row, map, kv) {
  for (const [k, v] of Object.entries(kv)) {
    if (!map[k]) continue;
    sheet.getRange(row, map[k]).setValue(v);
  }
}

function num_(v) {
  const n = Number(v);
  return isFinite(n) ? n : 0;
}
```

---

## 4) Sidebar.html (UX trong Google Sheet)

```html
<!DOCTYPE html>
<html>
  <head>
    <base target="_top">
    <style>
      body { font-family: system-ui, Arial; padding: 12px; }
      .box { border: 1px solid #ddd; border-radius: 10px; padding: 10px; margin-bottom: 10px; }
      .row { display: flex; justify-content: space-between; gap: 10px; margin: 4px 0; }
      .k { color: #666; }
      .v { font-weight: 600; text-align: right; }
      button { width: 100%; padding: 10px; border: 0; border-radius: 10px; cursor: pointer; font-weight: 700; }
      .btn { background: #111; color: #fff; }
      .btn2 { background: #f3f3f3; }
      .hint { color:#666; font-size: 12px; line-height: 1.4; }
      .ok { color: #0a7a0a; font-weight: 700; }
      .bad { color: #b00020; font-weight: 700; }
    </style>
  </head>
  <body>
    <div class="box">
      <div style="font-size:16px; font-weight:800;">Used Asset AI Panel</div>
      <div class="hint">Chọn 1 dòng trong tab <b>ASSETS</b> (row ≥ 2), rồi bấm Evaluate.</div>
    </div>

    <div id="status" class="hint"></div>

    <div class="box" id="previewBox" style="display:none;">
      <div class="row"><div class="k">Row</div><div class="v" id="p_row"></div></div>
      <div class="row"><div class="k">asset_id</div><div class="v" id="p_id"></div></div>
      <div class="row"><div class="k">mode</div><div class="v" id="p_mode"></div></div>
      <div class="row"><div class="k">category</div><div class="v" id="p_cat"></div></div>
      <div class="row"><div class="k">asking_price (U)</div><div class="v" id="p_ask"></div></div>
      <div class="row"><div class="k">purchase_target (W)</div><div class="v" id="p_w"></div></div>
      <div class="row"><div class="k">net_scrap_floor (AP)</div><div class="v" id="p_ap"></div></div>
      <div class="row"><div class="k">roi%</div><div class="v" id="p_roi"></div></div>
      <div class="row"><div class="k">risk</div><div class="v" id="p_risk"></div></div>
    </div>

    <button class="btn2" onclick="refresh()">Refresh Selected Row</button>
    <div style="height:8px;"></div>
    <button class="btn" onclick="evaluate()">AI Evaluate Selected Row</button>

    <div style="height:10px;"></div>
    <div class="hint">
      Tip: Nếu bạn thấy W = 0 thì bạn chưa nhập <b>asking_price (U)</b> hoặc công thức W bị ghi đè.
    </div>

    <script>
      function setStatus(msg, ok=true){
        const el = document.getElementById('status');
        el.innerHTML = ok ? `<span class="ok">${msg}</span>` : `<span class="bad">${msg}</span>`;
      }

      function refresh(){
        setStatus("Loading...", true);
        google.script.run.withSuccessHandler((res)=>{
          if(!res.ok){
            document.getElementById('previewBox').style.display = "none";
            setStatus(res.message, false);
            return;
          }
          document.getElementById('previewBox').style.display = "block";
          document.getElementById('p_row').textContent = res.row;
          document.getElementById('p_id').textContent = res.asset_id;
          document.getElementById('p_mode').textContent = res.evaluation_mode;
          document.getElementById('p_cat').textContent = res.category;
          document.getElementById('p_ask').textContent = res.asking_price;
          document.getElementById('p_w').textContent = res.purchase_price_target;
          document.getElementById('p_ap').textContent = res.net_scrap_floor;
          document.getElementById('p_roi').textContent = res.roi_percent;
          document.getElementById('p_risk').textContent = res.risk_score_total;
          setStatus("Ready.", true);
        }).getSelectedRowPreview();
      }

      function evaluate(){
        setStatus("Running AI... (check ai_status in sheet)", true);
        google.script.run.withSuccessHandler(()=>{
          setStatus("Done. Result written to ai_output / ai_counter_offer.", true);
          refresh();
        }).withFailureHandler((e)=>{
          setStatus("Error: " + (e && e.message ? e.message : e), false);
        }).evaluateSelectedRow();
      }

      refresh();
    </script>
  </body>
</html>
```

---

## 5) Cách dùng (UX flow)

1. **Used Asset AI → Set OpenAI API Key** (1 lần, lưu trong User Properties – không lưu trong sheet)
2. Vào tab **ASSETS** → chọn 1 dòng deal (row ≥ 2)
3. **Used Asset AI → Open AI Panel**
4. Bấm **AI Evaluate Selected Row**
5. Kết quả sẽ được ghi vào các cột:
   - `ai_status` = RUNNING/DONE
   - `ai_output` = nội dung phân tích AI
   - `ai_counter_offer` = giá đề xuất thương lượng (nếu parse được)
   - `ai_confidence` = mức tự tin (nếu parse được)
   - `ai_updated_at` = thời điểm chạy

---

## 6) Troubleshooting (lỗi thường gặp)

### 6.1 Không thấy menu “Used Asset AI”
- Refresh trang Google Sheets (F5)
- Đảm bảo project Apps Script đang gắn với đúng file sheet

### 6.2 Báo thiếu API key
- Chạy menu **Set OpenAI API Key** và nhập key

### 6.3 W (purchase_price_target) = 0
- Bạn chưa nhập `asking_price (U)` hoặc công thức `purchase_price_target` bị ghi đè
- Nhập U và V đúng (V=0.10 cho 10%)

### 6.4 Không ghi được kết quả vào cột AI
- Kiểm tra đã tạo header đúng tên:
  - ai_status, ai_output, ai_confidence, ai_counter_offer, ai_updated_at

### 6.5 API error (HTTP 401/429/500)
- 401: sai API key
- 429: vượt quota/rate limit → giảm tần suất chạy
- 500: lỗi phía API → thử lại sau

---

## 7) Khuyến nghị bảo vệ hệ thống (Governance)

- Protect range các cột AUTO trong ASSETS (ROI, total_cost, decision_suggested…)
- Chỉ cho edit các cột INPUT và AI columns
- Nếu nhiều user cùng thao tác, ưu tiên:
  - Evaluator: nhập dữ liệu
  - Owner/Reviewer: chạy AI + chốt decision_final

---

END OF DOCUMENT
