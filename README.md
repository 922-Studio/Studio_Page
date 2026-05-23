# 922-Studio

The public landing page and creative portfolio for [922-Studio](https://studio.922-studio.com) — a hobby dev collective building web apps, APIs, games, and self-hosted infrastructure.

Live at: **https://studio.922-studio.com**

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 16.1.6 (App Router, standalone output) |
| Language | TypeScript 5+, React 19.2.3 |
| Styling | Tailwind CSS 4.2.1 |
| i18n | next-intl 4.8.3 (`en` / `de`) |
| Content | next-mdx-remote 5.0.0, gray-matter 4.0.3, reading-time 1.5.0 |
| Icons | lucide-react 0.575.0 |
| Analytics | Google Analytics 4 (`NEXT_PUBLIC_GA_MEASUREMENT_ID`) |
| Unit tests | Vitest 2.1.9, @testing-library/react 16.3.2 |
| E2E tests | Playwright 1.58.2, allure-playwright 3.5.0 |
| Reporting | Allure (projects: `studio-unit`, `studio-e2e`) |
| Container | Docker, port 3000 (internal) |
| Proxy | Traefik on the `proxy` network |
| CI/CD | GitHub Actions → reusable workflows (922-Studio/workflows) |

## Content Management

Content lives in `content/` as MDX files with gray-matter frontmatter. Reading time is calculated automatically at build time using `reading-time`.

```
content/
  blog/        # Blog posts (YYYY-MM-<slug>.mdx)
  people/      # Collaborator profiles
  projects/    # Project showcase entries
  timeline/    # Studio timeline entries
```

Each MDX file has a YAML frontmatter block. Pages load content via `next-mdx-remote` at build time (SSG).

## Locale Routing

Supported locales: `en` (default), `de`. All routes are prefixed with `[locale]`.

```
src/app/[locale]/
  layout.tsx   # Locale-aware layout with metadata, fonts, GA4, ThemeProvider
  page.tsx     # Home page
  blog/        # Blog listing and detail pages
```

Translation strings live in `messages/en.json` and `messages/de.json`. The locale cookie persists for 1 year.

## Quick Start

### Docker (production-like)

```bash
docker compose up --build
```

The app is available at http://localhost:3000. Traefik labels are active only on the server; local Docker runs the container directly.

### Local development

```bash
npm install
npm run dev
```

Open http://localhost:3000.

## Commands

| Command | Description |
|---------|-------------|
| `npm run dev` | Start dev server |
| `npm run build` | Production build |
| `npm start` | Start production server |
| `npm run lint` | ESLint |
| `npm run test:unit` | Vitest (watch mode) |
| `npm run test:unit:ci` | Vitest (single run) |
| `npm run test:coverage` | Vitest with coverage |
| `npm run test` | Playwright E2E |
| `npm run test:install` | Install Playwright browsers |

## Testing

- **Unit tests** (`Vitest`): component tests co-located with components (`*.test.tsx`). Run with `npm run test:unit`.
- **E2E tests** (`Playwright`): browser tests in `tests/`. Run with `npm run test`.
- **Allure reporting**: results published to the home lab Allure server.
  - Unit project ID: `studio-unit`
  - E2E project ID: `studio-e2e`

## Deployment

The pipeline triggers on every push to `main` and on manual dispatch.

Pipeline stages (in order):

1. `cancel-previous-runs` — cancels any in-progress run for the same branch
2. `version` — bumps the version and creates a git tag
3. `build` + `tests` — Docker build and Vitest unit tests run in parallel
4. `push-prod` — pushes the image to `registry.922-studio.com` as `studio:prod-vX.Y.Z` and `studio:prod`
5. `kick-off-e2e` — triggers the separate E2E workflow
6. `notify-success` / `notify-failure` — Discord notification

The polaris runner executes `deploy.sh` for zero-downtime deployment. The container runs on the `proxy` Docker network. Traefik routes `studio.922-studio.com` to port 3000.

On failure, a GitHub issue is created automatically and a Discord failure alert is sent.

## Key Source Files

| File | Purpose |
|------|---------|
| `next.config.ts` | Standalone output, next-intl plugin, MDX config |
| `src/i18n/routing.ts` | Locale definitions (`en`, `de`) |
| `src/app/[locale]/layout.tsx` | Root layout: fonts, metadata, GA4, ThemeProvider |
| `src/app/[locale]/page.tsx` | Home page |
| `src/components/` | Shared components (Header, Footer, ThemeToggle, GoogleAnalytics, sections) |
| `messages/en.json` | English translation strings |
| `messages/de.json` | German translation strings |
| `content/` | MDX content files |
| `Dockerfile` | Multi-stage build |
| `docker-compose.yaml` | Container config with Traefik labels |
| `deploy.sh` | Zero-downtime deployment script |
| `.github/workflows/deploy.yml` | CI/CD pipeline |

## Infrastructure

- **Network**: `proxy` (external Docker network managed by Traefik / HomeStructure)
- **Domain**: `studio.922-studio.com` via Cloudflare Tunnel
- **Auth**: None — fully public site, no forward-auth middleware
- **Registry**: `registry.922-studio.com`
- **Server**: home lab (`ssh lab`)

For server details see `~/HomeStructure/docs/` on the server or the Planner repo `server.md`.
