# ZAHORSZKY

A simple public personal homepage: tiger artwork, centered ink calligraphy, a prominent wordmark, and contact details. Visitors open the page directly, with no sign-in.

Plain HTML and CSS. No JavaScript, framework, database, application backend, tracking scripts, or runtime dependencies. Cloudflare Workers hosts the static site, and GitHub stores its source and history.

**Status:** live at [zahorszky.com](https://zahorszky.com) and [www.zahorszky.com](https://www.zahorszky.com), with source in the public [`zahorszky/zahorszky.com`](https://github.com/zahorszky/zahorszky.com) repository. The contact address is set to `zoltan@zahorszky.com`.

## Files

```text
public/
  index.html          Wordmark, calligraphy, Contact email, image references
  styles.css          Colors, typography, responsive layout
  assets/
    tiger.avif        1536 × 1024 artwork, approximately 192 KB
    tiger.webp        WebP fallback, approximately 229 KB
    tiger-small.avif  900 × 600 version, approximately 75 KB
    tiger-small.webp  Smaller fallback, approximately 93 KB
    calligraphy.webp  Fine transparent ink characters, approximately 173 KB
    favicon.svg       Minimal Z monogram
  _headers            Security, cache, and indexing headers
  robots.txt          Discourage indexing until you make the site searchable
branding/
  tiger-background.png   Source painting for replacement exports
  calligraphy-source.png Current fine-brush transparent ink source
  calligraphy-source-bold.png Earlier heavier ink source
  artwork.md             Artwork provenance and generation prompts
wrangler.jsonc        Cloudflare static hosting configuration
.gitignore            Excludes credentials and local caches
README.md             This guide
```

Only `public/` is deployed. The README and configuration stay outside the served directory.

## Local preview

From the project directory, with Python 3 installed:

```sh
python3 -m http.server 8000 --bind 127.0.0.1 --directory public
```

Open <http://127.0.0.1:8000>. Stop with Ctrl+C. If the port is occupied, use 8001 instead. This previews the design; Python's server does not apply Cloudflare's `_headers` file.

To preview Cloudflare's routing and headers, with Node.js installed:

```sh
npx wrangler@4 dev
```

Open the local URL printed by Wrangler. Wrangler is the deployment tool, not a site dependency. No build step or `npm install` is needed for normal editing.

## Everyday edits

| Change | Location |
| --- | --- |
| Email | Change both the visible address and its `mailto:` link in `public/index.html` |
| Contact label | The bold `Contact` text in the footer |
| Name | `<h1>`, `<title>`, and description in `public/index.html` |
| Calligraphy | `public/assets/calligraphy.webp`, its `<img>` in `public/index.html`, and `.inscription` in `public/styles.css` |
| Colors and spacing | Custom properties at the top of `public/styles.css` |
| Background | The four `tiger*` image files, or their `<picture>` references |
| Favicon | `public/assets/favicon.svg` |

The email appears twice together in one footer block: once as the visible text and once in its `mailto:` link. The link opens the visitor's configured email application.

## Artwork and replacement images

The tiger background and separate calligraphy were generated from the supplied references as art direction; the originals were not edited or overwritten. Project source PNGs are in `branding/`. See [artwork notes](branding/artwork.md).

AVIF saves transfer size; WebP provides a widely supported fallback. The browser chooses a suitable width and format for the painting, not all four files. The painting and calligraphy have separate alternative text. System fonts require no downloads.

For replacements, use a 3:2 landscape with quiet upper/left space, and keep the tiger's face and paws away from the edges. Export at 1536 × 1024 and 900 × 600. Keep the larger export below about 450 KB when possible, checking brush detail at actual viewing size. Do not upscale a small source or repeatedly recompress an already compressed image. Retain your original separately.

An image editor with AVIF/WebP export is sufficient. Optional reproducible conversion using Pillow (an editing tool, not a site dependency):

```sh
python3 -m venv /tmp/zahorszky-images
/tmp/zahorszky-images/bin/pip install 'Pillow>=11.3'
/tmp/zahorszky-images/bin/python - <<'PY'
from PIL import Image, ImageOps

source = ImageOps.exif_transpose(Image.open('/absolute/path/to/replacement.png')).convert('RGB')
assert source.width * 2 == source.height * 3, 'Prepare a 3:2 source first'
for name, width in [('tiger', 1536), ('tiger-small', 900)]:
    image = source.resize((width, width * 2 // 3), Image.Resampling.LANCZOS)
    image.save(f'public/assets/{name}.avif', quality=68, speed=6)
    image.save(f'public/assets/{name}.webp', quality=86, method=6)
PY
```

Desktop uses a full canvas; narrow portrait screens reposition the tiger below the calligraphy. Ultrawide screens keep the image proportions and fade the left edge. The separate ink layer is centered on phones and shifted left on wide screens to keep the tiger visible. Check these layouts after replacing either asset.

## Publish with GitHub and Cloudflare

### 1. Repository

The source is stored in the public [`zahorszky/zahorszky.com`](https://github.com/zahorszky/zahorszky.com) repository. The production branch is `main`. Use your normal GitHub authentication and never put credentials in project files.

### 2. Deploy to Cloudflare

Authenticate once with `npx wrangler@4 login`, then deploy from the repository root:

```sh
npx wrangler@4 deploy
```

Cloudflare uploads `public/` as Workers Static Assets. The `wrangler.jsonc` file connects both `zahorszky.com` and `www.zahorszky.com` as custom domains. Cloudflare manages their DNS records and HTTPS certificates; there is no separate origin server, build step, environment variable, or application runtime to maintain.

The unrelated `_domainconnect` DNS record remains in place. Mail service must be configured separately with the MX and verification records supplied by the chosen email provider.

## Deploy updates

Preview your change, then:

```sh
git add .
git diff --cached
git commit -m "Update homepage"
git push
```

After pushing the source update, deploy it with `npx wrangler@4 deploy`. Check the active version under **Workers & Pages → zahorszky → Deployments**, then reload the live homepage.

To undo a bad commit:

```sh
git revert BAD_COMMIT_SHA
git push
```

For immediate recovery, use **Rollback** on a previous good version in the Worker's **Deployments** view. Revert the bad commit in Git too so the next push preserves the fix. [Rollback documentation](https://developers.cloudflare.com/workers/versions-and-deployments/rollbacks/)

## Indexing and headers

The homepage is publicly accessible. The existing noindex metadata, `X-Robots-Tag`, and `robots.txt` discourage search indexing until you decide to make the site searchable. They do not restrict visitors.

When you want search indexing, remove the robots meta tag in `index.html`, remove `X-Robots-Tag` from `_headers`, and change `Disallow: /` to `Allow: /` in `robots.txt`.

The response headers block scripts, framing, and unnecessary browser permissions. Public caching with revalidation lets browsers check for updates. No tracking scripts or cookies are added by the page.

## Verification and launch checklist

The design was inspected at phone, tablet, landscape, desktop, and ultrawide proportions. At 320×568, 390×844, 768×1024, 844×390, 1440×900, and 2560×1080, the artwork loaded without horizontal overflow. The contact link is keyboard accessible. No Lighthouse score is claimed.

Before launch:

- Test the Contact email link with a configured mail application.
- Check the live page opens directly in a fresh browser session.
- Verify HTTPS and the expected content on your chosen domain.
- Confirm `/README.md` and `/wrangler.jsonc` return 404.
- Decide whether to allow search engines to index the site.

The first production deployment and custom-domain setup were completed and verified on September 19, 2026.
