# rcodefs.com

The website for **Rcodefs**, a software factory. Static HTML, no build step, no dependencies, deployed with GitHub Pages.

Live at [rcodefs.com](https://rcodefs.com/).

---

## What's in here

| File | Purpose |
| --- | --- |
| `index.html` | The entire site: markup, CSS and JavaScript in one file. |
| `404.html` | Styled fallback page. GitHub Pages serves it automatically for unknown paths. |
| `favicon.svg` | Hand-written "R" monogram. No image assets required. |
| `CNAME` | Tells GitHub Pages to serve the site at `rcodefs.com`. |
| `.nojekyll` | Disables Jekyll processing so dotfiles and folders are published as-is. |
| `robots.txt` | Crawler policy, including explicit allowance for AI crawlers. |
| `sitemap.xml` | Single-URL sitemap pointing at the homepage. |
| `llms.txt` | Plain-Markdown summary of the company for AI agents and LLM crawlers. |
| `README.md` | This file. |
| `.github/workflows/pages.yml` | Builds the artifact and deploys it to Pages after approval. |

There is no package manager, no bundler, no CDN and no external font or image request. The page is a single ~35 KB document.

---

## Local preview

Any static server works. The page also behaves correctly when opened directly from disk, but a server matches production more closely (relative paths, `404.html`, `CNAME` handling).

```bash
# from the repository root
python3 -m http.server 8000
```

Then open <http://localhost:8000>.

On Node-based machines:

```bash
npx serve .
```

---

## Deploying with GitHub Pages

The site is published by `.github/workflows/pages.yml`, not by a branch build. Pages is configured with **Source: GitHub Actions**, and the deploy job targets the `github-pages` environment, which requires a manual approval from `@sergiohromano`.

How a release works:

1. Merge to `main` (direct pushes are blocked by the `Protect main` ruleset — open a PR instead).
2. The `Deploy site` workflow runs, uploads the site files as an artifact, then **waits**.
3. Approve the deployment in **Actions → Deploy site → Review deployments → Approve**, or from the run's URL Slack/email links to.
4. `actions/deploy-pages` publishes the approved artifact to `rcodefs.com`.

The workflow stages only the site files (`index.html`, `404.html`, `favicon.svg`, `robots.txt`, `sitemap.xml`, `llms.txt`, `CNAME`) into `_site`, so `.git`, the README and the workflow itself are never published.

One-time setup, if the repository is ever recreated:

1. Push to `main` on `github.com/rcodefs/website`.
2. **Settings → Pages → Source: GitHub Actions**.
3. **Settings → Environments → `github-pages` → Required reviewers**: add yourself.
4. **Settings → Pages → Custom domain**: `rcodefs.com` (already in `CNAME`).
5. Wait for the DNS check, then tick **Enforce HTTPS** once the certificate covers both apex and `www`.

### Publishing controls

- **Ruleset `Protect main`** — blocks deletions and force pushes, requires a pull request and linear history. Applied to the default branch only.
- **Environment gate** — no deployment ships without an approval on the `github-pages` environment, even if someone has push access.
- **Domain verification** — `rcodefs.com` should be claimed under Settings → Pages → Verified domains, so the domain cannot be attached to another GitHub account if this repo disappears.

### DNS

At your registrar, for the apex domain `rcodefs.com`:

```
A     @    185.199.108.153
A     @    185.199.109.153
A     @    185.199.110.153
A     @    185.199.111.153
```

And for the `www` subdomain:

```
CNAME www  rcodefs.github.io
```

`www.rcodefs.com` then redirects to the apex domain automatically once both are configured in Settings → Pages.

On Cloudflare, both records must be **DNS only** (grey cloud) — a proxied record answers the ACME challenge itself and the certificate never issues. Cloudflare's CNAME flattening makes the apex `CNAME` legal, so this works as-is:

```
CNAME  @    rcodefs.github.io    DNS only
CNAME  www  rcodefs.github.io    DNS only
```

Some registrars support `ALIAS`/`ANAME` records, which can point the apex at `rcodefs.github.io` instead of using the four `A` records. Either approach is fine.

### Email

The site's only conversion path is `hello@rcodefs.com` (`mailto:` links in the hero, contact card and 404 page). That address must actually receive mail — a real mailbox, a forwarder, or the registrar's email forwarding. If the address changes, update it in:

```bash
grep -n "hello@rcodefs.com" index.html 404.html llms.txt
```

Also update the `email` fields in the JSON-LD block at the top of `index.html`.

---

## Editing the content

Everything is in `index.html`. Sections are marked with banner comments:

```
<!-- ============================== HERO ============================== -->
<!-- ============================ SERVICES ============================ -->
<!-- ============================ PROCESS ============================ -->
<!-- ============================= STACK ============================= -->
<!-- ============================== WHY ============================== -->
<!-- ============================ CONTACT ============================ -->
```

Useful targets:

- **Brand colour** — the `:root` block at the top of `<style>`. `--accent` is the fill colour for buttons and glows; `--accent-text` is the lighter tint used for text on dark backgrounds. Change both together and check contrast.
- **Background / surfaces** — `--bg`, `--bg-soft`, `--surface`, `--surface-2`, `--line`.
- **Navigation** — the `<nav id="nav">` block in the header. Each link points at a section `id`; the active-link highlighting picks them up automatically.
- **Services, process steps, stack chips** — plain `<article>`/`<li>` markup. Copy an existing block to add one; the grids reflow on their own.
- **Footer year** — rendered by the inline script, so no yearly edit is needed.

### Adding social links later

The page deliberately has no LinkedIn/X/booking links yet. When they exist, add them to:

1. `llms.txt` (under `## Contact`)
2. The JSON-LD `Organization.sameAs` array in `index.html`
3. The `.contact-links` block in the contact section

---

## The machine-readable layer

### JSON-LD

A single `@graph` in the `<head>` of `index.html` contains four nodes linked by `@id`:

- `Organization` (`#org`) — company identity, email, GitHub, areas of expertise
- `WebSite` (`#website`) — site metadata, publisher → `#org`
- `ProfessionalService` (`#service`) — services as an `OfferCatalog`
- `BreadcrumbList` (`#breadcrumb`)

Validate changes at <https://validator.schema.org/> or with Google's Rich Results Test before pushing. Keep the `@id` values stable — they are absolute URLs and are what makes the nodes resolve to each other.

### `llms.txt`

Follows the emerging `llms.txt` convention: a Markdown summary at the root of the domain. It is linked from the page with:

```html
<link rel="alternate" type="text/markdown" href="/llms.txt">
```

It is not an official standard, so treat it as a best-effort hint for agents. Update it whenever services, contact details or positioning change — it is the cheapest way to keep AI answers about Rcodefs accurate.

### `robots.txt`

Allows everything and names the common AI crawlers explicitly, so the policy is visible rather than implied. Point it at the sitemap at the bottom of the file.

### `sitemap.xml`

One URL. Update `<lastmod>` when the page content changes materially.

---

## Design notes

- **No external assets.** System font stack, inline SVG icons, hand-written favicon. One HTTP request for the document, nothing to block rendering, nothing to break offline.
- **Accessibility.** Skip link, visible focus rings, `aria-expanded`/`aria-controls` on the mobile menu, `aria-current` on the active nav link, decorative SVGs marked `aria-hidden`, contrast-checked accent text, `prefers-reduced-motion` respected.
- **Responsive.** Fluid type with `clamp()`, grids that reflow via `auto-fit`, mobile nav below 820px, full-width buttons below 480px.
- **Progressive enhancement.** Without JavaScript the page is fully readable and navigable; only the scroll reveals, active-link highlighting and the mobile menu depend on it.

---

## Licence and contact

Site content © Rcodefs. Code in this repository may be reused for this project's own purposes.

hello@rcodefs.com · [github.com/rcodefs](https://github.com/rcodefs)
