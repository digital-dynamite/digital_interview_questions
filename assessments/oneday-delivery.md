# Technical Assessment — One Day Delivery

**Audience:** ecommerce architect candidates

**Format:** written design, 90 minutes (or a 45-minute whiteboard on the same brief), or 1 week as take-home

---

## 1. Scenario

We are adding **One Day Delivery** as a shipping method.

Offer it only when both are true:

- a fulfilling location within courier range has the variant available to promise, and
- a last-mile partner (Uber or equivalent) can deliver to the customer inside the promised window.

Many customers will fail one or both tests. They must still be able to shop and check out with the shipping methods we already offer.

The same decision must be available to:

- the web storefront (product detail page, cart, checkout),
- the mobile app,
- in-store point of sale.

## 2. Given systems

Do not replace these. You may add one new capability; name it and name its owner.

| System | What it is responsible for | Constraint |
| --- | --- | --- |
| **SFCC** (Headless / MRT + SCAPI) | Sellable product data: catalog, variants, price, images. Existing web basket and checkout. | Contains only country-wide aggregate inventory, not store-specific |
| **Enterprise Inventory API** | Server-to-server. Availability of a SKU at locations near a point or postal code. | Not callable from a browser, the mobile app binary, or the POS client. Rate limits are unpublished; state your assumption. |
| **Last-mile partner** | Zonal coverage, delivery promise, and booking. | A promise may be shown before a booking exists. An order must not be confirmed on an unbooked promise. |
| **Channels** | Web (SFCC), mobile app, POS. Already in production. | They must not each invent a private eligibility rule. |

POS knows the store it is running in. Web and app know a customer location (postal code or coordinates), not a store.

## 3. What the solution must satisfy

1. **Performance.** Safe to call from PDP, cart, and checkout. Give a latency budget for each surface. PDP is a read path: it must not wait on a live chain of inventory and courier calls.
2. **Peak scale.** Promo and holiday traffic multiplies PDP and cart volume far more than completed orders. Protect the Inventory API and the courier API. Say what is precomputed, what sheds load, and what degrades first.
3. **Unsupported places.** No location in range, or locations in range with no sellable inventory, are normal outcomes. Do not offer the method. Do not block add-to-cart, other shipping methods, or checkout. Define the eligibility states and which system decides each one.
4. **Cross-channel.** Web, app, and POS use one contract. Inputs differ (customer location vs. current store); the promise rules do not. Two channels must not sell the last unit.
4. **Secure authentication** Prevent unauthenticated clients from accessing API resources.

## 4. What to hand in

Three to five pages, or one diagram plus two pages. No code. No redesign of the catalog or the inventory platform.

1. **Context diagram.** Systems, the new capability, and who calls whom. Mark the trust boundary between clients (browser, app, POS) and server-side callers. Use C4 Modelling; providing C1 + C2 levels is sufficient for diagrams
2. **Decision contract.** Request and response for “can we offer One Day Delivery?” Include ineligible and unknown. Show how web, app, and POS call it.
3. **Three moments.** PDP view, checkout selection, POS sale. For each, say what is cached and what is live.
4. **Freshness and hold.** How old a “yes” may be on PDP versus checkout, and how inventory is held when the order is placed.
5. **Peak and failure.** Inventory API slow, inventory API down, courier API down. What the customer sees on each surface.
6. **Trade-offs.** At least three. For each, what you gain and what you give up.
7. **Measures.** The few indicators you would hold the design to (latency, false “available,” oversell, availability of the page when a dependency fails).
8. **Assumptions and questions.** What you need from merchandising, store operations, and the inventory team before build. State assumptions where you must proceed without an answer.

Address these cases inside the design, even if only as an assumption:

- A cart mixes eligible and ineligible lines.
- The customer’s location changes between PDP and checkout.
- The store can fulfill, but it closes before the courier can collect.
