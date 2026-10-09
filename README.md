# Lithos AI

A mining-research application for screening projects and companies, organizing watchlists, inspecting technical documents, and asking AI-assisted questions about the available data.

## Overview

Lithos AI combines a Next.js dashboard with Supabase-backed mining records, document extraction, map/globe views, news exploration, and contextual AI chat. It is an analytical application prototype: data availability and completeness depend on the configured database and external providers.

## Features

- Mining project and company screeners with filters and watchlists.
- Project detail and comparison interfaces with financial/resource fields.
- Geographic project visualization through map/globe components.
- PDF and HTML document viewing and AI-extracted highlights.
- Document upload and project-context chat.
- AI-assisted sensitivity explanations and investor-memo generation/export paths.
- News filtering/refresh routes and mining-focused web search.
- Scripts for population and FactSet filing/document experiments.

## Architecture

1. The signed-in Next.js interface loads project/company records and the user's watchlists from Supabase.
2. `lib/hooks/use-projects.ts` paginates project records, joins company names, and maps them to UI types; it can use dummy project data when a Supabase client is unavailable.
3. Document routes extract text and ask OpenAI for structured highlights, storing relevant results in Supabase.
4. Chat retrieves project context and processes supplied documents before calling OpenAI.
5. Firecrawl supplies web-search results. The search route can return explicitly marked mock fallback results if provider attempts fail.
6. News endpoints read configured tables and refresh pipelines; scripts perform separate import/extraction tasks.

Extraction uses GPT-4o and several chat/analysis routes use GPT-4o-mini in this snapshot. The sensitivity endpoint generates model-assisted analysis; it is not a validated financial valuation engine or a custom-trained geospatial model.

## Tech stack

| Layer | Implementation |
| --- | --- |
| Application | Next.js 15, React 19, TypeScript |
| UI and data tables | Tailwind CSS, Radix/shadcn, TanStack Table, Recharts |
| Identity and persistence | Supabase Auth, Postgres, Storage |
| AI and search | OpenAI, Firecrawl |
| Documents and exports | PDF.js/unpdf/pdf-lib, DOCX and spreadsheet utilities |
| Maps/3D | Leaflet, Mapbox-related components, Three.js/React Three Fiber |
| Data experiments | TypeScript scripts and FactSet SDKs |

## Project structure

- `app/` — dashboard, screeners, detail pages, authentication, and API routes.
- `components/` — project/company/news interfaces, document panels, charts, maps, and chat.
- `lib/` — Supabase clients, hooks, types, data mapping, and supporting services.
- `supabase/migrations/` — database schema changes.
- `scripts/` — migration runners, seed/import utilities, and provider experiments.
- `data/` — committed example datasets and data documentation.
- `downloads/` — document artifacts used by extraction experiments.

## Run locally

```bash
git clone https://github.com/anishkganesh/Lithos-AI.git
cd Lithos-AI
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

The main flows depend on project/company records, user profiles and watchlists, document/highlight records, and news tables. Migrations cover resources such as `projects`, `companies`, `pdf_highlights`, and watchlists; the news API queries `unified_news`.

Review migration dependencies and actual queries before applying them to a fresh project. Two migrations share the `009` prefix, and the repository is not a verified one-command database bootstrap. Storage buckets and policies must match upload routes. Imported files and seed records do not establish that every UI field is sourced or current; for example, the project hook supplies a default risk level.

`npm run populate:companies` and `npm run populate:projects` invoke the committed population scripts. Read them before running: they write to the configured database. Review `npm run migrate:db` similarly and confirm its schema assumptions. Do not run all experimental scripts indiscriminately.

FactSet experiments require provider access and script-specific credentials; they are not necessary for every dashboard screen. Map/search/news providers likewise require the settings used by the particular route or component. Keep privileged keys server-side and inspect every Supabase client's credentials: a client named “service” does not by itself imply service-role privileges.

## Usage

Sign in, explore the screeners, add projects to a watchlist, open a project detail panel, and inspect its documents alongside extracted highlights. Use contextual chat and exports as research aids, checking outputs against source documents. Enable news/search and import workflows only after provisioning their services and data.

Representative API routes include `/api/projects/[id]`, `/api/pdf/extract-highlights`, `/api/html/extract-highlights`, `/api/documents/upload`, `/api/chat`, `/api/sensitivity-analysis`, `/api/news`, and `/api/web-search`.

## Validation

```bash
npm run lint
npm run build
```

No single automated end-to-end test command is configured. Validate authentication, database loading, watchlist persistence, extraction/highlight viewing, chat, and exports with your own services. Distinguish real provider responses from mock fallback results. This documentation review did not execute the application or paid provider/import scripts.

## Deployment

Deploy as a Next.js application with provisioned Supabase resources and environment variables. Configure Auth redirects, storage access, document-processing limits, and any external news jobs for the deployment. Long extraction/import tasks may exceed serverless request limits and need a separate execution environment.

## Limitations

- Missing schema, policies, credentials, or imported data can leave individual screens incomplete.
- Some datasets and display values are examples/defaults; web search has a mock fallback.
- AI extraction and analytical responses require verification against original reports.
- Included scripts span multiple experiments and are not all active application paths.

## Attribution and license

Documents, data, and provider services retain their respective terms. No standalone project license file is included; this README does not grant a new license.
