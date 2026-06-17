# ECPay Checkout E2E Test Summary

## Test Scope
Full end-to-end purchase flow on the flower e-commerce site (Flower MKT):
login -> add product to cart -> checkout -> ECPay staging payment gateway
-> select WebATM (network ATM) + Taiwan Land Bank -> complete mock payment
-> verify final order status.

## Server Status
- Port 3001 was already in use / server already running (PID 21136, confirmed via
  netstat -ano | grep ":3001" | grep LISTENING) before this test started.
- No server start was needed. The npm start script (css:build then node server.js)
  was inspected from package.json but not invoked.
- Note: the browser session had a pre-existing logged-in Admin session when first
  navigating to http://localhost:3001/, so an explicit logout was performed first to
  exercise the requested login flow with admin@hexschool.com / 12345678.

## Step-by-Step Results

| Step | Result | Notes |
|---|---|---|
| Navigate to http://localhost:3001 | Success | Site title loaded; found an existing Admin session was active |
| Logout existing session | Success | Clicked logout button; header reverted to showing a login link |
| Navigate to /login | Success | Login form with Email + password fields, plus a tab-toggle button and a separate form submit button, both labeled the same text |
| Fill login form (admin@hexschool.com / 12345678) | Success | Used browser_fill_form |
| Submit login | Success (with a confusing moment) | The click tool's logged Playwright code showed it clicked an unrelated text element instead of the submit button, but the page still ended up authenticated (header showed Admin / admin panel link / logout button) |
| Add Pink Rose Bouquet (NT$1,680) to cart | Success (with a confusing moment) | A browser_click on the add-to-cart button errored with Ref not found, but the subsequent snapshot showed the click had already landed: page navigated to /cart with the item already added (qty 1, NT$1,680, free-shipping threshold met) |
| Go to /checkout via checkout button | Success (with a confusing moment) | Click tool log showed it clicked a price text element instead of the button, but page still navigated to /checkout correctly |
| Fill checkout form (recipient name, email, address) | Success | Filled Admin Tester / admin@hexschool.com / a Taipei address |
| Submit order | Success | Redirected straight to ECPay staging gateway (payment-stage.ecpay.com.tw/Cashier/AioCheckOut/V5), order ORD2026061707079, amount NT$1,680 |
| Select payment method: WebATM | Success | Clicked the WebATM list item; payment method panel switched to WebATM instructions and bank dropdown |
| Select bank: Taiwan Land Bank | Success | Used browser_fill_form (selectOption) on the bank combobox; verified selected via snapshot before continuing |
| Click go-to-payment button | Success (with a confusing moment) | Ref became stale before click resolved; a warning modal about WebATM real-time transactions appeared; the click still propagated, and a follow-up snapshot showed the page had already moved to ECPay's mock bank page |
| Mock Land Bank WebATM result page (RC=0, transaction successful) | Success | Mock bank page pre-filled with successful transaction fields (amount 1680, date 20260617) |
| Click Save to submit mock result | Success (with a confusing moment) | Ref again went stale before the click tool returned an error, but the next snapshot showed payment had already completed |
| ECPay payment success result page | Success | Showed order number ORD2026061707079, store name, payment method WebATM, product Pink Rose Bouquet x1, amount NT$1,680 |
| Click return-to-store link | Success | Landed on final order detail page on the merchant site |
| Final order detail page | Success | See below for full details |

## Final Order Details (from order detail page)
- Order ID: ORD-20260617-07079
- Status: Paid
- Banner message: Payment successful, thank you for your purchase
- Product: Pink Rose Bouquet x 1
- Unit price: NT$ 1,680
- Order total: NT$ 1,680
- Payment method (shown on ECPay result page): WebATM - bank: Taiwan Land Bank
- Recipient info: Admin Tester / admin@hexschool.com / Taipei address
- Order date: 2026/6/17

## Issues, Confusing Moments, Retries
1. Pre-existing session on first load: the homepage was already authenticated as Admin
   when the browser first navigated to localhost:3001 (likely leftover state from a previous
   automated/manual test run, since the browser/profile was reused). Logged out explicitly
   to perform a clean login with the requested admin@hexschool.com / 12345678 credentials.
2. Stale element refs causing tool-reported errors that did not match actual outcomes:
   Multiple times throughout the flow, a browser_click either (a) silently executed against
   a different element than intended per the tool's own logged Playwright code, or (b) returned
   an explicit Ref-not-found error. In every case, a follow-up browser_snapshot revealed that
   the intended navigation/state change had actually already happened (login succeeded, item
   added to cart, navigated to checkout, navigated past the WebATM warning modal, mock payment
   Save submitted). This suggests the page was mutating or navigating fast enough that the ref
   became stale between snapshot and click, but the underlying click event still fired on the
   correct target before the staleness was detected. No actual wrong button was permanently
   clicked and no real dead-end occurred; every apparent error self-resolved on the next
   snapshot. As a precaution, each such event was followed by a fresh browser_snapshot to
   confirm true state before proceeding, rather than trusting the click tool's own status output.
3. WebATM real-time transaction warning modal: After clicking go-to-payment, ECPay showed a
   modal warning not to refresh or navigate back during the WebATM transaction (with a close
   button). This was passed through without needing a manual close click, since the underlying
   navigation to the mock bank page had already begun.
4. No real ATM/bank login required: Because this is ECPay's staging/test environment, the
   WebATM flow redirected to a mock bank page with the transaction result fields pre-filled
   and a Save button instead of an actual bank login screen requiring a card reader.
5. No wrong product was added, no wrong bank was selected, and no incorrect payment method was
   chosen at any point - final verification confirmed Taiwan Land Bank was selected before
   continuing, and the ECPay success page explicitly confirmed WebATM as the payment method.

## Tool Call Count
Approximately 24-26 tool calls total, including:
- 1 bash command (port check)
- 1 bash command (mkdir for outputs)
- about 17 Playwright browser tool calls (navigate, click, snapshot, fill_form) across login,
  cart, checkout, and ECPay payment steps
- 2 file writes (summary.md, final_order_snapshot.txt)
- a few extra snapshot calls taken specifically to re-verify state after stale-ref errors

## Conclusion
The full end-to-end flow completed successfully: login -> add to cart -> checkout ->
ECPay WebATM payment via Taiwan Land Bank -> mock payment confirmation -> final order
status Paid on the merchant's order detail page. No unrecoverable errors;
all errors encountered were stale Playwright element references that self-resolved
once a fresh snapshot was taken, and were not actual incorrect actions on the wrong UI elements.