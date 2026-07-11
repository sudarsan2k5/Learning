# How to Create a New Checkout Session ID

## Option 1: You Already Have the `gf_sk_...` Secret Key

### 1. Export your merchant secret

```bash
export GOFASTRR_MERCHANT_SECRET="PASTE_YOUR_GF_SK_KEY_HERE"
```

(Optional) Verify that the environment variable is set:

```bash
echo ${GOFASTRR_MERCHANT_SECRET:0:12}
```

### 2. Create a Checkout Session

```bash
curl -s -X POST http://localhost:4000/api/v1/checkouts \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GOFASTRR_MERCHANT_SECRET" \
  -H "Idempotency-Key: test-checkout-$(date +%s)" \
  -d '{
    "line_items": [
      {
        "external_item_id": "test_product_1",
        "title": "Test Product",
        "quantity": 2,
        "unit_price_paise": 50000
      }
    ]
  }' | python3 -m json.tool
```

### Example Response

```json
{
  "checkoutSessionId": "b49bdae2-445c-461b-af.......",
  "url": "http://localhost:4000/checkout?session=b49bdae2-445c-................."
}
```

### Output

- **`checkoutSessionId`** – The unique ID of the newly created checkout session.
- **`url`** – The checkout URL that can be opened in a browser or shared with the customer.

---

# If You Get an `Unauthorized` Error

Your merchant secret may have expired or been rotated. Regenerate a new key.

From the repository root:

```text
goFasterr/GoFastrrBE
```

Run:

```bash
npx ts-node scripts/issue-merchant-key.ts \
  --merchant-id f327e30e-........................ \
  --apply
```

The command will print a new raw merchant secret like:

```text
gf_sk_...
```

Copy the full key locally. **Do not paste it into chat or commit it to source control.**

Export the new key:

```bash
export GOFASTRR_MERCHANT_SECRET="PASTE_NEW_GF_SK_KEY_HERE"
```

(Optional) Verify it:

```bash
echo ${GOFASTRR_MERCHANT_SECRET:0:12}
```

Create a new checkout session again:

```bash
curl -s -X POST http://localhost:4000/api/v1/checkouts \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GOFASTRR_MERCHANT_SECRET" \
  -H "Idempotency-Key: test-checkout-$(date +%s)" \
  -d '{
    "line_items": [
      {
        "external_item_id": "test_product_1",
        "title": "Test Product",
        "quantity": 2,
        "unit_price_paise": 50000
      }
    ]
  }' | python3 -m json.tool
```
