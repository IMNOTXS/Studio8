# Publishing studio8.com.kw on GitHub Pages

Repo: `IMNOTXS/Studio8` — values below are filled in.
Do the steps **in this order** — GitHub first, DNS second.

---

## Step 1 — GitHub (do this BEFORE touching DNS)

GitHub is explicit about the ordering: adding the domain at your DNS
provider before claiming it in GitHub can let someone else host a site
on your subdomain.

- [x] **Custom domain set.** GitHub already created the `CNAME` file on the
      remote containing `studio8.com.kw`. This is what caused the rejected
      push — see "Resolving the push rejection" below.
- [ ] **Confirm the publishing source.** Repo → **Settings** → **Pages** →
      Source: Deploy from a branch → `main` → `/ (root)` → **Save**
- [ ] **Confirm the build works** at `https://imnotxs.github.io/Studio8/`
      Check the Arabic homepage *and* click through to `/en/` before
      touching DNS.
- [ ] Leave **Enforce HTTPS** unchecked until DNS has propagated.

### Resolving the push rejection

The remote has two commits your local clone doesn't: GitHub's `CNAME`
commit, and your README edit. Your local commit only touches HTML and
`logo.png`, so there is no file overlap and no possible conflict.

```sh
git pull --rebase
git push
```

`--rebase` replays your commit on top of the remote's, keeping history
linear instead of adding a merge commit.

Note: pulling will also bring down your emptied `README.md`, replacing
the local 34-line version.

### While you're there — stop tracking .DS_Store

macOS junk files got committed by `git add .`. A `.gitignore` is now in
the folder; this untracks the two already committed:

```sh
git rm --cached .DS_Store assets/.DS_Store
git add .gitignore
git commit -m "Ignore .DS_Store"
```

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
| **Edit** | CNAME | `www.studio8.com.kw` | `imnotxs.github.io` *(currently points to `studio8.com.kw`)* |

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
`imnotxs.github.io` directly — no repo name.

---

## Step 3 — Verify

TTL on these records is 1 hour, so allow at least that. GitHub says
propagation can take up to 24 hours.

```sh
dig studio8.com.kw +noall +answer -t A
# expect the four 185.199.10x.153 addresses

dig www.studio8.com.kw +noall +answer
# expect CNAME -> imnotxs.github.io

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
