# Word Unscrambler (Astro)

**Historical subproject:** This folder preserves an earlier migration prototype. The production site now lives in the repository root. For Netlify, use the repository root as the base/package directory, `npm run build` as the build command, and `dist` as the publish directory. See [the root deployment guide](../DEPLOYMENT-GUIDE.md). The setup notes below apply only to this archived prototype.

Astro + Islands migration: fast first load, dictionary not in main bundle, sitemap/robots crawlable, structured data (WebSite/SearchAction, SoftwareApplication, FAQPage).

## Project structure

```
astro/
├── astro.config.mjs      # site, @astrojs/sitemap, @astrojs/react
├── package.json
├── tsconfig.json
├── public/
│   ├── robots.txt       # Allow /, Sitemap: https://lettersolver.net/sitemap.xml
│   ├── sitemap.xml      # Static fallback (/, /unscramble)
│   └── data/            # words_2.json … words_10.json (from build:dict)
├── scripts/
│   └── build-dict.ts    # words.txt → public/data/words_{len}.json
└── src/
    ├── env.d.ts
    ├── layouts/
    │   └── Layout.astro
    ├── components/
    │   ├── BaseHead.astro   # canonical, title, description, og
    │   ├── InternalLinks.astro
    │   └── UnscrambleTool.tsx  # client:load island
    ├── lib/
    │   ├── unscramble.ts   # pure unscramble (no dict import)
    │   ├── schema.ts       # WebSite, SoftwareApplication, FAQPage
    │   ├── meta.ts         # getMeta(), canonical
    │   └── internal-links.ts
    ├── pages/
    │   ├── index.astro     # home + WebSite schema
    │   └── unscramble.astro # tool + SoftwareApplication + FAQPage
    └── styles/
        └── global.css
```

## Setup

```bash
cd astro
npm install
```

## Build dictionary

Place a `words.txt` in the project root (one word per line). Then:

```bash
npm run build:dict
```

This writes `public/data/words_2.json` … `words_10.json`. If `words.txt` is missing, the script creates a sample file and sample JSONs.

## Run

```bash
npm run dev     # http://localhost:4321
npm run build   # build:dict + astro build → dist/
npm run preview # serve dist/
```

## Deploy

Production deployment is configured by `netlify.toml` at the repository root. The connected Netlify project builds the `main` branch using Node.js 24 and npm. Keep the base and package directories at the repository root; deploying this historical `astro/` folder would publish the old prototype.

The production site uses static output and needs no Netlify adapter. Follow [DEPLOYMENT-GUIDE.md](../DEPLOYMENT-GUIDE.md) for environment variables, domain migration, and verification.

## Verification

- **Sitemap:** `curl -I https://your-domain.com/sitemap.xml` → 200, `Content-Type: application/xml`
- **Robots:** `curl https://your-domain.com/robots.txt` → contains `Sitemap: https://lettersolver.net/sitemap.xml`
- **Structured data:** View source on `/` and `/unscramble` for `application/ld+json` (WebSite, SoftwareApplication, FAQPage).

## Optional: /api/unscramble

This project uses **static** output. For a server-side `/api/unscramble` with cache headers you’d need:

- Server output in `astro.config.mjs`, configured for the Astro version in use
- An adapter for the deployment platform (e.g. `@astrojs/netlify`)
- `src/pages/api/unscramble.ts` returning `Response` with `Cache-Control`

The current design keeps the main bundle small by loading `words_{len}.json` in the client and running unscramble in the browser.
