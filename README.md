# golfgames-web

Static marketing + legal site for [GolfGames](https://golfgames.gg).

Served at `https://golfgames.gg/`. Three pages:

- `/` — landing
- `/privacy/` — privacy policy
- `/terms/` — terms of service

The mobile app links to `/privacy` and `/terms` from the paywall and Settings → Legal (see `src/config/monetization.ts → LEGAL_URLS`). Apple verifies these URLs during App Store review. If they don't resolve over HTTPS at submission time, the binary is rejected.

## Layout

```
.
├── index.html          # landing
├── privacy/
│   └── index.html      # /privacy/ via clean URL
├── terms/
│   └── index.html      # /terms/  via clean URL
└── styles.css          # shared stylesheet
```

Plain HTML + CSS. No build step, no JS.

## ⚠️ Before publishing — review these

1. ~~**Governing law in `terms/index.html`**~~ — set to Missouri / Jackson County (Apr 2026).
2. **Effective date** — both pages are dated April 27, 2026. Update if you change anything before launch.
3. **Operator name** — `terms/index.html §1` says "the publisher listed in the App Store and Google Play listing". Intentionally left generic for now — revisit if/when an LLC is formed.
4. **Support email** — `support@golfgames.gg` is referenced throughout. Forwarding alias set up via Cloudflare Email Routing → `jamesjfisher@gmail.com` (see deploy steps below).
5. **Lawyer review** — these documents are reasonable boilerplate written specifically against what the GolfGames app does today (Supabase, RevenueCat, Apple/Google IAP, Resend, Expo notifications, no third-party analytics). They are NOT a substitute for review by an attorney licensed in your jurisdiction. Review before relying on them in any dispute.

## Deploying with Cloudflare Pages

1. **Push this repo to GitHub** (e.g. `jamesjfisher/golfgames-web`).
2. **Sign in to [Cloudflare](https://dash.cloudflare.com/)** → Workers & Pages → Create → Pages → Connect to Git.
3. Choose the `golfgames-web` repo. Build settings:
   - Framework preset: **None**
   - Build command: *(leave empty)*
   - Build output directory: `/`
4. Deploy. You'll get a `*.pages.dev` URL — test it.

### Custom domain

Once the `pages.dev` site works:

1. **Move the domain to Cloudflare** (recommended for simplest setup):
   - Cloudflare → Add a Site → enter `golfgames.gg`.
   - At Namecheap, change the nameservers to the two Cloudflare nameservers Cloudflare gives you. Propagation takes minutes to a few hours.
2. **Attach the domain to the Pages project**:
   - In the Pages project → Custom domains → Set up a custom domain → `golfgames.gg`.
   - Cloudflare auto-creates the DNS records and provisions an SSL cert.
3. (Optional) Add `www.golfgames.gg` and 301-redirect it to the apex.

If you don't want to move nameservers, you can keep DNS at Namecheap and add a CNAME from `golfgames.gg` (or `www`) to the `pages.dev` URL — but apex CNAMEs are awkward at Namecheap; moving NS to Cloudflare is the easier path.

### Verify

After DNS propagates:

```
curl -I https://golfgames.gg/
curl -I https://golfgames.gg/privacy/
curl -I https://golfgames.gg/terms/
```

All three should return `200 OK` over HTTPS.

## Future additions

When we tackle Universal Links (backlog item: Auth & Security → "Universal Links"), this site will also need to host:

- `/.well-known/apple-app-site-association` (no extension, served as `application/json`)
- `/.well-known/assetlinks.json`

Both go in a top-level `.well-known/` directory. Cloudflare Pages serves them with the correct content type automatically.
