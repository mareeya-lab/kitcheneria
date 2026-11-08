# Ate Jo's Kitcheneria

Pre-order website and back office for Ate Jo's Kitcheneria. This is still a single-file prototype
(`kitcheneria.html`). It is the reference for how the real Next.js + database build should behave
(see `docs/KITCHENERIA_SPEC.md`).

Open `kitcheneria.html` in a browser. The customer site starts at `#home`. The back office is at
`#admin` (there's also a "Staff: back office" link in the footer).

## What changed from v1 (`prototype/kitcheneria-v1.html`)

**Branding**
- The logo is now a transparent PNG (`assets/logo.png`), with a cream-ink version for dark mode
  (`assets/logo-dark.png`) and a small mark for the header and favicon (`assets/logo-mark.png`).
  All three are embedded in the HTML, so the file still works on its own.
- The colours come from the logo: napkin yellow and ink black. Headings use a slab serif that
  matches "Ate Jo's". The owner can upload a different logo and accent colour in Settings.

**Customer side**
- Customers no longer see Print or Save as Word. After checkout they get an **Order receipt**: an
  itemised table with #, item (with the dishes chosen), qty, unit price and amount, then subtotal,
  delivery fee, rush fee, total, down payment and balance due on delivery day.
- On phones the receipt fits the screen with no sideways scrolling. The unit price moves under
  each item ("2 × ₱130").
- The cart shows each item's name and dishes, a quantity stepper, and the unit price and line total.
- The order message sent by Viber, Messenger or SMS is itemised and numbered, so it's easy to read
  in a chat.
- When a customer edits an order, the down payment box shows what they already sent and the
  difference to send now.
- Dates and times always use Philippine time (Asia/Manila), whatever time zone the phone is set to.
- Mobile numbers are checked. Errors appear on the page next to the button, not in a popup.

**Back office**
- **Print** and **Word (.docx)** are now only here, for the **sales invoice** and the **kitchen
  slip**. Both are laid out as tables: a header with the logo, order details, an items table, totals
  and a payment-status stamp.
- You can **view** the invoice or slip on screen before you print or save it.
- The Word file has the logo, a real items table with a shaded header row, and the totals.
- **Price check**: every order you import is repriced from the current menu, combo table, tray
  prices, delivery fees, rush rule and down payment %. If the customer's totals don't match (an
  edited order code, or prices changed since they ordered), the order is flagged before you confirm it.
- If an updated order raises the down payment after you already ticked "received", you get a reminder
  to check the new reference number.
- At the top: counts for awaiting down payment, to cook, and due in the next 48 hours, plus the
  balance still to collect. You can filter orders by status and search by name, mobile number,
  KC/INV number or payment reference.
- "Add a sample order to try it" appears when there are no orders yet.
- Confirmations (such as delete) use an in-page dialog.

## Still a prototype
- Data is saved in the browser (`localStorage`). There's no staff login yet, so `#admin` is open
  to anyone who knows the address.
- Orders still arrive through the pasted `KC2:` order code.
- The real backend (spec §5) replaces both: a server database, staff accounts, server-side pricing
  (the price check above is the same logic), and invoice numbers from a database sequence.

## Open questions for the owners (from the spec, still open)
Rush fee amount · is 3 hours the earliest allowed time · real dishes, add-ons, tray prices, delivery
areas · GCash/bank details and proof-of-payment upload · what happens to the down payment on
cancellation · hours, address, map, photos, about text · Viber/Facebook usernames · is the invoice a
BIR-registered receipt (affects the wording, TIN and numbering).

`screenshots/` shows the customer side and the back office.
