```bash
python -m venv venv
source venv/Scripts/activate
pip install fastapi uvicorn python-multipart
```

- main.py
```py
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def home():
    return {"message": "Invoice AI Running 🚀"}
```

```bash
uvicorn main:app --reload
```
- open chrome => http://127.0.0.1:8000

- main.py
```py
from fastapi import FastAPI, UploadFile, File
import shutil
import os

app = FastAPI()

UPLOAD_DIR = "uploads"
os.makedirs(UPLOAD_DIR, exist_ok=True)

@app.post("/upload")
async def upload_invoice(file: UploadFile = File(...)):
    file_path = f"{UPLOAD_DIR}/{file.filename}"

    with open(file_path, "wb") as buffer:
        shutil.copyfileobj(file.file, buffer)

    return {
        "filename": file.filename,
        "path": file_path
    }
```

- Test in Swagger
- open chrome
http://127.0.0.1:8000/docs


```bash
python -m pip install pytesseract pillow
```
