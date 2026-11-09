# ai-food-amigos
Multi-restaurant food ordering application built with Laravel. It's designed to be a comprehensive e-commerce solution for a food delivery platform, featuring a multi-layered user system, multi-authentication, and a variety of advanced functionalities.

# Multi-Restaurant Food Ordering Platform

> Product Requirements Document (PRD) · Version 1.0 · Planning draft
>
> Status: Proposed requirements, not implemented functionality.

A web-first food ordering and delivery marketplace built around Laravel, Inertia.js, and Vue.js. This repository's README defines the product scope, workflows, requirements, acceptance criteria, and delivery plan.

All priorities, targets, architecture choices, and estimates are planning assumptions, not contractual commitments or claims about existing code.

## Contents

- [1. Product vision](#1-product-vision)
- [2. Release assumptions](#2-release-assumptions)
- [3. Users and permissions](#3-users-and-permissions)
- [4. Core user journeys](#4-core-user-journeys)
- [5. Functional requirements](#5-functional-requirements)
- [6. State machines](#6-state-machines)
- [7. Cancellation and exceptions](#7-cancellation-and-exceptions)
- [8. Key screens](#8-key-screens)
- [9. Technical architecture](#9-technical-architecture)
- [10. Non-functional requirements](#10-non-functional-requirements)
- [11. Analytics and success measures](#11-analytics-and-success-measures)
- [12. Acceptance criteria](#12-acceptance-criteria)
- [13. Delivery roadmap](#13-delivery-roadmap)
- [14. Risks](#14-risks)
- [15. Launch gates](#15-launch-gates)
- [16. Decisions required](#16-decisions-required)
- [17. Repository readiness](#17-repository-readiness)

## 1. Product vision

Create a food marketplace where customers discover restaurants, order meals, pay securely, and follow delivery progress; restaurants manage menus and incoming orders; riders accept and complete deliveries; and administrators oversee operations, payments, refunds, and disputes.

Every order must have a clear financial state, preparation state, delivery state, and accountable owner.

### Goals

1. Make ordering simple and transparent.
2. Give restaurants reliable control over incoming demand.
3. Provide riders with an unambiguous delivery workflow.
4. Give administrators visibility into operational exceptions and financial reconciliation.
5. Establish a modular foundation for additional gateways, advanced dispatch, and future multi-restaurant checkout.

### First-release non-goals

- Native iOS or Android applications.
- One checkout containing items from multiple restaurants.
- Multi-stop route optimization or delivery batching.
- AI dispatch, recommendations, or dynamic pricing.
- POS, kitchen hardware, or external delivery-network integrations.
- Loyalty subscriptions, advertising auctions, or restaurant bidding.
- Cryptocurrency or stablecoin payments.
- Multi-country tax and settlement operations.

The initial release supports multiple restaurants on the platform, but one restaurant per checkout. Cross-restaurant baskets and coordinated multi-pickup deliveries require a separate scope decision.

## 2. Release assumptions

| Area | Proposed first-release assumption |
|---|---|
| Region | One launch market and defined delivery service area |
| Currency | One transactional currency |
| Language | One launch language; localization-ready interface |
| Experience | Responsive web application |
| Restaurant structure | One location per operating account initially; extensible data model |
| Cart | One restaurant per active cart |
| Fulfillment | Immediate orders using platform-managed riders |
| Customer pickup | Optional; explicit launch decision required |
| Payments | One live gateway initially; extensible integration model |
| Cash on delivery | Excluded |
| Dispatch | Basic rider offers with administrator-assisted assignment |
| Tracking | Milestone-based; continuous GPS deferred |
| Promotions | Item sale prices and one eligible coupon per order |
| Authentication | Role-specific entry points, shared identity, explicit authorization |
| Hosting | Provider and environment selected during discovery |
| Invoices | PDF documents reflecting approved seller, tax, and invoicing model |

Select one Laravel major version before implementation. Laravel 11/12/13 represents alternatives, not simultaneous compatibility requirements.

Breeze is requested authentication scaffolding. Validate compatibility with the selected framework version and approve an alternative starter kit if necessary. Scaffolding does not implement the complete role, ownership, and tenant-isolation model.

## 3. Users and permissions

| Role | Capabilities | Boundaries |
|---|---|---|
| Customer | Browse, order, pay, apply coupons, manage addresses, save favorites, review completed orders | Own orders, documents, and personal data only |
| Rider | Manage availability, receive offers, accept tasks, record pickup/delivery, view earnings | Eligible offers and assigned delivery information only |
| Restaurant operator | Manage profile, menu, availability, coupons, orders, financial reports | Explicitly assigned restaurants only |
| Administrator | Approve accounts, manage settings, intervene in orders, manage refunds/reconciliation | Permission-controlled sensitive actions with audit records |

### Identity rules

- A person may hold multiple roles; role switching must not grant implicit permissions.
- Restaurant access requires explicit membership.
- Restaurants and riders require approval before operational use.
- Public registration never grants administrator privileges.
- Administrator access requires multi-factor authentication.
- Suspension prevents new operational actions while preserving history.
- Authorization is enforced server-side, not merely through hidden UI elements.
- Separate login screens do not require four unrelated identity databases.

## 4. Core user journeys

### 4.1 Customer ordering

1. Enter or select a delivery address.
2. Discover restaurants serving that address.
3. Choose products and modifiers.
4. Add items to the cart.
5. Review discounts, fees, taxes, total, and estimated timing.
6. Apply an eligible coupon.
7. Confirm address and payment.
8. Receive restaurant acceptance and preparation estimate.
9. Follow rider assignment and fulfillment milestones.
10. Receive the order.
11. Access the order document and submit an eligible review.

Selecting another restaurant while the cart contains items must prompt for replacement. Never silently remove the existing cart.

### 4.2 Restaurant operations

1. Submit restaurant details for approval.
2. Configure hours, menu, prices, preparation estimates, and availability.
3. Receive a new-order alert.
4. Accept or reject within the configured response period.
5. Prepare accepted orders.
6. Mark ready for pickup.
7. Confirm handoff and follow delivery status.
8. Review completed orders and settlement information.

### 4.3 Rider delivery

1. Sign in and check in as available.
2. Receive an offer with pickup/delivery areas, estimated distance, and compensation.
3. Accept or decline before expiry.
4. Travel to the restaurant and record arrival.
5. Verify the order reference and confirm pickup.
6. Travel to the customer.
7. Confirm delivery using the approved completion method.
8. Become available again or check out.

Riders accept delivery assignments, not the restaurant's decision to accept a food order.

### 4.4 Operational exception

1. Administrator receives an alert for acceptance, payment, dispatch, or delivery problems.
2. Review order, payment, and delivery history.
3. Perform an authorized intervention.
4. Record reason and outcome.
5. Notify affected users.

## 5. Functional requirements

### Priority definitions

- P0: Required for a controlled paid pilot.
- P1: Required for the full first release.
- P2: Later expansion.

### 5.1 Authentication and accounts

| ID | Requirement | Priority |
|---|---|---|
| AUTH-01 | Registration, login, logout, password reset, email verification | P0 |
| AUTH-02 | Role-appropriate dashboards and navigation | P0 |
| AUTH-03 | Server-side roles, ownership, and restaurant membership checks | P0 |
| AUTH-04 | Administrator approval and suspension of restaurants/riders | P0 |
| AUTH-05 | Administrator MFA and sensitive-action audit logs | P0 |
| AUTH-06 | Profile updates and saved addresses | P0 |
| AUTH-07 | Configurable restaurant staff permissions | P1 |

### 5.2 Discovery and restaurant profiles

| ID | Requirement | Priority |
|---|---|---|
| DISC-01 | Show restaurants eligible for the delivery location | P0 |
| DISC-02 | Show names, images, cuisines, hours, availability, fees, estimates | P0 |
| DISC-03 | Restaurant search and cuisine/open-status filters | P0 |
| DISC-04 | Menu categories, products, descriptions, prices, availability | P0 |
| DISC-05 | Save and remove favorite restaurants | P1 |
| DISC-06 | Approved review summaries and ratings | P1 |
| DISC-07 | Preserve filters/search across pagination | P1 |

Closed restaurants may remain discoverable. Disable immediate checkout unless scheduled ordering is explicitly added.

### 5.3 Menu and media

| ID | Requirement | Priority |
|---|---|---|
| MENU-01 | Create/edit categories and products | P0 |
| MENU-02 | Base prices, temporary sale prices, availability | P0 |
| MENU-03 | Required/optional modifiers and selection limits | P0 |
| MENU-04 | Single profile/cover images and multiple product images | P0 |
| MENU-05 | Validate upload type, size, and ownership | P0 |
| MENU-06 | Pause ordering and mark products sold out | P0 |
| MENU-07 | CSV/Excel menu imports with preview, validation, errors | P1 |
| MENU-08 | Authorized menu exports | P1 |

Accommodate ingredient, allergen, and dietary information. Confirm mandatory launch-market fields before release.

### 5.4 Custom cart and checkout

| ID | Requirement | Priority |
|---|---|---|
| CART-01 | Custom cart without an external shopping-cart package | P0 |
| CART-02 | Persist authenticated carts across sessions | P0 |
| CART-03 | Quantities, modifiers, removals, order notes | P0 |
| CART-04 | One restaurant per cart | P0 |
| CART-05 | Server-side price and eligibility recalculation | P0 |
| CART-06 | Revalidate stock availability, restaurant status, zone, minimum order | P0 |
| CART-07 | Itemized totals before payment confirmation | P0 |
| CART-08 | Prevent duplicate submissions/payment attempts | P0 |
| CART-09 | Require reconfirmation when quoted totals change | P0 |

#### Pricing contract

Distinguish product/modifier subtotal, item discounts, coupon discount, delivery fee, optional service fee, taxes or included-tax disclosures, optional tip, and final payable amount.

Snapshot prices, selected modifiers, tax treatment, and discount allocations into the order. Menu edits must not change historical transactions.

### 5.5 Coupons and dynamic discounts

| ID | Requirement | Priority |
|---|---|---|
| PROMO-01 | Administrator-created platform-funded coupons | P1 |
| PROMO-02 | Restaurant-created coupons scoped to owned restaurants | P1 |
| PROMO-03 | Percentage and fixed discounts | P1 |
| PROMO-04 | Validity, minimum spend, global/per-customer usage limits | P1 |
| PROMO-05 | Record funding party and discount allocation | P1 |
| PROMO-06 | Atomic usage-limit enforcement | P1 |

Initial rules:

- One coupon per order.
- Apply sale prices before coupons.
- Apply coupons to eligible food subtotal, not fees/tips, unless configured otherwise.
- Never reduce eligible charges below zero.
- Allow maximum discount caps on percentage coupons.
- Release cancelled-order redemptions according to documented policy.

### 5.6 Restaurant orders

| ID | Requirement | Priority |
|---|---|---|
| ORDER-01 | Live incoming-order queue and durable alerts | P0 |
| ORDER-02 | Accept/reject with reason | P0 |
| ORDER-03 | Update preparation estimate | P0 |
| ORDER-04 | Preparing and ready-for-pickup milestones | P0 |
| ORDER-05 | Configurable unaccepted-order escalation | P0 |
| ORDER-06 | Append-only status/actor history | P0 |
| ORDER-07 | Historical searches and filters | P1 |

Orders must remain visible after refreshes and notification or real-time connection failures.

### 5.7 Rider operations

| ID | Requirement | Priority |
|---|---|---|
| RIDER-01 | Approved onboarding and active/inactive status | P0 |
| RIDER-02 | Availability check-in/check-out | P0 |
| RIDER-03 | Offers with expiry, route context, compensation | P0 |
| RIDER-04 | Atomic single-rider assignment | P0 |
| RIDER-05 | Arrival, pickup, en-route, completion milestones | P0 |
| RIDER-06 | Decline, reassignment, incident reporting | P0 |
| RIDER-07 | Administrator assignment/reassignment | P0 |
| RIDER-08 | Delivery history and earnings breakdown | P1 |
| RIDER-09 | Continuous location tracking | P2 |

Pilot riders may have only one active delivery. Checking out stops new offers but does not abandon an active task.

### 5.8 Payments, refunds, settlements

| ID | Requirement | Priority |
|---|---|---|
| PAY-01 | One approved live online gateway | P0 |
| PAY-02 | Adapter contract for additional gateways | P0 |
| PAY-03 | Payment confirmation through verified provider events | P0 |
| PAY-04 | Idempotent payment/event processing | P0 |
| PAY-05 | Failed-payment recovery without duplicate charges/orders | P0 |
| PAY-06 | Authorized full/partial refunds with audit history | P0 |
| PAY-07 | Record commissions, compensation, discounts, fees, refunds, balances | P0 |
| PAY-08 | Settlement reports and controlled payout process | P0 |
| PAY-09 | Reconcile internal records against gateway records | P0 |
| PAY-10 | Activate a second gateway | P1 |

Orders, payments, deliveries, and payouts are separate linked records with independent lifecycles. Browser redirects alone must not establish payment success.

Approve before implementation:

- Seller and invoice issuer.
- Whether the platform collects on behalf of restaurants.
- Authorization and capture timing.
- Restaurant/rider financial onboarding.
- Commission basis and rounding.
- Discount and refund funding.
- Settlement timing, reserves, dispute handling.

These are launch-blocking decisions.

### 5.9 Reviews and favorites

| ID | Requirement | Priority |
|---|---|---|
| REVIEW-01 | Reviews only after delivered orders | P1 |
| REVIEW-02 | One review per eligible customer order | P1 |
| REVIEW-03 | Rating from 1–5 and optional text | P1 |
| REVIEW-04 | Administrator moderation with recorded reason | P1 |
| REVIEW-05 | Aggregates from eligible visible reviews | P1 |
| FAV-01 | Authenticated favorite management | P1 |

Restaurants cannot edit customer ratings or directly delete unfavorable reviews.

### 5.10 Notifications

| ID | Requirement | Priority |
|---|---|---|
| NOTIF-01 | Verification and password-reset emails | P0 |
| NOTIF-02 | Customer confirmation, rejection, cancellation, completion alerts | P0 |
| NOTIF-03 | Restaurant order and operational alerts | P0 |
| NOTIF-04 | Rider offer and assignment alerts | P0 |
| NOTIF-05 | Retries and operational visibility of persistent failures | P0 |
| NOTIF-06 | Templates and appropriate event preferences | P1 |

In-app operational alerts are mandatory. Email is supplementary, not the sole order-acceptance or dispatch channel.

### 5.11 Documents and data tools

| ID | Requirement | Priority |
|---|---|---|
| DOC-01 | Downloadable PDFs from immutable order data | P1 |
| DOC-02 | Approved issuer details, items, taxes, discounts, fees, currency, identifiers | P1 |
| DOC-03 | Role/ownership-controlled document access | P1 |
| DATA-01 | CSV/Excel imports for approved menu fields | P1 |
| DATA-02 | Dry-run validation, row errors, import summaries | P1 |
| DATA-03 | Scoped menu/order/settlement exports | P1 |
| DATA-04 | Sensitive-export audit and spreadsheet-content protection | P1 |
| PAGE-01 | Reusable pagination preserving sorting, filters, size limits | P1 |

Custom pagination means a consistent UI and reusable components, not replacing the framework's underlying pagination behavior.

## 6. State machines

### 6.1 Order lifecycle

```text
Awaiting payment
  -> Pending restaurant acceptance
    -> Accepted
      -> Preparing
        -> Ready for pickup
          -> Picked up
            -> Delivered
```

Alternative outcomes:

```text
Awaiting payment -> Payment failed / Expired
Pending acceptance -> Rejected / Timed out / Cancelled
Accepted or later -> Controlled cancellation / Operational exception
```

### 6.2 Payment lifecycle

```text
Initiated -> Pending -> Authorized or Captured
                    -> Failed
                    -> Cancelled

Captured -> Partially refunded -> Refunded
Captured -> Disputed
```

Authorization/capture behavior follows the approved payment model and gateway.

### 6.3 Delivery lifecycle

```text
Unassigned -> Offered -> Assigned -> At restaurant
  -> Picked up -> En route -> Delivered
```

Exceptions include offer expiry, decline, reassignment, failed delivery, and administrator intervention.

### Transition rules

- Record actor, timestamp, previous state, and reason where applicable.
- Reject invalid transitions server-side.
- Restaurant rejection starts appropriate void/refund handling.
- Payment success alone cannot mark an order delivered.
- Cancellation cannot imply a completed refund before provider confirmation.
- Recover safely from late or out-of-order payment events.

## 7. Cancellation and exceptions

| Scenario | Proposed behavior |
|---|---|
| Customer cancels before acceptance | Cancel and start void/refund handling |
| Restaurant rejects or times out | Notify customer; release/refund payment |
| Product unavailable before acceptance | Reject; substitutions out of scope |
| Cancellation requested after acceptance | Authorized operational decision |
| No rider accepts | Alert operations, reassign, cancel at approved cutoff |
| Rider fails before pickup | Reassign; retain history |
| Incident after pickup | Open incident; do not automatically redispatch the same food |
| Customer unreachable | Follow contact and failed-delivery procedure |
| Refund fails | Pending/failed status, safe retry, operations alert |

Approve timeouts, compensation, refund entitlements, and food-remake policies before launch.

## 8. Key screens

| Area | Screens |
|---|---|
| Customer P0 | Discovery, restaurant menu, cart, checkout, payment outcome, tracking, history/detail, profile, addresses |
| Customer P1 | Favorites, review submission, PDF access |
| Restaurant P0 | Onboarding, dashboard, incoming orders, detail, menu editor, hours/availability, profile |
| Restaurant P1 | Coupons, imports/exports, financial reports |
| Rider P0 | Approval status, availability, offers, active delivery, incident reporting |
| Rider P1 | History and earnings |
| Administrator P0 | Operations, approvals, users, restaurants, orders, dispatch, payments, refunds, configuration, audit |
| Administrator P1 | Moderation, promotions, exports, settlement reporting |

Include loading, empty, error, stale-data, and permission-denied states.

## 9. Technical architecture

### Proposed application structure

Use a modular Laravel application with:

- Inertia.js and Vue.js role-specific experiences.
- A relational transactional database.
- Queue workers for notifications, documents, imports, and background work.
- Shared cache and locks for concurrency-sensitive operations.
- Object storage for images and documents.
- Private real-time channels for order/delivery updates.
- Payment, email, maps, and geocoding adapters.
- Central logs, metrics, alerts, and error tracking.

### Domain modules

1. Identity and Access.
2. Restaurants and Memberships.
3. Catalog and Availability.
4. Cart and Pricing.
5. Orders.
6. Payments and Accounting.
7. Dispatch and Delivery.
8. Promotions.
9. Reviews and Favorites.
10. Notifications and Documents.
11. Administration and Reporting.

### Constraints

- Keep business rules in application/domain services, not Vue components.
- Treat frontend totals as informational until server validation.
- Make background retries safe.
- Do not roll back valid orders because notifications fail.
- Protect assignment and payment invariants using constraints and locking.
- Make external-event processing idempotent.
- Apply restaurant isolation to queries, downloads, exports, and real-time subscriptions.
- Store monetary values using currency-aware integer units, not floating-point arithmetic.

### Core entities

| Domain | Entities |
|---|---|
| Identity | User, Role, Permission, RestaurantMembership |
| Customer | Address, Favorite |
| Restaurant | Restaurant, Location, OpeningHours, DeliveryZone |
| Catalog | Category, Product, ModifierGroup, ModifierOption, Media |
| Commerce | Cart, CartItem, CheckoutQuote, Order, OrderItem |
| Promotions | Coupon, Redemption, DiscountAllocation |
| Payments | PaymentAttempt, PaymentEvent, Refund |
| Accounting | LedgerEntry, CommissionRule, Settlement, Payout |
| Delivery | RiderProfile, AvailabilitySession, DeliveryTask, DeliveryOffer, DeliveryEvent |
| Trust | Review, ModerationAction |
| Operations | Notification, AuditLog, ImportJob, ExportJob, Document |

## 10. Non-functional requirements

All numerical targets are proposed and require validation against an agreed load profile.

| Area | Target or requirement |
|---|---|
| Availability | 99.5% monthly during pilot |
| Standard reads | P95 server response below 750 ms under agreed load |
| Checkout | P95 internal processing below 2 seconds, excluding external payment interaction |
| Freshness | Order/delivery updates visible within 5 seconds normally |
| Accessibility | Target WCAG 2.2 AA on critical journeys |
| Responsive design | Supported mobile browsers for customer, restaurant, rider |
| Security | Server authorization, rate limits, CSRF protection, secure sessions, upload validation |
| Privacy | Minimized collection; role/need-based access |
| Recovery | Proposed RPO 1 hour and RTO 4 hours, rehearsed |
| Observability | Correlated order, payment, delivery, and job identifiers |
| Integrity | No duplicate successful capture or active assignment in acceptance tests |
| Time | Consistent timestamp storage; operating-timezone display |
| Testing | Critical permissions, pricing, state transitions, payments automated |

Proposed pilot load-test baseline: 100 concurrent customer sessions, 20 restaurant sessions, and 50 active rider sessions. These are test assumptions, not proven capacity.

## 11. Analytics and success measures

### Primary outcome

Completed paid orders delivered within the customer-promised delivery window.

### Supporting metrics

| Area | Metrics |
|---|---|
| Acquisition | Visits resulting in restaurant views |
| Conversion | Menu-to-cart; checkout-to-paid-order |
| Restaurant operations | Acceptance rate and time |
| Preparation | Actual versus estimated preparation |
| Dispatch | Assignment time and offer acceptance |
| Delivery | Pickup wait, duration, on-time rate |
| Reliability | Payment failures, duplicate events, failed jobs, unresolved exceptions |
| Customer experience | Cancellations, refunds, ratings, repeat orders |
| Economics | Gross order value, revenue, discount spend, payment costs, contribution per order |

Approve definitions for gross order value, revenue, and contribution so customer spending is not confused with platform earnings.

### Proposed pilot targets

- At least 90% of eligible orders accepted within 2 minutes.
- At least 90% of delivered orders within the displayed delivery window.
- Fewer than 2% of paid orders cancelled for platform-controlled operational reasons.
- No unexplained payment/order reconciliation differences after next-business-day review.
- Zero critical restaurant-isolation or duplicate-payment defects.

Recalibrate targets using observed pilot demand and delivery conditions.

## 12. Acceptance criteria

| ID | Scenario | Pass condition |
|---|---|---|
| AC-01 | Restaurant isolation | Operator A cannot access Restaurant B orders, edits, exports, documents, private channels |
| AC-02 | Stale cart price | Server recalculates; customer reconfirms before charging |
| AC-03 | Duplicate checkout | At most one confirmed order and successful capture per logical purchase |
| AC-04 | Duplicate provider event | Financial and operational effects applied once |
| AC-05 | Paid order rejection | Accurate customer rejection/refund status; no payout released |
| AC-06 | Concurrent rider acceptance | Only one assigned rider |
| AC-07 | Unauthorized pickup | Unassigned rider cannot mark pickup or delivery |
| AC-08 | Last coupon redemption | Concurrent checkouts cannot exceed usage limit |
| AC-09 | Partial refund | Payment, restaurant balance, commission, allocation remain policy-consistent |
| AC-10 | Notification failure | Valid order remains visible and operational |
| AC-11 | Historical document | Issued transaction details unchanged by later menu/profile edits |
| AC-12 | Invalid import rows | Row errors shown; no silent unintended partial update |

## 13. Delivery roadmap

### Staffing and scope assumptions

- Three experienced full-stack engineers.
- Fractional product, design, QA, and DevOps support.
- Responsive web, one market/currency/gateway.
- Basic dispatch and manual operational fallback.
- No continuous GPS, cross-restaurant basket, native apps, or POS integration.

| Phase | Scope | Calendar estimate |
|---|---|---|
| 0 | Discovery: funds flow, fulfillment, permissions, UX, technical validation | 2–3 weeks |
| 1 | Foundation: environments, auth, authorization, onboarding, catalog | 3–4 weeks |
| 2 | Commerce: cart, pricing, checkout, payment events, restaurant workflows | 5–6 weeks |
| 3 | Delivery: availability, offers, assignment, milestones, operations | 4–6 weeks |
| 4 | Full V1: coupons, reviews, favorites, PDFs, imports/exports, reports | 3–5 weeks |
| 5 | Hardening: security, reconciliation, recovery, load tests, training | 3–4 weeks |

Indicative total: 20–28 calendar weeks with overlapping activities.

A controlled P0 pilot may be possible in approximately 16–20 weeks only when financial controls, dispatch fallback, monitoring, and critical acceptance criteria are complete. Do not accelerate launch by deferring payment or authorization safeguards.

These are rough-order-of-magnitude estimates, not commitments. Provider onboarding, tax review, incomplete designs, and fulfillment changes can materially alter them.

## 14. Risks

| Risk | Impact | Mitigation |
|---|---|---|
| Cross-restaurant cart added mid-build | Reworks checkout, refunds, settlement, delivery | Keep P2 or rebaseline |
| Unresolved payment model | Onboarding/invoicing/accounting rework | Approve in discovery |
| Insufficient riders | Unfulfilled paid orders | Restrict zones/hours, manual dispatch |
| Missed restaurant orders | Rejections/refunds | Persistent alerts, timeouts, training |
| Stale availability | Payment for unavailable food | Checkout revalidation, sold-out controls |
| Data leakage | Privacy/commercial exposure | Ownership rules, isolation tests |
| Incorrect settlements | Loss/disputes | Immutable records, reconciliation, payout review |
| Harmful imports | Menu disruption | Dry-run validation, deliberate commit |
| Browser interruptions | Missed offers/stale status | Durable server state, reconnect, polling fallback |

## 15. Launch gates

- [ ] Seller, invoicing, payment, refund, and settlement model approved.
- [ ] Restaurants and riders onboarded and trained.
- [ ] Zones, fees, hours, exception policies configured.
- [ ] Order, payment, rejection, refund, assignment, delivery tests passed.
- [ ] Critical authorization and isolation tests passed.
- [ ] Unaccepted-order and dispatch-failure detection operational.
- [ ] Reconciliation and payout controls tested.
- [ ] Backup, restore, rollback, monitoring, alerting rehearsed.
- [ ] Support ownership and escalation documented.
- [ ] Required privacy, terms, food-information, and operating-policy reviews complete.
- [ ] No unresolved critical or high-severity launch-blocking defects.

## 16. Decisions required

| Decision | Proposed default |
|---|---|
| Checkout scope | One restaurant per checkout |
| Delivery responsibility | Platform-managed riders |
| Native rider app | Responsive web first |
| Continuous tracking | Deferred; milestones first |
| Guest checkout | Deferred; account-based ordering |
| Cash on delivery | Deferred |
| Scheduled orders | Deferred |
| Substitutions | Deferred; reject/refund initially |
| Gateways | One for pilot; second in full V1 |
| Dispatch | Basic offers and manual fallback |
| Restaurant staff | Explicit memberships; granular P1 permissions |
| Framework version | Select one after compatibility validation |
| Seller/invoice issuer | Resolve in discovery |
| Restaurant/rider payout model | Resolve before payment implementation |

## 17. Repository readiness

This document specifies planned behavior. It does not imply that application code, database migrations, CI, infrastructure, or deployments already exist.

Before adding installation instructions, record and verify:

- Selected Laravel, PHP, Node.js, Vue.js, and Inertia.js versions.
- Database, cache, queue, storage, and real-time providers.
- Authentication starter kit and authorization approach.
- Local development commands and required services.
- Environment-variable template without credentials.
- Migration, seed, build, worker, scheduler, and test commands.
- CI checks and deployment/rollback procedures.
- Maintainers, contribution process, and approved license.

Do not commit secrets, payment credentials, customer exports, or production environment files.

### Definition of done

A requirement is complete when its acceptance criteria pass, authorization is enforced, automated tests cover critical behavior, failure states are handled, required monitoring exists, and relevant operational documentation is updated.

### Product boundary

This platform combines a restaurant marketplace with a delivery operations system. Each needs explicit workflows, permissions, state machines, and acceptance tests. Neither should be treated as an incidental extension of the other.
