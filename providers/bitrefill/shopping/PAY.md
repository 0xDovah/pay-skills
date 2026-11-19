---
name: shopping
title: "Bitrefill"
description: "Buy gift cards, eSIMs, mobile top-ups and prepaid cards with USDC via x402. 10,000+ brands across 180+ countries. No API keys or Bitrefill account required — wallet payment and optional SIWX sign-in only."
use_case: "Use when the user wants to browse or buy gift cards, mobile top-ups or eSIMs on Bitrefill, or manage Bitrefill orders."
category: shopping
service_url: https://api.bitrefill.com
openapi:
  path: openapi.json
---

Agent commerce without a Bitrefill account: gift cards, mobile top-ups and eSIMs
for 10,000+ brands in 180+ countries. pay pays in Solana USDC, so keep USDC in
the wallet. Full agent guide: https://www.bitrefill.com/agents/SKILL.md

## How to buy

1. Sign in once (recommended): `POST /x402/connect` is free, and pay signs in
   with the wallet. It returns a `token` valid for about 2 hours. Send it as
   `X-Access-Token` (raw token, no `Bearer`) on later calls: search, details,
   invoice and status are then free. Without it, each of those calls is paid
   separately ($0.001 to $0.002).
2. Search: `GET /x402/gift-cards/search?q=<brand>&country=<code>`,
   `GET /x402/esims/search?q=<country>` or `GET /x402/topups/search?q=<operator>`.
   Country codes are 2 capital letters (`US`, `IT`).
3. Get the price: `GET /x402/products/detail?slug=<slug>` lists the packages and
   whether a recipient is needed (`refill_input`: phone, email, etc.).
4. Prepare the order: `POST /x402/invoice/create` with
   `{"items":[{"product_id":"<slug>","package_value":"<value>"}]}`.
   `package_value` is the exact string from `packages[].package_value`: `"10"`
   for gift cards and top-ups, `"1GB, 7 Days"` for eSIMs. A wrong value returns
   HTTP 500, with no charge. The reply has `invoice_id`, `price_usdc`,
   `price_usd` and `expires_in_minutes`. An expired invoice can't be paid:
   create a new one.
5. Confirm with the user: product, amount, top-up recipient and the exact USDC
   total. Wait for explicit approval.
6. Pay: `POST /x402/invoice/pay` with `{"invoice_id":"<id>"}`. The payment must
   equal the confirmed total.
7. Poll `GET /x402/invoice/status?invoice_id=<id>` until `delivery_status` is
   `all_delivered`. Payment is not delivery.
8. The redemption code is in each order's `redemption_info`. Only the wallet
   that paid can see it: if it's missing, call status again without the token,
   so pay signs in with that wallet.

## Good to know

- The pay call (step 6) is not in pay's catalog, because its price is the
  invoice amount. Call it with `pay curl`.
- `INVOICE_NOT_PAYABLE` from the pay call means the invoice is already paid:
  poll status.
- Signed in, `GET /x402/my/orders` and `GET /x402/my/esims` list the wallet's
  orders and eSIMs.
- Not delivered after 3 hours? Go to https://help.bitrefill.com.
- Codes are not refundable once issued.
- With the user, use plain shopping words: "sign in with your wallet",
  "approve the payment", "your code is ready". Don't say x402, SIWX, JWT or
  endpoint.

## Links

- Docs: https://docs.bitrefill.com and https://github.com/bitrefill/agents
- Terms: https://www.bitrefill.com/terms/
- Refunds: https://docs.bitrefill.com/docs/refunds
- Support: https://help.bitrefill.com
