# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal portfolio site (philipaz.com) built with Next.js 16 (App Router), React 19, TypeScript, Tailwind CSS v4, shadcn/ui (new-york style), and Framer Motion. It is a fully static export deployed to GitHub Pages.

## Commands

Package manager is **pnpm**.

```bash
pnpm install          # install deps
pnpm dev              # dev server at http://localhost:3000
pnpm build            # static export to ./out
pnpm lint             # eslint --fix, then eslint (next core-web-vitals + typescript configs)
pnpm format           # prettier over js/ts/tsx/css/md
pnpm exec tsc --noEmit  # type check (CI runs this but with continue-on-error)
```

There is no test suite.

Prettier config: 4-space indentation, double quotes, semicolons, 80-char width.

## Architecture

- **Single-page site.** `app/page.tsx` composes the homepage from section components in `components/sections/` (hero, about, projects, experience, education, research, testimonials) plus `components/github-activity.tsx` and `components/cta-section.tsx`. Each section renders a `<section id="...">` that the anchor links in `navigation` (in `data/site.ts`) scroll to.
- **Content lives in `data/site.ts`.** All personal info, nav items, tech stack, projects, experiences, education, honors, research, and testimonials are exported typed constants there. Sections import them directly and derive their prop types from the data (e.g. `(typeof experiences)[0]`), so changing content usually means editing only `data/site.ts`; changing the shape of an entry means updating the consuming section too.
- **Static export constraints** (`next.config.ts`): `output: "export"`, `trailingSlash: true`, unoptimized images. Avoid features that need a server (API routes, dynamic rendering, `next/image` optimization, middleware). Metadata routes like `app/sitemap.ts` must set `export const dynamic = "force-static"`.
- **basePath handling.** `NEXT_PUBLIC_PAGES_BASE_PATH` sets Next's `basePath`/`assetPrefix` and is exposed via `appConfig.basePath` in `lib/config.ts`. Raw asset URLs that bypass Next's routing (e.g. `<img src>`, metadata icons/manifest in `app/layout.tsx`) must be prefixed manually with `appConfig.basePath`. It is currently empty in production because the site is served from a custom domain (`CNAME`).
- **Client components.** Sections using Framer Motion, `next-themes`, or interactive state are `"use client"`. `components/animated-section.tsx` is the shared scroll-reveal wrapper. Theming is class-based via `next-themes` with CSS variables defined in `app/globals.css` (`:root` / `.dark`).
- **GitHub activity** is rendered client-side with `react-github-calendar` for the hardcoded username `epigos`.
- **MDX** is wired in `next.config.ts` (`@next/mdx`, `.md/.mdx` page extensions) but no MDX pages currently exist. The README's references to `app/research/`, `components/mdx/`, and `data/github.ts` are outdated — those paths do not exist.
- shadcn/ui primitives live in `components/ui/`; add new ones via the shadcn CLI (config in `components.json`, `@/` path alias).

## Deployment

`.github/workflows/deploy.yml` builds on every push to `main` (pnpm 9, Node 20) and deploys `./out` to GitHub Pages. The Google Analytics ID is injected at build time via `NEXT_PUBLIC_GOOGLE_ANALYTICS_ID` in the workflow; locally it is empty.
