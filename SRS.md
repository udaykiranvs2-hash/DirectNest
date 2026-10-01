# Software Requirements Specification

## Direct-to-Owner Multilingual Property Marketplace (working name: "DirectNest")

| Field | Value |
| --- | --- |
| Version | 1.0 (Draft for review) |
| Date | 1 October 2026 |
| Market | India (initial); architecture is geography-neutral |
| Languages | English, Telugu, Hindi (extensible) |
| Status | Requirements baseline. No source code, SQL or UI code is included. |

> **Legal/compliance notice.** Items marked **\[LEGAL\]** have regulatory implications (DPDP Act 2023, IT Act 2000, state rent-control and stamp-duty laws, RERA, e-signature rules, RBI payment rules, Fair-housing/anti-discrimination law, tourism/hotel regulation). They are flagged for legal review and are not legal advice.

---

# 1. Executive Summary

DirectNest is a mobile-first marketplace where customers discover, evaluate, and transact for properties **directly with verified owners, with no mandatory broker**. Residential renting is the primary use case. The same reusable workflows extend to PG/hostel/co-living, hotels, hourly and short stays, commercial space, and land (rent, lease, sale).

**Differentiators:** (1) verified owners and properties, (2) a **freshness system** that keeps stale listings out of results, (3) a **true-cost** view (monthly and move-in), (4) a **grounded AI assistant** that never invents data, (5) map and nearby-facility discovery, (6) full in-app flow from chat to visit to application to agreement to payment, (7) English/Telugu/Hindi support throughout.

**MVP (P0)** covers authentication, owner verification, listing creation, search/filters/map, nearby facilities, property detail, favourites, chat, visits, reporting, reviews, multilingual UI, admin verification, a basic rental application flow, and the AI assistant. Payments, e-agreements, hotels, commercial and land transactions, and roommate matching follow in Phases 2 and 3 (see §42–45).

**Key risks:** fake listings, regulatory exposure (privacy, e-sign, payments), location/POI data quality, and AI hallucination. Mitigations are in §46.

---

# 2. Product Vision

A trusted, broker-free way for anyone in India to find and secure a place to live, work, or stay, in their own language, with confidence that what they see is real, current, and priced honestly.

# 3. Problem Statement

| # | Problem | Platform response |
| --- | --- | --- |
| P1 | Brokers add cost and friction | Direct owner–customer chat, visits, applications |
| P2 | Fake or already-rented listings | Verification + freshness system + reporting |
| P3 | Advertised rent hides true cost | True Cost Calculator |
| P4 | Non-English users underserved | Telugu/Hindi first-class localization |
| P5 | Hard to judge location quality | Map, nearby facilities, commute filters |
| P6 | Fragmented process | One flow: discover → visit → apply → agree → pay |

# 4. Goals and Objectives

| ID | Goal | Measure (target, configurable) |
| --- | --- | --- |
| G-01 | Trustworthy supply | ≥ 95% of public listings verified; fake-listing report rate \< 1% of views-to-contact |
| G-02 | Fresh inventory | ≥ 90% of public listings confirmed within the last 30 days |
| G-03 | Efficient discovery | Median time from first search to first enquiry \< 10 min |
| G-04 | Localization | 100% of customer-facing strings available in en/te/hi at release |
| G-05 | Reliability | 99.9% monthly availability for core read paths |
| G-06 | Responsiveness | Median owner first response \< 4 hours |

# 5. Scope

**In scope:** all categories in §10; roles in §8; discovery, AI, verification, visits, chat, applications, agreements, payments, bookings, reviews, reports, support, notifications, analytics, admin tooling, multilingual, security, privacy, observability.

# 6. Out-of-Scope Items

- Physical inspection services performed by platform staff (Phase 3 option).
- Mortgage/loan origination, tenant insurance, moving, furniture rental, property management (future roadmap, §45).
- Binding legal, financial, or valuation advice (AI and calculators are informational only).
- Native desktop apps. Source code, SQL, and UI code (excluded from this document).

# 7. Target Users

| Persona | Description | Key needs |
| --- | --- | --- |
| Family tenant | Relocating household | Verified homes, schools/hospitals nearby, true cost |
| Student | Budget-sensitive, shared stay | PG/room, distance to college, roommates |
| Working professional | Commute-driven | Metro/workplace distance, furnished |
| Small business | Shop/office seeker | Footfall, zoning, lease terms |
| Traveller | Short/hourly stay | Instant availability, clear cancellation |
| Buyer/investor | Land or property | Documents, location, price |
| Owner/landlord | Individual or small portfolio | Qualified leads, low effort, payment safety |
| Employees/Admins | Trust & safety, support | Efficient queues, audit trail |

---

# 8. User Roles and Permissions

## 8.1 Role definitions (IDs ROLE-xxx)

| ID | Role | Summary |
| --- | --- | --- |
| ROLE-01 | Customer/Tenant | Discover, communicate, visit, apply, book, pay, review, report |
| ROLE-02 | Owner/Landlord | Verify, list, manage enquiries, visits, applications, bookings, payouts |
| ROLE-03 | Verification Employee | Review owners, documents, listings, media, freshness |
| ROLE-04 | Support Employee | Tickets, disputes, escalations |
| ROLE-05 | Moderation Employee | Reports, fraud, content, reviews, account restrictions |
| ROLE-06 | Admin | Operational management of platform data and queues |
| ROLE-07 | Super Admin | Role/employee management, security and platform configuration |

A user may hold both Customer and Owner roles (one account, role switch). Employee roles are separate accounts and cannot be self-registered.

## 8.2 Role-Permission Matrix

Legend: C=create, R=read, U=update, D=delete/archive, A=approve/decide, O=own records only, —=no access.

| Capability | Customer | Owner | Verif. | Support | Moder. | Admin | Super Admin |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Register/login/profile | CRUD(O) | CRUD(O) | R(O) | R(O) | R(O) | R(O) | R(O) |
| Search/browse public listings | R | R | R | R | R | R | R |
| Create/edit listing | — | CRU(O) | R, U(limited: flag) | R | R, U(hide) | CRUD | CRUD |
| Approve/reject listing | — | — | A | — | — | A (override) | A |
| Owner/doc verification | — | submit(O) | A | — | — | A (override) | A |
| View ownership documents | — | R(O) | R (assigned) | — | R (if flagged) | R (audited) | R (audited) |
| Favourites/collections/comparison | CRUD(O) | — | — | — | — | — | — |
| Chat | CRUD(O) | CRUD(O) | — | R (with consent/ticket) | R (reported only) | R (audited) | R (audited) |
| Visits | CRU(O) | CRUA(O) | — | R | — | R | R |
| Rental applications | CRU(O) | RUA(O) | — | R | — | R | R |
| Agreements | R, sign(O) | CRU, sign(O) | — | R | — | R | R |
| Payments/refunds | C, R(O) | R(O) | — | R (masked) | — | R, refund A | R, config |
| Reviews | CRU(O) | R, respond | — | R | R, A (moderate) | RA | RA |
| Reports | C, R(O) | R (against own) | R | R | CRUA | RA | RA |
| Support tickets | CR(O) | CR(O) | — | CRUA | R | RA | RA |
| Categories/amenities/locations | R | R | R | R | R | CRUD | CRUD |
| Analytics (own listings) | — | R(O) | — | — | — | — | — |
| Platform analytics | — | — | R(queue) | R(queue) | R(queue) | R | R |
| Monetization config | — | R(own plan) | — | — | — | CRU | CRUD |
| Employee/role management | — | — | — | — | — | R | CRUD |
| Security/platform settings | — | — | — | — | — | R | CRUD |
| Audit logs | R(own activity) | R(own activity) | R(own) | R(own) | R(own) | R | R |

**Requirements**

- **AUTH-RBAC-001 (P0):** All access is enforced server-side via role- and attribute-based checks (ownership, assignment, status). UI hiding is never the sole control.
- **AUTH-RBAC-002 (P0):** Admin overrides of verification decisions require a reason and produce an audit record.
- **AUTH-RBAC-003 (P0):** Employees may access personal data only for assigned cases; every access to documents is logged.
- **AUTH-RBAC-004 (P0):** Super Admin actions (role grants, security settings) require MFA re-authentication and are dual-logged.
- **AUTH-RBAC-005 (P1):** Support for custom employee roles with granular permissions (permission sets configurable by Super Admin).

---

# 9. Product Architecture Overview

Modular, API-first, vendor-neutral, horizontally scalable.

| Layer | Components |
| --- | --- |
| Clients | Responsive PWA web app (mobile-first); Android/iOS apps (Phase 2, or wrapper in MVP); Employee/Admin console |
| Edge | CDN, WAF, API gateway (auth, rate limit, routing) |
| Core services | Identity & Access; User/Profile; Owner Verification; Property/Listing; Media; Freshness; Search & Ranking; Geo/Map; Nearby Facilities; AI Assistant; Matching; Chat; Visits; Applications; Agreements; Booking; Payments & Ledger; Reviews; Reports/Moderation; Support; Notifications; Analytics; Admin/Config; Audit; Localization |
| Data | Transactional DB; search index; geospatial index; object storage (public media, private documents); cache; message queue/event bus; data warehouse (analytics) |
| Integration adapters | Map/geocoding, POI, payments, SMS/OTP, email, push, identity verification, e-signature, LLM, cloud storage, malware scanning, image moderation |
| Cross-cutting | Observability, secrets, feature flags, config, i18n, audit |

**Architectural requirements**

- **ARCH-001 (P0):** Every third-party capability is accessed through an adapter interface with at least a defined contract, timeout, retry, circuit breaker, and fallback. No vendor named in business logic.
- **ARCH-002 (P0):** Services communicate via versioned APIs and asynchronous events; state changes publish domain events (e.g., `ListingVerified`, `PaymentConfirmed`).
- **ARCH-003 (P0):** Data is partitionable by region/country (`region_id`) from day one.
- **ARCH-004 (P0):** A modular monolith with clear module boundaries is acceptable for MVP provided boundaries map to the services above. *(Assumption A-04)*
- **ARCH-005 (P0):** All user-facing text, formats, and currencies are resolved via a localization layer.
- **ARCH-006 (P1):** Feature flags control category enablement per city/region.

---

# 10. Property Taxonomy

## 10.1 Hierarchy

| Group | Types | Transaction modes |
| --- | --- | --- |
| Residential | Apartment, Flat, Independent house, Villa, Studio, Room, Shared room; BHK variants (1/2/3/4+) as attribute | Rent, Lease, Sale |
| Student/Shared | PG, Hostel, Co-living, Student housing, Shared accommodation | Rent (per bed/room), Short-term |
| Short stay | Hotel, Room booking, Hourly stay, Vacation stay, Short-term rental | Booking |
| Commercial | Shop, Office, Warehouse, Commercial building, Retail space, Business space | Rent, Lease, Sale |
| Land | Residential plot, Commercial plot, Agricultural land, Other eligible land | Sale, Lease (where legal) |

- **TAX-001 (P0):** Taxonomy is data-driven (admin-managed), versioned, and supports parent/child types and category-specific attribute sets.
- **TAX-002 (P0):** Each category definition declares: required fields, optional fields, filters, allowed transaction modes, verification documents, workflow template, and pricing model.
- **TAX-003 (P1):** A listing may contain multiple **units** (e.g., commercial building with several shops, PG with rooms/beds).
- **TAX-004 (P0):** Agricultural land listings are enabled only for regions where legal review confirms permitted flows **\[LEGAL\]** (agricultural land transfer restrictions vary by state).

## 10.2 Category applicability matrix

| Aspect | Residential | PG/Hostel/Co-living | Short stay | Commercial | Land |
| --- | --- | --- | --- | --- | --- |
| Bedrooms/bathrooms | ✔ | ✔ (per room) | ✔ | — | — |
| Furnishing/amenities | ✔ | ✔ | ✔ | Partial | — |
| Occupancy rules | ✔ | ✔ (gender/occupancy) | Guests | — | — |
| Pricing | Rent, deposit, maintenance | Per bed/room, deposit | Hourly/daily, taxes | Rent/lease/sale, CAM | Sale/lease per unit area |
| Availability | Move-in date | Beds available | Calendar/slots | Date | Status only |
| Visits | ✔ | ✔ | Optional | ✔ | Site visit |
| Application | ✔ | ✔ | — | ✔ (EOI) | Enquiry/EOI |
| Agreement | Rental | Rental/PG | Booking terms | Lease | Sale-intent (not a sale deed) |
| Payment | Deposit, rent (P2) | Deposit, rent | Full/part | Token/deposit | Token (optional) |
| Reviews | ✔ | ✔ | ✔ | Optional | — |
| Extra docs | Ownership | Ownership, PG licence where applicable **\[LEGAL\]** | Trade/GST/tourism registration **\[LEGAL\]** | Ownership, NOC | Title, survey, encumbrance **\[LEGAL\]** |

---

# 11. Functional Requirements

Priority: P0 MVP, P1 Phase 2, P2 Phase 3 enhancement, P3 future. All features are retained; priority is sequencing only.

## 11.1 Authentication & Account (AUTH)

| ID | Requirement | Pri |
| --- | --- | --- |
| AUTH-001 | Register by phone (OTP) or email (verification link/OTP). Phone is primary for India. | P0 |
| AUTH-002 | Password login with salted adaptive hashing (e.g., Argon2id/bcrypt); passwordless OTP login optional. | P0 |
| AUTH-003 | Social login (Google; others via adapter) with account linking and verified-email check. | P1 |
| AUTH-004 | OTP: 6 digits, 5-min expiry, single use, max 5 attempts, resend cooldown 30 s, max 5 sends/hour/identifier. | P0 |
| AUTH-005 | Sessions: short-lived access token + rotating refresh token; device list; revoke sessions; idle timeout configurable (default 30 days mobile, 30 min admin). | P0 |
| AUTH-006 | Account recovery via verified phone/email with cooldown; recovery triggers security notification. | P0 |
| AUTH-007 | Suspicious-login detection (new device, impossible travel, velocity); step-up OTP, notify user. | P1 |
| AUTH-008 | Account deletion request with 14-day grace, then erasure/anonymization per retention rules (§30). Blocked while active bookings, payments or disputes exist, with clear explanation. | P0 |
| AUTH-009 | MFA mandatory for employees/admins; optional for owners (recommended before payout setup). | P0 |
| AUTH-010 | Onboarding: language, city/location prefs, property prefs, budget, move-in date, optional employment/student info, privacy prefs. All optional except language; skippable. | P0 |
| AUTH-011 | Account states: Pending → Active → Restricted → Suspended → Deactivated → Deleted. | P0 |

## 11.2 Owner Onboarding & Verification (VER)

| ID | Requirement | Pri |
| --- | --- | --- |
| VER-001 | Owner profile: legal name, display name, phone/email verified, address, photo. | P0 |
| VER-002 | Identity verification via adapter (government ID, selfie/liveness where supported) or manual document review. Government ID numbers stored masked/encrypted; full numbers not displayed to anyone. **\[LEGAL\]** | P0 |
| VER-003 | Ownership documents per category (e.g., sale deed, property tax receipt, utility bill, society NOC, authorization letter for agents/power of attorney). Admin-configurable list. | P0 |
| VER-004 | Owner declares right to list (checkbox + timestamped legal declaration). | P0 |
| VER-005 | Bank/UPI details collected only when payouts are enabled; verified via penny-drop or adapter. | P1 |
| VER-006 | Owner verification states: Draft → Submitted → Under Review → More Information Required → Verified / Rejected → Suspended. Verified may expire (default 24 months) → Re-verification Required. | P0 |
| VER-007 | Property verification is separate from owner verification (states in §39). | P0 |
| VER-008 | Workflow: owner submits → auto-checks (format, duplicates, name match, blacklist) → queue → Verification Employee decides → rejections/escalations to Admin; Admin may override with reason. SLA default 48 h, configurable. | P0 |
| VER-009 | Rejection must include a reason code and plain-language message (localized); owner may resubmit (max configurable attempts, then support escalation). | P0 |
| VER-010 | Brokers/agents cannot list as owners; authorized representatives allowed with authorization document and labelled "Authorized Representative". | P1 |
| VER-011 | Multiple co-owners supported; at least one authorized signatory declared. | P1 |

## 11.3 Property Listing (PROP)

| ID | Requirement | Pri |
| --- | --- | --- |
| PROP-001 | Multi-step listing wizard with autosave drafts, resumable, per-step validation, mobile-friendly. | P0 |
| PROP-002 | Basic fields: title, type, category, transaction mode, description, address, locality, city, state, PIN, map pin (lat/long, adjustable), size, built-up, carpet, floor, total floors. | P0 |
| PROP-003 | Residential fields: bedrooms, bathrooms, balconies, kitchen, furnishing level (unfurnished/semi/fully, with item list), parking (2W/4W, count), property age, facing, water, electricity, power backup. | P0 |
| PROP-004 | Pricing: rent, sale price, lease amount, security deposit, maintenance, other charges, electricity/internet estimates (labelled estimates), booking/hourly/daily price, additional charges (each with label, amount, frequency, mandatory flag). | P0 |
| PROP-005 | Availability: available-from, occupancy, min/max stay, lease duration, move-in date, status (Available / Reserved / Rented / Unavailable). | P0 |
| PROP-006 | Rules: family, students, working professionals, bachelors, gender preference, pets, smoking, visitors, noise, curfew, food, parking, custom rules. | P0 |
| PROP-007 | Restrictions on religion, caste, ethnicity, marital status, disability or similar attributes are **not offered** as selectable rules; free-text and media are scanned for such language and blocked/flagged. Gender-based preference is allowed only where category rules permit (e.g., women-only PG/hostel) **\[LEGAL\]**. | P0 |
| PROP-008 | Commercial/land fields per §29 (zoning/use, frontage, plot size, expected rental income, documents). | P1 |
| PROP-009 | Listing may be saved as draft; submission triggers validation + verification pipeline (§41). | P0 |
| PROP-010 | Listing states: Draft → Submitted → Under Review → Changes Requested → Approved/Published → Paused → Rented/Sold/Unavailable → Stale → Hidden → Archived; Rejected; Suspended. | P0 |
| PROP-011 | Edits to material fields (price, address, area, type, ownership) on a published listing trigger re-review (configurable); minor edits publish immediately and are logged. | P0 |
| PROP-012 | Duplicate-listing detection (address/geo + area + media similarity); suspected duplicates held for review. | P0 |
| PROP-013 | Owners can clone a listing, pause, mark rented, relist, and archive. | P1 |
| PROP-014 | Multi-unit listings (building → units) with unit-level availability and price. | P1 |

## 11.4 Media (MEDIA)

| ID | Requirement | Pri |
| --- | --- | --- |
| MEDIA-001 | Images: JPEG/PNG/WebP/HEIC (converted); min 800×600; max 15 MB each; up to 30 images; videos MP4/MOV max 200 MB, up to 3, ≤ 3 min (limits configurable). | P0 |
| MEDIA-002 | Room/area tags (bedroom, bathroom, kitchen, exterior, building, amenity, nearby), caption, cover image, drag/reorder. | P0 |
| MEDIA-003 | Malware scan, file-type sniffing (not extension only), EXIF GPS stripped from public output, re-encoding. | P0 |
| MEDIA-004 | Moderation: nudity/violence, watermark/third-party-contact overlays, stock/stolen image detection (reverse/perceptual hash against platform and known sets where available), duplicate detection across listings/owners. | P0 |
| MEDIA-005 | Each asset stores uploaded-at, uploader, status (Pending/Approved/Rejected), derivatives (thumbnail/medium/large, adaptive video). | P0 |
| MEDIA-006 | Display "Last media update" on listing. At least N (default 5) approved images required to publish residential listings. | P0 |
| MEDIA-007 | Upload resumable on weak networks, with retry and progress; failures never lose other uploaded items. | P0 |

## 11.5 Freshness (FRESH)

Every listing tracks: `last_owner_confirmation_at`, `last_media_update_at`, `last_info_update_at`, `last_verified_at`, `last_availability_confirmation_at`.

| Indicator | Default rule (configurable per category/city) |
| --- | --- |
| Recently Updated | Any info/media update ≤ 14 days |
| Recently Verified | Platform verification ≤ 90 days |
| Needs Confirmation | No availability confirmation 15–29 days |
| Stale Listing | No confirmation ≥ 30 days |

| ID | Requirement | Pri |
| --- | --- | --- |
| FRESH-001 | Prompt owner "Is this property still available?" at day 14 (push/SMS/email/in-app), with one-tap Yes / Rented / Edit. | P0 |
| FRESH-002 | Reminder cadence: day 14, 21, 27 (configurable). | P0 |
| FRESH-003 | Escalation ladder: Warn (day 15) → Deprioritize in ranking (day 30) → Temporarily hide from search but link remains with "may be unavailable" notice (day 45) → Auto-archive (day 90). Owner can restore by confirming. | P0 |
| FRESH-004 | Hours-based rules for hotels/hourly stays: availability must be synchronized or confirmed daily (configurable). | P1 |
| FRESH-005 | Customer-reported "already rented" reports trigger immediate owner confirmation request; 3 independent reports within 72 h hides the listing pending confirmation. | P0 |
| FRESH-006 | Freshness status and dates are computed server-side and displayed on cards, detail pages, comparisons. | P0 |
| FRESH-007 | Freshness is a ranking input (§15, §55) but never overrides hard filters. | P0 |

## 11.6 Badges (BADGE)

| Badge | Meaning | Granted by |
| --- | --- | --- |
| Verified Owner | Identity verified and right-to-list documents accepted | Verification workflow only |
| Property Verified | Listing details and media reviewed; location confirmed (remote or on-site as configured) | Verification workflow only |
| Documents Verified | Ownership documents reviewed and accepted | Verification workflow only |
| Recently Updated | Updated within window (FRESH) | System |
| Recently Available | Availability confirmed within window | System |
| Platform Verified | All three verification badges valid and current | System |

- **BADGE-001 (P0):** Badges are system-derived; no owner-editable field or API can set them. Description text is scanned to block claims such as "verified" in titles.
- **BADGE-002 (P0):** Each badge shows a tooltip with its meaning, date granted, and expiry. Expired verification removes the badge automatically.

## 11.7 Owner Profile (OWN)

- **OWN-001 (P0):** Shows display name, verification status, number of properties, active listings, response rate and time (rolling 90 days), years on platform, ratings where eligible, contact options.
- **OWN-002 (P0):** Never shows legal name beyond display policy, government IDs, documents, bank details, personal phone/email (unless §24 reveal rules are met).
- **OWN-003 (P0):** Response rate = enquiries replied within 24 h ÷ enquiries received; response time = median first-response time (definitions fixed and shown in tooltip).

---

# 12. User Journeys

Each journey lists the main path; exception paths link to §37.

| # | Journey | Main flow |
| --- | --- | --- |
| J1 | Customer finds and rents a house | Onboard → search/AI → filter → view detail (true cost, nearby, freshness) → favourite/compare → chat → book visit → visit (checklist) → Apply to Rent → owner accepts → agreement → sign → deposit payment → move-in checklist → property status Rented |
| J2 | Room/PG | Search PG filters (gender, sharing, food) → per-bed pricing → visit → application → deposit → roommate option |
| J3 | Hotel | Search destination/dates/guests → select room type → price with taxes → pay → confirmation voucher → cancellation/refund per policy |
| J4 | Hourly stay | Select slot (e.g., 3/6/12 h) → availability lock (10-min hold) → pay → check-in code → auto-checkout reminder |
| J5 | Commercial lease | Search (zoning/business type) → enquiry → visit → Expression of Interest with business info → owner terms → lease agreement (legal review config) → deposit |
| J6 | Land purchase | Search plot filters → document view (shared on consent) → site visit → Offer/EOI → owner response → token amount optional → hand-off to legal sale process off-platform. Platform does **not** execute title transfer **\[LEGAL\]** |
| J7 | Owner creates and verifies | Register → ID verification → submit ownership docs → create listing → media → submit → review → published |
| J8 | Employee verification | Pick queue item → view docs/media/map → checklists → approve/reject/request info → audit |
| J9 | Report fake property | Detail → Report → category/evidence → Report ID → triage → action → reporter notified |
| J10 | AI assistant | Describe needs → clarifying questions → structured criteria shown → results with explanations → refine → save search |
| J11 | Compare | Add 2–4 → compare → highlight best-in-row → share/shortlist |
| J12 | Schedule visit | Pick slots → owner accepts/counters → reminders → check-in/out feedback |
| J13 | Digital agreement | Application accepted → agreement draft → both review → e-sign via adapter → stored |
| J14 | Payment | Select payable item → breakdown → gateway → webhook confirmation → receipt → ledger |
| J15 | Roommate matching | Opt-in → preferences → matches (consent-based) → intro chat → accept/decline |

---

# 13. Use Cases (summary table)

| UC | Actor | Pre-condition | Main success | Key alternates |
| --- | --- | --- | --- | --- |
| UC-01 Publish listing | Owner | Verified owner | Listing approved and live | Rejection, changes requested |
| UC-02 Search | Customer | — | Relevant results | Zero results → alternatives |
| UC-03 Book visit | Customer | Listing Available | Owner confirms | Reject, reschedule, no-show |
| UC-04 Apply to rent | Customer | Logged in, listing Available | Application Accepted | Withdrawn, rejected, listing rented |
| UC-05 Sign agreement | Both | Application Accepted | Both signed | Expired, declined |
| UC-06 Pay deposit | Customer | Agreement/booking in state | Payment Confirmed | Failed, pending, refund |
| UC-07 Report listing | Customer | Logged in | Report logged | Duplicate report |
| UC-08 Verify | Employee | Item in queue | Decision recorded | Escalation |
| UC-09 Moderate review | Moderator | Flagged review | Decision | Appeal |
| UC-10 Handle ticket | Support | Ticket open | Resolved | Escalated |

---

# 14. Detailed Feature Requirements

## 14.1 Detail Page (DET)

- **DET-001 (P0):** Shows title, gallery, video, price, deposit, estimated monthly cost, type, size, rooms, amenities, rules, availability, location/map, nearby facilities, owner profile, badges, last updated/verified, reviews/ratings, and actions (compare, favourite, share, report, contact, chat, schedule visit, apply/book).
- **DET-002 (P0):** Actions reflect state: if Rented/Unavailable, Apply/Book/Visit are disabled with explanation and "Find similar".
- **DET-003 (P0):** Shared links render localized previews (title, cover, price) and never include owner contact data.
- **DET-004 (P0):** Exact address is shown only after a visit is confirmed or application accepted (configurable); before that, show locality and approximate map circle.

## 14.2 Favourites & Collections (FAV)

- **FAV-001 (P0):** One-tap favourite/unfavourite; syncs across devices.
- **FAV-002 (P1):** Collections: create, rename, delete, add/remove, reorder; default "Shortlist". Limits configurable (default 50 collections, 500 items).
- **FAV-003 (P1):** Alerts for favourites: price change, availability change, freshness/verification change, removal.
- **FAV-004 (P1):** Collection compare shortcut; collections may be shared as read-only link (opt-in).

## 14.3 Comparison (CMP)

- **CMP-001 (P1):** Compare 2–4 properties: rent, deposit, true monthly cost, move-in cost, area, rooms, furnishing, amenities, parking, distances (to chosen anchors), nearby facilities, owner verification, freshness, reviews, rules, availability.
- **CMP-002 (P1):** Differences highlighted; "best value" shown only as factual min/max, no opaque scoring. Comparison persists per user and is shareable.
- **CMP-003 (P1):** Mixed categories permitted but only common attributes compared; others marked "Not applicable".

## 14.4 True Cost Calculator (COST)

| ID | Requirement | Pri |
| --- | --- | --- |
| COST-001 | Estimated monthly cost = rent + maintenance + recurring mandatory charges + parking + estimated electricity + estimated internet + other recurring. | P0 |
| COST-002 | Estimated move-in cost = first period rent (if required) + security deposit + one-time fees + platform fee (if any) + advance/brokerage (shown as ₹0 for no-brokerage). | P0 |
| COST-003 | Every line is labelled **Actual (owner-stated)** or **Estimated** with source (owner estimate, area default, user override). | P0 |
| COST-004 | Area defaults for utilities admin-configurable per city/property size; owner estimate takes precedence; user can override. | P1 |
| COST-005 | Currency/number formatting localized; amounts in minor units internally. | P0 |
| COST-006 | Taxes (e.g., GST on certain commercial rents/hotel rates) shown where applicable **\[LEGAL\]**. | P1 |

## 14.5 Affordability (AFF)

- **AFF-001 (P2):** Inputs: monthly income, commitments, savings target. Output: indicative budget range and rent-to-income ratio using transparent formula (default: housing ≤ configurable % of income after commitments).
- **AFF-002 (P2):** Disclaimer: informational only, not financial advice. Inputs are not stored unless user opts in; never shared with owners.

## 14.6 Visits (VIS)

| ID | Requirement | Pri |
| --- | --- | --- |
| VIS-001 | Customer requests visit: date, time slot, attendees (count + optional names), message. | P0 |
| VIS-002 | Owner defines availability windows; system proposes slots; conflicts prevented. | P0 |
| VIS-003 | Owner: accept, reject (reason), suggest alternate, reschedule, cancel. Customer: cancel, reschedule. | P0 |
| VIS-004 | Reminders at 24 h and 2 h (configurable); notifications in user language. | P0 |
| VIS-005 | Post-visit prompt: attended? feedback; no-show tracking for both parties (affects trust metrics, not penalties without review). | P1 |
| VIS-006 | Visit safety: shows safety tips; first visit address disclosure per DET-004; optional share-visit-details with a trusted contact. | P1 |
| VIS-007 | Statuses: Requested, Accepted, Rejected, Reschedule Proposed, Rescheduled, Cancelled by Customer, Cancelled by Owner, Completed, No-Show (customer/owner), Expired. | P0 |

## 14.7 Chat (CHAT)

| ID | Requirement | Pri |
| --- | --- | --- |
| CHAT-001 | One conversation per customer–listing pair; text, emoji, attachments (image/PDF, scanned), system messages (visit, application). | P0 |
| CHAT-002 | Delivery and read status; push notifications; offline queueing. | P0 |
| CHAT-003 | Contact masking: phone/email hidden by default. Reveal only by mutual consent or at defined stages (visit accepted/application accepted). Detection of phone numbers/emails/payment links in messages with warning; repeated attempts to move off-platform for payment flagged. | P0 |
| CHAT-004 | Report message/user; block user; blocked users cannot start new conversations. | P0 |
| CHAT-005 | Spam controls: new-account message limits, link restrictions, duplicate message detection, rate limits. | P0 |
| CHAT-006 | Document sharing only through "Request document" flow with expiry and view-only mode where feasible; sensitive IDs discouraged by warning. | P1 |
| CHAT-007 | Retention: messages retained per §30; reported conversations preserved for investigations. | P0 |
| CHAT-008 | Auto-translation of messages (en/te/hi) as optional AI feature, clearly labelled machine-translated; original always viewable. | P2 |
| CHAT-009 | Staff can read conversations only when reported or under ticket with logged justification. | P0 |

## 14.8 Rental Application (APP)

| ID | Requirement | Pri |
| --- | --- | --- |
| APP-001 | "Apply to Rent": personal info, employment/student info, move-in date, number of occupants, supporting documents, message. | P0 (basic), P1 (documents) |
| APP-002 | Data minimization: only fields required by owner template; ID documents optional until acceptance stage. | P0 |
| APP-003 | Statuses: Draft, Submitted, Viewed, Information Requested, Shortlisted, Accepted, Rejected, Withdrawn, Expired, Cancelled (listing unavailable), Converted to Agreement. | P0 |
| APP-004 | Owner may have multiple applications open; accepting one places listing in **Reserved** (configurable hold, default 72 h pending agreement/deposit) and auto-notifies others upon rental. | P0 |
| APP-005 | A customer may have a limited number of concurrent active applications (default 5). | P1 |
| APP-006 | Rejection reasons are chosen from a non-discriminatory list. | P0 |
| APP-007 | Applications cannot be created for Rented/Unavailable/Archived listings. | P0 |

## 14.9 Digital Agreement (AGR)

| ID | Requirement | Pri |
| --- | --- | --- |
| AGR-001 | Generate from approved, region-specific templates (admin-managed, versioned, localized). Fields: tenant, owner, property, rent, deposit, tenure, notice period, rules, responsibilities, escalation, special clauses (reviewed). | P1 |
| AGR-002 | Templates undergo configurable legal review/approval before activation **\[LEGAL\]**. | P1 |
| AGR-003 | E-signature via adapter; must comply with IT Act 2000 / applicable e-sign requirements (e.g., Aadhaar eSign or other legally recognized methods as supported by chosen provider) **\[LEGAL\]**. | P1 |
| AGR-004 | Stamp duty and registration: system displays region-specific requirements and supports e-stamping integration where available; platform does not assert enforceability **\[LEGAL\]**. Agreements over statutory thresholds flagged for registration. | P1 |
| AGR-005 | Statuses: Draft, Pending Owner Review, Pending Customer Review, Pending Signatures, Partially Signed, Fully Executed, Expired, Cancelled, Terminated, Amended. | P1 |
| AGR-006 | Immutable executed copy with hash, timestamps, signer metadata; both parties can download; stored encrypted. | P1 |
| AGR-007 | Amendments create new versions; no silent edits after signature. | P1 |
| AGR-008 | MVP provides a basic agreement-summary and downloadable draft for off-platform signing. | P0 |

## 14.10 Rooms, PG & Roommate Matching (ROOM)

- **ROOM-001 (P1):** PG/hostel inventory modelled as property → room → bed; per-bed availability and pricing; gender-specific accommodation as permitted.
- **ROOM-002 (P3):** Roommate matching is **opt-in**, separate profile, explicit consent per attribute shared; attributes: budget, location, lifestyle, sleep schedule, food, smoking, pets, work/study schedule, sharing preference.
- **ROOM-003 (P3):** Matching must not use or infer protected attributes (religion, caste, ethnicity, etc.) **\[LEGAL\]**; identity details hidden until mutual match; users can leave, block, report; safety guidance; minors excluded (18+ only).
- **ROOM-004 (P3):** Transparent compatibility explanation per attribute.

## 14.11 Swipe Discovery & Modern UX (UX)

- **UX-001 (P2):** Swipe mode: right = favourite, left = skip, tap = details. Undo last action. Skipped items down-ranked for that user only. Keyboard/button alternatives for accessibility.
- **UX-002 (P1):** Personalized home feed (recommended, recently updated, nearby, price drops, continue where you left off).
- **UX-003 (P2):** Shareable property cards; AI summaries (grounded); recently viewed; price-drop alerts; commute-based recommendations; move-in countdown; rental checklist (documents, utilities, inspection).
- **UX-004 (P0):** List, grid, and map views always available; swipe is never the only path.

## 14.12 Owner Analytics (ANA)

- **ANA-001 (P1):** Per-listing: views, unique visitors, search appearances, favourites, enquiries, chats started, visit requests, applications, bookings, conversion funnel, freshness status; date ranges, export CSV.
- **ANA-002 (P1):** Aggregated/pseudonymous only; no identification of individual viewers; minimum cohort thresholds to avoid re-identification.

## 14.13 Reviews (REV)

| ID | Requirement | Pri |
| --- | --- | --- |
| REV-001 | Customer → Property ratings: accuracy, cleanliness, amenities, location, overall (1–5) + text. | P0 |
| REV-002 | Customer → Owner: communication, responsiveness, rental experience. | P1 |
| REV-003 | Owner → Customer feedback (post-rental, structured, limited to rent payment, property care, communication; no free-text personal data; shown only to owners on later applications with the customer's right to respond). **\[LEGAL\]** | P2 |
| REV-004 | Eligibility: completed visit (verified by both parties), or active/completed booking/rental. One review per transaction; editable within 14 days. | P0 |
| REV-005 | Anti-abuse: simultaneous (blind) release for two-way reviews, retaliation detection, profanity/PII filter, moderation queue, owner "dispute review" flow, appeal. | P1 |
| REV-006 | Owner responses permitted publicly once. | P1 |
| REV-007 | Aggregates exclude removed/disputed reviews; show review count with recency. | P0 |

## 14.14 Reporting & Fraud (REP)

| ID | Requirement | Pri |
| --- | --- | --- |
| REP-001 | Categories: fake property, fake owner, fake photos, incorrect info, wrong price, already rented, fraud/scam, inappropriate content, suspicious behaviour, other. | P0 |
| REP-002 | Report has ID, category, description, evidence (≤5 files), reporter, target, status, assignee, resolution, audit history. | P0 |
| REP-003 | Statuses: Submitted, Triaged, In Review, Awaiting Info, Action Taken, Dismissed, Escalated, Closed. SLA default 24 h triage for fraud. | P0 |
| REP-004 | Automated signals: price far below locality median; duplicate media/address; new account with many listings; off-platform contact pushes; advance-payment requests; ID/name mismatches; velocity; many reports; IP/device clustering; recycled phone numbers. Signals produce a risk score and queue priority, never an automatic permanent ban without human review (except clear policy-defined cases e.g., malware). | P0 |
| REP-005 | Actions: warn, hide listing, suspend owner, restrict chat, remove media, escalate, refer to authorities **\[LEGAL\]**. | P0 |
| REP-006 | Reporter is notified of outcome without disclosing private info about the accused. Reporter identity not disclosed to the accused. | P0 |
| REP-007 | Abuse of reporting (false mass reports) tracked and rate-limited. | P1 |

---

# 15. Search, Discovery & Matching (SRCH)

## 15.1 Discovery channels

Search bar (keyword, location, natural language), categories, map, recommended, recently added/updated, nearby, saved searches, personalized feed, AI assistant, collections.

## 15.2 Requirements

| ID | Requirement | Pri |
| --- | --- | --- |
| SRCH-001 | Full-text search across title/description/locality/landmarks with typo tolerance and en/te/hi support, including transliteration (e.g., "Kukatpally" typed in Telugu or Hindi script). | P0 |
| SRCH-002 | Location search via geocoding adapter: city, locality, landmark, PIN, "near ". | P0 |
| SRCH-003 | Natural-language search converts text to structured criteria through the AI assistant; the parsed criteria are displayed and editable. | P0 |
| SRCH-004 | Only public, Available (or per-setting, Reserved) listings appear; unverified/rejected/suspended/stale-hidden excluded. | P0 |
| SRCH-005 | Sorting: Relevance (default), Newest, Recently updated, Price low/high, Estimated total cost, Distance, Rating. | P0 |
| SRCH-006 | Pagination via cursor; filter counts shown; result retention 30 s consistency window. | P0 |
| SRCH-007 | Saved searches with alert frequency (instant/daily/weekly). Limits default 20. | P1 |
| SRCH-008 | Zero-results: suggest relaxing specific filters (showing impact count), nearby areas, similar properties. | P0 |
| SRCH-009 | Recently viewed and search history, user-clearable. | P1 |
| SRCH-010 | Promoted listings are labelled "Promoted", limited to a fixed number of slots per page, must still satisfy all filters, and cannot outrank organically by more than configured rules; promoted status excluded from relevance score. | P1 |

## 15.3 Filters

| Group | Filters |
| --- | --- |
| Property | Type, BHK, room type, area range, furnishing, floor, age, parking, balcony, AC, water, power backup, pets, family/student/professional, availability date |
| Financial | Rent range, deposit, maintenance, brokerage (none), other charges, total estimated monthly cost |
| Location | City, locality, radius, distance from a selected point, college, workplace, metro, hospital, supermarket; commute time to anchor (mode: walk/bike/car/transit where supported) |
| Trust | Verified owner, verified property, recently updated, recently verified, no brokerage |
| Category-specific | PG: gender, sharing, food; Hotel: dates, guests, star, cancellation; Commercial: zoning, frontage, business type; Land: plot size, road width, use |

- **FLT-001 (P0):** Filters combine with AND across groups, OR within multi-select; URL-encoded, shareable.
- **FLT-002 (P0):** Filter set per category defined by taxonomy configuration.
- **FLT-003 (P1):** Commute-time filters rely on routing adapter; if unavailable, fall back to straight-line distance with a visible notice.

## 15.4 Map (MAP)

- **MAP-001 (P0):** Interactive map with markers (price labels), clustering, search-this-area, radius circle, draw boundary (P2), dynamic filtering sync with list.
- **MAP-002 (P0):** List ↔ Map toggle preserves filters and scroll.
- **MAP-003 (P0):** Marker selection shows mini-card; tap opens detail.
- **MAP-004 (P1):** Route/distance/travel-time estimation to chosen destinations via routing adapter; labelled "estimated".
- **MAP-005 (P0):** Map failure degrades to list view with message.
- **MAP-006 (P0):** Marker coordinates for pre-contact display are privacy-offset (≈ 100–200 m) if the exact address is hidden (DET-004).

## 15.5 Nearby Facilities (NEAR)

- **NEAR-001 (P0):** Categories: hospitals, pharmacies, supermarkets, schools, colleges, metro, bus stops, railway, airports (where ≤ configurable distance), restaurants, cafes, gyms, banks, ATMs, petrol stations, parks, malls, entertainment, tourist attractions, user-requested custom facility.
- **NEAR-002 (P0):** Per facility: name, category, straight-line distance, estimated travel time (where available), map location, data source and "last refreshed".
- **NEAR-003 (P0):** Facility data obtained via POI adapter, cached and refreshed on schedule (default 30 days) with fallback to cached data if provider is unavailable. Platform never fabricates facilities.
- **NEAR-004 (P0):** Properties filterable by presence/distance of facilities (e.g., metro ≤ 1 km), computed from pre-indexed geospatial data.
- **NEAR-005 (P1):** Owner can suggest corrections; staff review.

## 15.6 Transparent Matching (MATCH)

Matching score (0–100) is shown with a visible breakdown. Default weights (admin-configurable, versioned):

| Factor | Weight | Calculation |
| --- | --- | --- |
| Budget | 25 | 100% if total cost ≤ budget; linear decay to 0 at +25% over |
| Location/commute | 20 | Based on distance/commute vs. user maximum |
| Property type & size | 15 | Exact match = full; adjacent type (e.g., 2BHK vs 3BHK) = partial |
| Amenities & furnishing | 15 | Fraction of must-have/nice-to-have satisfied (must-haves are hard filters) |
| Nearby facilities | 10 | Fraction of requested facilities within thresholds |
| Availability/move-in | 10 | Available by desired date = full; decay per week |
| Trust & freshness | 5 | Verified + recently updated |

- **MATCH-001 (P0):** Hard constraints (must-haves, max budget declared as strict) exclude rather than score.
- **MATCH-002 (P0):** Each result lists matched criteria, partially met criteria, unmet criteria, and trade-offs, each backed by a platform data field.
- **MATCH-003 (P0):** Score never includes paid promotion, owner subscription tier, or protected attributes.
- **MATCH-004 (P1):** Weight changes are logged, A/B tested, and previous versions reproducible.

## 15.7 Ranking & Recommendation (RANK)

- **RANK-001 (P0):** Ranking signals: text relevance, geo-relevance, filter match, freshness, verification, availability, completeness, response rate, user preference.
- **RANK-002 (P1):** Personalization uses behaviour (views, favourites, saved searches) with an opt-out toggle and a "Why am I seeing this?" explanation.
- **RANK-003 (P0):** Diversity: avoid more than N consecutive listings from one owner.
- **RANK-004 (P1):** Stale listings deprioritized per FRESH-003.

---

# 16. AI Requirements (AI)

## 16.1 Capabilities

| ID | Requirement | Pri |
| --- | --- | --- |
| AI-001 | Conversational assistant in en/te/hi (text; voice P2) understanding free-form requirements. | P0 |
| AI-002 | Extracts structured criteria (budget, type, furnishing, location/anchor, distance/commute, amenities, move-in date, occupants) into a visible chip/filter set. | P0 |
| AI-003 | Asks at most 2 clarifying questions per turn when critical fields are ambiguous (budget, city/area, purpose); otherwise proceeds with stated assumptions. | P0 |
| AI-004 | Executes search through internal Search API (tool/function calls); it cannot query raw database tables. | P0 |
| AI-005 | Ranks using the transparent matching engine (MATCH). The LLM explains, it does not compute scores. | P0 |
| AI-006 | For each result, generates an explanation citing platform fields (e.g., "Rent ₹22,000 (within budget); Metro station 800 m (source: facility data, refreshed 12 Sep)"). | P0 |
| AI-007 | Identifies trade-offs (e.g., cheaper but farther). | P0 |
| AI-008 | Supports refining ("make it ₹20,000", "closer to metro"), saving preferences as a saved search, and alternatives when no exact match. | P0 |
| AI-009 | Summaries of listings and compare outputs are generated from structured fields only. | P1 |
| AI-010 | Can guide support/FAQ questions (AI support assistant) with handoff to humans. | P1 |

## 16.2 Safety & Reliability (AI-SAFE)

| ID | Requirement | Pri |
| --- | --- | --- |
| AI-SAFE-001 | Grounding: every property claim (price, availability, verification, distance, amenities) must come from a retrieved platform record in the same turn. Claims without a source are blocked by an output validator that cross-checks numbers and entities against retrieved data. | P0 |
| AI-SAFE-002 | The assistant never fabricates availability, verification, prices, reviews, or owner details; when data is missing it says "not provided by the owner". | P0 |
| AI-SAFE-003 | No legal guarantees (agreement validity, tenant rights outcomes) or financial guarantees; shows disclaimer and suggests professional advice for legal/financial questions. | P0 |
| AI-SAFE-004 | No exposure of personal data of other users; prompt-injection defence: listing text and chat content treated as untrusted data, never instructions. | P0 |
| AI-SAFE-005 | Does not take irreversible actions (payments, applications, agreements, bookings) on behalf of the user; may prepare drafts for explicit user confirmation. | P0 |
| AI-SAFE-006 | Bias controls: no filtering/ranking on protected attributes; refuse discriminatory requests (e.g., "only a particular religion/caste tenant") and explain why. | P0 |
| AI-SAFE-007 | Confidence/uncertainty: states uncertainty and data freshness (e.g., last updated date). | P0 |
| AI-SAFE-008 | Fallback: if LLM unavailable or validation fails, the user gets structured filter UI with last-known-parsed criteria, keyword search, and message "Assistant temporarily unavailable". | P0 |
| AI-SAFE-009 | Logging: prompts/outputs stored for quality/safety with PII minimization, retention default 90 days, user opt-out for training use. | P0 |
| AI-SAFE-010 | Cost/usage controls: per-user quotas, caching, timeouts (default 15 s), token budgets. | P0 |
| AI-SAFE-011 | Evaluation: offline test sets per language (grounding accuracy ≥ 99% on claim checks, criteria-extraction F1 ≥ 0.90, refusal accuracy for discriminatory requests ≥ 99%), regression gates in CI. | P0 |

---

# 17. Verification and Trust (TRUST)

## 17.1 Verification pipeline for a listing

| Step | Type | Description |
| --- | --- | --- |
| 1 | Automated | Schema/completeness, banned content, price plausibility vs. locality, duplicate address/geo/media, owner status |
| 2 | Automated | Media: malware, moderation, perceptual-hash duplicates, resolution, EXIF checks |
| 3 | Manual | Verification Employee reviews details, media, map pin vs. address, ownership docs (name/address match, validity) |
| 4 | Optional | Availability confirmation call/message; on-site verification (Phase 3) |
| 5 | Decision | Approve → Published (Property Verified + Documents Verified badges); Request changes; Reject (reason); Escalate to Admin |
| 6 | Ongoing | Periodic re-verification (default 12 months), freshness checks, re-review on material edits |

- **TRUST-001 (P0):** Listings are never publicly searchable before steps 1–3 are complete (Business Rule BR-01).
- **TRUST-002 (P0):** Reviewers use checklists; each decision includes checklist results and reason code.
- **TRUST-003 (P0):** Four-eyes principle: Verification Employee cannot approve own submissions, and high-risk (risk score ≥ threshold) approvals require a second reviewer or Admin.
- **TRUST-004 (P0):** Ownership documents stored in a private encrypted store; viewed through short-lived signed URLs, with watermarking in the viewer, no download for reviewers by default.
- **TRUST-005 (P0):** Document expiry (e.g., authorization letters) triggers re-verification workflows.
- **TRUST-006 (P1):** Appeals by owners go to Admin.

# 18. Booking / Rental / Lease / Sale Workflows

## 18.1 Workflow templates

| Template | Used for | Stages |
| --- | --- | --- |
| WF-RENT | Residential rent | Enquiry → Visit → Application → Acceptance → Agreement → Deposit/payment → Move-in → Active tenancy → Renewal/Exit |
| WF-ROOM | PG/hostel/co-living | Enquiry → Visit → Bed reservation → Deposit → Move-in |
| WF-STAY | Hotel/hourly/short/vacation | Search → Select → Hold → Pay → Confirm → Check-in/out → Review |
| WF-LEASE | Commercial lease | Enquiry → Visit → EOI → Terms negotiation → Lease agreement → Deposit → Handover |
| WF-SALE | Land/property sale | Enquiry → Site visit → Document access → Offer/EOI → Owner response → Off-platform legal completion (platform records status only) |

## 18.2 Short-stay specifics (STAY)

| ID | Requirement | Pri |
| --- | --- | --- |
| STAY-001 | Inventory model: property → room type → room/inventory count; calendar availability with date and hourly slots. | P1 |
| STAY-002 | Booking inputs: check-in/out date-time, guests (adults/children), room type, special requests. | P1 |
| STAY-003 | Hourly stays: slot durations (e.g., 3/6/12 h) defined by owner; minimum turnaround time; no overlapping bookings. | P2 |
| STAY-004 | Inventory hold during checkout (default 10 min) using atomic reservation; expires automatically. | P1 |
| STAY-005 | Pricing: base, taxes (GST as applicable), service fees, dynamic/seasonal rules; all shown before payment. **\[LEGAL\]** | P1 |
| STAY-006 | Cancellation policies (flexible/moderate/strict/custom) shown pre-booking; refund computed automatically. | P1 |
| STAY-007 | Confirmation voucher (localized), check-in code/QR, guest ID requirements per local regulations (e.g., guest ID capture at hotel) **\[LEGAL\]**. | P1 |
| STAY-008 | Statuses: Held, Pending Payment, Confirmed, Checked-In, Completed, Cancelled (guest/host), No-Show, Refunded, Disputed, Expired. | P1 |
| STAY-009 | Optional channel manager/iCal sync adapter to avoid double-bookings. | P2 |

# 19. Payment Requirements (PAY)

Payment provider is abstracted (implementation option: any RBI-authorized gateway; none assumed).

| ID | Requirement | Pri |
| --- | --- | --- |
| PAY-001 | Payment Gateway Adapter exposes: create order, capture/verify, refund, fetch status, webhook parse/verify, payout (if supported). | P0 (contract), P1 (live) |
| PAY-002 | Payables: booking amount, rental application fee (where applicable), security deposit, rent, hotel/hourly booking, lease payments, platform fees, subscriptions. | P1 |
| PAY-003 | Statuses: Initiated, Pending, Authorized, Captured/Paid, Failed, Cancelled, Refund Pending, Partially Refunded, Refunded, Chargeback/Disputed, Settled, Reconciled. | P1 |
| PAY-004 | **Payment status derives only from server-verified gateway confirmation (signed webhook or server-to-server status check), never from client redirects.** | P1 |
| PAY-005 | Webhooks: signature verification, idempotency keys, replay protection, retries with backoff, dead-letter queue. | P1 |
| PAY-006 | Idempotent payment creation; one payable ↔ at most one successful payment. | P1 |
| PAY-007 | Ledger: double-entry style records for customer payments, owner payables, platform fees, taxes, refunds; immutable entries. | P1 |
| PAY-008 | Receipts/invoices (GST-compliant where applicable) generated and downloadable **\[LEGAL\]**. | P1 |
| PAY-009 | Refund rules per policy; partial refunds; refund to original method; ownership of refund decisions by policy engine, with manual Admin override audited. | P1 |
| PAY-010 | Payment holding/escrow-like flows (hold deposit until move-in confirmation) may require RBI payment aggregator/nodal/escrow arrangements; platform shall use a compliant provider **\[LEGAL\]**. | P1 |
| PAY-011 | Reconciliation: daily automated match of gateway settlement files vs. ledger; discrepancy queue for Finance/Admin. | P1 |
| PAY-012 | Fraud controls: velocity limits, 3-D Secure/2FA per regulations, blocklists, high-value review, mismatch detection. | P1 |
| PAY-013 | PCI-DSS scope minimization: platform never stores card data (tokenised/hosted fields). | P0 |
| PAY-014 | Owner payouts require verified bank account and KYC; payout schedule configurable; TDS/GST handling configurable **\[LEGAL\]**. | P2 |
| PAY-015 | Rent collection and recurring payments (autopay mandates) are Phase 3. | P3 |
| PAY-016 | MVP: no in-platform money movement; booking/deposit workflows are recorded as statuses with "pay offline" guidance and warnings against advance payments before visit. | P0 |

# 20. Communication (COMM)

Covers chat (§14.7), notifications (§37 below as N-xxx), visits, in-app system messages.

- **COMM-001 (P0):** All system messages are templates with i18n keys and variables; stored per language.
- **COMM-002 (P0):** Contact reveal rules: phone/email shown only after (a) mutual consent, or (b) accepted application/visit if configured; reveal events audited.
- **COMM-003 (P1):** Masked calling/virtual numbers via telephony adapter (optional).

# 21. Reviews and Reporting

See REV (§14.13) and REP (§14.14).

# 22. Support (SUP)

| ID | Requirement | Pri |
| --- | --- | --- |
| SUP-001 | Help centre with searchable FAQs (en/te/hi), categories for customers and owners. | P0 |
| SUP-002 | Ticket creation from any screen with context auto-attached (listing ID, booking ID); attachments scanned. | P0 |
| SUP-003 | Ticket statuses: New, Open, Pending Customer, Pending Internal, Escalated, Resolved, Closed, Reopened. | P0 |
| SUP-004 | Assignment: auto-routing by category/language/skill; SLA timers (first response 4 h business, resolution 48 h; configurable); escalation at breach. | P0 |
| SUP-005 | Live chat during business hours (P2); AI support assistant (P1) with handoff. | P1/P2 |
| SUP-006 | CSAT survey on resolution; ticket history visible to user. | P1 |
| SUP-007 | Dispute management for bookings/deposits: evidence collection from both parties, timeline, decision, refund trigger. | P1 |

# 23. Multilingual Requirements (I18N)

| ID | Requirement | Pri |
| --- | --- | --- |
| I18N-001 | Languages: English (en-IN), Telugu (te-IN), Hindi (hi-IN). Language chosen at onboarding, switchable anytime; persisted to profile; default from device. | P0 |
| I18N-002 | No hard-coded strings; ICU message format with plurals; translation management workflow with versioning and missing-key fallback to English. | P0 |
| I18N-003 | Localized UI, emails, SMS (DLT-template-compliant for India), push, notifications, errors, PDF documents and agreements. | P0 |
| I18N-004 | Date/time (IST default, timezone-aware), numbers (Indian digit grouping lakh/crore), currency (₹ default; multi-currency-ready). | P0 |
| I18N-005 | Search in all languages with transliteration and synonyms; localized place names. | P0 |
| I18N-006 | User-generated content (listings/messages) is stored in the original language; optional machine translation labelled as such; owners may provide multi-language descriptions (P1). | P1 |
| I18N-007 | Fonts: Noto Sans (or equivalent) with full Telugu/Devanagari support; layout tested for text expansion (+35%). | P0 |
| I18N-008 | Adding a language requires only translation resources, locale config, and search analyzers; no code change. | P0 |
| I18N-009 | AI assistant responds in the user's chosen/detected language. | P0 |

# 24. Admin / Employee Requirements (ADMIN)

| ID | Requirement | Pri |
| --- | --- | --- |
| ADMIN-001 | Verification console: queues (owners, listings, media, freshness), filters, SLA timers, checklists, doc viewer, decision + reason, bulk assign. | P0 |
| ADMIN-002 | Moderation console: reports, fraud signals, suspicious users, reviews, content; actions with templates. | P0 |
| ADMIN-003 | Support console: tickets, macros, customer context, escalation. | P0/P1 |
| ADMIN-004 | Dashboards: total users, owners, properties, active listings, pending verification, reports, bookings, rentals, applications, revenue, tickets, suspicious activity, platform performance; filters by city, category, date, status. | P0 (core metrics), P1 (revenue) |
| ADMIN-005 | Master data: categories, attributes, locations, amenities, languages, translations, rules, templates, badges, freshness thresholds, ranking weights, promotion/pricing plans. | P0 |
| ADMIN-006 | User/owner management: search, restrict, suspend, reinstate with reason. | P0 |
| ADMIN-007 | Employee management (Super Admin): create, roles, MFA enforcement, deactivate, session control. | P0 |
| ADMIN-008 | Audit log viewer with filters/export; immutable. | P0 |
| ADMIN-009 | Configuration management with versioning, approval workflow for risky changes (e.g., ranking weights), and rollback. | P1 |
| ADMIN-010 | Admin console accessible only from allowlisted networks/devices where configured; mandatory MFA. | P0 |

---

# 25. Data Model (Conceptual)

All entities carry: `id` (UUID), `created_at`, `updated_at`, `created_by`, `region_id`, soft-delete flag where relevant. Money is stored as integer minor units + currency code. Lifecycle states in §39.

| Entity | Key fields | Relationships | Lifecycle |
| --- | --- | --- | --- |
| User | phone, email, password_hash, language, status, roles, last_login | 1–1 Customer/Owner profile; 1–\* Sessions | Pending→Active→Restricted→Suspended→Deactivated→Deleted |
| Customer | prefs (budget, locations, move-in), employment/student info (optional), privacy settings | User 1–1 | — |
| Owner | legal name, display name, verification_status, bank ref (tokenised), declaration timestamp | User 1–1; 1–\* Property, OwnershipDocument | Owner verification states |
| Employee/Admin | employee_id, role set, MFA status | User 1–1 | Active/Disabled |
| Property | owner_id, category_id, transaction_mode, title, description, address, location_id, geo point, areas, floors, specs (JSON by category), status, freshness timestamps, badges | Owner *–1; 1–* Media, Amenity, Rule, Availability, Verification, Unit | Property states |
| PropertyCategory | parent_id, attribute schema, filters, workflow, doc requirements, version | self 1–\* | Active/Retired |
| PropertyUnit | property_id, label, type, price, availability | Property 1–\* | Available/Reserved/Rented |
| PropertyMedia | property_id, type, URL ref, tag, caption, order, is_cover, status, hash, uploaded_at | Property 1–\* | Pending/Approved/Rejected |
| PropertyAmenity | property_id, amenity_id | M–N | — |
| PropertyRule | property_id, rule_code, value, text | 1–\* | — |
| PropertyAvailability | property_id/unit_id, from, to, status, hourly slots, last_confirmed_at | 1–\* | — |
| PropertyVerification | property_id, reviewer_id, checklist result, decision, reason, expiry | 1–\* | Verification states |
| OwnershipDocument | owner_id, property_id, type, encrypted store ref, status, expiry | Owner/Property | Submitted→Accepted/Rejected/Expired |
| Location | hierarchy (country, state, city, locality), geo polygon, localized names | self 1–\* | — |
| NearbyFacility | property_id/location, category, name, geo, source, distance, travel_time, refreshed_at | Property 1–\* | — |
| Favourite | user_id, property_id, collection_id | M–N | — |
| Collection | user_id, name, sharing flag | 1–\* Favourite | — |
| SavedSearch | user_id, criteria JSON, alert frequency, last_run | User 1–\* | Active/Paused |
| PropertyComparison | user_id, property_ids (2–4) | — | — |
| Conversation / Message | property_id, customer_id, owner_id; message body, attachments, status, flags | Conversation 1–\* Message | Active/Blocked/Archived |
| Visit | property_id, customer_id, slot, attendees, status | — | Visit states |
| RentalApplication | property_id/unit_id, applicant_id, form data, docs, status | — | Application states |
| RentalAgreement | application_id, template_version, terms, signatures, hash, status | — | Agreement states |
| Booking | property_id/unit, guest_id, check-in/out, guests, price breakdown, policy, status | — | Booking states |
| Payment | payable_ref, amount, currency, provider ref, status, idempotency key | Booking/Agreement | Payment states |
| Refund | payment_id, amount, reason, status | Payment 1–\* | Refund states |
| LedgerEntry | account, debit/credit, ref | Payment | Immutable |
| Review / Rating | reviewer, subject, ratings, text, status | Property/Owner/Customer | Review states |
| Report | reporter, target, category, evidence, status, assignee, resolution | — | Report states |
| SupportTicket | requester, category, priority, status, assignee, SLA | — | Ticket states |
| Notification | user_id, type, channel, payload, status, locale | — | Queued→Sent→Delivered/Failed→Read |
| Subscription | owner_id, plan_id, period, status | — | Subscription states |
| ListingPromotion | property_id, type, slots, start/end, status, payment_ref | — | Promotion states |
| AuditLog | actor, action, entity, before/after, timestamp, IP/device ref, reason | — | Immutable |
| Consent | user_id, purpose, version, granted_at, revoked_at | — | — |
| Translation | key, locale, text, version | — | — |

Key relationships: Owner 1–\* Property; Property 1–\* Media/Unit/Availability/Rule; Customer *–* Property (Favourite); Property 1–\* Visit/Application/Booking; Application 1–1 Agreement; Agreement/Booking 1–\* Payment; Payment 1–\* Refund; Property 1–\* Review.

# 26. API Requirements

Style: REST (JSON, versioned `/v1`), with GraphQL optional for aggregated read models (P2). Auth: OAuth2/JWT bearer with rotating refresh; RBAC/ABAC enforced. Standard error envelope: `code`, localized `message`, `request_id`, `details[]` (field errors). Pagination: cursor. Idempotency-Key header on all payment/booking POSTs. Default rate limit: 100 req/min/user, 20 req/min unauthenticated/IP unless noted.

| API group | Purpose | Request / Response (summary) | AuthN/AuthZ | Validation | Key errors | Rate limit |
| --- | --- | --- | --- | --- | --- | --- |
| Authentication | Register, OTP, login, refresh, logout, recovery | credentials/OTP → tokens | Public / session | Phone/email format, OTP rules | 401, 429, 423 (locked) | 5/min per identifier |
| Users | Profile, preferences, privacy, deletion | profile JSON | Self | Field rules | 403, 409 | 60/min |
| Owners | Onboarding, documents, verification status | doc upload (multipart) → status | Owner; staff read | File type/size, required docs | 413, 422 | 30/min |
| Properties | CRUD, publish, pause, status | listing JSON ↔ listing | Owner (own); staff | Category schema validation | 403 (unverified owner), 409 | 60/min |
| Media | Signed upload URLs, tagging, ordering | upload token → asset | Owner | Type/size/limits | 415, 413 | 60/min |
| Search | Query, sort, pagination, NL parse | query+filters → results+facets | Public | Param whitelist | 400 | 60/min |
| Filters | Config of filters by category | → filter schema | Public | — | — | 120/min |
| Map | Bounds query, clusters, route estimate | bbox/zoom → clusters | Public | Bounds limits | 400, 503 (degraded) | 120/min |
| Nearby | POI by property/location | → facilities | Public | Category enum | 503 fallback | 60/min |
| AI Assistant | Chat turn, criteria, explanations | message → reply + criteria + results + sources | Optional auth | Length ≤ 2,000 chars | 429, 503 | 20/min |
| Favourites / Collections | CRUD | ids → lists | Customer | Limits | 409 | 60/min |
| Comparisons | Create/get | ids (2–4) → table | Customer | Count 2–4 | 422 | 30/min |
| Chat | Conversations, messages, read receipts (WebSocket + REST) | message → ack | Participants | Length, attachment scan | 403 (blocked) | 30 msgs/min |
| Visits | Request, respond, reschedule, cancel | slot → visit | Customer/Owner | Slot validity | 409 (conflict) | 30/min |
| Applications | Submit, review, withdraw | form → application | Customer/Owner | Required fields, status | 409 (not available) | 20/min |
| Agreements | Generate, review, sign, download | → document | Parties | Version/status | 409, 422 | 20/min |
| Bookings | Quote, hold, confirm, cancel | → booking | Customer/Owner | Availability | 409 | 20/min |
| Payments | Create order, status, receipt, refund; webhook endpoint | → payment | Customer; webhook signature | Idempotency | 402, 409 | 20/min; webhook allowlisted |
| Reviews | Create, edit, respond, dispute | → review | Eligible users | Eligibility | 403 | 20/min |
| Reports | Create, status | → report ID | Authenticated | Category, evidence | 429 | 10/min |
| Support | Tickets, comments, CSAT | → ticket | Requester; staff | — | — | 30/min |
| Notifications | List, preferences, device tokens | → items | Self | — | — | 60/min |
| Admin | Queues, decisions, config, users | → results | Staff roles | Role checks, reason required | 403 | 120/min |
| Analytics | Owner and platform metrics | → aggregates | Owner(own)/Admin | Range limits | — | 30/min |

---

# 27. Frontend Requirements (FE)

## 27.1 Customer screens

Landing; Login/Registration/OTP; Onboarding; Home feed; Search + Filters (drawer on mobile); List/Grid/Map; Property details; AI assistant; Favourites; Collections; Comparison; Messages; Visits; Applications; Bookings; Agreements; Payments; Notifications; Reviews; Support/Help centre; Profile; Settings (privacy, notifications, language); Language selection; Swipe mode (P2); Affordability tool (P2); Roommate (P3).

## 27.2 Owner screens

Dashboard; Verification centre; Add/Edit property wizard; Media manager; Availability calendar; Enquiries; Chat; Visits; Applications; Bookings; Payments/Payouts; Analytics; Reviews; Subscription/Promotions (P1); Support; Profile/Settings.

## 27.3 Employee/Admin screens

Verification queue and detail; Moderation queue; Report detail; Support console; Admin dashboard; Master data; User/Owner management; Employee/Role management; Audit logs; Config; Reports/exports.

## 27.4 UI/UX & Accessibility requirements

- **FE-001 (P0):** Mobile-first responsive (320 px–1920 px); PWA with offline shell, installable.
- **FE-002 (P0):** Property cards show price, type, location, cover image, bedrooms, key amenities, verification badges, freshness, no-brokerage tag, estimated monthly cost, promoted label.
- **FE-003 (P0):** Skeleton loaders, optimistic updates, inline validation, clear empty/error states.
- **FE-004 (P0):** Design system with tokens, light/dark (P2), components documented.
- **FE-005 (P0):** Accessibility target WCAG 2.2 AA: keyboard operability, visible focus, screen reader labels (ARIA), semantic HTML, contrast ≥ 4.5:1, resizable text to 200%, accessible forms/errors, alt text for images (owner-provided/auto-suggested), reduced-motion support, ≥ 44 px touch targets.
- **FE-006 (P0):** Low-bandwidth mode: lazy-load, responsive images (WebP/AVIF), data-saver.
- **FE-007 (P0):** Supported browsers: last 2 versions of Chrome, Edge, Firefox, Safari; Android 8+, iOS 15+ (assumption).

# 28. Backend Requirements (BE)

- **BE-001 (P0):** Domain-driven modules; transactional integrity for bookings/applications using optimistic locking and unique constraints on (unit, slot).
- **BE-002 (P0):** Event-driven workflows (outbox pattern) with at-least-once delivery and idempotent consumers.
- **BE-003 (P0):** Background jobs: freshness scheduler, notification dispatcher, media processing, POI refresh, saved-search matcher, expiry handlers, reconciliation.
- **BE-004 (P0):** Configuration service for thresholds (freshness, limits, SLA, weights).
- **BE-005 (P0):** Search index updated within 60 s of listing changes (near-real-time); removal on unpublish within 10 s.
- **BE-006 (P0):** Feature flags and kill switches for AI, payments, promotions.
- **BE-007 (P0):** Time handled in UTC internally; localized for display.

---

# 29. Commercial and Land Requirements (COM / LAND)

| ID | Requirement | Pri |
| --- | --- | --- |
| COM-001 | Fields: plot size, built-up area, road frontage, zoning/use, parking, electricity (load kW), water, accessibility, suitable business types, lease duration, lock-in, escalation %, sale price, expected rental income, CAM charges, documents. | P1 |
| COM-002 | Filters: area, frontage, zoning, business type, floor, parking, price per sq ft. | P1 |
| COM-003 | Multi-unit commercial buildings with unit-level listing and availability. | P1 |
| LAND-001 | Land fields: survey number, plot size/unit conversion, boundaries, approvals (DTCP/HMDA/RERA/etc. as applicable), encumbrance, litigation declaration, ownership chain documents, road width, soil/irrigation (agri). | P1 |
| LAND-002 | Land listings require enhanced verification and legal disclaimers; status "Verification of title is the buyer's responsibility" displayed. **\[LEGAL\]** | P1 |
| LAND-003 | Platform does not process sale deed registration; transaction ends with EOI/offer tracking. | P1 |

# 30. Privacy (PRIV)

Assumption: DPDP Act 2023 and rules apply; final obligations confirmed by legal counsel **\[LEGAL\]**.

| ID | Requirement | Pri |
| --- | --- | --- |
| PRIV-001 | Notice and consent at collection, in en/te/hi; granular, withdrawable consent per purpose (service, marketing, analytics, personalization, AI training); consent records versioned. | P0 |
| PRIV-002 | Data minimization: collect only fields required for the function; optional data clearly marked. | P0 |
| PRIV-003 | Data principal rights: access (export), correction, erasure, grievance redressal, nominate; response SLA default 30 days (configurable per law). Grievance officer contact displayed. | P0 |
| PRIV-004 | Retention schedule (configurable): chat 24 months after last message; ID documents 12 months after verification decision (or as legally required); financial records 8 years; logs 12 months; reports/fraud evidence per case; deleted accounts anonymized after grace period except legal holds. | P0 |
| PRIV-005 | Children: 18+ only; no knowing collection from minors (verifiable parental consent not offered in MVP). | P0 |
| PRIV-006 | Cross-border transfers restricted per law; data residency in India by default for personal data. | P0 |
| PRIV-007 | Third-party processors: contractual DPAs, inventory of integrations, purpose limitation. | P0 |
| PRIV-008 | Privacy settings page: visibility of profile, personalization, communication preferences, data export, delete account. | P0 |
| PRIV-009 | Breach response: detection, containment, notification to authority/users per law (target ≤ 72 h) **\[LEGAL\]**. | P0 |
| PRIV-010 | Owner/customer personal contacts masked per COMM-002. | P0 |

# 31. Security (SEC)

| ID | Requirement | Pri |
| --- | --- | --- |
| SEC-001 | TLS 1.2+ everywhere, HSTS; encryption at rest (AES-256) with KMS-managed keys; field-level encryption for IDs, bank refs. | P0 |
| SEC-002 | Passwords: min 8 chars, breach-list check, Argon2id/bcrypt, no password hints; lockout/backoff after repeated failures. | P0 |
| SEC-003 | OTP: cryptographically random, hashed at rest, attempt/expiry limits, SMS pumping protection, anti-automation (CAPTCHA/risk-based). | P0 |
| SEC-004 | RBAC/ABAC server-side; least privilege; object-level authorization on every resource (prevent IDOR). | P0 |
| SEC-005 | OWASP ASVS L2 baseline: input validation, output encoding, parameterized queries/ORM (no string-built queries), NoSQL injection prevention, XSS (CSP, sanitization of rich text), CSRF tokens/SameSite for cookie auth, SSRF protection, secure headers. | P0 |
| SEC-006 | File uploads: type sniffing, size limits, malware scanning, storage outside web root, random names, no executable content, signed URLs. | P0 |
| SEC-007 | Documents private by default; access via short-lived signed URLs; access logged; no public listing of buckets. | P0 |
| SEC-008 | Secrets in a managed secret store; rotated; none in code/config repos. | P0 |
| SEC-009 | API: rate limiting, WAF, bot protection, request size limits, schema validation, replay protection for webhooks. | P0 |
| SEC-010 | Admin: MFA, IP allowlist optional, session limits, privileged action re-auth, break-glass process. | P0 |
| SEC-011 | Audit logs append-only, tamper-evident (hash chain), time-synchronized. | P0 |
| SEC-012 | Dependency scanning (SCA), SAST/DAST in CI, container scanning, annual third-party penetration test and pre-launch test. | P0 |
| SEC-013 | Payment: PCI-DSS SAQ-A scope via hosted fields; webhook signature checks; payout dual control. | P1 |
| SEC-014 | LLM security: prompt-injection mitigations, output filtering, no secrets in prompts, tool permission scoping. | P0 |
| SEC-015 | Abuse prevention: device fingerprint signals, account velocity limits, anomaly detection. | P1 |
| SEC-016 | Vulnerability disclosure policy and incident response runbook. | P0 |

---

# 32. Non-Functional Requirements

| Category | ID | Requirement / target | Configurable |
| --- | --- | --- | --- |
| Performance | NFR-PERF-01 | Mobile LCP ≤ 2.5 s (4G, p75); TTI ≤ 4 s | No |
|  | NFR-PERF-02 | API p95 ≤ 400 ms (reads), ≤ 800 ms (writes) excluding media/AI | Yes |
|  | NFR-PERF-03 | Search p95 ≤ 600 ms; autocomplete ≤ 200 ms | Yes |
|  | NFR-PERF-04 | Map initial render ≤ 3 s; marker refresh ≤ 1 s | Yes |
|  | NFR-PERF-05 | Image delivery p95 ≤ 1 s via CDN; thumbnails ≤ 50 KB | Yes |
|  | NFR-PERF-06 | AI first token ≤ 3 s; full response p95 ≤ 12 s | Yes |
|  | NFR-PERF-07 | Concurrent users: MVP 10,000; design for 100,000+ | Yes |
|  | NFR-PERF-08 | Uploads: 15 MB image ≤ 30 s on 4G; resumable | Yes |
|  | NFR-PERF-09 | Push/in-app delivery ≤ 5 s p95; SMS OTP ≤ 30 s p95; email ≤ 2 min | Yes |
| Scalability | NFR-SCAL-01 | Stateless services scale horizontally; DB read replicas, partitioning by region; search index sharding | — |
|  | NFR-SCAL-02 | Support ≥ 10 M listings, ≥ 50 M media assets, 1,000 search QPS peak (design target) | Yes |
| Availability | NFR-AVL-01 | 99.9% monthly (core), 99.5% for AI/auxiliary; planned maintenance windows off-peak | — |
| Reliability | NFR-REL-01 | Zero lost confirmed bookings/payments; graceful degradation for adapters | — |
| Disaster recovery | NFR-DR-01 | RPO ≤ 15 min, RTO ≤ 4 h; cross-zone redundancy, cross-region backups; DR test twice yearly | Yes |
| Security | NFR-SEC-01 | Per §31; zero critical/high vulnerabilities open at release | — |
| Privacy | NFR-PRIV-01 | Per §30 | — |
| Maintainability | NFR-MNT-01 | Unit coverage ≥ 80% critical modules; modular code; documented APIs (OpenAPI); zero-downtime migrations | — |
| Accessibility | NFR-ACC-01 | WCAG 2.2 AA | — |
| Localization | NFR-LOC-01 | 100% strings localized; add language without code change | — |
| Usability | NFR-USE-01 | First-time user completes search → shortlist in ≤ 3 min (usability test, 80% success) | Yes |
| Observability | NFR-OBS-01 | Per §34 | — |
| Compatibility | NFR-CMP-01 | Browsers/OS per FE-007; screen readers (TalkBack, VoiceOver, NVDA) | — |

# 33. Scalability (SCAL)

- **SCAL-001:** Region-aware data partitioning, per-region search indices, CDN for media, async processing queues, autoscaling policies.
- **SCAL-002:** Multi-currency/multi-country: currency, tax, legal-template, language and category configuration per `region_id`; no India-specific logic in core code paths.
- **SCAL-003:** Media pipeline scales via queue-based workers; storage tiering (hot/cold).
- **SCAL-004:** Cache strategy for search facets, POI, configuration; cache invalidation on events.

# 34. Observability (OBS)

| ID | Requirement |
| --- | --- |
| OBS-001 | Structured logs (app, API, error, security, audit) with correlation/request IDs; PII redaction. |
| OBS-002 | Metrics (RED/USE), distributed tracing, dashboards for DB, search, queue lag, AI usage/cost/latency/validation-failure rate, payment success rate, notification delivery rate. |
| OBS-003 | Alerts (paging): availability \< SLO, error rate > 2% for 5 min, payment webhook failures, queue lag > threshold, OTP delivery failure > 10%, AI grounding failure spike, security anomalies (auth failures, privilege escalation), storage malware detection, backup failure. |
| OBS-004 | Error tracking with release correlation; runbooks per alert; on-call rotation; post-incident reviews. |
| OBS-005 | Log retention: security/audit 12+ months; app logs 90 days. |

---

# 35. Testing Strategy (TEST)

| Type | Scope | Tooling-neutral targets |
| --- | --- | --- |
| Unit | Business rules, calculators (true cost, matching), state machines | ≥ 80% coverage on critical modules |
| Integration | Service + DB + adapters (sandboxed/mocked) | All adapter contracts have contract tests |
| API | Every endpoint: authz matrix, validation, error envelopes, rate limits | 100% endpoints, negative tests for IDOR |
| UI | Component and flow tests, visual regression | Key screens in 3 languages |
| End-to-end | J1–J15 critical paths on mobile + desktop viewports | Run on every release candidate |
| Security | SAST, DAST, SCA, pen test, authz fuzzing, file-upload abuse, OTP abuse, prompt injection | No open critical/high at release |
| Performance | Load, stress, soak, spike for search/map/chat/booking | NFR-PERF targets verified |
| Accessibility | Automated (axe-class) + manual screen-reader audit | WCAG 2.2 AA |
| Localization | Pseudo-localization, text expansion, script rendering, transliteration search | en/te/hi parity |
| Payment | Sandbox: success, failure, timeout, duplicate webhook, out-of-order webhook, refund, reconciliation | All PAY states reachable |
| AI evaluation | Golden sets per language, grounding checks, red-teaming (injection, discrimination, hallucination), regression gates | AI-SAFE-011 thresholds |
| Fraud/moderation | Seeded fake listings/images, price anomaly, duplicates; precision/recall tracking | Recall ≥ 85% on seeded set (initial target) |
| Cross-browser | Browser/OS matrix FE-007 | — |
| Mobile/responsive | Device lab, 320 px upward, low-end Android, throttled network | — |

# 36. Error Handling (ERR)

General rules: localized, actionable, non-technical messages; show a `request_id` for support; never expose stack traces, SQL, tokens, internal hostnames, or other users' data; log full detail server-side.

| ID | Situation | Required behaviour |
| --- | --- | --- |
| ERR-001 | Invalid login | Generic "credentials incorrect"; no account enumeration; lockout/backoff messaging |
| ERR-002 | OTP failure/expired | Show remaining attempts, resend option, alternative channel |
| ERR-003 | Network failure | Offline banner; queue drafts/messages; retry with backoff |
| ERR-004 | Payment failure | Show reason category, no booking confirmation, retry option, funds-deducted reassurance with auto-reconciliation notice |
| ERR-005 | Upload failure | Per-file error, retry, other files preserved |
| ERR-006 | Verification rejection | Reason code + guidance + resubmit path |
| ERR-007 | Property already rented | Disable actions; suggest similar; notify applicants |
| ERR-008 | Booking conflict | "Just booked by someone else"; alternatives |
| ERR-009 | Application rejection | Neutral message; similar listings |
| ERR-010 | AI failure | Fallback per AI-SAFE-008 |
| ERR-011 | Search failure | Retry, cached results, simplified search |
| ERR-012 | Map failure | List view fallback |
| ERR-013 | Notification failure | Retry across alternate channel; in-app copy always created |

# 37. Edge Cases (EDGE) and Notifications (N)

| ID | Case | Required behaviour |
| --- | --- | --- |
| EDGE-01 | Property rented while customer is viewing | On action attempt, server re-checks status; show "no longer available"; preserve favourite with "Rented" tag; offer similar |
| EDGE-02 | Two users book simultaneously | Atomic inventory reservation; first successful hold wins; second receives conflict with alternatives; holds expire |
| EDGE-03 | Owner deletes property after application | Deletion is soft/blocked while active applications/bookings; if archived, applicants notified; any payments refunded per policy |
| EDGE-04 | Owner suspended | All listings hidden; active bookings follow safeguarding process (support contacts customers; refunds where applicable); chats frozen read-only |
| EDGE-05 | Verification expires | Badge removed; listing deprioritized after grace (default 14 days); hidden after 30 days if not renewed |
| EDGE-06 | Incorrect location | Users can flag; staff review; listing hidden if map pin mismatch confirmed; owner prompted to correct |
| EDGE-07 | Duplicate listings | Auto-detected; duplicate set merged or earlier one retained; second flagged to owner/moderator |
| EDGE-08 | Fake images | Media removed, listing suspended pending review, owner risk score raised |
| EDGE-09 | Stale property | FRESH ladder |
| EDGE-10 | Payment succeeds but booking fails | Payment marked Captured/Unfulfilled; automatic retry of fulfilment; if not resolved in 15 min, auto-refund or manual-review ticket; customer notified |
| EDGE-11 | Payment fails but booking appears successful | Prohibited by PAY-004: booking confirmation only after server-verified payment; reconciliation job reverts erroneous confirmations and notifies |
| EDGE-12 | Owner unreachable | After 72 h no response to application/visit, auto-notify customer, escalate to support, reduce responsiveness score, may auto-expire request and recommend alternatives |
| EDGE-13 | Customer cancels | Policy-driven refund; owner notified; availability restored |
| EDGE-14 | Owner cancels | Customer full refund; owner penalty per policy; support assistance; repeat cancellations affect trust metrics |
| EDGE-15 | Disputed transaction | Dispute workflow (SUP-007); funds hold per provider; evidence timeline |
| EDGE-16 | Review abuse | Moderation, blind release, dispute, appeal |
| EDGE-17 | Fraud reports | Threshold-based auto-hide; human decision within SLA |
| EDGE-18 | Multiple owners per property | Co-owner model; one primary contact; all co-owners acknowledge declaration; disputes escalate to support |
| EDGE-19 | Commercial multi-unit | Unit-level availability/pricing; building-level verification |
| EDGE-20 | Duplicate account/phone reuse | Recycled numbers re-verified; account merge flow |
| EDGE-21 | Price change during checkout | Price snapshot at hold; if changed, user must re-confirm |
| EDGE-22 | Time zone / DST | Stored UTC; displayed IST |

**Notifications (N-xxx, P0 unless noted):** channels push, email, SMS/OTP, in-app. Types: new matching property (P1), price change (P1), property update (P1), favourite change (P1), new message (P0), visit confirmation/reminder (P0), application status (P0), agreement status (P1), payment/booking (P1), rent due (P3), support ticket (P0), verification status (P0), freshness prompts (P0). Users control preferences per type/channel; transactional/security messages cannot be fully disabled; quiet hours supported; SMS follows DLT/TRAI rules **\[LEGAL\]**; delivery tracking and fallback channel.

# 38. Business Rules (BR)

| ID | Rule |
| --- | --- |
| BR-01 | Only verified owners can publish public listings; listings are not searchable until required verification is complete. |
| BR-02 | Listing information must be periodically confirmed (default every 30 days). |
| BR-03 | Stale listings are deprioritized, hidden, then archived per FRESH-003. |
| BR-04 | Rented/unavailable/archived properties cannot accept new applications, visits, or bookings. |
| BR-05 | Only authorized Verification Employees may verify listings; only Admins may override, with reasons. |
| BR-06 | Payment status is set only by verified gateway confirmation. |
| BR-07 | Reviews require eligibility (completed visit or booking/rental) and are limited to one per transaction. |
| BR-08 | Reported listings follow moderation workflow; high-risk reports hide listing pending review. |
| BR-09 | Sensitive data is exposed only on a need-to-know, consented, audited basis. |
| BR-10 | AI cannot invent property information or take binding actions. |
| BR-11 | Verification badges are system-assigned only. |
| BR-12 | Promoted listings are labelled and cannot bypass filters or verification. |
| BR-13 | Discriminatory restrictions (religion, caste, ethnicity, disability, etc.) are not permitted in listing rules, applications, reviews, roommate matching. **\[LEGAL\]** |
| BR-14 | One property cannot have two active published listings; duplicates require owner resolution. |
| BR-15 | A unit/slot can have at most one confirmed booking at any time. |
| BR-16 | Customers may not be charged without displaying the full price breakdown and cancellation terms. |
| BR-17 | Owners cannot edit signed agreements; amendments create new versions. |
| BR-18 | Staff access to messages/documents requires assignment and is logged. |
| BR-19 | Platform fees, taxes, and policies are configuration-driven by region and category. |
| BR-20 | Accounts under 18 are not permitted. |
| BR-21 | Advance payment outside the platform is discouraged; messages requesting such payment are flagged. |
| BR-22 | Deposit refund timelines follow agreement terms and applicable law **\[LEGAL\]**. |
| BR-23 | Users cannot contact owners/customers who have blocked them. |
| BR-24 | Suspended users cannot publish, message, book, or apply. |

# 39. State Machines

Notation: `State → State [trigger/actor]`.

| Entity | States and transitions |
| --- | --- |
| **User** | Pending → Active \[verified\]; Active → Restricted \[moderation\]; Restricted → Active \[lifted\]; Active/Restricted → Suspended \[admin\]; Suspended → Active \[reinstated\]; Active → Deactivated \[user\]; Deactivated → Active \[login\]; any → Deletion Pending \[request\] → Deleted \[grace end\] |
| **Owner verification** | Draft → Submitted → Under Review → (More Info Required ↔ Under Review) → Verified / Rejected; Verified → Suspended / Re-verification Required; Rejected → Draft \[resubmit\] |
| **Property** | Draft → Submitted → Under Review → Approved/Published; Under Review → Changes Requested → Submitted; Under Review → Rejected; Published → Paused/Reserved/Rented/Sold; Published → Stale → Hidden → Archived; any → Suspended; Rented → Published \[relist\] |
| **Property verification** | Pending → Auto-checks Passed → In Manual Review → Verified / Changes Requested / Rejected / Escalated; Verified → Expired |
| **Rental application** | Draft → Submitted → Viewed → (Info Requested ↔ Submitted) → Shortlisted → Accepted → Converted to Agreement; Submitted/Viewed/Shortlisted → Rejected; any open → Withdrawn / Expired / Cancelled |
| **Visit** | Requested → Accepted → Completed; Requested → Rejected / Expired; Requested/Accepted → Reschedule Proposed → Accepted; Accepted → Cancelled / No-Show |
| **Booking** | Held → Pending Payment → Confirmed → Checked-In → Completed; Held → Expired; Confirmed → Cancelled → Refunded; Confirmed → No-Show; any paid → Disputed |
| **Payment** | Initiated → Pending → Paid; Pending → Failed / Cancelled; Paid → Refund Pending → Refunded / Partially Refunded; Paid → Disputed → Resolved; Paid → Settled → Reconciled |
| **Rental agreement** | Draft → Pending Owner Review ↔ Pending Customer Review → Pending Signatures → Partially Signed → Fully Executed; Draft/Pending → Cancelled/Expired; Executed → Amended / Terminated / Expired |
| **Support ticket** | New → Open → Pending Customer ↔ Open → Escalated → Resolved → Closed; Resolved → Reopened → Open |
| **Report** | Submitted → Triaged → In Review → (Awaiting Info ↔ In Review) → Action Taken / Dismissed → Closed; In Review → Escalated |
| **Review** | Submitted → Pending Moderation → Published / Rejected; Published → Disputed → Upheld / Removed; Published → Edited |
| **Subscription** | Pending Payment → Active → Past Due → Active/Cancelled; Active → Paused → Active; Active → Expired |
| **Listing promotion** | Draft → Pending Payment → Scheduled → Active → Completed; Active → Paused/Cancelled; any → Rejected (policy) |

---

# 40. Requirements Traceability Matrix (representative; full matrix maintained in the requirements tool)

| Req ID | Feature | Role | API / Component | Test case | Acceptance |
| --- | --- | --- | --- | --- | --- |
| AUTH-001/004 | Phone OTP registration | Customer, Owner | Auth API / Identity svc | TC-AUTH-001..012 | AC-AUTH-01 |
| VER-008 | Owner verification workflow | Owner, Verif. | Owners API / Verification svc | TC-VER-001..020 | AC-VER-01 |
| PROP-001/009 | Listing creation & submit | Owner | Properties API | TC-PROP-001..030 | AC-PROP-01/02 |
| TRUST-001 / BR-01 | No public listing before verification | System | Search index gate | TC-TRUST-001..006 | AC-TRUST-01 |
| FRESH-001..003 | Freshness ladder | Owner, System | Freshness job | TC-FRESH-001..015 | AC-FRESH-01 |
| SRCH-001..006 | Search | Customer | Search API | TC-SRCH-001..040 | AC-SRCH-01..03 |
| FLT-001 | Filters | Customer | Search/Filters API | TC-FLT-001..030 | AC-FLT-01 |
| MAP-001..003 | Map & list toggle | Customer | Map API | TC-MAP-001..015 | AC-MAP-01 |
| NEAR-001..004 | Nearby facilities | Customer | Nearby API | TC-NEAR-001..012 | AC-NEAR-01 |
| AI-002..006, AI-SAFE-001 | AI assistant & grounding | Customer | AI API / AI svc | TC-AI-001..060 | AC-AI-01..03 |
| MATCH-002/003 | Transparent matching | Customer | Matching svc | TC-MATCH-001..015 | AC-MATCH-01 |
| FAV-001 | Favourites | Customer | Favourites API | TC-FAV-001..008 | AC-FAV-01 |
| CMP-001 | Comparison | Customer | Comparisons API | TC-CMP-001..008 | AC-CMP-01 |
| COST-001..003 | True cost | Customer | Cost svc | TC-COST-001..012 | AC-COST-01 |
| CHAT-001..005 | Chat & privacy | Customer, Owner | Chat API | TC-CHAT-001..025 | AC-CHAT-01/02 |
| VIS-001..004 | Visit scheduling | Customer, Owner | Visits API | TC-VIS-001..020 | AC-VIS-01 |
| APP-001..004 | Rental application | Customer, Owner | Applications API | TC-APP-001..020 | AC-APP-01 |
| AGR-001..006 | Agreement | Both | Agreements API | TC-AGR-001..018 | AC-AGR-01 |
| PAY-004/005/011 | Payment integrity | Customer, System | Payments API / Webhooks | TC-PAY-001..040 | AC-PAY-01..03 |
| REV-001/004 | Reviews | Customer | Reviews API | TC-REV-001..015 | AC-REV-01 |
| REP-001..005 | Reporting | Customer, Moder. | Reports API | TC-REP-001..020 | AC-REP-01 |
| I18N-001..008 | Languages | All | i18n layer | TC-I18N-001..030 | AC-I18N-01 |
| SEC-004/007 | AuthZ & document privacy | All | Gateway / Storage | TC-SEC-001..050 | AC-SEC-01 |
| ADMIN-001..008 | Admin consoles | Staff | Admin API | TC-ADMIN-001..030 | AC-ADMIN-01 |
| PRIV-003 | Data subject rights | Customer, Owner | Users API | TC-PRIV-001..010 | AC-PRIV-01 |

# 41. Acceptance Criteria (AC)

| ID | Criterion (Given/When/Then, measurable) |
| --- | --- |
| AC-AUTH-01 | Given a valid phone, when OTP is entered within 5 min, then the account activates; 6th wrong attempt locks OTP for 15 min. |
| AC-VER-01 | A submitted owner reaches a decision state, with reason, in ≤ 48 h SLA for ≥ 90% of cases; every decision has an audit record. |
| AC-PROP-01 | A verified owner can create, submit, and, once approved, publish a listing that appears in search within 60 s. |
| AC-PROP-02 | An unverified owner receives 403 on publish and sees guidance to complete verification. |
| AC-TRUST-01 | No listing in Draft/Submitted/Under Review/Rejected/Suspended appears in any public search, map, API or share preview. |
| AC-FRESH-01 | A listing with no confirmation for 30 days displays "Stale Listing" and drops below fresh listings in default ranking; at 45 days it is excluded from search. |
| AC-SRCH-01 | Rent filter returns only listings within range; counts match results. |
| AC-SRCH-02 | The query "furnished 2BHK under ₹25,000 near metro" yields parsed chips (furnished, 2BHK, ≤25,000, metro) and matching results. |
| AC-SRCH-03 | Search works with Telugu and Hindi script input and transliterated Latin input for ≥ 95% of the test corpus. |
| AC-MAP-01 | Toggling list↔map preserves filters; clusters appear when > 50 markers in view. |
| AC-NEAR-01 | Detail page shows facilities with distance and category; filter "metro ≤ 1 km" returns only listings meeting it. |
| AC-AI-01 | 100% of price, availability, verification and distance claims in a golden set match platform data. |
| AC-AI-02 | When the LLM is unavailable, users see fallback search UI within 3 s. |
| AC-AI-03 | Discriminatory request prompts are refused in ≥ 99% of red-team cases. |
| AC-MATCH-01 | Match breakdown sums to the displayed score; promoted status has no effect (tested). |
| AC-FAV-01 | Favourite persists across devices; removal updates immediately. |
| AC-CMP-01 | 2–4 properties compare; 5th is blocked with message. |
| AC-COST-01 | Move-in cost equals deposit + first rent + one-time fees as configured; each line is labelled Actual/Estimated. |
| AC-CHAT-01 | Phone/email are not visible in chat or profile until reveal conditions are met. |
| AC-CHAT-02 | Blocked users cannot send messages; reports create Report IDs. |
| AC-VIS-01 | Owner accept/reject/reschedule updates both sides in ≤ 5 s with notifications; double-booking of a slot is prevented. |
| AC-APP-01 | Applications are impossible on Rented listings; accepted application sets listing to Reserved. |
| AC-AGR-01 | Executed agreement is immutable, hash-verified, and downloadable by both parties. |
| AC-PAY-01 | Payment becomes Paid only after verified webhook/server check; replayed webhook doesn't duplicate state. |
| AC-PAY-02 | Reconciliation detects an injected mismatch within one cycle. |
| AC-PAY-03 | Payment succeeded + fulfilment failed triggers auto-refund/ticket within 15 min. |
| AC-REV-01 | Only eligible users can submit a review; duplicates rejected. |
| AC-REP-01 | A report generates an ID, enters triage within SLA, and the reporter sees status updates. |
| AC-I18N-01 | Switching language updates all UI, notifications and AI replies; no untranslated keys in en/te/hi release builds. |
| AC-SEC-01 | Users cannot access others' resources by ID manipulation (tested across all endpoints); documents inaccessible without valid signed URL. |
| AC-ADMIN-01 | Admin overrides require reason and appear in audit log with before/after values. |
| AC-PRIV-01 | Data export delivered within 30 days (target ≤ 7 days); account deletion removes/anonymizes data per retention schedule. |

---

# 42. MVP (Phase 1 – P0)

| # | Capability | Included scope |
| --- | --- | --- |
| 1 | Authentication | Phone/email OTP, password, sessions, onboarding, language |
| 2 | Owner verification | ID/documents, manual verification console, states |
| 3 | Property listing | Residential (apartment, house, room, PG basic); media; freshness basics |
| 4 | Search | Keyword, location, saved search (basic) |
| 5 | Filtering | Property, financial, trust, location |
| 6 | Property details | Full detail page, badges, true cost (monthly/move-in) |
| 7 | Map | List/Map toggle, clustering |
| 8 | Nearby facilities | Core POI categories via adapter |
| 9 | Favourites | Favourite + basic collection |
| 10 | Communication | In-app chat, notifications (push/in-app/SMS/email) |
| 11 | Visits | Request/accept/reschedule/cancel + reminders |
| 12 | Reporting | Reports + moderation queue |
| 13 | Reviews | Customer → property |
| 14 | Multilingual | en/te/hi UI and notifications |
| 15 | Admin | Verification, moderation, dashboard basics, audit |
| 16 | Basic rental workflow | Application + owner decision + agreement summary + offline payment guidance |
| 17 | AI assistant | NL search, grounded explanations, fallback |

MVP exclusions: online payments, e-sign, hotels/hourly, commercial and land transactions, roommates, swipe, subscriptions, affordability.

# 43. Phase 2 (P1)

Payments (gateway live, receipts, refunds, reconciliation), digital agreements with e-sign, comparison, collections sharing, saved-search alerts, personalization, owner analytics, hotel/short-stay (STAY), PG room/bed inventory, owner reviews of properties, AI support assistant, promoted listings and subscriptions, commercial listings, land listing (enhanced verification), native mobile apps, dispute management.

# 44. Phase 3 (P2–P3)

Hourly stays, swipe discovery, roommate matching, affordability calculator, owner→customer feedback, chat auto-translation, voice assistant, on-site verification, payouts, rent collection/autopay, channel manager sync, additional Indian languages, multi-currency/international pilot.

# 45. Future Roadmap

Mortgage/loan integrations, home services, moving, furniture rental, utilities setup, tenant insurance, maintenance management, property management, smart-home integration, corporate housing, employee relocation, investment analytics. Architecture supports them through modular services and adapters without MVP complexity.

# 46. Risks and Mitigations

| ID | Risk | Impact | Mitigation |
| --- | --- | --- | --- |
| R-01 | Fake listings and scams | Trust loss | Verification, fraud signals, reporting, payment guidance |
| R-02 | Cold-start supply | Low value | City-by-city launch, owner incentives, bulk onboarding tools |
| R-03 | Regulatory (DPDP, e-sign, RBI, rent laws, RERA) | Delays/penalties | Legal review gates, configurable templates, compliant providers |
| R-04 | AI hallucination | Misinformation | Grounding, validators, evaluation, fallback |
| R-05 | POI/geo data inaccuracy | Wrong guidance | Source labelling, refresh, user corrections |
| R-06 | Owner non-responsiveness | Poor experience | Response metrics, escalation, ranking effects |
| R-07 | Off-platform leakage | Revenue/safety | Value-added workflows, detection warnings |
| R-08 | Discrimination in rules/reviews | Legal/ethical | Allowed-rule catalog, content scanning, moderation |
| R-09 | Vendor lock-in | Cost/risk | Adapters |
| R-10 | Localization quality | Adoption | Native-speaker review, translation QA |
| R-11 | Verification bottleneck | Slow publishing | Automation, SLAs, staffing, queue analytics |
| R-12 | Data breach | Severe | Security controls, minimization, IR plan |

# 47. Assumptions

| ID | Assumption |
| --- | --- |
| A-01 | Launch markets: selected Indian cities (e.g., Hyderabad first), expanding. |
| A-02 | Mobile web/PWA is the primary client at MVP; native apps in Phase 2. |
| A-03 | Third-party providers (maps, SMS, KYC, payments, e-sign, LLM) are available through compliant Indian vendors; none is selected here. |
| A-04 | Modular monolith is acceptable for MVP provided module boundaries allow later extraction. |
| A-05 | Owners are individuals or small landlords; enterprise bulk APIs are future scope. |
| A-06 | Platform acts as an intermediary/marketplace (IT Act intermediary status to be confirmed by counsel) **\[LEGAL\]**. |
| A-07 | Default thresholds (freshness, SLAs, limits) are starting values subject to pilot tuning. |
| A-08 | Hotel regulations (guest ID, GST, local licences) vary by state and require legal review before Phase 2. |
| A-09 | No in-platform money movement in MVP. |
| A-10 | Gender-restricted accommodation (e.g., women-only PG) is permitted where law allows. |

# 48. Dependencies

Map/geocoding/routing/POI provider; SMS (DLT registration) and email providers; push services; identity verification/KYC provider; e-signature and e-stamping provider; payment aggregator (RBI-authorized); LLM provider; object storage/CDN; malware and image-moderation services; legal counsel for templates and policies; translation specialists (Telugu/Hindi); verification staff hiring; cloud hosting with Indian region.

# 49. Glossary

| Term | Definition |
| --- | --- |
| BHK | Bedroom, Hall, Kitchen |
| PG | Paying Guest accommodation |
| Freshness | Recency of confirmed/updated/verified listing data |
| True Cost | Estimated monthly and move-in cost including all recurring and one-time charges |
| Adapter | Vendor-neutral interface to an external service |
| Grounding | Restricting AI claims to retrieved platform data |
| RBAC/ABAC | Role/Attribute-based access control |
| RPO/RTO | Recovery Point/Time Objective |
| DPDP | Digital Personal Data Protection Act, 2023 (India) |
| EOI | Expression of Interest |
| POI | Point of Interest |
| CAM | Common Area Maintenance |
| DLT | Distributed Ledger Technology registration for Indian commercial SMS |
| SLA/SLO | Service Level Agreement/Objective |
| IDOR | Insecure Direct Object Reference |

---

# Appendix A — Complete Feature Matrix

| Feature | Customer | Owner | Staff/Admin | Priority | Phase |
| --- | --- | --- | --- | --- | --- |
| Registration/Login/OTP | ✔ | ✔ | ✔ (MFA) | P0 | 1 |
| Owner verification |  | ✔ | ✔ | P0 | 1 |
| Listing creation & media |  | ✔ | review | P0 | 1 |
| Freshness system | view | confirm | monitor | P0 | 1 |
| Search, filters, map, nearby | ✔ |  |  | P0 | 1 |
| Property detail, badges, true cost | ✔ |  |  | P0 | 1 |
| Favourites | ✔ |  |  | P0 | 1 |
| Collections | ✔ |  |  | P0 basic / P1 full | 1–2 |
| Compare | ✔ |  |  | P1 | 2 |
| Chat | ✔ | ✔ | moderated | P0 | 1 |
| Visits | ✔ | ✔ | view | P0 | 1 |
| Rental application | ✔ | ✔ | view | P0 | 1 |
| Digital agreement | ✔ | ✔ | view | P0 basic / P1 | 1–2 |
| Payments | ✔ | ✔ | refunds | P1 | 2 |
| Hotel/short-stay | ✔ | ✔ |  | P1 | 2 |
| Hourly stay | ✔ | ✔ |  | P2 | 3 |
| Commercial/land | ✔ | ✔ | enhanced verification | P1 | 2 |
| Roommate matching | ✔ |  | moderation | P3 | 3 |
| Swipe discovery | ✔ |  |  | P2 | 3 |
| AI assistant | ✔ |  | evaluation | P0 | 1 |
| AI support assistant | ✔ | ✔ |  | P1 | 2 |
| Reviews | ✔ | respond | moderate | P0 | 1 |
| Reports & fraud | ✔ |  | ✔ | P0 | 1 |
| Support | ✔ | ✔ | ✔ | P0 | 1 |
| Multilingual | ✔ | ✔ | ✔ | P0 | 1 |
| Notifications | ✔ | ✔ | ✔ | P0 | 1 |
| Saved searches/alerts | ✔ |  |  | P0 basic / P1 alerts | 1–2 |
| Owner analytics |  | ✔ |  | P1 | 2 |
| Admin dashboards |  |  | ✔ | P0 | 1 |
| Monetization (promotions, subscriptions) |  | ✔ | config | P1 | 2 |
| Audit logs |  |  | ✔ | P0 | 1 |
| Affordability calculator | ✔ |  |  | P2 | 3 |

# Appendix B — Prioritized Roadmap

| Phase | Focus | Exit criteria |
| --- | --- | --- |
| 1 (MVP) | Trust, discovery, communication, AI, multilingual | All P0 acceptance criteria pass; security test clean; 1 city pilot |
| 2 | Transactions: payments, e-sign, short-stay, commercial/land, analytics, monetization | PAY and AGR criteria pass; legal sign-off |
| 3 | Differentiators and scale: hourly, roommate, swipe, payouts, rent collection, more languages | Multi-city scale tests pass |
| Future | Ecosystem services | Per business case |

*End of document.*