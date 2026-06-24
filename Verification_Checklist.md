# GoFastrr Backend Startup & Verification Checklist

## 1. Start PostgreSQL

Check PostgreSQL status:

```bash
pg_isready
```

Expected output:

```text
accepting connections
```

If PostgreSQL is not running:

```bash
brew services start postgresql@15
```

> Replace `15` with the version you installed if different.

---

## 2. Verify Database Exists

```bash
psql -l
```

You should see:

```text
gofastrr_dev
```

---

## 3. Open GoFastrr Project

```bash
cd /path/to/GoFastrrBE
```

---

## 4. Check `.env`

Make sure the following variables exist:

```env
DATABASE_URL=postgresql://zod@localhost:5432/gofastrr_dev

APP_URL=https://scolding-thirteen-penholder.ngrok-free.dev

SHOPIFY_CLIENT_ID=...
SHOPIFY_CLIENT_SECRET=...

SMSALA_API_TOKEN=...
SMSALA_OTP_TEMPLATE_ID=...
```

---

## 5. Start ngrok

Open a new terminal:

```bash
ngrok http 4000
```

Copy the generated HTTPS URL.

If ngrok gives a new URL, update:

```env
APP_URL=https://new-ngrok-url.ngrok-free.app
```

Then restart the backend.

---

## 6. Start Backend

Open a new terminal:

```bash
npm run dev
```

Expected output:

```text
🚀 Server running on port 4000
```

---

## 7. Quick Health Check

```bash
curl http://localhost:4000/health
```

Expected response:

```json
{"status":"ok"}
```

---

## 8. Verify Tables Still Exist

Connect to the database:

```bash
psql gofastrr_dev
```

Check the following tables:

```sql
\d checkout_sessions
\d checkout_line_items
\d otp_codes
\d shopify_tokens
```

Exit PostgreSQL:

```sql
\q
```

---

## 9. Create a Fresh Checkout Session

Use the Shopify product:

```bash
curl -X POST http://localhost:4000/checkout/start \
  -H "Content-Type: application/json" \
  -d '{
    "shop": "gofastrr-dev.myshopify.com",
    "lineItems": [
      {
        "variantId": 51283967967452,
        "quantity": 1
      }
    ]
  }'
```

Copy the returned session ID:

```json
{
  "checkoutSessionId": "..."
}
```

---

## 10. Test OTP Send

Replace `NEW_SESSION_ID` with the value returned in Step 9:

```bash
curl -X POST http://localhost:4000/auth/otp/send \
  -H "Content-Type: application/json" \
  -d '{
    "checkoutSessionId":"NEW_SESSION_ID",
    "phone":"7205281825"
  }'
```

Expected response:

```json
{
  "success": true
}
```


---

## 11. Verify OTP

Use the OTP received on WhatsApp:

```bash
curl -X POST http://localhost:4000/auth/otp/verify \
  -H "Content-Type: application/json" \
  -d '{
    "checkoutSessionId":"ffe1c7d0-4f72-48d9-b4d9-d240f16aad47",
    "phone":"7205281825",
    "otp":"PASTE_OTP_HERE"
  }'
```

Expected response:

```json
{
  "success": true
}
```

---

## 12. Check Checkout Details

```bash
curl "http://localhost:4000/checkout/details/ffe1c7d0-4f72-48d9-b4d9-d240f16aad47"
```

Verify that the checkout session state has changed from:

```text
OTP_SENT
```

to:

```text
OTP_VERIFIED
```

Expected example:

```json
{
  "checkoutSessionId": "ffe1c7d0-4f72-48d9-b4d9-d240f16aad47",
  "step": "OTP_VERIFIED"
}
```
