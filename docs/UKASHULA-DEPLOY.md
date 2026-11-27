# Ukashula private deploy of bolt.diy

The Worker `ukashula-bolt` runs on Ukashula's Cloudflare account, behind Cloudflare Access, for the team only.

- No API keys are stored on the server; each user enters their own AI key in the app.
- **Testing only (prototype/POC).** Staff must not use it for real work until StackBlitz confirms in writing that a licence is not needed, or Ukashula buys one.

## WebContainers licence (checked 2026-11-27)
| # | Finding | Source | Status |
|---|---|---|---|
| 1 | The npm package `@webcontainer/api@1.6.5-internal.2` is MIT: "Copyright (c) 2022-25 StackBlitz Inc. Permission is hereby granted, free of charge…" | the package's LICENSE file in node_modules | Verified |
| 2 | "bolt.diy is open-source under the MIT License. It utilizes the WebContainers API, which has specific commercial licensing terms for enterprise deployment." | README.md, Licensing | Verified |
| 3 | A licence is required for "production usage in a for-profit setting". | webcontainers.io/enterprise, seen in a search snippet only | Not verified |
| 4 | "Prototypes or POCs do not require a commercial license." | an earlier bolt.diy README, seen in a search snippet only | Not verified |
| 5 | A licence is needed when the API is used "to meet the needs of your customers, prospective customers, and/or employees". If so, a staff-only tool counts as production use. | an earlier bolt.diy README, quoted on Medium | Not verified |

**Conflict:** finding 1 (MIT, free) disagrees with findings 3 and 5 (paid licence for for-profit production use). One possible reading is that MIT covers the client code, while StackBlitz's terms govern the hosted runtime it connects to. That reading is not confirmed. No price is published; the reported route is to contact StackBlitz.

**Question to send to StackBlitz (via webcontainers.io/enterprise):**
> Ukashula is a small for-profit company. We want to run bolt.diy, which uses the WebContainer API, as an internal tool for about [TO BE CONFIRMED] staff. It will not be offered to customers. Does this need a WebContainer API licence, and if so, what does it cost?

## Verified before deploy (2026-11-27, commit 72db5ce)
- `pnpm install --frozen-lockfile`: exit 0.
- `pnpm run build`: exit 0.
- `wrangler deploy --dry-run --name ukashula-bolt`: total upload 6,338.61 KiB, gzip 1,400.70 KiB.

The free-plan Worker size limit has not been checked against Cloudflare's docs. If the deploy is refused for size, that is the cause.

## 1. Deploy (on the Mac)
No token is shared: wrangler signs in through the browser.
```bash
git clone https://github.com/Ukashula/bolt.diy ~/bolt.diy && cd ~/bolt.diy && git checkout ukashula/private-deploy
```
```bash
pnpm install && npx wrangler login
```
Sign in to the **Ukashula** Cloudflare account, not Yendra.
```bash
pnpm run deploy
```
It prints the URL, `ukashula-bolt.<subdomain>.workers.dev`. Do not share the URL yet.

## 2. Lock it down (Cloudflare dashboard)
1. Zero Trust → Access → Applications → Add an application → Self-hosted. Domain: the URL from step 1.
2. Add a policy named "Ukashula team": Action Allow, Include Emails, listing the team's addresses.
3. Workers & Pages → ukashula-bolt → Settings → Domains & Routes: turn off Preview URLs.

## 3. Verify
- In a private browser window, the URL must ask for the Access sign-in before anything loads.
- A listed email gets the app; an unlisted email is refused.
- Workers & Pages → ukashula-bolt → Settings → Variables and Secrets shows no API keys.

## Rollback
```bash
npx wrangler delete ukashula-bolt
```
Then delete the Access application.
