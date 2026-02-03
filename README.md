
---

## Domain Model (Django)

### Catalog hierarchy (confirmed)
`StoneType > Category > SubCategory > Product > ProductVariant`

### Core entities
- **StoneType** — top-level classification (localized fields, icon, active/sort)
- **Category** — belongs to StoneType (localized, image, slug, active/sort)
- **SubCategory** — belongs to Category (localized, image, slug, active/sort, JSON config)
- **Product** — marketing layer (localized titles/descriptions, SEO, flags, preview image); holds filter attributes and relations
- **ProductVariant** — sellable unit (SKU, quantity, size/weight, gem attributes, custom JSON)
- **VariantImage** — ordered images per variant with a single main image enforced

### Dictionaries (filters + “picker” UX)
- **Shape**, **Cut**, **Color** — localized dictionaries used by filters and picker pages
- **CutMedia** — gallery images for cut pages
- **ZodiacSign** + **ZodiacStoneRecommendation** — zodiac data and curated product recommendations

### Pricing & currency
- **Currency** — UAH/USD/EUR (base pricing currency: USD)
- **ExchangeRate** — NBU or manual rate per date (cached + synced)
- **VariantPrice** — USD price + price type (per piece/carat/gram), optional sale window

### Orders & payments
- **Order** — cart/checkout + delivery/payment fields (totals stored in UAH)
- **OrderItem** — fixed price snapshot per variant at purchase time
- **Payment** — provider, status, amount (UAH), external ids

### Users
- **User** — role, profile, discount flags, email verification
- **UserFavorite** — wishlist items
- **UserCompareItem** — compare list items

---

## Architecture & Data Flow

### Frontend → Backend
- Next.js calls DRF endpoints via `NEXT_PUBLIC_API_URL`.
- In dev, an optional API proxy (`/app/api/[...path]`) forwards requests to reduce CORS/cookie friction.

### Authentication
- Access token (JWT) stored in localStorage and attached as `Authorization: Bearer ...` via Axios interceptor.
- Refresh token stored as **HttpOnly cookie** (set by backend).
- Refresh flow handled by `/api/users/token/refresh/` with a retry queue to prevent multiple simultaneous refreshes.

> Note: exact token behavior can differ between dev/prod depending on HTTPS and cookie settings.

### Pricing
- Base prices are stored in USD (`VariantPrice`).
- Final checkout totals are stored in **UAH**; calculated using NBU rates and saved on `Order` / `OrderItem` for auditability.

### Filtering strategy
- Server-side filtering via `django-filter` with query params.
- Supports: search, hierarchy slugs, dictionary slugs, multi-selects, JSON array tags (e.g. genders/purposes/zodiac), and flags.

### Media pipeline
- Django stores media in S3-compatible R2 (production).
- Next.js image config whitelists R2 and optional custom media domains.

---

## Key Features & User Flows

### Public storefront
- Landing, catalog, product details
- Advanced filters (shape/cut/color + multi-select + tags)
- “Picker” experience (zodiac, color, shape, cut)
- Sale listings + featured sections
- Utility/content pages (calculator `/calc`, packaging, policy, contacts, about, etc.)

### User flows
- Email verification during signup
- Wishlist (favorites) and compare list
- Cart + checkout
- Profile & order history

### Admin workflows
- **Django Admin**: CRUD for catalog entities, dictionaries, variants, pricing, orders, payments, zodiac data, site settings
- **Custom Admin (Next.js)**: dashboard, catalog management, pricing, orders, users, site settings
- Admin access can be controlled via an allowlist (configured via env)

---

## Integrations

### NBU exchange rates
- Rates pulled from the National Bank of Ukraine endpoint and cached.
- `maybe_sync_nbu_rates()` uses cache-based locking to limit sync frequency.

### Nova Poshta (delivery)
- City/warehouse search endpoints
- Admin order actions to generate waybills and download PDFs
- Sender profile resolves via NP APIs (configured via env)

### Cloudflare R2
- S3-compatible storage (`django-storages` + `boto3`)
- Next.js image loader whitelists R2 domains and optional custom media host

### Hosting
- Render build/start commands documented in `DEPLOY.md` / `DEPLOYMENT.md`
- Production DB via Neon Postgres using `DATABASE_URL`

---

## API Overview (DRF)
Grouped by route prefix (high-level summary).

- `/api/catalog/` — hierarchy, products, dictionaries, zodiac, sale, admin search
- `/api/inventory/` — variants, variant images
- `/api/pricing/` — currencies, exchange rates, variant prices, quote/reprice/sync tools
- `/api/orders/` — cart actions, checkout, staff CRUD, Nova Poshta waybill/pdf
- `/api/users/` — auth, email verify, refresh, profile, favorites/compare, admin stats
- `/api/site-settings/` — footer/contacts/socials (public + admin update)
- `/api/delivery/` — NP city/warehouse search
- `/api/payments/` — payment records

**OpenAPI/Swagger:** not enabled in the current repository state.

---

## Notable Decisions & Tradeoffs
- **Product vs Variant split**: marketing data on Product; sellable specifics, stock and images on Variant.
- **USD base pricing + UAH checkout**: stable global pricing; stored UAH totals for auditability.
- **JSON tag fields**: flexible tagging with custom filtering logic.
- **Refresh token cookie**: reduces XSS exposure for refresh tokens while keeping access token short-lived.
- **Server-side filtering**: consistent backend-driven catalog behavior (more API complexity, but predictable UX).

---

## Local Development

### Prerequisites
- Python 3.11+
- Node.js 18+
- Optional: Postgres (SQLite is used by default for local dev)

### Backend
```bash
cd backend
python -m venv venv
# Windows: venv\Scripts\activate
# macOS/Linux: source venv/bin/activate
pip install -r requirements.txt
python manage.py migrate
python manage.py runserver
