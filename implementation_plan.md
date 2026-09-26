# Implementation Plan

## Production checklist

- Set `NODE_ENV=production`, a unique 32+ character `JWT_SECRET`, scoped `CORS_ORIGINS`, and `PUBLIC_APP_URL`.
- Configure SMTP variables before enabling account verification and password reset in production.
- Configure Razorpay `RAZORPAY_WEBHOOK_SECRET` and register `/api/payments/webhook` for `payment.captured` and `order.paid` events.
- Run the SQLite database on persistent encrypted storage for a demo deployment; migrate to managed PostgreSQL before high traffic or multi-instance deployment.
- Put the API behind an HTTPS reverse proxy/WAF, restrict inbound ports, and configure daily encrypted backups with restore testing.
- Send application logs to a redacted centralized service and alert on repeated login failures, 5xx responses, webhook failures, and admin actions.
- Create the first administrator by setting `users.role = 'admin'` through a controlled migration or database console. Never expose role assignment to the public API.
- Run `npm audit`, review the Vite/esbuild upgrade, and perform a dependency update before production release.

## Goal
Create a full‑stack e‑commerce experience for the WINDRUNNER ’86 site:
- Backend API (Express + SQLite) with products, users, cart, orders, reviews.
- Front‑end integration: product catalogue, product detail, shopping cart, checkout flow.
- Opening page animation (already added) and a polished UI that follows the premium tailoring aesthetic.
- Development workflow that runs the Vite client and Express server concurrently.
- Basic store info (owner name/address) displayed in the footer.

## User Review Required
> [!IMPORTANT] Confirm that you are happy with the chosen backend (SQLite + Express) and the simplified cart implementation (guest session + authenticated users). If you prefer a different DB (MongoDB) let us know now.

## Open Questions
- Do you need user registration/login flows fully exposed in the UI? (We have API endpoints ready.)
- Shipping/payment: will “cash on delivery” be sufficient for the demo, or do you want a Stripe integration?
- Should the product catalogue be a separate page or stay within the scroll‑driven story? (We can add a grid view after the story section.)

## Proposed Changes
---
### Backend (`server/`)
- **[MODIFY] `server/index.cjs`** – expose routes under `/api/*` (already done).
- **[NEW] `server/middleware/auth.cjs`** – tiny JWT auth helper (optional, extracted for clarity).
- **[NEW] `server/router/products.cjs`**, `router/cart.cjs`, `router/orders.cjs`, `router/users.cjs` – split routes for maintainability.
- **[MODIFY] `package.json`** – added `concurrently` script to run client and server together (already updated).
- **[NEW] `.env`** – optional env vars for `PORT` and `JWT_SECRET`.

---
### Front‑end (`src/`)
- **[NEW] `src/components/OpeningAnimation.jsx` & CSS** – already created.
- **[MODIFY] `src/App.jsx`** – import `OpeningAnimation`, render it conditionally on first load, then show the rest of the site.
- **[NEW] `src/components/ProductGrid.jsx`** – grid view fetching `/api/products` and displaying cards with image, name, price, and “Add to Cart”.
- **[NEW] `src/components/ProductDetail.jsx`** – page for a single product (`/product/:id`) with larger images, description, size selector, reviews, and “Add to Cart”.
- **[NEW] `src/components/CartDrawer.jsx`** – slide‑in drawer showing cart items, totals, and a “Checkout” button.
- **[NEW] `src/components/Checkout.jsx`** – simple form collecting shipping name/address/phone and submitting to `/api/orders`.
- **[NEW] `src/contexts/CartContext.jsx`** – React context managing cart state (persisted in `localStorage` for guests, synced with backend when logged in).
- **[NEW] `src/components/Footer.jsx`** – fetch `/api/store` and display owner name, address, contact.
- **[MODIFY] `src/index.css`** – add utility classes for grid cards, buttons, and modal overlay.
- **[NEW] `src/pages/Store.jsx`** – main page that renders `OpeningAnimation` then the scroll‑driven story followed by the product grid.
- **[UPDATE] Vite config** – proxy `/api` to `http://localhost:4000` during dev.

---
### Dependencies
- Install `framer-motion` (already added for animation).
- Install `axios` or use native `fetch` for API calls.
- Ensure `react-router-dom` is present for navigation (already a dependency of Vite React template).

## Verification Plan
### Automated Tests
- Run `npm run build` – should succeed with no compile errors.
- Start dev with `npm run dev` – both client (Vite) and server (Express) should be live.
- Manually visit `http://localhost:3000`:
  1. Opening animation appears then fades.
  2. Scroll story works as before.
  3. Product grid loads 3 seeded items.
  4. Adding to cart updates the cart drawer.
  5. Checkout creates an order (check server console for order log).
  6. Footer displays “Abdul Rahuman – Thuvarankurichy, Trichy dist‑621314, India”.

### Manual Verification
- Verify responsive layout on mobile.
- Test guest cart persistence across refreshes.
- Test logged‑in flow (register, login, cart sync).
- Confirm that API routes return `200` and data shapes are as expected.

---
**Next Steps**
1. Implement the front‑end components listed above.
2. Split backend routes into modules for clarity.
3. Add Vite proxy config.
4. Run full dev session and verify.

Please review the plan and confirm or answer the open questions so we can proceed with the implementation.
