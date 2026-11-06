# Hello World — public static assets on Cloudflare Workers

A minimal cyberpunk landing page and a public, unauthenticated file-download root served with **Cloudflare Workers Static Assets**.

## What this project does

- Serves `public/index.html` at `/`.
- Serves any files placed directly in `public/` at the matching URL path.
- Adds `X-Robots-Tag: noindex, nofollow, noarchive, nosnippet` to all static responses.
- Includes `robots.txt` that allows crawlers to fetch the site and see the `noindex` directive.
- Uses no external JavaScript, CSS framework, font, analytics, or third-party runtime.

## Important: public means public

Anyone who knows or discovers a file URL can download it. There is no authentication, authorization, access control, or secrecy. Do not publish credentials, private keys, personal data, confidential material, or software you are not authorized to distribute.

`noindex` is a request to compliant search engines, not access control or a guarantee that a URL will never appear in search results. Search engines may discover URLs from links or other sources. A public GitHub repository is independently crawlable and searchable; the site's HTTP headers do not control indexing of GitHub itself.

## Add ZIP files

Place ZIP files directly in `public/` (not in a subdirectory) so their URLs are at the domain root.

Example:

```text
public/
├── index.html
├── robots.txt
├── _headers
├── checksums.sha256
├── linux-tool-v1.0.zip
└── setup-script-v2.1.zip
```

Then users can download them without an HTML directory listing:

```bash
wget https://YOUR-DOMAIN.example/linux-tool-v1.0.zip
curl -fLO https://YOUR-DOMAIN.example/linux-tool-v1.0.zip
```

Replace `YOUR-DOMAIN.example` with your deployed hostname.

### Generate and verify SHA-256 checksums

From the repository root, after adding your ZIP files:

```bash
find public -maxdepth 1 -type f -name '*.zip' -print0 \
  | sort -z \
  | xargs -0 -r sha256sum > public/checksums.sha256
```

After downloading a ZIP and the checksum file:

```bash
wget https://YOUR-DOMAIN.example/checksums.sha256
wget https://YOUR-DOMAIN.example/linux-tool-v1.0.zip
sha256sum -c checksums.sha256
```

The checksum file should be distributed over the same trusted channel as the files. For stronger authenticity, sign release checksums with a signing key whose public key users already trust. A checksum alone detects accidental corruption; it does not prove who published a file if an attacker can replace both the ZIP and its checksum.

## Cloudflare Workers deployment

### Prerequisites

- A Cloudflare account.
- Node.js and npm installed on the deployment machine.
- A GitHub repository containing this project.

### Local preview

From the repository root:

```bash
npx wrangler@latest dev
```

Open the local URL printed by Wrangler.

### Deploy from the command line

Authenticate Wrangler:

```bash
npx wrangler@latest login
```

Deploy:

```bash
npx wrangler@latest deploy
```

The configuration in `wrangler.jsonc` points Wrangler to `./public`.

### GitHub automatic deployments

1. Push this project to a GitHub repository.
2. In the Cloudflare dashboard, open **Workers & Pages** and create a Worker using the Git repository integration (the exact labels in the dashboard may change).
3. Select the repository and configure the build/deploy command as `npx wrangler@latest deploy`.
4. Set the root directory to the repository root.
5. Ensure the deployment uses the committed `wrangler.jsonc`.
6. Complete the first deployment and configure your custom domain in the Worker settings if desired.

Do not configure a framework build command; this project is plain static assets.

## Publishing changes

Add or replace files in `public/`, then commit and push:

```bash
git add public/
git commit -m "Publish downloadable files"
git push
```

If GitHub integration is configured, the push triggers a new deployment.

## Search-engine exclusion

Three layers are used:

1. HTML robots meta directives in `public/index.html`.
2. `X-Robots-Tag` response header in `public/_headers`, applied to all static assets.
3. `robots.txt` allows crawling so search bots can fetch the page and observe `noindex`.

Do **not** change `robots.txt` to `Disallow: /` if you rely on crawlers reading the `noindex` directives. Blocking crawling can prevent search engines from seeing those directives.

No sitemap is provided.

## Cloudflare Static Assets limits and behavior

At the time this project was prepared, Cloudflare documents a **25 MiB per-file limit** for Workers Static Assets. Static asset responses do not currently provide HTTP byte-range support: a request with a `Range` header receives the full file with `200 OK`, rather than `206 Partial Content`. Consequently, clients generally cannot resume interrupted downloads with `wget -c` or `curl -C -` from this static-assets setup.

If you later need files larger than the limit, resumable downloads, or object-storage features, evaluate Cloudflare R2 and a suitable delivery design. Recheck Cloudflare's current limits before publishing large files.

## Security notes

- All files in `public/` are downloadable by anyone who has the URL.
- No directory listing is generated; this is not a confidentiality feature.
- Use versioned filenames instead of silently replacing released binaries.
- Scan and test scripts/binaries before publishing.
- Publish SHA-256 checksums and, for higher assurance, signed release manifests.
- `X-Content-Type-Options: nosniff` is applied to static responses.
- The page uses a restrictive Content Security Policy via `_headers` only if you choose to add one. The current page has inline CSS, so a strict `style-src` policy needs to allow inline styles or move the CSS into a separate file.

## Files

- `public/index.html` — landing page.
- `public/robots.txt` — crawler guidance.
- `public/_headers` — response headers for Cloudflare Workers Static Assets.
- `public/checksums.sha256` — starter checksum manifest.
- `wrangler.jsonc` — Wrangler configuration.
