# Word Unscrambler | Online Word Unscramble Tool

**Word Unscrambler** is a fast, free online word unscramble tool that helps you solve anagrams, word scrambles, and spelling challenges. Generate valid English words from scrambled letters, filter by length, and quickly find the best matches for learning, games, and teaching.

The site is built with [Astro](https://astro.build) (static output) only. All routes (tool pages, words-by-length, words-start-with, words-ending-in, N-letter-words-with-X, etc.) are generated from `src/pages/`.

## 🚀 Key Features

- **Unscramble letters** into all possible words
- **Length filters** to narrow down results
- **Dictionary scope** options (common, advanced, full)
- **Instant search** with sortable results
- **Mobile-friendly** responsive UI

## 🔍 SEO Keywords

> word unscrambler, unscramble words, anagram solver, word scramble solver, letter solver, english word finder, anagram generator, word game helper, vocabulary practice

## 📍 Target Audience (US)

This project targets **US** users: students, teachers, word-game players, and anyone who needs to unscramble letters or solve anagrams in English. Ideal for Wordle, Scrabble, Words with Friends, classroom activities, and spelling practice.

## 🧭 Use Cases

- English learning: spelling practice and vocabulary building
- Word games: Anagram, Word Scramble, Wordscapes
- Teaching: classroom activities and word challenges
- Content creation: wordplay and anagram ideas

## 🛠️ Local Development (Astro)

```bash
npm ci
npm run dev       # http://localhost:4321
npm run build     # builds dict + Astro → dist/
npm run preview   # serve dist/
```

- **Dictionary**: Put `words.txt` (one word per line) in the project root, or leave it out to use the built-in sample. Run `npm run build:dict` to regenerate `public/data/words_*.json`. For a much larger 2–10 letter dictionary, run once (with network): `npx tsx scripts/download-words-alpha.ts` to fetch a public word list into `words_alpha.txt`; the build will merge it automatically. Use `SKIP_REMOTE_DICT=1` to skip network fetch and use only `words.txt` plus optional `words_alpha.txt` or the bundled subset.

## 📦 Deployment

Deploy the root Astro project to **Netlify**. The repository is connected to Netlify; pushing to the production branch (`main`) triggers a build and deployment. Build settings, redirects, and response headers are defined in the root `netlify.toml`.

| Setting | Value |
| --- | --- |
| Base directory | Repository root (`.`) |
| Build command | `npm run build` |
| Publish directory | `dist` |
| Node.js | `24` |
| Package manager | npm, using `package-lock.json` |

Use the repository root for both the base and package directory settings; the `astro/` folder is a historical subproject. This site uses static output and needs no Netlify adapter. `NPM_FLAGS=--include=dev` keeps the build tools available during dependency installation.

Optional build environment variables are `PUBLIC_ADSENSE_CLIENT` and `PUBLIC_ADSENSE_SLOT`. Keep `SKIP_REMOTE_DICT` unset for normal production builds: setting it to `1` skips the remote dictionary and can reduce word coverage when no full local dictionary is available.

After a successful Netlify deployment, add `lettersolver.net` in domain management and update DNS using the records shown by Netlify. See [DEPLOYMENT-GUIDE.md](DEPLOYMENT-GUIDE.md) for migration steps and verification commands. Astro's [Netlify deployment guide](https://docs.astro.build/en/guides/deploy/netlify/) explains static hosting support.

### Nginx (project-integrated)

This repository now includes Nginx config for serving the generated static site:

- `nginx/nginx.conf`: Nginx server config (SPA fallback + static asset caching + gzip + rate limiting)
- `docker-compose.nginx.yml`: one-command Nginx runtime
- Proxy cache control component included at `/proxy-cache/*`
- Rate limiting component included for page and proxy routes
- Brotli example included at `nginx/brotli.conf.example`

Build first, then run Nginx:

```bash
npm run build
docker compose -f docker-compose.nginx.yml up -d
```

Visit: `http://localhost:8080`

### Nginx Proxy Cache 控制

Project now includes a proxy cache control block in `nginx/nginx.conf`:

- Cache endpoint: `/proxy-cache/*` (demo upstream `https://word.tips/`)
- Cache status headers:
  - `X-Proxy-Cache: HIT | MISS | BYPASS | EXPIRED`
  - `X-Proxy-Cache-Key: ...`
- Cache bypass rules:
  - query: `?nocache=1`
  - request header: `Cache-Control: no-cache` / `no-store`
  - any request with `Authorization` header
  - write methods (`POST|PUT|PATCH|DELETE`)

Example:

```bash
curl -I "http://localhost:8080/proxy-cache/word-scramble/"
curl -I "http://localhost:8080/proxy-cache/word-scramble/?nocache=1"
```

### Nginx Rate Limiting 组件

Rate limiting is enabled by default in `nginx/nginx.conf`:

- `location /` uses `page_rate_zone`:
  - rate: `8 req/s` per IP
  - burst: `40`
- `location /proxy-cache/` uses `api_rate_zone`:
  - rate: `20 req/s` per IP
  - burst: `80`
- Connection cap per IP:
  - `/`: `30`
  - `/proxy-cache/`: `40`
- Exceeded requests return `429` with JSON body.

Tune values in:

- `limit_req_zone ... rate=...`
- `limit_req ... burst=...`
- `limit_conn ...`

### Nginx Gzip / Brotli 压缩控制组件

Gzip is enabled by default:

- `gzip on`
- `gzip_comp_level 6`
- `gzip_min_length 1024`
- `gzip_types` includes text/css/js/json/xml/svg

Brotli is provided as an optional component:

- Example file: `nginx/brotli.conf.example`
- Your Nginx build must include `ngx_brotli` module before enabling Brotli directives.
- If supported, include the file and reload Nginx.

Stop service:

```bash
docker compose -f docker-compose.nginx.yml down
```

## 📄 License

MIT License
