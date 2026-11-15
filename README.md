# CatchUp 🌵

**Catch what's happening on campus. Remember what you learned there.**

A Pokémon Go–style app for ASU students: career fairs, club tabling, workshops, free food and socials show up on a campus map as critters. Walk over and catch them. Then (the part that matters) spend 30 seconds on **Field Notes** while it's fresh (who you met, the recruiter's email, the next step) and get a **follow-up reminder** that actually turns an event into an outcome.

**Plume** 🪶, an original Gambel's quail guide, is the app's AI assistant: ask in plain words or by voice ("free food near me right now?"), hear answers read aloud, and dictate Field Notes instead of typing. New events from **Sun Devil Central** show up on the map in real time.

> Working name. Not affiliated with or endorsed by Arizona State University. Don't use ASU marks (logos, Sparky, the pitchfork) without approval from ASU Trademark Licensing.

## What's in here

```
catchup/
├── assets/critters/        Original critter art (SVG). Source of truth for all art.
├── assets/guide/plume.svg  Plume, the AI guide
├── packages/shared/        Game rules shared by server, app and prototype (pure TS, fully tested)
│   └── src/
│       ├── critters.ts     Roster, rarity, evolution stages
│       ├── classify.ts     Event → category (career / tabling / food / …), perks, flagship
│       ├── campus.ts       Tempe building gazetteer: "MU 2nd floor" → lat/lng
│       ├── spawning.ts     Event → spawns (with grace windows, legendary window)
│       ├── rules.ts        Geofence + anti-cheat validation, catch odds, XP, levels, quests, Field Notes prompts
│       └── art.generated.ts  (generated) SVG strings + silhouette helper
├── apps/api/               Fastify + Drizzle + Postgres
│   ├── src/routes/         auth · spawns · catch · me · organizer · admin · ai · live (WebSocket)
│   ├── src/ai/             Claude provider (tool-use agent, enrichment, notes) + basic fallback
│   ├── src/live/           Real-time bus → /live WebSocket (new events, crowd counts)
│   ├── src/ingest/         Adapters (Sun Devil Central, Engage API, iCal) → normalize → upsert → spawns; scheduler
│   ├── drizzle/            SQL migrations
│   └── test/               End-to-end API tests on an embedded Postgres
├── apps/mobile/            Expo (React Native) app, SDK 58 + expo-router
├── prototype/              Clickable web demo built from the same shared rules
└── docs/                   ARCHITECTURE.md · GAME_DESIGN.md
```

## Quick start

Requires Node 22+.

```bash
npm install
cp .env.example .env

# API: embedded Postgres (PGlite), no Docker needed
npm run seed          # a realistic "today on the Tempe campus"
npm run dev:api       # http://localhost:4000

# Mobile
cd apps/mobile
npx expo install --fix        # align native module versions with your Expo Go
EXPO_PUBLIC_API_URL=http://<your-LAN-IP>:4000 npx expo start
```

Set `ANTHROPIC_API_KEY` in `.env` for Claude-powered Plume (without it, Plume runs in a rule-based basic mode). Sign in with any `@asu.edu` address. In dev, the API returns the magic-link token directly and the login screen shows a **Dev: sign in now** button. The seed also creates `organizer@asu.edu` so you can test check-in codes.

On a phone, the map centers on wherever you actually are. To see spawns from off campus, temporarily change `center` in `apps/mobile/app/(tabs)/index.tsx` to the Tempe coordinates, or use a simulator location.

### Other commands

| Command | What it does |
|---|---|
| `npm test` | Shared rules (20 tests) + API end-to-end, live and AI (22 tests) |
| `npm run typecheck` | Strict TS across shared + API (`cd apps/mobile && npm run typecheck` for the app) |
| `npm run art` | Regenerate `art.generated.ts` after editing any SVG in `assets/critters/` |
| `npm run prototype` | Build the clickable demo → `prototype/dist/index.html` |
| `npm run db:generate -w @catchup/api` | New migration after changing `schema.ts` |
| `ICS_FEEDS="asu_events\|https://…" npm run ingest -w @catchup/api` | Pull real events from iCal feeds |

## The critters

| # | Critter | Real animal | Spawns at | Rarity |
|---|---|---|---|---|
| 001 | Tabbit | Antelope jackrabbit | Tabling, club fairs | Common |
| 002 | Tieguaro | Saguaro | Career fairs, info sessions | Uncommon |
| 003 | Javelunch | Javelina | Free food | Common |
| 004 | Hootorial | Elf owl | Talks, workshops, lectures | Common |
| 005 | Groovila | Gila monster | Socials, concerts, culture nights | Uncommon |
| 006 | Bytescorp | Bark scorpion | Hackathons, tech events | Rare |
| 007 | Dashrunner | Roadrunner | Rec, intramurals, fitness | Common |
| 008 | Kindwing | Costa's hummingbird | Volunteering, service | Uncommon |
| 009 | Monsoonix | Monsoon storm | First 90 min of flagship events | Legendary |

Each evolves at 5 and 15 catches (Tieguaro → Suitguaro → CEOguaro). Details in [docs/GAME_DESIGN.md](docs/GAME_DESIGN.md).

## AI, accessibility and real time (short version)

| Feature | Where | How it stays trustworthy |
|---|---|---|
| **Ask Plume** (type or dictate; answers read aloud) | `apps/api/src/ai/claude.ts`, `app/ask.tsx` | Claude can only cite events returned by DB-backed tools; invented IDs are dropped |
| **Plain-language summaries + access info** on every listing | `ai/enrich.ts` (runs after each ingest) | Access details are copied from the listing, never guessed; "not listed → ask the host" |
| **Voice Field Notes** ("just say what happened") | `POST /ai/notes`, notes screen | Fills the form for review; never saves on its own |
| **Easy catch, reduce motion, read aloud**, VoiceOver/TalkBack labels | `src/settings.tsx` | Defaults follow the phone's own accessibility settings |
| **Live Sun Devil Central feed** | `ingest/adapters/sunDevilCentral.ts`, `ingest/scheduler.ts`, `routes/live.ts` | Polls every 5 min, pushes new events + "🔥 N here now" over WebSocket; no personal data in pushes |

## Where to start reading

1. `packages/shared/src/rules.ts`: every game number and rule in one file
2. `apps/api/src/routes/catch.ts`: the core loop on the server
3. `apps/api/test/api.test.ts`: the whole product, as a test
4. [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md): system design, data sources, privacy, roadmap
