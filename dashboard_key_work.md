GoFastrr Merchant Key: What issue-merchant-key.ts --apply Does

It’s important to understand what issue-merchant-key.ts --apply actually does, because that key is essentially the Dashboard's access credential for this merchant.

What This Command Does
npx ts-node scripts/issue-merchant-key.ts --shop gjvtwa-nx.myshopify.com --apply

Breakdown
npx
 ↓
runs the package/tool without needing a global installation

ts-node
 ↓
runs the TypeScript script directly

scripts/issue-merchant-key.ts
 ↓
GoFastrr's merchant-key generation script

--shop gjvtwa-nx.myshopify.com
 ↓
selects WHICH Shopify merchant receives the key

--apply
 ↓
actually writes/updates the key in the database


The important part is that this command does not create a Shopify checkout key and it does not create a Shopify API token.

It creates a GoFastrr Dashboard merchant API key.

1. Merchant ID

Your output showed:

Merchant ID:
f58b349f-c30d-468d-842e-fb3dd63882a2


This is the internal GoFastrr merchant identifier.

Conceptually:

gjvtwa-nx.myshopify.com
        ↓
GoFastrr merchant record
        ↓
merchant_id


The Dashboard doesn't need you to manually provide this ID.

That's important for security.

When you log in with the gf_sk_... key, the backend determines:

"Which merchant does this key belong to?"

and obtains the merchant ID itself.

So the browser should not be able to say:

merchant_id = someone else's merchant


and access their data.

2. Shop Domain

Your output:

Shop Domain:
gjvtwa-nx.myshopify.com


This associates the key with your friend's Shopify store.

So the relationship is essentially:

gf_sk_...
   ↓
merchant authentication
   ↓
Merchant ID
   ↓
gjvtwa-nx.myshopify.com


This is why you used:

--shop gjvtwa-nx.myshopify.com


rather than just generating a generic key.

3. Platform

Your output:

Platform:
shopify


This tells GoFastrr which commerce platform the merchant is associated with.

In your current setup:

Platform = Shopify


This becomes useful because GoFastrr's merchant architecture can potentially support different platforms later.

For example, conceptually:

Merchant
├── Shopify
├── WooCommerce
├── ...


Your friend's merchant is currently Shopify.

4. Key ID

Your output showed:

The key identifier is:
1e94ce4ce76b358a7f0b4bad8f29d9aa


This is not the secret key itself.

Think of it as the identifier for the particular credential.

The generated key looks approximately like:

gf_sk_<key_id>_<secret>


The backend can use the key ID to locate the appropriate credential record before verifying the secret portion.

Your audit described the backend model as storing the secret_key_id together with a bcrypt hash rather than storing the raw Dashboard secret.

That's why the raw key needs to be saved when it's issued.

5. The Actual gf_sk_... Key

This is the important credential:

gf_sk_...


This is what you entered into:

https://gofastrr-dashboard.vercel.app/login


It acts as the Dashboard's Bearer authentication credential.

Conceptually:

Dashboard
   │
   │ Authorization: Bearer gf_sk_...
   ▼
GoFastrr Backend
   │
   ├── identify key
   ├── verify key
   ├── identify merchant
   └── authorize request


The backend then knows:

This request belongs to merchant
f58b349f-c30d-468d-842e-fb3dd63882a2

6. Why the Key Is Never Shown Again

Your output said:

Save this key immediately.
It will NEVER be shown again.


That's intentional.

The system doesn't want the database or normal API responses to contain the raw secret.

Instead, the backend keeps the information needed to verify the credential.

So:

RAW KEY
gf_sk_...
     ↓
shown once
     ↓
you save it


After that, the Dashboard uses the key you saved.

If you lose it, you don't retrieve the old raw secret — you issue/rotate another one.

7. What --apply Actually Changes

This is the important difference:

Without:

--apply


the script can be used in a non-destructive/checking mode depending on its implementation.

With:

--apply


it actually commits the newly generated credential to the database.

Your output confirmed:

SUCCESS: New key issued and committed to database.


So the database now recognizes the new key.

8. Why You Got the Rotation Warning

You saw:

WARNING: Existing key_id (32 chars) will be rotated and invalidated!


This means that merchant already had a Dashboard credential.

Running:

--apply


again doesn't simply create another active key alongside it.

It rotates the existing credential.

Conceptually:

OLD KEY
   ↓
INVALIDATED ❌

NEW KEY
   ↓
ACTIVE ✅


That's why you should not run the command again right now unless you intentionally want to rotate the credential.

9. What Happens When You Log Into Dashboard

This is the whole flow:

You
 │
 │ gf_sk_...
 ▼
Dashboard Login
 │
 │ GET /api/v1/merchant/settings/shopify
 │ Authorization: Bearer gf_sk_...
 ▼
GoFastrr Backend
 │
 ├── validate key format
 │
 ├── find key identifier
 │
 ├── verify secret
 │
 ├── check key is active
 │
 └── resolve merchant
       │
       ▼
 Merchant
 gjvtwa-nx.myshopify.com
       │
       ▼
 Dashboard access


Then when you open Orders:

Dashboard
   │
   │ GET /api/v1/merchant/orders
   │ Bearer gf_sk_...
   ▼
Backend
   │
   ├── authenticate key
   ├── resolve merchant_id
   └── query orders WHERE merchant_id = that merchant
   ▼
Your friend's orders


The frontend doesn't need to send:

merchant_id=f58b...


for authorization.

That's a very important part of the architecture.

10. What This Key Does NOT Give You

The gf_sk_... key is not:

❌ Shopify Admin access token

❌ Razorpay secret

❌ Shopify storefront token

❌ Customer password

❌ Checkout session ID

❌ Database password

❌ Your friend's Shopify login

It is specifically the GoFastrr merchant Dashboard API credential.

11. What Dashboard Functionality Depends on This Key

Once authenticated, the Dashboard can call the merchant APIs that your backend exposes.

From the audit, that includes:

GET    /api/v1/merchant/dashboard/overview

GET    /api/v1/merchant/orders
GET    /api/v1/merchant/orders/:id

GET    /api/v1/merchant/payments

GET    /api/v1/merchant/upsells
POST   /api/v1/merchant/upsells
PATCH  /api/v1/merchant/upsells/:id
DELETE /api/v1/merchant/upsells/:id

POST   /api/v1/merchant/upsells/:id/approve
POST   /api/v1/merchant/upsells/:id/unapprove

GET    /api/v1/merchant/shopify/products/search


So the key is basically the passport that lets the Dashboard access those merchant-scoped APIs.

The audit confirms these APIs resolve the merchant from the authenticated key and scope their database queries accordingly.

The Most Important Thing to Understand

Your current architecture is:

                    ┌──────────────────────┐
                    │ Shopify Store        │
                    │ gjvtwa-nx...         │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ GoFastrr Checkout    │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ GoFastrr Backend     │
                    │ Render               │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ Neon PostgreSQL      │
                    └──────────┬───────────┘
                               │
                  merchant_id  │
                               ▼
                    ┌──────────────────────┐
                    │ GoFastrr Dashboard   │
                    │ Vercel               │
                    └──────────────────────┘


The merchant key is the bridge that tells the backend which merchant the Dashboard session belongs to.

And now that you've successfully logged in, we have confirmed that this authentication bridge is working.

The next job is exactly what you suggested: go through the Dashboard one section at a time and correct the data/functionality rather than changing everything at once.
