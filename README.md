# Wook Scraper — Automated School Textbook Collection and Management System

A system that automates the collection, cataloguing, and lookup of school textbooks adopted by Portuguese schools, sourced from [Wook](https://www.wook.pt). Wook does not expose a public API, so this project replaces the manual navigation (year, teaching cycle, district, city, school, course, subjects) with an automated pipeline backed by a database cache.

Final year project, Computer Engineering, ISTEC Porto.

## Project repositories

This repository is the index for the system. The code lives in three independent repositories, managed here as Git submodules:

| Repository | Role | Stack |
| :--- | :--- | :--- |
| [WebScrapperApi](https://github.com/ricardo4457/WebScrapperApi) | Central orchestrator. The only component with database access, exposes the public API, and dispatches scrape requests. | Laravel 12, MySQL |
| [WebScrapper](https://github.com/ricardo4457/WebScrapper) | Browser automation microservice. Consumes jobs from a queue, runs the navigation against Wook, and reports results back via callback. | Node.js, Playwright / Camoufox, BullMQ, Redis |
| [WebScrapper-Frontend](https://github.com/ricardo4457/WebScrapper-Frontend) | Guided search wizard used by end users, with an interactive map of Portugal and price history. | Vue 3, Pinia, Vuetify |

Each repository has its own README with details on internal architecture, endpoints, environment variables, and setup instructions. This document only covers the system-level view.

## Architecture

Laravel is the single source of truth. It decides when a scrape is needed, sends the task to the Node service through a queue, and is the only component with direct database access. Node never touches the database; the only contract between the two is an authenticated HTTP callback.

![Context diagram](./docs/Diagrama%20de%20Contexto.drawio.png)

- **Website (Vue)** sends search requests and polls scraping status through the **Laravel API**.
- **Laravel API** checks the database first; on a cache miss, it sends the parameters to the **Job Queue** (BullMQ / Redis).
- The **Scrapper** (Node.js / Playwright) consumes the queue, collects raw data from **Wook**, and sends the processed data back to the Laravel API via callback.
- The Laravel API persists the result, and the Website then serves the response straight from the database.

## Use cases

![Use case diagram](./docs/Diagrama%20Use%20case.drawio.png)

- **Website User**: searches for books, looks up textbooks adopted by a school, and checks price history.
- **API User**: checks the status of a scraping operation through the public API.
- **WebScrapper API / Scraping Service**: internal flow covering start, extraction, callback validation, and persistence of the scraped books, including updates to existing records and price history.

## Security and communication flow

| From | To | Purpose | Mechanism |
| :--- | :--- | :--- | :--- |
| Vue.js | Laravel | Trigger scraping, check status, and search books | API key, CORS, rate limiting |
| Laravel | Node.js | Send scraping tasks to the queue | Isolated internal network |
| Node.js | Laravel | Report results and update job status | Shared token, `VerifyNodeApiKey` middleware |

## Getting started

Clone the repository together with its submodules:

```bash
git clone --recurse-submodules https://github.com/ricardo4457/Projecto_MainRepo.git
```

If you already cloned it without `--recurse-submodules`:

```bash
git submodule update --init --recursive
```

There is no single root-level `docker-compose` that brings up all three services together. Each service starts independently, following its own README:

1. **WebScrapperApi** (Laravel): see the [README](https://github.com/ricardo4457/WebScrapperApi#getting-started) for migrations, `.env`, and `php artisan serve`.
2. **WebScrapper** (Node.js): ships with its own `docker-compose.yml` including Redis and RedisInsight, see the [README](https://github.com/ricardo4457/WebScrapper#readme).
3. **WebScrapper-Frontend** (Vue): `npm install && npm run dev`, see the [README](https://github.com/ricardo4457/WebScrapper-Frontend#readme).

The three services share secrets with each other (API key, Node ↔ Laravel token) that need to be configured manually in each `.env`.

## Testing

Each repository runs its own test suite:

```bash
# WebScrapperApi
php artisan test

# WebScrapper
npm test

# WebScrapper-Frontend
npx vitest run
```

## License

MIT — see [LICENSE](LICENSE).