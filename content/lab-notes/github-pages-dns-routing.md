+++
date = '2026-06-17T19:19:07-03:00'
draft = false
title = 'Hosting a Static Site on GitHub Pages with a Custom Domain'
tags = ['github', 'dns', 'devops', 'hosting']
+++

GitHub Pages is a free static hosting service built into every GitHub repository.
Combined with a custom domain and a handful of DNS records, you can serve a
production site with HTTPS for nothing beyond the cost of the domain name.

This note covers the full path: repo setup, Pages configuration, DNS records,
and TLS enforcement.

## Prerequisites

- A GitHub account and a repository containing your static site output (HTML/CSS/JS)
- A registered domain name managed at any DNS registrar (examples use Porkbun)
- `gh` CLI installed and authenticated (`gh auth login`)

---

## 1. Prepare the repository

Your repo needs to end up with built HTML at a known location. Two options:

**Option A — build in CI (recommended)**
Use a GitHub Actions workflow to build your site and deploy the output. The
workflow publishes to Pages directly; nothing gets committed. Set Pages source to
**GitHub Actions** in repo Settings → Pages.

**Option B — publish from a branch**
Commit the built output to a branch (commonly `gh-pages`). Set Pages source to
that branch + folder in repo Settings → Pages. Simpler but mixes source and
output.

Option A keeps the repo clean and rebuilds on every push. Option B is fine for
sites with no build step.

---

## 2. Enable GitHub Pages

1. Go to **repo Settings → Pages**
2. Under **Source**, select **GitHub Actions** (or your branch if using Option B)
3. Leave **Custom domain** blank for now — set it after DNS is ready

At this point `https://<username>.github.io/<repo>` will serve your site once
the first build completes.

> **Note:** For an organisation or user root site (repo named
> `<username>.github.io`), the site serves at `https://<username>.github.io`
> with no path suffix.

---

## 3. Add a CNAME file to your repo

GitHub Pages reads a file named `CNAME` at the root of the published output to
know which custom domain to expect. Create it in your source tree so it gets
included in every build.

For a plain HTML site, add it at the repo root:

```text
your-domain.com
```

For a static site generator, place it where the generator copies static files
verbatim into the output (e.g. `static/CNAME` for Hugo, `public/CNAME` for
Gatsby). The file must contain exactly one line: the bare domain, no protocol,
no trailing slash.

Commit and push. Confirm the file appears in the deployed output.

---

## 4. Configure DNS at your registrar

GitHub Pages requires specific DNS records depending on whether you want the apex
domain (`example.com`), a subdomain (`www.example.com`), or both.

### Apex domain — four A records

GitHub Pages serves from four IP addresses. Add an A record for each:

| Type | Host | Value           |
|------|------|-----------------|
| A    | @    | 185.199.108.153 |
| A    | @    | 185.199.109.153 |
| A    | @    | 185.199.110.153 |
| A    | @    | 185.199.111.153 |

`@` means the root of the domain. Some registrars use the domain name itself or
leave the field blank for the apex — check your registrar's UI.

### www subdomain — one CNAME record

| Type  | Host | Value                    |
|-------|------|--------------------------|
| CNAME | www  | `<username>.github.io.`  |

Replace `<username>` with your GitHub username or organisation name. The
trailing dot is part of the DNS record; most registrars add it automatically.

### Subdomain only (e.g. `app.example.com`)

Skip the A records and add a single CNAME pointing the subdomain at
`<username>.github.io.`. No apex records needed.

---

## 5. Verify DNS propagation

DNS changes can take minutes to hours to propagate. Check with:

```bash
dig your-domain.com A +short
# should return the four 185.199.x.153 addresses

dig www.your-domain.com CNAME +short
# should return <username>.github.io.
```

Or use Google's public resolver to bypass local caching:

```bash
dig @8.8.8.8 your-domain.com A +short
```

Wait until you see the expected records before proceeding.

---

## 6. Set the custom domain in GitHub Pages settings

1. Go to **repo Settings → Pages → Custom domain**
2. Enter your domain (`your-domain.com`) and click **Save**
3. GitHub runs a DNS check immediately — a green tick means it can see your A records

This step also writes a `CNAME` entry to the Pages configuration on GitHub's
side, which is separate from the file in your repo. Both must match.

---

## 7. Enforce HTTPS

After the DNS check passes, GitHub provisions a TLS certificate via Let's
Encrypt. This typically takes 5–30 minutes.

Once the cert is issued:

1. Reload **repo Settings → Pages**
2. Check **Enforce HTTPS**

From this point all HTTP requests redirect to HTTPS automatically. GitHub renews
the cert before expiry — no manual renewal needed.

---

## Verification checklist

```bash
# DNS resolves correctly
dig @8.8.8.8 your-domain.com A +short
# → four 185.199.x.153 lines

# Site responds over HTTPS
curl -sI https://your-domain.com | head -3
# → HTTP/2 200

# No HTTP leakage (should redirect)
curl -sI http://your-domain.com | head -3
# → HTTP/1.1 301 or 308 to https://
```

---

## Common problems

**DNS check fails in GitHub UI**
The A records exist but the check fails. Usually means the CNAME file is missing
from the deployed output, or the domain in Settings doesn't exactly match the
CNAME file. Check both.

**Site serves at `github.io` URL but not the custom domain**
Custom domain not set in Settings → Pages, or DNS hasn't propagated yet. Run
`dig` to confirm records are live before blaming GitHub.

**HTTPS not available / cert pending indefinitely**
GitHub can't verify domain ownership if the A records don't point to its IPs, or
if CAA records at your registrar block Let's Encrypt. Check for a CAA record and
either remove it or add `letsencrypt.org` as an allowed issuer.

**`www` redirects but apex doesn't (or vice versa)**
You need both the A records (apex) and the CNAME (www) configured. GitHub Pages
will canonicalise to whichever you set as the custom domain and redirect the other.

---

## Cost

| Item            | Cost                                                                 |
|-----------------|----------------------------------------------------------------------|
| GitHub Pages    | Free                                                                 |
| TLS certificate | Free (Let's Encrypt via GitHub)                                      |
| Domain name     | Varies by registrar and TLD; typically $10–20/year for `.com`/`.org` |
