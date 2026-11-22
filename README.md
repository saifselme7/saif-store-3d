# saif-store-3d

Arabic-first RTL apparel storefront and admin console built with React, TypeScript, Three.js (React Three Fiber), Framer Motion, and Supabase (PostgreSQL RLS + Edge Functions).

Live: [demo3shop.vercel.app](https://demo3shop.vercel.app)

## Architecture Notes

- **Private payment-proof upload flow:** Payment screenshots for Vodafone Cash and InstaPay are submitted through the `create-order` Supabase Edge Function (`supabase/functions/create-order/index.ts`). Using only the publishable key, the function creates a signed upload URL scoped to `pending/` (where `anon` has `INSERT`-only storage access and zero `SELECT` policy), uploads the validated file, and invokes the `create_order` RPC, which verifies the storage object exists before mutating stock or writing the order.
- **Server-side order integrity & scoped tracking:** Prices, variant stock, totals, and delivery fields are validated inside PostgreSQL RPCs. Admins inspect payment screenshots through short-lived signed URLs authorized by `public.is_admin()`. Customers track orders via `get_public_order_status(order_number, phone)`, which returns non-sensitive status fields only when both identifiers match.
- **Offline preview fallback:** When Supabase environment variables are absent, the storefront runs against a local catalog and browser-persisted cart/order state so the UI can be previewed without backend credentials.
- **3D & media stack:** Integrates React Three Fiber / Drei for the 3D hero scene alongside Framer Motion transitions and `hls.js` support.

## Stack

- **Frontend:** React 18, TypeScript, Vite, Tailwind CSS
- **3D & Motion:** Three.js, `@react-three/fiber`, `@react-three/drei`, Framer Motion, `hls.js`
- **Backend:** Supabase (PostgreSQL, RLS, RPCs, Private Storage, Deno Edge Functions)

## Routes

- Public storefront: `/`, `/products`, `/category/:slug`, `/product/:slug`, `/cart`, `/checkout`, `/success/:orderNumber`, `/track`
- Admin console: `/admin` (unlisted in public nav; requires an authenticated user with `profiles.role = 'admin'`)

## Local Setup

```bash
npm install
cp .env.example .env.local
# Optional: set NEXT_PUBLIC_SUPABASE_URL and NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY
# (vite.config.ts exposes NEXT_PUBLIC_ via envPrefix)
npm run dev
```

Verification:

```bash
npm run typecheck
npm run build
```

For full database schema, RLS policies, Edge Function deployment, and first-admin provisioning, see [`supabase/README.md`](./supabase/README.md).

## License

[MIT](./LICENSE)
