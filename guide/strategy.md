## 🚀 1. SYSTEM ARCHITECTURE (Big Picture)
```text
React Frontend
     ↓
FastAPI Backend
     ↓
OCR Layer (Tesseract / API)
     ↓
LLM Parsing Layer (OpenAI / Gemini)
     ↓
Validation Layer
     ↓
Supabase (DB + Storage)
     ↓
Analytics Engine
```

## 🧠 2. CORE FLOW (End-to-End)
1. User uploads invoice (PDF/Image)
2. File stored in Supabase Storage
3. OCR extracts raw text
4. LLM converts text → structured JSON
5. Validation layer checks correctness
6. Store structured data in Supabase DB
7. Analytics dashboard reads from DB
## 🗂️ 3. DATABASE DESIGN (Supabase)
Tables:
#### 👤 users
- id (uuid)
- email
- created_at
#### 📁 invoices
- id
- user_id
- file_url
- upload_date
- format_hash (for reuse detection)
- vendor_name
- invoice_date
- total_amount
- currency
- raw_text
#### 📊 invoice_items
- id
- invoice_id
- description
- quantity
- price
#### 🧠 parsed_data
- id
- invoice_id
- json_data
- confidence_score
## ⚡ 4. BACKEND (FastAPI)
#### 📦 Folder Structure
```text
backend/
 ├── main.py
 ├── routes/
 ├── services/
 │    ├── ocr.py
 │    ├── llm_parser.py
 │    ├── validator.py
 │    ├── format_detector.py
 ├── db/
 ├── models/
 └── utils/
 ```
#### 🧾 File Upload API
```py
@app.post("/upload")
async def upload_invoice(file: UploadFile):
    file_path = save_temp(file)

    # Upload to Supabase Storage
    file_url = upload_to_supabase(file_path)

    raw_text = extract_text(file_path)

    structured = parse_invoice(raw_text)

    validated = validate(structured)

    save_to_db(file_url, raw_text, validated)

    return validated
```
#### 🔍 5. OCR LAYER

Option 1: Tesseract (Free)
```py
import pytesseract
from PIL import Image

def extract_text(file_path):
    return pytesseract.image_to_string(Image.open(file_path))
```
Option 2 (Better): API
- Google Vision
- AWS Textract

👉 Use API if accuracy matters.

#### 🤖 6. LLM PARSING (MOST IMPORTANT PART)
🔥 Prompt (VERY IMPORTANT)
```text
You are an AI that extracts structured data from invoices.

Extract the following fields:
- vendor_name
- invoice_number
- invoice_date
- total_amount
- currency
- line_items (array with description, quantity, price)

Rules:
- Return ONLY valid JSON
- If missing, use null
- Do NOT hallucinate

Invoice Text:
{{OCR_TEXT}}
```
#### 🧠 Function
```py
def parse_invoice(text):
    response = openai.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": prompt.format(OCR_TEXT=text)}],
        temperature=0
    )
    return json.loads(response.choices[0].message.content)
```

#### ✅ 7. VALIDATION LAYER
```py
def validate(data):
    if not isinstance(data.get("total_amount"), (int, float)):
        data["total_amount"] = None

    if not data.get("vendor_name"):
        data["vendor_name"] = "Unknown"

    return data
```
👉 BONUS: Use Pydantic schema

#### 🔁 8. FORMAT DETECTION (SMART FEATURE)
Idea:
- Hash invoice structure
- Compare with previous formats
```py
import hashlib

def get_format_hash(text):
    return hashlib.md5(text[:500].encode()).hexdigest()
```

Store in DB → reuse parsing template.

#### 📊 9. ANALYTICS QUERIES
💰 Total Spend by Vendor
```sql
SELECT vendor_name, SUM(total_amount)
FROM invoices
GROUP BY vendor_name;
```
📅 Monthly Spend
```sql
SELECT DATE_TRUNC('month', invoice_date), SUM(total_amount)
FROM invoices
GROUP BY 1;
```
🌍 Currency-wise
```sql
SELECT currency, SUM(total_amount)
FROM invoices
GROUP BY currency;
```
#### 🎨 10. FRONTEND (React)
Pages:
- Upload Page
- Invoice List
- Analytics Dashboard
Upload Example:
```js
<input type="file" onChange={handleUpload} />
```
📈 Dashboard Features
- Vendor spend chart
- Monthly trends
- Invoice count

👉 Use:

- Chart.js
- Recharts

#### 🧠 11. BONUS FEATURES (HIGH IMPACT)
✅ Confidence Score
```py
confidence = len(valid_fields) / total_fields
```
🔁 Retry Logic
- If parsing fails → retry with stricter prompt

🔍 Duplicate Detection
```sql
SELECT * FROM invoices
WHERE total_amount = X AND vendor_name = Y;
```
🧾 Vendor Normalization
- "Amazon Pvt Ltd" → "Amazon"

#### 🚀 12. DEPLOYMENT
Backend:
- Render / Railway

Frontend:
- Vercel

DB:
- Supabase

#### ⏱️ 13. 48-HOUR STRATEGY (IMPORTANT)
#### 🕐 Day 1
- Backend setup
- OCR working
- LLM parsing working
- Supabase integration
#### 🕐 Day 2
- Frontend UI
- Analytics
- Format detection
- Deployment
- README + video

#### 📄 14. README STRUCTURE

Include:

- Architecture diagram
- Tech choices (WHY FastAPI? WHY Supabase?)
- Prompt design explanation
- Edge cases handled
- Future improvements

#### 🧠 15. WHAT WILL IMPRESS THEM

👉 Focus on these:

✔ Clean JSON output
✔ Strong prompt (no hallucination)
✔ Retry + fallback
✔ Format reuse system
✔ Analytics dashboard
✔ Fast response time
✔ Clean UI

💡 FINAL ADVICE

This is NOT just coding.

👉 This is:

AI product thinking
System design
Real-world robustness