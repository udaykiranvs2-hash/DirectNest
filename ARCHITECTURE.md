# Architecture (decisions are fixed unless I say otherwise)

## Product
Direct-to-owner property marketplace for India. Mobile-first. Languages: English, Telugu, Hindi.
Full requirements are in docs/SRS.md. Build only what each task asks for.

## Tech choices
- Framework: Next.js (App Router) with TypeScript
- Styling: Tailwind CSS
- Database, login, file storage: Supabase (PostgreSQL)
- Translations: next-intl
- Maps (later): Leaflet + OpenStreetMap
- Hosting: Vercel
- Code history: Git + GitHub

## Folder layout
- /src/app        pages and routes
- /src/components reusable UI pieces
- /src/lib        helpers (database client, utilities)
- /messages       translation files: en.json, te.json, hi.json
- /supabase       database migrations (SQL files)
- /docs           SRS.md, ARCHITECTURE.md, PROGRESS.md

## Key decisions
1. All visible text comes from /messages files. No hard-coded text in pages.
2. Every database table has Row Level Security (RLS) turned on, with policies written.
3. Three user roles: customer, owner, admin. Roles are stored in the database and checked on the server, not just hidden in the page.
4. Only listings with status "approved" are visible to the public.
5. Third-party services (maps, SMS, AI, payments) are used through small wrapper files in /src/lib, so they can be swapped later.
6. Secrets live in .env.local only. .env.local is never committed.
7. Money is stored as whole numbers in paise (rupees x 100) to avoid rounding errors.