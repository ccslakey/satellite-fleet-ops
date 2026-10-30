# Satellite Fleet Operations

A fleet operations console for a ride-share satellite operator. It propagates live sub-satellite positions
from TLEs, draws ground tracks and visibility footprints on a contact-planning map, predicts passes over
contracted ground stations, schedules contacts with double-booking detection, and tracks incident reports
with threaded comments.

React, TypeScript, Apollo Client and GraphQL Codegen on the client; a mock GraphQL API on the server; SGP4
propagation via satellite.js; 92 Vitest tests and a seven-job CI pipeline.

## Features

- **Live fleet view.** Satellite positions recomputed from TLEs every second on the server and polled every 5s
  by the client, with filter, sort, and search state kept in the URL.
- **Contact-planning map.** Client-side SGP4 propagation, ground tracks, visibility footprints, station links,
  and next-contact windows, loaded as its own lazy chunk.
- **Pass prediction and scheduling.** Computed pass windows per satellite and station, with double-booking
  warnings before a contact is saved.
- **Incident reports.** Raise reports and comment on them, with an optimistic `createComment` write.
- **Scale profile.** `SEED_PROFILE=large pnpm dev` seeds ~60 satellites from real CelesTrak TLEs (Iridium NEXT,
  Planet, Spire, OneWeb) and 17 ground stations to exercise filters, map density, and list truncation.

## Routes

Eight routes under a persistent shell:

| Route | View |
| --- | --- |
| `/fleet` | Satellite list with live positions, pass timelines, and URL-backed filter/sort/search |
| `/fleet/:satelliteId` | Satellite detail: position, orbit, spacecraft, launch, payloads, upcoming passes, contacts, reports |
| `/ground-stations` | Contracted antenna sites |
| `/ground-stations/:stationId` | Station detail: upcoming passes across the fleet and its contact history |
| `/map` | Contact-planning map: live client-propagated positions, ground tracks, footprints, links, next-contact windows |
| `/contacts` | Scheduled contacts grouped into in-progress, upcoming, and past |
| `/contacts/new` | Schedule a contact: computed pass windows with double-booking warnings |
| `/reports` | Raise reports and comment on them |

## Stack

| Layer | Choice |
| --- | --- |
| UI | React 18, TypeScript, React Router |
| Data | Apollo Client with GraphQL Codegen (client preset); types generated from a committed `schema.graphql` snapshot so type-checking and CI run without a live server |
| State | Apollo's normalized cache plus URL state. No separate store |
| Orbital mechanics | satellite.js for SGP4 propagation |
| Styling | SCSS Modules with a token and mixin layer; saturated colour reserved for state |
| API | Mock GraphQL server (`json-graphql-server`) in `react/apps/server` |
| Testing | Vitest, 92 tests across 9 files |
| Tooling | pnpm workspace, Turborepo, Vite, ESLint, Stylelint, Prettier, Docker |

## Quality gates

Every push and pull request runs seven CI jobs in parallel: `test`, `lint`, `lint:styles`, `format:check`,
`type-check`, `build`, and `codegen-drift`. The last one regenerates `schema.graphql` and the typed documents
from live introspection and fails on any diff, so a server shape change cannot compile green against stale
committed types.

| Command | Result |
| --- | --- |
| `pnpm test` | 92 tests across 9 files |
| `pnpm type-check` | `tsc` over the dashboard (tests included) and the server |
| `pnpm lint` | ESLint with `--max-warnings 0` |
| `pnpm lint:styles` | Stylelint over the SCSS |
| `pnpm format:check` | Prettier |
| `pnpm build` | Type-check and production build |
| `pnpm codegen` | Regenerates typed documents; works with no server running |

## Running it

```sh
cd react
make dev          # builds the dev image, starts the container, drops you into a shell
pnpm install      # inside the container
pnpm dev          # dashboard on :8080, API on :3000/graphql
```

Everything runs inside Docker because the workspace pins Node `>=20.10.0 <21`.

## Where to look

| Document | What it covers |
| --- | --- |
| [`WORK_SUMMARY.md`](./WORK_SUMMARY.md) | What was built, architecture, changes to the template, testing, decisions |
| [`ASSUMPTIONS.md`](./ASSUMPTIONS.md) | Assumptions baked into the code, each pointing at where it lives |
| [`react/README.md`](./react/README.md) | Workspace layout and every available script |
| [`react/apps/dashboard/README.md`](./react/apps/dashboard/README.md) | Front-end architecture: data layer, routing, styling, design direction |

The mock GraphQL API in [`react/apps/server`](./react/apps/server) and the Docker dev environment started from
a starter template; [`WORK_SUMMARY.md`](./WORK_SUMMARY.md) lists every change made to them.

## AI usage

Claude Code was used substantially throughout. Specifically:

- **Environment and verification.** Bringing up the Docker environment, and running the build, type-check, tests,
  and browser checks after each change (side-by-side testing).
- **Scaffolding.** The Apollo client and codegen configuration, the router, the SCSS token system, and the route
  components were largely AI-written under direction.
- **Investigation.** API quirks, SGP4 math, and the satellite.js interface.
- **Review and correction.** Several defects were caught this way, including various clipped displays and a
  TypeScript directive being deleted during a comment cleanup.
