# Altegra — Gemstone E-Commerce Platform | Portfolio Case Study

A production-grade gemstone e-commerce platform built end-to-end with a modern **Next.js** storefront and a **Django / DRF** backend.  
This repository is a **public case study** (README + screenshots). The production source code and configuration live in a private repository and can be shared on request.

**Live demo:** https://altegra-frontend.onrender.com/en  
**Disclaimer:** The project is still in active development. The catalog may contain demo data. Please don’t create fake orders or spam the system.

---

## Why this project is strong
- ✅ **100% solo, full-cycle:** architecture, UI/UX, frontend, backend, database modeling, integrations, deployment
- ✅ **Fully responsive:** mobile / tablet / desktop (adaptive layout across all screens)
- ✅ **Real e-commerce flows:** catalog discovery → selection → cart → checkout → orders
- ✅ **Admin tooling:** operational workflows for managing catalog, pricing, content, and orders
- ✅ **Localization + currencies:** UA/EN and UAH/USD/EUR with live rate sync
- ✅ **Production integrations:** Nova Poshta logistics, Cloudflare R2 media storage, Render hosting, Neon Postgres

---

## Tech stack (confirmed)

### Frontend
- Next.js **15.4.3** (App Router), React **18.3.1**, TypeScript
- Tailwind CSS, Framer Motion, Swiper, next-intl
- Axios (API client)

### Backend
- Django **6.0**, Django REST Framework **3.16.1**
- Simple JWT, django-filter, django-cors-headers
- django-storages + boto3 (S3-compatible storage for Cloudflare R2)

### Infrastructure
- Render (frontend + backend), Neon Postgres (prod), SQLite (local), Cloudflare R2 (media)

---

## Product scope

### Public storefront
- Landing page with curated sections and guided discovery
- Catalog with advanced filters and sorting
- Product details with **variant selection**
- **Picker** flow for users who don’t know what to choose:
  - Zodiac selection
  - Color palette selection
  - Shape / cut selection
- Utility & content pages (examples):
  - Calculator (`/calc`)
  - Packaging / policies / contacts / about
  - News / exclusive jewelry mini-catalog
  - Discount program (planned)

### User flows
- Registration + login with **email verification**
- **Favorites (wishlist)**
- **Compare list**
- **Cart** and **checkout**
- Profile + order history

### Admin workflows
- Catalog management (hierarchy + dictionaries)
- Products & variants management (pricing, stock, media)
- Orders management + delivery workflow
- Site settings / content management

---

## Key UX modules

### 1) Catalog discovery
The catalog is designed around fast “find what you want” flows:
- server-driven filters (shape / cut / color + tags)
- multi-select filtering
- curated / featured blocks

### 2) Smart Picker
A guided selection experience that converts “I don’t know what to buy” into a short list:
- **Zodiac → recommendations**
- **Color palette → matching stones**
- **Shape / cut → visual preference**

### 3) Variant-based selling
A product is the “marketing shell”, while the **variant** is what the customer buys:
- cleaner pricing logic
- real stock tracking per SKU/variant
- independent media galleries per variant

---

## Domain model (high-level)

Catalog hierarchy:
`StoneType → Category → SubCategory → Product → ProductVariant`

Supporting modules (examples):
- Dictionaries: `Shape`, `Cut`, `Color`, `CutMedia`
- Zodiac: `ZodiacSign`, `ZodiacStoneRecommendation`
- Pricing: `Currency`, `ExchangeRate`, `VariantPrice`
- Orders: `Order`, `OrderItem`
- Payments: `Payment`
- Users: `User` + `UserFavorite` + `UserCompareItem`

---

## Architecture overview

### Frontend → Backend
- Next.js consumes DRF endpoints (configured via environment variables in production)
- In development, an optional proxy route can be used to reduce CORS/cookie friction

### Authentication (high-level)
- JWT access token used for authenticated calls
- Refresh token stored via secure cookie (depending on environment settings)
- Centralized refresh flow to avoid parallel refresh requests

### Pricing (high-level)
- Base pricing stored in **USD**
- Checkout totals stored in **UAH** using NBU exchange rates
- Order items store a price snapshot at purchase time for auditability

### Filtering strategy
- server-side filtering with `django-filter`
- supports search, slug-based hierarchy, dictionary filters, multi-selects, and tag arrays

### Media pipeline
- Django stores media in **Cloudflare R2** (S3-compatible)
- Next.js image configuration allows remote media domains

---

## Integrations (high-level)

### NBU exchange rates
- Exchange rates are synced from the National Bank of Ukraine
- Cached / throttled sync to avoid unnecessary requests

### Nova Poshta (delivery)
- City / warehouse search
- Order workflow supporting shipping document / waybill actions (admin side)

### Cloudflare R2
- S3-compatible object storage for product/variant media
- Supports scalable media delivery without storing large assets on the app servers

### Hosting (Render + Neon)
- Frontend and backend are deployed as services on Render
- Production database is Neon Postgres

---

## API surface (grouped, high-level)
> This is a conceptual overview (not a full endpoint dump).

- `/api/catalog/` — hierarchy, products, dictionaries, zodiac, sale, admin search
- `/api/inventory/` — variants + variant images
- `/api/pricing/` — currencies, exchange rates, variant prices, quote/reprice tools
- `/api/orders/` — cart actions, checkout, staff order management, Nova Poshta workflow
- `/api/users/` — auth, email verification, profile, favorites, compare, admin stats
- `/api/site-settings/` — footer/contacts/socials (public + admin update)
- `/api/delivery/` — Nova Poshta city/warehouse search
- `/api/payments/` — payment records

---

## Screenshots

All screenshots are stored in the `screens/` folder.

### Home
- `screens/home1.png`
- `screens/home2.png`
- `screens/home3.png`

### Catalog / Categories
- `screens/cat1.png`
- `screens/cat2.png`
- `screens/jewCat.png` — exclusive jewelry / mini-catalog

### Product details
- `screens/details1.png`
- `screens/details2.png`

### Sale
- `screens/sale.png`

### Auth
- `screens/login.png`
- `screens/reg.png`

### User flows
- `screens/cart.png`
- `screens/checkout.png`
- `screens/fav.png`
- `screens/compare1.png`
- `screens/compare2.png`
- `screens/profile.png`

### Admin
- `screens/admin.png`

> Tip: keep the same browser zoom/device scale for all screenshots for a consistent portfolio look.  
> If any screen contains sensitive data (emails, addresses, IDs) — blur/cover it.

---

## Notes on privacy & security
- Secrets and production configuration are stored in environment variables and **not** published here.
- This repository intentionally omits private implementation details while keeping the product scope and architecture clear.

---

## Code access
The production source code is private because it contains operational configuration and integration details.  
**Private code access can be shared upon request.**
