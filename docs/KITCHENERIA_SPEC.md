# Kitcheneria: Project Brief for Claude Code

Kitcheneria is a small eatery in the Philippines whose customers are mostly online. We are turning a working single-file prototype into a real website with a backend. The prototype is `kitcheneria.html` (attach or place it in the repo root). It contains all UI, pricing logic and document generation, and is the reference for behaviour.

## 1. Goal

A public website plus a back office:

- **Customer site:** catalog, party trays, about, contact/FAQ, and a pre-order flow.
- **Back office:** the owners manage menu, prices, website content, orders, payments, order slips and sales invoices.
- **Scope now:** **pre-orders only**. A daily-orders module will be added later, so keep the code modular for it.

## 2. Business rules (source of truth)

### 2.1 Combo meals
- Every combo is a bento box with rice, built from 2 or 3 **specific dishes**.
- The customer picks specific dishes; the system derives the price from the dishes' **categories**.
- The same dish may be picked twice (e.g. Chicken Adobo + Chicken Adobo).
- **Combo price = base price for the category combination + sum of each selected dish's add-on.**
  - Example: Lechon Kawali carries +10 in every combo it appears in.
- Order of dishes does not matter: the combination is a multiset of categories, so use a sorted key.
- A combination not in the table is **not offered**: show a message, don't guess.

Base price table (PHP), exact data from the owners:

| Combination | Price |
|---|---|
| Chicken & Vegetables | 110 |
| Pork & Vegetables | 110 |
| Chicken & Chicken | 120 |
| Chicken & Pork | 120 |
| Pork & Pork | 130 |
| Beef & Vegetables | 130 |
| Beef & Chicken | 130 |
| Beef & Pork | 130 |
| Beef & Seafood | 160 |
| Chicken & Seafood | 140 |
| Pork & Seafood | 140 |
| Seafood & Vegetables | 140 |
| Chicken & Chicken & Vegetables | 145 |
| Chicken & Pork & Vegetables | 145 |
| Beef & Beef | 150 |
| Seafood & Seafood | 150 |
| Pork & Pork & Vegetables | 150 |
| Beef & Chicken & Vegetables | 165 |
| Beef & Pork & Vegetables | 165 |
| Chicken & Seafood & Vegetables | 165 |
| Pork & Seafood & Vegetables | 165 |
| Beef & Seafood & Vegetables | 185 |
| Beef & Beef & Vegetables | 185 |
| Seafood & Seafood & Vegetables | 185 |

Categories: Chicken, Pork, Beef, Seafood, Vegetables (combo categories) and Desserts (à la carte).

### 2.2 Other menu items
- **Desserts** (and any non-combo category) are sold individually at their own price.
- Dishes can have an optional photo and description, and an **Available** flag. Unavailable dishes show as "Not available now" and cannot be ordered.

### 2.3 Party trays
- Sizes: 10, 20, 30, 40, 50 pax (owner can add more).
- **One fixed price per size.** The customer picks any dishes; **no add-ons apply to trays**.
- Customers can order **multiple trays**.
- Tray prices are not yet provided. They are 0 (shown as "ask for a quote") until the owners enter them.

### 2.4 Quantities
All items (combos, trays, desserts) have quantities that can be changed in the cart.

### 2.5 Pre-order timing and rush fee
- Orders are accepted **at least 2 days ahead** (setting `lead`).
- Customers must pick a date and time needed.
- If it is **less than 2 days but at least `minH` hours** ahead, show a warning and require the customer to agree to a **rush fee** before continuing. The fee is added to the total.
- Less than `minH` hours ahead is blocked.
- Defaults: `lead = 2`, `minH = 3` (my assumption, please confirm), `rush = 0` (**owner has not given the amount**).

### 2.6 Payment (online, at order time)
- Methods: **GCash** and **bank transfer**. Account details are owner-editable.
- **Down payment = 70% of total** (setting `dp`, rounded up), charged at order time.
- The **balance is paid on delivery day.** This is intentional, to build trust.
- **The order is not processed until at least the 70% down payment is received/confirmed.**
- Total = items + delivery fee + rush fee. The down payment is computed on the total.
- In the prototype, the customer pays manually and enters a reference number; the owner ticks "Down payment received".

### 2.7 Fulfilment
- Pickup or delivery. **Delivery fee depends on area/distance** (owner-managed list of areas and fees). The owners haven't provided areas or fees yet.

### 2.8 Order intake channels
- The owners receive orders **only via Viber, the Facebook page (Messenger), and SMS**.
- Prototype approach: after checkout, the customer taps a channel button that opens the chat with the order text prepared. The text ends with an encoded order code. The back office pastes the message to import the order.

### 2.9 Editing orders
- Customers can **edit an order after sending it** (modify items, or replace dishes that became unavailable).
- Each edit increments a **revision**. The updated order is sent again; the back office replaces the earlier version (same order id, same order/invoice number).
- Carts with items whose dishes are now unavailable must be flagged, and checkout blocked until they're removed.
- If the new down payment is higher, the customer sends the difference with a new reference number.

### 2.10 Documents
- **Order slip** (kitchen): order no, customer, pickup/delivery details, needed date/time, items with quantities, notes, and down-payment status.
- **Sales invoice**, auto-numbered sequentially: `INV-0001`, linked to order `KC-0001`. It shows items, delivery fee, rush fee, total, down payment (received or awaiting), balance due on delivery day, and PAID IN FULL or unpaid.
- **Order summary** for customers (no invoice number).
- Every document has **Print** and **Save as Word (.docx)** buttons, for customers (summary) and the back office (slip, invoice).

### 2.11 Statuses
Awaiting down payment → Confirmed → Preparing → Ready → Out for delivery → Done (or Cancelled). Ticking "Down payment received" moves Awaiting to Confirmed. A separate "Paid in full" flag tracks the balance.

## 3. Pages

**Customer website** (hash routes in the prototype; use real routes):
- Home: hero image, pre-order notice, how to order, favorites, gallery, about teaser.
- Menu: combo builder, full catalog by category with images and descriptions.
- Party trays: tray sizes with image, description and fixed price, plus a dish picker.
- About: story, image, gallery.
- Contact: hours, address and map link, message buttons (Viber, Messenger, SMS), payment details, FAQ.
- Order: cart, quantities, customer details, date/time, rush warning, payment method and reference, send buttons, edit order.

**Back office** (needs real authentication):
- Orders: import, status, payment flags, documents.
- Menu: dishes with photo, description, category, add-on or price, availability.
- Combo prices.
- Website content: hero, about, gallery, hours, map, FAQ.
- Settings: business info, order channels, pre-order rules (lead days, min hours, rush fee, down payment %), payment methods, trays, delivery areas, categories.

**Everything must stay configurable** (owner asked for flexibility): categories, combos and prices, add-ons, tray sizes and prices, delivery areas and fees, lead time, rush fee, down payment %, payment methods, channel numbers, currency symbol, accent color, invoice header/footer.

## 4. What the prototype does and doesn't do

Prototype:
- It is a single HTML file with plain JS.
- State lives in `localStorage`; there is a thin data layer (`getCfg/setCfg/getOrders/setOrders`) meant to be replaced.
- It runs as a Claude artifact, where a "Publish to customers" button bakes the config into the page. **Drop that mechanism** once there is a real backend.
- Orders are imported by pasting a message that contains an encoded `KC2:` JSON code (base64). This is the prototype's workaround for having no server.
- `.docx` is generated client-side with JSZip (minimal WordprocessingML).
- Back office `#admin` has **no authentication**.

## 5. Suggested target architecture (adjust if you have better ideas)

- Next.js (App Router) + TypeScript + Tailwind; Postgres (Supabase or Neon) with Prisma/Drizzle.
- Auth for back office: owner/staff accounts (Supabase Auth or Auth.js).
- Image storage: object storage (Supabase Storage / S3 / Cloudinary), with resizing on upload.
- Orders stored server-side with **revision history**, the customer's edit link (signed token) so edits don't need the pasted-code workaround.
- **Server-side price calculation.** Never trust client totals. Recompute combo, tray, delivery, rush and down payment on the server.
- Payments: start with manual verification (reference number plus screenshot upload). Phase 2: PayMongo (GCash) and/or bank-transfer matching.
- Notifications to owners: Messenger and Viber webhooks/APIs, SMS gateway for the Philippines (e.g. Semaphore). Keep the "customer taps a channel button" path as a fallback.
- Invoice numbering via a database sequence; documents generated server-side (`docx` + PDF).
- Keep the pre-order domain separate from a future `daily orders` module (different availability, no lead time).

## 6. Suggested work plan

1. Scaffold the project; define the schema (categories, dishes, combos, trays, areas, settings, orders, order_revisions, payments, invoices).
2. Port the pricing engine from the prototype and cover it with unit tests (use the table above).
3. Build the public pages and the cart/checkout with server-validated totals.
4. Back office: auth, order management, payment confirmation, documents (print and .docx).
5. Content management: images, descriptions, website pages, all settings.
6. Order intake: channel buttons first, then webhooks and notifications.
7. Payments automation, deployment, backups.

## 7. Open questions for the owners

1. Rush fee amount (and is it one fee, or different for 1 day vs. hours before)?
2. Is `minH = 3` hours acceptable as the earliest allowed time?
3. Real dish list with categories and add-ons; real tray prices; delivery areas and fees.
4. GCash and bank account details; do they want screenshot upload of proof of payment?
5. What happens to the down payment if the customer cancels?
6. Operating hours, address, map link, logo and photos, "about" copy.
7. Do they have a Viber business account and Facebook page username for the channel links?
8. Is the invoice a BIR-registered receipt or an informal sales invoice (affects wording, TIN, numbering)?

## 8. Notes for Claude Code

- Read `kitcheneria.html` first; match its behaviour, then improve.
- Currency is Philippine peso (₱). Time zone is Asia/Manila (UTC+8). Be careful with date/time-needed comparisons.
- Sample data in the prototype (dish names, delivery areas, account numbers) is placeholder content. Do not treat it as real.
- Ask me before inventing business rules not listed here.
