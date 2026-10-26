# SRMU Campus Recover – FastAPI Backend

Simple backend: FastAPI + SQLite + JWT login. Python 3.9+ chahiye.

## Run

```bash
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt

# optional: ye emails admin (security desk) banengi jab register karengi
export ADMIN_EMAILS="security@srmu.ac.in"      # Windows: set ADMIN_EMAILS=...
export SECRET_KEY="koi-lamba-random-string"

uvicorn app.main:app --reload
```

Swagger docs (testing ke liye): http://127.0.0.1:8000/docs
Upar "Authorize" button se login karke saare endpoints try kar sakte ho.

## Endpoints

| Method | URL | Kaun | Kaam |
|---|---|---|---|
| POST | /auth/register | public | account banao (JSON) |
| POST | /auth/login | public | login (form: username=email, password) |
| GET | /auth/me | login | apni profile |
| POST | /items | login | lost/found report (multipart form + optional image) |
| GET | /items | public | feed; filters: type, category, status, location, q, skip, limit |
| GET | /items/mine | login | meri reports |
| GET | /items/{id} | public | detail (secret_identifier sirf owner/admin ko) |
| DELETE | /items/{id} | owner/admin | report delete |
| POST | /items/{id}/claim | login | "Claim Item" / "I Found This" (proof_answer bhejo) |
| GET | /claims/mine | login | mere claims ka status |
| GET | /admin/claims | admin | pending claims + secret identifier (match karne ke liye) |
| POST | /admin/claims/{id}/review | admin | approve/reject |
| GET | /stats | public | homepage counters |

## Flow (UI se mapping)

1. Student register/login karta hai.
2. "Report Lost/Found" form -> `POST /items` (title, category, location, event_time, secret_identifier, image).
3. Feed aur search bar -> `GET /items?type=found&q=calculator`.
4. Koi claim kare -> `POST /items/{id}/claim` me secret detail wala jawab.
5. Security desk `GET /admin/claims` me dekhta hai ki `proof_answer` aur `item_secret_identifier` match karte hain ya nahi, phir `review` se approve/reject.
6. Approve hone par item `claimed` ho jata hai (UI me "Resolved").

## Frontend se call karne ka example

```js
// login
const form = new URLSearchParams({ username: email, password });
const { access_token } = await (await fetch("http://127.0.0.1:8000/auth/login", { method: "POST", body: form })).json();

// feed
const items = await (await fetch("http://127.0.0.1:8000/items?type=found")).json();

// report
const fd = new FormData();
fd.append("type", "lost"); fd.append("title", "Casio fx-991CW"); fd.append("category", "gadgets");
fd.append("location", "Library"); fd.append("event_time", "2026-10-24T11:30");
fd.append("secret_identifier", "Spider-Man sticker"); fd.append("image", fileInput.files[0]);
await fetch("http://127.0.0.1:8000/items", { method: "POST", headers: { Authorization: `Bearer ${access_token}` }, body: fd });
```

Images `/uploads/<file>` par serve hoti hain (item ke `image_url` me poora path milta hai).
CORS me sirf localhost origins allowed hain; deploy karte waqt `app/main.py` me apna frontend URL add karna.
