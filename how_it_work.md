# GoFastrr Shopify Integration Plan

Think of **GoFastrr** as a hosted checkout service that merchants connect to their online store.

For the initial release, **Shopify is the first priority**.

---

# Merchant Journey

The ideal merchant flow should be:

```text
Merchant installs GoFastrr Shopify App
        ↓
Connects their Shopify store
        ↓
Enables GoFastrr Checkout Button / Theme App Embed
        ↓
Customer clicks "Checkout with GoFastrr"
        ↓
GoFastrr Checkout opens
        ↓
Customer completes payment via Razorpay
        ↓
GoFastrr syncs the order back to Shopify
        ↓
Merchant fulfils the order from Shopify or the GoFastrr Dashboard
```

---

# 1. What We Provide to Shopify Merchants

Merchants should install a **Shopify App** instead of manually adding code.

Each merchant receives:

1. Shopify App installation
2. GoFastrr Merchant Dashboard
3. Test Mode / Live Mode
4. Checkout customization settings
5. Payment settings
6. Orders & Payments dashboard
7. Theme App Embed / App Block configuration

## Recommended Shopify Architecture

```text
GoFastrr Shopify App
        +
Theme App Extension
        +
App Embed / App Block
        +
GoFastrr Hosted Checkout
```

---

# 2. Customer Checkout Flow

## Version 1 (Recommended)

```text
Customer adds products to Shopify cart
        ↓
Clicks "Checkout with GoFastrr"
        ↓
GoFastrr creates Checkout Session
        ↓
Hosted GoFastrr Checkout opens
        ↓
Phone OTP verification
        ↓
Shipping Address
        ↓
Payment Method
        ↓
Razorpay Payment
        ↓
Payment Success
        ↓
GoFastrr syncs order to Shopify
```

The merchant can keep Shopify's default checkout button while offering GoFastrr as an alternative fast checkout.

## Future Enhancement

Eventually support:

```text
Replace Shopify Checkout completely
```

### Product Roadmap

- ✅ Add GoFastrr Checkout button (V1)
- ✅ Replace Shopify Checkout later (Future)

---

# 3. Order Sync After Payment

After a successful Razorpay payment:

```text
Razorpay Webhook
        ↓
GoFastrr marks Checkout as Paid
        ↓
Create / Sync Shopify Order
        ↓
Store Order, Payment & Customer in GoFastrr Database
        ↓
Merchant views order in Dashboard
        ↓
Merchant fulfils order from Shopify
```

## Recommended V1 Strategy

```text
Payment completed in GoFastrr
        ↓
Create Shopify Order (or Draft Order)
        ↓
Mark as Paid / External Payment
```

### Things to Validate

- Inventory updates
- Taxes
- Shipping
- Discounts
- Payment records
- Order status

---

# 4. Merchant Dashboard

The dashboard should manage all merchant operations.

## Pages

- Overview
- Orders
- Payments
- Customers
- Abandoned Checkouts
- Checkout Customization
- Shopify Connection
- API Keys
- Webhooks
- Settings

## Order Details

Each order should display:

- Order ID
- Customer Name
- Phone
- Email
- Products
- Order Amount
- Payment Status
- Shopify Sync Status
- Razorpay Payment ID
- Fulfilment Status
- Created Time

---

# 5. Custom Website Integration

For merchants not using Shopify, GoFastrr should provide an API + JavaScript SDK.

## Backend

```http
POST /api/v1/checkouts
Authorization: Bearer gf_sk_xxx
```

Response:

```json
{
  "checkoutSessionId": "...",
  "checkoutUrl": "https://checkout.gofastrr.io/checkout?session=..."
}
```

## Frontend

```html
<script src="https://cdn.gofastrr.io/checkout.js"></script>
```

```javascript
GoFastrrCheckout.open({
  checkoutSessionId: "...",
  onSuccess(order) {},
  onClose() {},
  onError(error) {}
});
```

### Customer Flow

```text
Merchant Website Cart
        ↓
Merchant Backend creates Checkout Session
        ↓
GoFastrr Checkout opens
        ↓
Customer completes payment
        ↓
Webhook sent to Merchant Backend
        ↓
Merchant fulfils order in their own system
```

---

# 6. Shopify vs Custom Website

| Shopify | Custom Website |
|----------|----------------|
| Install Shopify App | Integrate via API |
| Shopify OAuth | API Keys |
| Theme App Extension | JavaScript SDK |
| Orders sync back to Shopify | Merchant stores orders in own system |
| Minimal setup | Developer integration required |

---

# 7. Shopify Development Roadmap

## Phase 1 — Shopify App Foundation

- Shopify OAuth
- Install Flow
- Store Access Token securely

---

## Phase 2 — Checkout Button

- Theme App Extension
- App Embed
- "Checkout with GoFastrr" button

---

## Phase 3 — Checkout Session

```text
Shopify Cart
        ↓
Create GoFastrr Checkout Session
        ↓
Open Hosted Checkout
```

---

## Phase 4 — Payment & Order Sync

```text
Razorpay Webhook
        ↓
Payment Verified
        ↓
Create Shopify Order
        ↓
Sync Status Updated
```

---

## Phase 5 — Merchant Dashboard

Dashboard includes:

- Orders
- Payments
- Customers
- Analytics
- Checkout Settings
- Shopify Connection
- API Keys
- Webhooks

---

# Complete Shopify Flow

```text
Merchant installs GoFastrr App
        ↓
OAuth Installation
        ↓
Theme App Extension Enabled
        ↓
Checkout Button Added
        ↓
Customer adds products to cart
        ↓
Customer clicks Checkout with GoFastrr
        ↓
Checkout Session Created
        ↓
Hosted Checkout Opens
        ↓
Phone OTP
        ↓
Address
        ↓
Payment via Razorpay
        ↓
Payment Success
        ↓
Webhook Received
        ↓
GoFastrr Creates Shopify Order
        ↓
Merchant fulfils order
```

---

# Recommended Product Strategy

## Version 1

Focus on:

- ✅ Shopify App
- ✅ Alternative GoFastrr Checkout Button
- ✅ Hosted GoFastrr Checkout
- ✅ Razorpay Integration
- ✅ Shopify Order Sync
- ✅ Core Order & Payment APIs
- ✅ Merchant Dashboard

Avoid replacing Shopify's default checkout initially, as it can introduce:

- Theme conflicts
- Checkout conflicts
- Order synchronization issues
- Compatibility challenges

Instead, position GoFastrr as:

> **"Add GoFastrr Fast Checkout to your Shopify Store."**

Once the platform is stable and battle-tested, introduce:

> **"Replace the default Shopify Checkout with GoFastrr."**

This provides the safest rollout path while minimizing risk for merchants.
