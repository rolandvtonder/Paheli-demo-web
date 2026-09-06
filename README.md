# Paheli Weddings & Events — website

A seven-page static site for Paheli Weddings & Events, Alberton, Gauteng.
No frameworks, no build server, no database. The `.html` files in the project
root are the finished site — upload them and it works.

---

## 1. What's here

| Page | File | Contains |
|---|---|---|
| Home | `index.html` | Scroll-driven photo hero, positioning statement, about teaser, stats, services preview, gallery preview, review carousel, CTA |
| About | `about.html` | Preeshani's story, credentials, stats, "Four things we hold to" |
| Services | `services.html` | All six services in detail, plus the four-stage process |
| Packages | `packages.html` | The Essentials / The Signature / The Bespoke, add-ons, FAQ |
| Real Weddings | `gallery.html` | 30 photographs across five weddings, filterable, with a lightbox |
| Reviews | `reviews.html` | All ten Google reviews |
| Contact | `contact.html` | Enquiry form, contact details, office hours, map |

Supporting files: `sitemap.xml`, `robots.txt`, `assets/`.

---

## 2. Editing content

**Do not edit the root `.html` files directly — they are generated and will be
overwritten.** Edit the sources instead:

```
src/layout.html      header, footer, nav, <head> — shared by every page
src/pages/*.html     the body of each page
```

Each page file starts with a small JSON block that sets its title, meta
description and social-share image:

```html
<!--{
  "slug": "about",
  "title": "About Preeshani Goven — Paheli Weddings & Events, Alberton",
  "desc": "Meet Preeshani Goven, founder and head planner…"
}-->
```

Then rebuild:

```bash
node tools/build.mjs
```

That regenerates all seven pages plus `sitemap.xml` and `robots.txt`.

### Preview it locally

```bash
node tools/serve.mjs
```

Then open <http://localhost:4321>.

---

## 3. Adding or replacing photographs

1. Drop full-resolution originals into the matching folder:
   `assets/img/hero/`, `assets/img/gallery/`, or `assets/img/about/`
2. Run the optimiser:

```bash
node tools/optimise.mjs
```

It writes resized WebP versions into `assets/opt/` (960/1600/2400px for hero,
480/900/1600px for gallery) and never upscales. The current set went from
196 MB of originals to 16 MB of WebP — a 92% reduction.

3. For gallery images, add the new filename to the `PHOTOS` array near the top
   of `assets/js/main.js`. Each entry is
   `['file-stem', 'album-key', unused, largestWidthAvailable]`.

`assets/img/` holds the originals and does **not** need to be uploaded to the
web server — only `assets/opt/`, `assets/brand/`, `assets/css/` and
`assets/js/` are used by the live site.

---

## 4. Making the contact form send

Right now the form validates properly and then opens the visitor's email app
with everything pre-filled. That works today with no setup, but it depends on
the visitor having an email client configured.

To have enquiries delivered straight to your inbox instead, sign up for a form
service (Formspree, Web3Forms and Basin all have free tiers), then put your
endpoint URL into `src/pages/contact.html`:

```html
<form class="form" id="enquiryForm" novalidate data-endpoint="https://formspree.io/f/XXXXXXX">
```

Rebuild, and the form will POST JSON there instead. No other change is needed —
success and failure messages are already handled.

---

## 5. Things still to fill in

These are deliberately left for you. Each is marked with a `TODO` comment in the
source.

- **Package prices** — `src/pages/packages.html`. All three cards currently say
  "Request a quote". Replace the `<small>` line in each with a real figure
  (e.g. `From R12 500`) when you're ready to publish them.
- **A photograph of Preeshani** — `src/pages/about.html` and
  `src/pages/home.html`. The old site had no portrait of her (the image labelled
  "Preeshani" there is a table centrepiece), so both pages currently show one of
  her weddings, described accurately. Drop a headshot into `assets/img/about/`,
  re-run `node tools/optimise.mjs`, and point the image marked `HEADSHOT SLOT`
  at it.
- **Street address** — `src/pages/contact.html`. The map currently centres on
  Alberton. If you want the exact address shown, change the `q=` value in the
  Google Maps iframe.
- **Photographer credits** — two hero photographs carry a visible
  "Hemisha Bhana Photography" watermark. They are used as-is, which credits the
  photographer, but swap them if you'd prefer unwatermarked frames.

Note on the stats band: "8+ years", "5.0 Google rating", "100% bespoke plans"
and "7 days a week on the day" are all supportable. If you want a
"weddings planned" number, send it and I'll add it — I didn't want to invent one.

---

## 6. Deploying

The site is plain static files. Upload the following to your web root:

```
*.html  sitemap.xml  robots.txt
assets/brand/  assets/css/  assets/js/  assets/opt/
```

Skip `src/`, `tools/`, `assets/img/`, `node_modules/` and `package*.json` —
those are build-time only.

- **cPanel / shared hosting:** upload via File Manager or FTP into `public_html`.
- **Netlify / Vercel / Cloudflare Pages:** drag the folder in, or connect the
  repo with no build command and the project root as the publish directory.

After it's live, update the `SITE` constant at the top of `tools/build.mjs` if
the domain ever changes, then rebuild so canonical URLs and the sitemap match.

---

## 7. Design system

Colours are sampled from the Paheli logo and defined once as CSS custom
properties at the top of `assets/css/styles.css`:

| Token | Value | Used for |
|---|---|---|
| `--wine-600` | `#9B1E44` | Primary — buttons, links, accents |
| `--wine-900` | `#400C1E` | Dark sections, footer, nav overlay |
| `--gold-500` | `#C8963E` | Decorative rules, stars, large numerals |
| `--gold-700` | `#8C601F` | Gold *text* at body size (passes contrast) |
| `--rose-500` | `#E0705F` | Floral accent |
| `--sage-500` | `#93A56B` | Botanical accent |
| `--ivory` / `--cream` | `#FBF7F2` / `#F4EDE4` | Page and alternating backgrounds |
| `--ink-900` | `#2A1B20` | Body text |

Typography: **Cormorant Garamond** for display, **Jost** for body and UI.

> Gold is the one colour to be careful with. `--gold-500` does not meet the
> 4.5:1 contrast minimum on a light background, so it is only used for
> decoration and large display numerals. Any gold text at body size uses
> `--gold-700`.

### Accessibility

Built to WCAG 2.2 AA and verified with `tools/audit.js`, which checks colour
contrast, heading order, alt text, form labels, accessible names, duplicate IDs
and horizontal overflow. All seven pages pass clean. Also handled:

- Full keyboard support — lightbox and menu trap focus, close on `Escape`, and
  return focus to where it came from
- `prefers-reduced-motion` disables the hero sequence, scroll reveals and the
  review autoplay
- The review carousel has a visible pause control and stops on hover and focus
- Form errors appear inline *and* in a focusable summary linked to each field
- Works without JavaScript: every page renders and reads correctly; only the
  gallery filter and lightbox need it

To re-run the audit, serve the site, open it in a browser, and in the console:

```js
fetch('tools/audit.js').then(r=>r.text()).then(eval).then(()=>console.table(__audit().issues))
```

---

## 8. A note on the hero

You asked for a hero video that moves with the scroll, then chose photo parallax
instead — so the home page hero is built as a scroll-driven photo sequence. It
pins to the viewport and cross-fades through four photographs as you scroll,
each one slowly pushing in, which reads like footage without shipping a video
file (and without the bandwidth cost on mobile).

If you later want real video there, the structure is ready for it: the frames
live in `.hero__stage` in `src/pages/home.html`, and the scrub logic in
`assets/js/main.js` drives whatever is inside.
