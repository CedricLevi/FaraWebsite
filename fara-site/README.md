# Fara website (faraneuro.com)

Static site served by a Cloudflare Worker, same setup as cedriclevi.com.

```
wrangler.jsonc        Worker config — serves everything in Public/
Public/
  index.html          the whole site (HTML + CSS + JS in one file)
  fara-implant.glb    3D model of the implant (rendered live with three.js)
  favicon.svg / favicon.ico / apple-touch-icon.png / icon-512.png
brand/                logo files (not served) — SVG masters + 4x PNGs
```

## Editing

Everything visible is in `Public/index.html`. Brand colors are at the top of the `<style>` block:

| Token   | Hex       | Use                                   |
|---------|-----------|---------------------------------------|
| navy    | `#0E2530` | logo disc, main background            |
| cream   | `#F4F1EA` | wordmark, light sections              |
| coral   | `#F08A6C` | seizure activity / the problem        |
| teal    | `#4CC3B1` | Fara / the barrier / buttons          |

Before going public, check: the timeline statuses, team bios,
and anything about unpublished data (e.g. the human-data step).

## Domains

- **faraneuro.com** — main site (Worker custom domain)
- **www.faraneuro.com** — 301 → faraneuro.com
- **fara-neuro.com**, **faraimplants.com** — 301 → faraneuro.com (keeps the path)

### 1. Add each domain to Cloudflare
Cloudflare dashboard → **Add a domain** → `faraneuro.com` → Free plan. Delete any imported
Porkbun parking records (ALIAS/CNAME to `pixie.porkbun.com`, `uixie.porkbun.com`).
Note the two nameservers Cloudflare gives you. Repeat for `fara-neuro.com` and `faraimplants.com`.

### 2. Point Porkbun at Cloudflare
Porkbun → **Account → Domain Management** → domain → **Details** → **Nameservers** (edit) →
remove the Porkbun entries, paste Cloudflare's two → **Save Nameservers**.
If DNSSEC is on in Porkbun, turn it off first. Remove any Porkbun URL forwarding.
Cloudflare emails you when each domain is **Active** (usually under a few hours, up to 48).

### 3. Deploy the site
Create a GitHub repo (e.g. `CedricLevi/fara-site`) with these files →
Cloudflare **Workers & Pages → Create → Import a repository** → pick the repo → Deploy.
Then Worker → **Settings → Domains & Routes → Add → Custom domain** → `faraneuro.com`.

### 4. www → apex (faraneuro.com zone)
DNS: `A  www  192.0.2.1  Proxied`.
**Rules → Redirect Rules** → template **Redirect from WWW to root** (or custom:
hostname equals `www.faraneuro.com` → Dynamic `concat("https://faraneuro.com", http.request.uri.path)`, 301, preserve query string).

### 5. Redirect the other two domains (do in each zone)
DNS: `A  @  192.0.2.1  Proxied` and `A  www  192.0.2.1  Proxied`.
**Rules → Redirect Rules → Create rule** → **All incoming requests** →
Type **Dynamic**, expression `concat("https://faraneuro.com", http.request.uri.path)`,
status **301**, **Preserve query string** on.

### 6. Test
`https://fara-neuro.com/anything` and `https://www.faraimplants.com` should both land on
`https://faraneuro.com/...`.

### Optional: email
Cloudflare **Email → Email Routing** on faraneuro.com can forward `connect@faraneuro.com`
to your inbox for free.
