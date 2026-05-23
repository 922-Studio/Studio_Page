# Project: 922-Studio

## Overview
- **Type**: app (public website)
- **Path**: /Users/gregor/dev/922/studio
- **Status**: active
- **Description**: Public landing page and creative portfolio at studio.922-studio.com. A fully public Next.js 16 site with no authentication, supporting `en` (default) and `de` locales via next-intl. Content lives as MDX files in `content/` with gray-matter frontmatter. Deployed as a Docker container on the `proxy` network, routed by Traefik.

## Tech Stack
- **Language(s)**: TypeScript 5+ (strict), React 19
- **Framework(s)**: Next.js 16, next-intl, Tailwind CSS 4, next-mdx-remote, gray-matter
- **Infrastructure**: Docker, Traefik (`proxy` network), Cloudflare Tunnel
- **CI/CD**: GitHub Actions (reusable workflows from `922-Studio/workflows`), polaris runner → `deploy.sh`

## Key Files to Read

| File | Purpose | When to read |
|------|---------|--------------|
| `package.json` | Dependencies and scripts | Before dependency/script changes |
| `next.config.ts` | Build, MDX, i18n plugin config | Before build config changes |
| `src/i18n/routing.ts` | Locale config | Before i18n / new locale changes |
| `src/app/[locale]/layout.tsx` | Layout, fonts, metadata, GA4 | Before layout changes |
| `src/app/[locale]/page.tsx` | Home page | Before home page edits |
| `messages/en.json` + `messages/de.json` | Translations — keep both in sync | Before any copy change |
| `content/` | MDX content + frontmatter schema | Before content/schema changes |
| `src/components/` | Shared components | Before component changes |
| `Dockerfile` | Multi-stage build | Before build pipeline changes |
| `docker-compose.yaml` | Container + Traefik labels | Before deployment changes |
| `deploy.sh` | Zero-downtime deploy script | Before deployment logic changes |
| `.github/workflows/deploy.yml` | CI/CD pipeline | Before pipeline changes |

## Best Practices
- All routes live under `src/app/[locale]/` — never add routes outside the locale segment.
- Use `generateStaticParams` for all locale routes (SSG).
- Tailwind CSS 4 utility classes — no custom CSS unless absolutely necessary.
- `@/*` path alias for `src/`.
- MDX files in `content/<section>/` with YAML frontmatter; reading-time computed at build.
- Co-locate test files (`*.test.tsx`) next to the component.
- Public site — do NOT add forward-auth or auth middleware to Traefik labels.
- Always update both `messages/en.json` and `messages/de.json` together.

## Testing Strategy
- **Unit tests**: Vitest — `npm run test:unit` (watch), `npm run test:unit:ci` (single run), `npm run test:coverage`
- **E2E tests**: Playwright — `npm run test` (browsers via `npm run test:install`)
- **Allure project IDs**: `studio-unit` (unit), `studio-e2e` (E2E)
- **Lint**: `npm run lint`

## Documentation
- **Where**: `README.md`, translation files in `messages/`
- **Update rule**: Keep `en.json` and `de.json` in sync on every copy change; update `README.md` on structural changes.

## Pipeline & Deployment
- **CI trigger**: Push to `main` + manual dispatch
- **Pipeline**: cancel-previous → version → tests → deploy → Discord notify
- **Deploy**: GitHub Actions → polaris runner → `./deploy.sh` (zero-downtime, build-first)
- **Container**: `studio` on `proxy` network, internal port 3000, Traefik-routed
- **Monitor after push**: CI green, Discord notification, https://studio.922-studio.com reachable

## Dependencies on Other Projects
- **HomeStructure**: Traefik `proxy` network + Cloudflare Tunnel routing
- **922-Studio/workflows**: Reusable CI/CD workflows

## Notes
- Live domain: studio.922-studio.com (public, no login)
- Port 3000 is internal only — Traefik handles external routing
