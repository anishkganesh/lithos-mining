# Lithos AI

[Demo video](https://www.youtube.com/watch?v=X2_BryBCPTg)

A mining-research application for screening projects and companies, organizing watchlists, inspecting technical documents, and asking AI-assisted questions about the available data.

## Overview

Lithos AI combines a Next.js dashboard with Supabase-backed mining records, document extraction, map/globe views, news exploration, and contextual AI chat. Data coverage depends on the configured database and external providers.

## Features

- Mining project and company screeners with filters and watchlists.
- Project detail and comparison interfaces with financial/resource fields.
- Geographic project visualization through map/globe components.
- PDF and HTML document viewing and AI-extracted highlights.
- Document upload and project-context chat.
- AI-assisted sensitivity explanations and investor-memo generation/export paths.
- News filtering/refresh routes and mining-focused web search.
- Scripts that load company and project records and run the database migration.

## Architecture

1. The signed-in Next.js interface loads project/company records and the user's watchlists from Supabase.
2. `lib/hooks/use-projects.ts` paginates project records, joins company names, and maps them to UI types; it can use dummy project data when a Supabase client is unavailable.
3. Document routes extract text and ask OpenAI for structured highlights, storing relevant results in Supabase.
4. Chat retrieves project context and processes supplied documents before calling OpenAI.
5. Firecrawl supplies web-search results. The search route can return explicitly marked mock fallback results if provider attempts fail.
6. The news page reads the configured news tables, and a refresh endpoint updates them.

Extraction uses GPT-4o and several chat/analysis routes use GPT-4o-mini in this snapshot. The sensitivity endpoint generates model-assisted analysis; it is not a validated financial valuation engine or a custom-trained geospatial model.

## Tech stack

| Layer | Implementation |
| --- | --- |
| Application | Next.js 15, React 19, TypeScript |
| UI and data tables | Tailwind CSS, Radix/shadcn, TanStack Table, Recharts |
| Identity and persistence | Supabase Auth, Postgres, Storage |
| AI and search | OpenAI, Firecrawl |
| Documents and exports | PDF.js/unpdf/pdf-lib, DOCX and spreadsheet utilities |
| Maps/3D | Mapbox, Three.js/React Three Fiber |
| Data loading | TypeScript scripts |

## Project structure

- `app/` — dashboard, screeners, detail pages, authentication, and API routes.
- `components/` — project/company/news interfaces, document panels, charts, maps, and chat.
- `lib/` — Supabase clients, hooks, types, data mapping, and supporting services.
- `supabase/migrations/` — database schema changes.
- `scripts/` — the database migration runner and company/project population scripts.
- `data/` — committed example datasets and data documentation.

## Run locally

```bash
git clone https://github.com/anishkganesh/lithos-mining.git
cd lithos-mining
npm ci
```

Create `.env.local` with your own service settings:

```dotenv
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-supabase-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-server-only-service-role-key
OPENAI_API_KEY=your-openai-key
FIRECRAWL_API_KEY=your-firecrawl-key
```

Provision the schema/data and configure Supabase Auth before starting:

```bash
npm run dev
```

Open `http://localhost:3000`, register/sign in, and inspect the loaded dataset. Production commands are `npm run build` and `npm start`.

## Configuration and data

The main flows depend on project/company records, user profiles and watchlists, document/highlight records, and news tables. Migrations cover resources such as `projects`, `companies`, `pdf_highlights`, and watchlists.

Review migration dependencies before applying them to a fresh project; two migrations share the `009` prefix. Storage buckets and policies must match the upload routes.

`npm run populate:companies` and `npm run populate:projects` invoke the committed population scripts. Read them before running: they write to the configured database. Review `npm run migrate:db` similarly and confirm its schema assumptions.

Map, search, and news providers require the settings used by each route or component. Keep privileged keys server-side.

## Usage

Sign in, explore the screeners, add projects to a watchlist, open a project detail panel, and inspect its documents alongside extracted highlights. Use contextual chat and exports as research aids, checking outputs against source documents. Enable news/search and import workflows only after provisioning their services and data.

Representative API routes include `/api/projects/[id]`, `/api/pdf/extract-highlights`, `/api/html/extract-highlights`, `/api/documents/upload`, `/api/chat`, `/api/sensitivity-analysis`, `/api/news/refresh`, and `/api/web-search`.

## Validation

```bash
npm run lint
npm run build
```

No automated end-to-end tests are configured. Check sign-in, database loading, watchlists, highlight extraction, chat, and exports against your own services.

## Deployment

Deploy as a Next.js application with provisioned Supabase resources and environment variables. Configure Auth redirects, storage access, document-processing limits, and any external news jobs for the deployment. Long extraction/import tasks may exceed serverless request limits and need a separate execution environment.

## Limitations

- Missing schema, policies, credentials, or imported data can leave individual screens incomplete.
- Some datasets and display values are examples/defaults; web search has a mock fallback.
- AI extraction and analytical responses require verification against original reports.

## Attribution and license

Documents, data, and provider services keep their own terms. No license file is included.
