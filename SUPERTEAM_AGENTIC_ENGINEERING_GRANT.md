# Superteam Agentic Engineering Grant Draft

## 1. Project Overview

Storesolly Mobile POS is a mobile-first retail checkout application for small and medium merchants that want to accept Solana-powered payments at the point of sale. The product focuses on fast checkout, simple item entry, QR-based Solana Pay payment requests, realtime payment verification, and a lightweight merchant workflow that works from a phone instead of requiring expensive POS hardware.

The MVP goal is to let a merchant create a cart, quote the total in local currency, generate a Solana Pay QR code, verify settlement in realtime, and issue a clear receipt/order record. Storesolly is designed for retail environments where speed, trust, and payment confirmation matter more than complex enterprise POS features.

## 2. Real-World Problem

Many small merchants still depend on cash, bank transfers, manual payment confirmation, or expensive card terminals. These flows create friction:

- Payment confirmation can be slow or manual.
- Merchants may have to check bank apps during checkout.
- Customers may abandon purchases when card terminals fail.
- Small merchants often cannot afford full POS hardware.
- Crypto checkout is still too technical for everyday retail use.

Storesolly solves this by turning a merchant phone into a Solana-enabled checkout terminal. The app abstracts wallet addresses, payment links, transaction signatures, and chain verification behind a familiar retail checkout flow.

## 3. Mobile Checkout Workflow

1. Merchant opens the mobile app and starts a new sale.
2. Merchant adds products from catalog, scans/searches items, or enters a custom amount.
3. App calculates subtotal, discounts, taxes, and final payable amount.
4. Merchant selects payment method: Solana Pay.
5. App creates a Solana Pay payment request with recipient wallet, amount, reference, label, message, and optional memo/order id.
6. Customer scans the QR code using a Solana-compatible wallet.
7. Customer approves payment.
8. App watches the Solana network for a transaction matching the unique reference.
9. App validates recipient, amount, token, confirmation status, and reference.
10. Checkout screen updates to paid in realtime.
11. App records the order, payment signature, timestamp, and receipt details.
12. Merchant can share or show the receipt.

The merchant should never need to inspect a block explorer during checkout. Payment state must be visible as clear statuses: `Awaiting payment`, `Detected`, `Confirming`, `Paid`, `Expired`, or `Failed`.

## 4. Solana Pay Integration Architecture

### Client Layer

- Mobile app generates Solana Pay URLs using `@solana/pay`.
- Each checkout creates a unique payment reference public key.
- QR code is rendered from the Solana Pay URL.
- The checkout screen starts a realtime verification loop after QR generation.

### Payment Request Data

Each payment request should include:

- Merchant recipient wallet address.
- Amount denominated in SOL, USDC, or supported SPL token.
- Unique reference key for matching transaction.
- Merchant label, store name, and checkout message.
- Internal order id in memo or backend metadata.

### Verification Layer

The app or backend checks Solana RPC for transactions matching the reference:

- Use `findReference` to detect candidate transactions.
- Use `validateTransfer` to verify recipient, amount, SPL token, and reference.
- Require a practical confirmation threshold before marking an order as paid.
- Store transaction signature and verification result.

### Backend Layer

For MVP speed, the app can start with a lightweight backend service that handles:

- Merchant profile and wallet configuration.
- Order creation and payment intent records.
- RPC verification endpoint.
- Websocket or polling-based checkout status updates.
- Receipt and sales history persistence.

The backend should own final payment status to prevent client-side tampering. The mobile app can detect payment optimistically, but launch readiness requires server-side verification before fulfillment.

### Data Model

Core entities:

- `Merchant`: business profile, wallet address, settlement token, store settings.
- `Product`: name, price, SKU/barcode, image, stock count.
- `CartItem`: product id, quantity, price snapshot.
- `Order`: merchant id, items, total, currency, status.
- `PaymentIntent`: order id, recipient, amount, token, reference, expiry.
- `PaymentTransaction`: signature, slot, confirmation status, validation result.

## 5. Suggested MVP Feature Scope

### Must Ship

- Merchant onboarding with store name and Solana wallet setup.
- Product list with add/edit/delete and quick custom amount.
- Mobile cart and checkout screen.
- Solana Pay QR generation.
- Realtime payment detection and verification.
- Paid/failed/expired payment states.
- Order history with payment signature.
- Basic receipt screen/share action.
- Local currency display with crypto settlement amount.

### Should Ship If Time Allows

- USDC-first checkout for price stability.
- Barcode/SKU search.
- Basic inventory decrement after successful payment.
- Merchant dashboard summary: daily sales, successful payments, failed/expired payments.
- Offline cart draft support.

### Defer

- Multi-branch support.
- Staff permissions.
- Advanced inventory management.
- Accounting exports.
- Loyalty programs.
- Complex tax configuration.
- Hardware printer integration.

## 6. Four-Week Shipping Roadmap

### Week 1: Foundation and Checkout Skeleton

- Finalize mobile app structure and navigation.
- Build merchant setup flow.
- Implement product catalog and custom amount entry.
- Build cart state and checkout UI.
- Define order and payment intent models.
- Set up backend persistence if not already available.

Deliverable: merchant can create a cart and reach a checkout screen.

### Week 2: Solana Pay Payment Flow

- Integrate `@solana/pay` URL creation.
- Generate unique payment references per checkout.
- Render QR code on mobile checkout screen.
- Add payment expiry timer and retry/regenerate flow.
- Implement RPC-based reference lookup.
- Validate recipient, amount, token, and reference.

Deliverable: merchant can request and verify a real Solana Pay payment on devnet, then mainnet-beta once configured.

### Week 3: Merchant-Ready POS Workflow

- Add order history and receipt screen.
- Persist transaction signature and payment status.
- Add realtime checkout state updates through polling or websocket.
- Improve error handling for underpayment, wrong recipient, expired payment, and RPC failure.
- Add USDC/SPL token support if MVP requires price stability.
- Tighten mobile layout for one-handed merchant use.

Deliverable: end-to-end checkout is reliable enough for internal pilot testing.

### Week 4: Pilot, Hardening, and Launch Prep

- Run pilot tests with 3-5 merchants.
- Fix checkout friction from real merchant feedback.
- Add basic analytics for checkout started, QR shown, paid, expired, failed.
- Add production wallet/RPC configuration.
- Add security checks around merchant wallet and payment verification.
- Prepare demo video, grant update, and launch checklist.

Deliverable: production-ready MVP for limited merchant pilot.

## 7. AI-Assisted Engineering Workflow Using Codex CLI

Storesolly can use Codex CLI as an agentic engineering workflow to compress MVP shipping time:

- Use Codex to inspect the repository, identify missing POS flows, and generate implementation plans.
- Build feature slices end-to-end: catalog, cart, checkout, payment verification, receipt.
- Ask Codex to write focused tests for payment parsing, order state transitions, and Solana Pay validation.
- Use Codex for refactors that preserve behavior while improving folder structure and app scaling.
- Generate migration scripts, API contracts, and typed models from product requirements.
- Let Codex review checkout UX edge cases, especially failed payment states and realtime verification behavior.
- Use Codex to maintain launch checklists, grant updates, and pilot feedback summaries.

Recommended loop:

1. Create a small task: "Implement Solana Pay QR checkout for existing cart."
2. Let Codex inspect relevant files and propose changes.
3. Apply the code changes.
4. Run tests/lint/build.
5. Ask Codex to fix failures.
6. Manually test on mobile.
7. Commit one vertical slice.

This keeps the project moving through small, reviewable increments instead of broad rewrites.

## 8. Suggested Folder Architecture

```text
src/
  app/
    navigation/
    providers/
    screens/
  features/
    auth/
    merchant/
    catalog/
    cart/
    checkout/
    payments/
    orders/
    receipts/
  components/
    ui/
    forms/
    layout/
  services/
    api/
    solana/
    storage/
    analytics/
  domain/
    models/
    money/
    order-state/
  hooks/
  utils/
  config/
  assets/
```

Payment-specific structure:

```text
src/features/payments/
  components/
    SolanaPayQRCode.tsx
    PaymentStatusPanel.tsx
  hooks/
    useSolanaPaymentIntent.ts
    usePaymentVerification.ts
  services/
    createSolanaPayUrl.ts
    verifySolanaPayment.ts
  types/
    paymentIntent.ts
    paymentStatus.ts
```

This architecture keeps merchant UX features separate from low-level Solana services. The app can scale without mixing UI screens, RPC logic, and order state transitions in the same files.

## 9. Pilot Merchant Onboarding Strategy

Start with a small, hands-on pilot rather than a broad public launch.

Target merchants:

- Small retail shops.
- Pop-up vendors.
- Campus merchants.
- Event sellers.
- Phone-first merchants without existing POS hardware.

Pilot process:

1. Recruit 3-5 merchants who already accept digital payments.
2. Help each merchant set up a Solana wallet and choose settlement token.
3. Add 10-30 common products to their catalog.
4. Run a training session focused on starting a sale, showing QR, and confirming payment.
5. Observe live checkout sessions and record friction.
6. Track QR generation, scan success, payment confirmation time, and failed checkouts.
7. Iterate weekly on merchant feedback.

Success metrics:

- Merchant can complete checkout without developer support.
- Payment confirmation is visible in under a few seconds after transaction finalization.
- Merchant trusts the in-app paid status.
- At least 80% of pilot transactions complete without manual intervention.

## 10. Technical Milestones Before Launch

- Production Solana RPC provider selected and configured.
- Merchant wallet setup flow completed.
- Unique payment reference generated for every checkout.
- Server-side payment verification implemented.
- Payment validation checks recipient, amount, token, reference, and confirmation status.
- Checkout handles expired, duplicate, underpaid, and failed payments.
- Orders and transaction signatures are persisted.
- Receipt and order history are available to merchants.
- Mobile UI tested on common Android screen sizes.
- Basic telemetry added for checkout conversion and failures.
- Error logs capture RPC failures and validation errors.
- Pilot merchant data can be reset or migrated cleanly.
- App configuration separates devnet, test, and production environments.
- Security review completed for wallet addresses, payment status updates, and backend APIs.

## Grant Positioning Summary

Storesolly Mobile POS is a practical agentic engineering project that brings Solana Pay into everyday retail checkout. The grant would support a focused four-week MVP push: building a merchant-ready mobile checkout app, integrating realtime Solana Pay verification, piloting with real merchants, and proving that Solana can power fast, low-friction in-person commerce from a standard smartphone.

The strongest MVP thesis is simple: merchants do not need another crypto demo; they need a checkout flow that feels as reliable as a card terminal and as lightweight as a mobile app.
