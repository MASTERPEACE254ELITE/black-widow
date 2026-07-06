[# Hotspot Console — MikroTik Hotspot Billing Platform

A multi-tenant SaaS platform for MikroTik hotspot resellers ("sellers") to manage their routers,
issue WiFi vouchers, and get paid, while you (the platform owner) bill sellers a recurring
subscription via **Paystack**.

## What's included

- `backend/` — Node.js/Express API, PostgreSQL via Prisma, WireGuard + RouterOS API integration,
  RADIUS control layer, Paystack billing + webhooks, subscription-expiry enforcement cron.
- `frontend-php/` — plain PHP pages (no framework) for the seller dashboard and admin dashboard.
  Pages call the Node API server-side via cURL and render HTML directly — no JS build step, no
  Node needed to serve the UI. Session-based login (JWT from the API is stored in `$_SESSION`).
- `docker-compose.yml` — local Postgres for development.

## What you need to run this for real (not just locally)

This app talks to real infrastructure that this code can't spin up on its own. Set these up once:

### 1. A small Ubuntu VPS (this is your "platform server")
Any $5–10/mo VPS (Nairobi/Europe region for low latency to Kenya is fine). This is where your
backend, WireGuard server, and RADIUS server will run.

### 2. WireGuard server (lets you reach routers behind CGNAT/dynamic IP)
```bash
sudo apt install wireguard
wg genkey | tee server_private.key | wg pubkey > server_public.key
```
Create `/etc/wireguard/wg0.conf`:
```
[Interface]
PrivateKey = <contents of server_private.key>
Address = 10.20.0.1/24
ListenPort = 51820
```
```bash
sudo systemctl enable --now wg-quick@wg0
```
Put the server's **public** key and your VPS's public IP into `backend/.env`
(`WG_SERVER_PUBLIC_KEY`, `WG_SERVER_ENDPOINT`). The backend process needs permission to run
`wg` commands (run it as root, or grant passwordless sudo for the `wg` binary to its user) since
`src/services/wireguard.js` calls the `wg` CLI directly to add/remove peers.

### 3. FreeRADIUS + a small admin API in front of it
Install FreeRADIUS on the same VPS. FreeRADIUS itself has no REST API, so `src/services/radius.js`
expects a small internal HTTP service (write ~100 lines of Express that read/write FreeRADIUS's
SQL tables for NAS clients and users) listening on `RADIUS_ADMIN_URL`. This indirection is
intentional — it keeps FreeRADIUS's config file surface out of your main app and lets you swap in
a different RADIUS setup later without touching the rest of the platform.

### 4. Paystack account
- Create subscription **Plans** in your Paystack dashboard (one per pricing tier) and put their
  plan codes into `backend/.env` (`PAYSTACK_PLAN_STARTER` etc).
- Set your webhook URL in the Paystack dashboard to `https://yourdomain.com/api/paystack/webhook`.
- Put your secret/public keys into `backend/.env`.

## Local development setup

```bash
# 1. Start Postgres
docker compose up -d

# 2. Backend
cd backend
cp .env.example .env    # fill in real values as you get them
npm install
npx prisma migrate dev --name init
npm run prisma:generate
node prisma/seed.js you@yourplatform.com yourpassword   # creates your first admin login
npm run dev              # API on http://localhost:4000

# 3. Subscription-expiry cron (separate process)
npm run cron

# 4. Frontend (PHP — no build step, no npm needed)
cd ../frontend-php/public
php -S localhost:8000
# Visit http://localhost:8000
```

The PHP frontend talks to the backend at the URL set in the `API_BASE_URL` environment variable
(defaults to `http://localhost:4000/api`). Set `FRONTEND_URL` to wherever the PHP site is
reachable (e.g. `http://localhost:8000` locally, or `https://app.yourdomain.com` in production) —
the backend uses it to build the Paystack checkout return link.

In production, point your web server (Apache/Nginx + PHP-FPM) at `frontend-php/public/` as the
document root, and keep `frontend-php/includes/` outside the web root (or block it via
`.htaccess`/Nginx `location` rules) since it isn't meant to be served directly.

Note: `wg`, RouterOS API calls, and RADIUS calls will fail locally unless you're actually running
those services — that's expected. You can still develop and test the dashboards, auth, voucher
generation, and Paystack checkout flow without them; those failures are caught and logged rather
than crashing the app.

## How the pieces connect, end to end

1. Seller signs up → gets a 7-day trial subscription automatically (or redeems an admin-issued
   trial token for a longer period).
2. Seller adds a router in their dashboard → backend generates a WireGuard keypair + config
   script → seller pastes it into the MikroTik terminal once → router dials out and appears
   "online" in the dashboard.
3. Seller clicks "Provision hotspot" → backend talks to the router over the WireGuard tunnel via
   the RouterOS API, sets up the hotspot server + captive portal, and points it at your central
   RADIUS server.
4. Seller creates packages and generates vouchers → vouchers become valid RADIUS logins → end
   customers buy/enter a voucher code on the captive portal to get online.
5. Seller subscribes via Paystack → webhook marks their subscription `ACTIVE` and re-enables
   RADIUS access for their routers.
6. A cron job checks every 15 minutes for expired trials/subscriptions and immediately disables
   RADIUS access for any seller who hasn't paid — their end customers can no longer authenticate
   until they renew.
7. You (admin) can see every seller, their revenue, suspend/reactivate them, and issue trial
   tokens from `/admin`.

## Suggested next steps once this is live

- Add M-Pesa STK Push as a second payment method for end-customer voucher purchases (the
  `Payment` model and RADIUS voucher flow are already structured to support a second payment
  provider without a rewrite).
- Add SMS notifications (e.g. via Africa's Talking) for low-data/low-time warnings.
- Build the actual captive portal login page (a simple branded HTML page served by RouterOS
  itself, customized per seller using `brandLogoUrl`/`brandColor` from the Seller model).
](http://localhost:8080)
