# Studio 8 — bilingual static site

## Structure
```
/                    Arabic (RTL) — default
  index.html  about.html  platforms.html  services.html
/en/                 English (LTR)
  index.html  about.html  platforms.html  services.html
/assets/img/         shared images (cached across all 8 pages)
```

Both language folders use identical relative filenames, so every
in-language link is just `about.html`, `platforms.html`, etc.
Only the language switcher differs:
  AR page -> `en/<same-page>.html`
  EN page -> `../<same-page>.html`

## Deploying
No build step. Upload this folder as-is.

- **Netlify** — drag the folder onto app.netlify.com/drop
- **Cloudflare Pages** — Create project > Direct Upload
- **Any host** — copy to the web root

Arabic is the default; visitors landing on `/` get the RTL site.

## Then point DNS
In ClouDNS, change the A record for `studio8.com.kw` from `35.214.54.37`
to the new host's IP, or swap it for a CNAME if the host gives you a
hostname. Leave the MX records alone — they run Google Workspace email.

## Still to come
- `services.html` / `en/services.html` are placeholders. Drop the real
  pages in at those exact paths and the nav links already work.
