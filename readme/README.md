**Project Overview**
- **Name:**: `mmeraki-backend1` — backend for the Mmeraki platform.
- **Purpose:**: This repository contains the Node/TypeScript backend that powers the Mmeraki website and its APIs (user auth, experiences, cart, wishlist, orders).
- **Live site:**: `https://mmeraki.com/`
- **Backend URL:**: `https://mmeraki-backend1.vercel.app/`

**Quick Summary**
- **Hosted:**: Vercel
- **Database:**: Supabase (Postgres)
- **Authentication:**: Firebase (for login/identity), JWT (for API tokenization)
- **Language / Runtime:**: TypeScript / Node.js

**Repository Layout**
- **API entry:**: `api/[[...slug]].ts` (Vercel serverless entry)
- **Server:**: `src/server.ts`
- **Routes:**: `src/routes/` (auth, cart, experience, order, wishlist, seed)
- **Controllers:**: `src/controllers/`
- **Services:**: `src/services/`
- **Utils:**: `src/utils/` (JWT helpers, Supabase connector, slug generator)

**Tech Stack & Services**
- **Frontend (live):**: `https://mmeraki.com/` (separate repo)
- **Backend Hosting:**: Vercel (serverless functions)
- **Database:**: Supabase (Postgres) — used for storing users, experiences, carts, orders, wishlist
- **Authentication provider:**: Firebase Authentication (handles login flows; provider may be email/password, OAuth etc.)
- **Tokenization:**: JWT tokens are issued/validated by the backend for API authorization
- **Language & Frameworks:**: TypeScript, Node.js, likely Express-like or direct serverless handlers

**Environment & Setup (for local development)**
- **Env template:**: see `env.template` for required variables. Typical values:
  - `SUPABASE_URL` and `SUPABASE_KEY`
  - `FIREBASE_*` (Firebase config / service account as required)
  - `JWT_SECRET` (secret used to sign tokens)
  - `VERCEL_*` (optional deployment environment variables)

Install & run locally (example):
```powershell
# install
npm install

# build + run (if using ts-node or compiled setup)
npm run dev
```

**Deployment**
- Vercel: connect this Git repository to Vercel and set environment variables in the Vercel dashboard (matching `env.template`).
- On push to `main` (or configured branch) Vercel will deploy the serverless functions and the API will be available at `https://mmeraki-backend1.vercel.app/`.

**Database (Supabase)**
- The SQL schema files are available in the `database/` folder: `schema.sql`, `user_schema_simple.sql`, `wishlist_cart_schema.sql`.
- Create a Supabase project, run the SQL to create tables, then set `SUPABASE_URL` and `SUPABASE_KEY` in env.

**Authentication & JWT Flow**
- The project uses Firebase Authentication for login (email/password or OAuth).
- After the user authenticates with Firebase, the backend creates/validates local user records and issues a signed JWT for API access.
- The JWT should be included in requests via the `Authorization` header: `Authorization: Bearer <JWT>`.
- Example JWT usage:
  - Client logs in via Firebase and obtains an ID token (optional)
  - Client calls backend `/auth/login` (or direct token-exchange route) with Firebase token
  - Backend verifies Firebase token, creates/updates user, and signs a JWT using `JWT_SECRET`
  - Client stores the JWT (HTTP-only cookie or secure storage) and uses it to call protected endpoints

**API Endpoints**
Note: exact route prefixes depend on how routes are mounted. Below are common endpoints based on the repository structure.

- **Auth**
  - `POST /api/auth/register` — Register a new user
    - Body: `{"email":"user@example.com","password":"secret","name":"Full Name"}`
    - Success: `201` with created user (or message)

  - `POST /api/auth/login` — Login (Firebase identity exchange / backend login)
    - Body (example): `{"email":"user@example.com","password":"secret"}` or send a `firebaseIdToken` depending on your flow
    - Success: `200` with `{ "token": "<JWT>", "user": { ... } }`

  - `GET /api/auth/profile` — Get logged-in user profile (protected)
    - Headers: `Authorization: Bearer <JWT>`

  - `POST /api/auth/verify` — (Optional) verify email or token exchange

- **Experiences**
  - `GET /api/experiences` — List experiences
  - `GET /api/experiences/:id` — Get experience details
  - `POST /api/experiences` — Create experience (protected/admin)

- **Cart**
  - `GET /api/cart` — Get current user cart (protected)
  - `POST /api/cart` — Add item to cart
  - `PUT /api/cart/:itemId` — Update item quantity
  - `DELETE /api/cart/:itemId` — Remove item

- **Wishlist**
  - `GET /api/wishlist` — List wishlist items (protected)
  - `POST /api/wishlist` — Add to wishlist
  - `DELETE /api/wishlist/:id` — Remove from wishlist

- **Orders**
  - `GET /api/orders` — List orders for user (protected)
  - `POST /api/orders` — Create new order (protected)

**Postman / cURL Examples**

- Environment variables (Postman):
  - `baseUrl = https://mmeraki-backend1.vercel.app`
  - `jwt` (set after login)

- Register (cURL):
```bash
curl -X POST "https://mmeraki-backend1.vercel.app/api/auth/register" \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"secret","name":"Test User"}'
```

- Login (cURL):
```bash
curl -X POST "https://mmeraki-backend1.vercel.app/api/auth/login" \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"secret"}'

# Response contains token; set in Postman: {{jwt}}
```

- Protected request example (get profile):
```bash
curl -X GET "https://mmeraki-backend1.vercel.app/api/auth/profile" \
  -H "Authorization: Bearer {{jwt}}"
```

- Create order (example):
```bash
curl -X POST "https://mmeraki-backend1.vercel.app/api/orders" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {{jwt}}" \
  -d '{"items":[{"experienceId":"exp_123","qty":2}],"total":100}'
```

**Postman Collection (manual steps)**
- Create a new collection called `mmeraki-backend1`.
- Add environment variable `baseUrl` = `https://mmeraki-backend1.vercel.app` and `jwt` empty.
- Add requests for `POST {{baseUrl}}/api/auth/login`, `GET {{baseUrl}}/api/auth/profile`, and others above.
- After login, copy token from response and set `jwt` environment variable. Use `Authorization` header value `Bearer {{jwt}}` in protected requests.

**Security Notes**
- Keep `JWT_SECRET` secure (Vercel environment variable). Use strong random secret.
- Use HTTPS (Vercel provides it) for all traffic.
- Prefer storing tokens in secure, HTTP-only cookies to mitigate XSS. Use refresh tokens if implementing long-lived sessions.

**Helpful Links**
- Live frontend: `https://mmeraki.com/`
- Backend (this repo) endpoint: `https://mmeraki-backend1.vercel.app/`
- Supabase: https://supabase.com/
- Firebase Auth: https://firebase.google.com/products/auth
- Vercel: https://vercel.com/

**Notes & Next Steps**
- If you want, I can:
  - export a Postman collection JSON for you
  - add a `README.md` to repository root in addition to this `readme/README.md`
  - create example `.env.local` for local testing

---
