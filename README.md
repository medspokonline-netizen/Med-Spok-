# Medspok Solutions — website

The website for **Med Spok Solutions Pvt Ltd**, Porur, Chennai: medical equipment for home
and hospital, on rent or on sale.

Six pages of plain HTML, one stylesheet, one small JavaScript file. No framework, no build
step, no database. It loads fast on a phone on mobile data, costs nothing to host, and will
still work in five years without anyone reinstalling anything.

```
├── index.html          Home
├── products.html       Full range, with spec tables and photos
├── rent-or-buy.html    Rent vs buy, with the situation picker
├── services.html       Installation, AMC/CMC, consumables, setup film
├── about.html          Company, premises, brand partners
├── contact.html        Enquiry form, phone numbers, map
├── assets/
│   ├── style.css       All styling
│   ├── site.js         Menu, picker, photo viewer, enquiry form
│   ├── products/       Product photography
│   ├── premises/       Shopfront and stockroom
│   ├── brands/         Manufacturer logos
│   ├── icons/          Favicons
│   └── video/          Concentrator setup guide
├── standalone/         The whole site as one self-contained HTML file
├── tools/              Link checker, test suite, single-file builder
└── .github/workflows/  Deploys to GitHub Pages on push
```

## Working on it locally

Open `index.html` in a browser — there is nothing to compile. For the Google map on the
contact page, and for anything involving the video, serve it over HTTP instead:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

## Before you push

```bash
python3 tools/checklinks.py
```

Checks every page for unclosed tags, broken internal links, missing files, duplicate IDs,
images without alt text, and heading levels that skip. It also runs in CI, so a broken link
fails the deploy instead of reaching the public site.

There is also a behavioural test that drives a real browser and clicks things — the
rent-or-buy picker, the photo viewer, the enquiry form, the mobile menu, the logos:

```bash
pip install playwright && python3 -m playwright install chromium
python3 tools/testsite.py
```

Worth running after any change to `assets/site.js`. A syntax check alone will not tell you a
feature has stopped working — that mistake has already been made once on this site, and two
features sat dead for several builds before a test caught them.

## The standalone builds

Two single-file copies of the whole site can be built. The light one is committed;
the complete one is 31 MB and is deliberately **not** in the repository — it is a build
artifact, regenerated in one command.

| File | Size | Needs alongside it |
|---|---|---|
| `medspok-website.html` | 1.8 MB | `setup-guide.mp4`, `medspok-catalogue-2026.pdf` |
| `medspok-website-complete.html` | 21.5 MB | nothing at all |

The **complete** build carries everything inside the one file — all 78 photographs, the
logos, the 6½-minute setup film and the catalogue PDF, as base64. Double-click it on any
machine, online or off, and the entire site works. Useful on a laptop at a hospital meeting,
or to hand someone on a pen drive.

It comes in under GitHub's 25 MB browser-upload limit, so it can be attached to a release
or emailed as a single artifact.

The trade is weight. 21.5 MB is still far too heavy to put on a web server: a data URI cannot stream,
so the film has to arrive whole before it plays, and the browser parses the lot up front.
**Host the folder version, and treat the complete file as an offline copy.**

Rebuild either from the repository root:

```bash
python3 tools/bundle.py                  # light build, external media
python3 tools/bundle.py --embed-video    # complete build, everything inside
```

For the complete build the media is re-encoded to fit the budget: the film goes from 20 MB
to 12 MB, and the catalogue PDF from 8.5 MB to 2.6 MB via Ghostscript at its `/printer`
setting — measured at 34 dB PSNR against the original, which is indistinguishable on screen
and still fine to print. Without that, the page would be nearer 50 MB. Both builds store each asset once and apply it at load, rather
than pasting the same base64 string in at every reference.

### Vercel

`vercel.json` is included and sets the project up as a plain static site: no build command,
no framework, served from the repository root, with long cache headers on `/assets/`.

Import the **GitHub repository** in Vercel rather than dragging files in. Drag-and-drop
often uploads only the files you selected and leaves the `assets/` folder behind — which
produces a page with no styling, no logo and blue underlined links, because
`assets/style.css` is missing.

If the deployed site looks unstyled, open `https://your-project.vercel.app/assets/style.css`
directly. A 404 confirms the folder never uploaded; redeploy from Git and it will be there.
In project settings, Framework Preset should read **Other**, with Build Command and Output
Directory left empty.

## Publishing

**GitHub Pages via Actions (recommended).** Push to `main`, then set *Settings → Pages →
Source* to **GitHub Actions**. The workflow runs the link check and publishes.

**GitHub Pages from a branch.** *Settings → Pages → Source: Deploy from a branch*, branch
`main`, folder `/ (root)`. Simpler, but skips the link check.

**Anywhere else.** Cloudflare Pages, Netlify and Vercel all take this repository as-is with
an empty build command and `/` as the output directory.

### Custom domain

Add a file named `CNAME` at the repository root containing just the domain:

```
www.medspok.in
```

and point a CNAME record at `<username>.github.io`. Then update the URLs in `sitemap.xml`,
`robots.txt`, and the `<link rel="canonical">` and Open Graph tags at the top of each page.
They currently read `https://www.medspok.in/`, which is a placeholder until the domain is
registered.

## Repository size

The catalogue PDF (~9 MB) and the setup film (~20 MB) are almost all of the ~32 MB
repository. That is well inside GitHub's limits, but each future change to those files
stores another full copy in the history. If they are updated often, move them to
[Git LFS](https://git-lfs.com), or put the film on YouTube and embed it.

## Third-party material

This repository contains other companies' trademarks and photography — Home Medix, BMC,
ResMed and SVASTEK logos and product images, and a setup film produced by Home Medix —
included on the basis that Med Spok Solutions is an authorized dealer.

Confirm permission with each manufacturer before the site goes public. Note that making this
repository **public** publishes those assets too; a **private** repository still deploys to
GitHub Pages on the free plan, which is the safer default until permissions are settled.

`MAINTENANCE.md` covers how the content is put together, what still needs checking, and how
to change things.
