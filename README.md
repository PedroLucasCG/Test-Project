# Amazon Product Scraper — Full Stack (Backend + Frontend in one repo)

Scrape Amazon search results via an Express API (runs great on **Bun**) and visualize them with a Vite + Vanilla JS frontend.

> If you want to check the commits for each project go to the following repos:
> Frontend: https://github.com/PedroLucasCG/scraperApp
> Backend: https://github.com/PedroLucasCG/scraper

---

## Quick Start (Windows + Bun)

1. **Install Bun**

```powershell
irm bun.sh/install.ps1 | iex
# reopen your terminal so bun is on PATH
bun --version
```

2. **Clone & install dependencies**

```powershell
git clone https://github.com/PedroLucasCG/Test-Project.git scraper
cd scraper
```

```powershell
cd scraperAPI
bun install
cd ..
cd scrapperAPP
bun install
```

Copy from the example and adjust values for each project:

```powershell
# cd into each folder and run the command, fill the enviroment variables as indicated
Copy-Item .env.example .env
```

**Backend `.env` (root)** — common defaults:

```env
PORT=8082
SCRAPING_URI=https://www.amazon.com
MAX_PAGES=3
CORS_ORIGINS=http://localhost:5173
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX=300
MORGAN_FORMAT=dev
DEBUG=0
```

**Frontend `.env` (for Vite)** — make sure it includes `/api`:

```env
VITE_API_BASE=http://localhost:8082/api
```

4. **Run the servers (two terminals)**

**Terminal A — backend**

```powershell
bun run server.js
# or, if you add the script below:
# bun run dev:server
```

**Terminal B — frontend**

```powershell
bunx vite
# or, with the script below:
# bun run dev:client
```

Open: [http://localhost:5173](http://localhost:5173)

---

## Repository Structure

```
.
Test-Project/
├─ scraperAPI/                     # Backend (Express/Bun)
│  ├─ server.js
│  ├─ routes/
│  │  ├─ index.js
│  │  └─ scraper.js
│  ├─ services/
│  │  └─ scraper.js               # buildUrl, fetchHtml, parseProducts, findTotalPages
│  ├─ .env.example
|  ├─ .gitignore
|  ├─ .env                        # (gitignored)
│  └─ package.json                
│
└─ scrapperAPP/                    # Frontend (Vite + Vanilla JS)
   ├─ index.html
   ├─ package.json
   ├─ README.md
   ├─ .gitignore
   ├─ .env.example                      # contains VITE_API_BASE
   ├─ .env                              # (gitignored)
   │
   ├─ public/
   │  ├─ background.png
   │  ├─ example.jpg
   │  ├─ icon.svg
   │  ├─ link.svg
   │  ├─ message-bubble.svg
   │  ├─ shopping-cart.svg
   │  ├─ star-half.svg
   │  └─ star.svg
   │
   └─ src/
      ├─ main.js
      ├─ apiModel.js
      ├─ style.css
      ├─ resultsBuilder.js
      └─ cardBuilder.js
---

## `package.json` Scripts (Bun)

```json
{
  "scripts": {
    "dev:server": "bun run server.js",
    "dev:client": "bunx vite",
    "dev": "bunx concurrently \"bun run dev:server\" \"bunx vite\"",

    "build": "bunx vite build",
    "preview": "bunx vite preview --port 5173",

    "lint": "bunx eslint .",
    "format": "bunx prettier -w ."
  }
}
```

> Bun auto-loads `.env` for the backend when you run `bun run ...`. Frontend config must use `VITE_*` variables.

---

## Backend API

### Endpoint

`GET /api/scrape?keyword=…&page=1&pages=1`

**Query params**

* `keyword` **(required)** — search term
* `page` *(default: 1)* — which Amazon page to fetch
* `pages` *(default: 1, max = `MAX_PAGES`)* — how many consecutive pages to fetch this request

**Response**

```json
{
  "keyword": "notebook",
  "page": 1,
  "currentPage": 1,
  "totalPages": 7,
  "pagesFetched": 1,
  "hasNext": true,
  "nextPage": 2,
  "count": 24,
  "products": [
    {
      "asin": "B0XXXXXXX",
      "title": "Great Notebook 13\u201d",
      "rating": 4.6,
      "reviews": 2897,
      "image": "https://…/image.jpg",
      "url": "https://www.amazon.com/dp/B0XXXXXXX"
    }
  ]
}
```

**Errors**

```json
{ "error": "Missing ?keyword=" }
```

```json
{ "error": "Scraping of https://www.amazon.com failed.", "details": "HTTP 503" }
```

### Notes

* Pagination parsed from Amazon `.s-pagination-strip`.
* Sponsored results filtered out.
* CORS allowlist via `CORS_ORIGINS` (use `http://localhost:5173` for Vite).
* Rate limiting with `express-rate-limit` using IPv6‑safe `ipKeyGenerator`.

---

## Frontend Usage

* Enter a keyword (e.g., `wireless headphones`).
* Pick how many pages the **backend** should fetch (`1–3`).
* Click **Search Products**.
* Navigate with **Prev / Next** or page numbers.

**UI features**

* Multi-line ellipsis for long titles
* Accessible pagination (`aria-current`)
* Clean cards: image, title, stars, review count, “View on Amazon”
