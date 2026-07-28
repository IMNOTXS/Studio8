# Publishing studio8.com.kw on GitHub Pages

Replace `USERNAME` below with your GitHub username (or org name).
Do the steps **in this order** — GitHub first, DNS second.

---

## Step 1 — GitHub (do this BEFORE touching DNS)

GitHub is explicit about the ordering: adding the domain at your DNS
provider before claiming it in GitHub can let someone else host a site
on your subdomain.

1. Commit `.nojekyll` (already in this folder) and push.
   It skips the Jekyll build step — faster deploys, no surprises.
2. Repo → **Settings** → **Pages**
3. **Source**: Deploy from a branch → `main` → `/ (root)` → **Save**
4. Wait ~1 minute, then confirm the site loads at
   `https://USERNAME.github.io/REPO/`
   Check the Arabic homepage *and* click through to `/en/` before continuing.
5. Under **Custom domain**, enter `studio8.com.kw` → **Save**
   - This commits a `CNAME` file to the repo root
   - GitHub will report "DNS check in progress" — expected, DNS isn't changed yet
   - Leave **Enforce HTTPS** unchecked for now

---

## Step 2 — ClouDNS

Zone: `studio8.com.kw`

### Change these

| Action | Type | Host | Points to |
|---|---|---|---|
| **Delete** | A | `studio8.com.kw` | `35.214.54.37` *(the old WordPress site)* |
| **Add** | A | `studio8.com.kw` | `185.199.108.153` |
| **Add** | A | `studio8.com.kw` | `185.199.109.153` |
| **Add** | A | `studio8.com.kw` | `185.199.110.153` |
| **Add** | A | `studio8.com.kw` | `185.199.111.153` |
| **Edit** | CNAME | `www.studio8.com.kw` | `USERNAME.github.io` *(currently points to `studio8.com.kw`)* |

All four A records are required — GitHub load-balances across them.

### Do NOT touch

| Type | Host | Points to | Why |
|---|---|---|---|
| MX | `studio8.com.kw` | `1 smtp.google.com` | **Live Google Workspace email.** Deleting this breaks mail immediately. |
| NS | `studio8.com.kw` | `ns41–ns44.cloudns.net` | Your nameservers. |

### Why the www record must change

It currently points at the apex. GitHub's docs warn that pointing a
subdomain at your apex domain causes HTTPS enforcement to fail and the
subdomain may not reach the site at all. It must point to
`USERNAME.github.io` directly — no repo name.

---

## Step 3 — Verify

TTL on these records is 1 hour, so allow at least that. GitHub says
propagation can take up to 24 hours.

```sh
dig studio8.com.kw +noall +answer -t A
# expect the four 185.199.10x.153 addresses

dig www.studio8.com.kw +noall +answer
# expect CNAME -> USERNAME.github.io

dig studio8.com.kw +noall +answer -t MX
# expect 1 smtp.google.com  — confirms email is untouched
```

---

## Step 4 — HTTPS

Once GitHub Settings → Pages shows the domain as verified, tick
**Enforce HTTPS**. The certificate is issued automatically and free.
The option can take up to 24 hours to become available.

---

## Rollback

If anything goes wrong, set the apex A record back to `35.214.54.37`
and the old WordPress site returns. Nothing about this change is
destructive to the old site — it stays running on its own server until
someone shuts it down.

---

## Notes

- **Repo must be public** for GitHub Pages on a free account. Private
  repos need GitHub Pro.
- `game.studio8.com.kw` had no DNS record in the zone. If that subdomain
  is still meant to work, it needs its own record — unrelated to this change.
- The old site keeps running and keeps costing someone money. Worth
  telling the client to decommission it once the new site is confirmed live.
