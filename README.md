# ScholaU

**Your university courses. Clearer lessons. Focused practice. Competitive learning.**

[Live application](https://scholau.com) · [John Nwaokolo](https://github.com/JohnNwaokolo) · [Organization repository](https://github.com/D-GENIUSES/SCHOLAU) · [Technical documentation](docs/README.md)

ScholaU is a university learning platform that brings curriculum-based study, exam preparation, and social competition into one application. Students follow courses relevant to their academic profile, study structured lessons, practise with feedback, take mock assessments, and challenge other students in timed quiz battles.

The platform also gives approved lecturers tools for their assigned courses and administrators tools to review content, manage access, and operate the academic catalogue. This repository contains the application source, shared TypeScript packages, database migrations, and technical documentation.

## Project ownership and this repository

This is **John Nwaokolo's personal portfolio copy of ScholaU**, a project developed within **D-GENIUSES**. It presents the application and its engineering work under a personal GitHub URL while retaining the project's original Git history and contributor attribution.

The [D-GENIUSES repository](https://github.com/D-GENIUSES/SCHOLAU) remains the upstream development repository. This copy is maintained separately; updates do not synchronize automatically. Its `main` branch starts from upstream `main`, with a portfolio-specific README. The upstream `staging` branch is retained separately for reference. The commit history records individual contributions; this copy does not imply exclusive authorship of every contribution.

## The problem ScholaU addresses

University study often means moving between course outlines, lecture materials, revision questions, and separate tools for tracking progress. Generic question banks can also miss the student's actual programme, level, or semester.

ScholaU organizes learning around the student's academic context. Content access follows university, programme, level, semester, and course enrollment. Students can move from a course lesson to a knowledge check, revisit difficult topics, prepare under assessment conditions, and practise with classmates in the same learning environment.

## Student experience

### Academic onboarding and course selection

- Register with email and password, verify an email address, and recover account access when needed.
- Use Google authentication when the hosted provider is configured; new Google identities complete registration before accessing protected learning features.
- Set an academic profile so the application can present relevant course offerings.
- Manage personal profile details, profile imagery, and ranking visibility.

### Structured course learning

- Browse course offerings and open the lessons available for each course.
- Study structured lesson content, including explanations and supported learning assets.
- Record lesson progress and use lesson-linked knowledge checks to assess understanding.
- Return to scheduled topic reviews as part of the mastery workflow.

Lesson completion and mastery are distinct: completing a lesson records progress, while qualifying assessment performance drives learning rewards. Content must be published and available to the student's academic context before it can be accessed.

### Practice, mock exams, and results

- Use review assessments to check understanding of a lesson or topic.
- Practise course questions and use mock assessments for exam preparation.
- Work through the assessment player, submit attempts, and review results and available explanations.
- Keep attempt history and progress tied to the authenticated learner.

Assessment access, submission, grading, and rewards are validated through backend contracts. The browser presents the experience; it is not the authority for marks or XP.

### Friends and live quiz battles

- Find students by name or handle, send friend requests, and manage friendships.
- Challenge eligible friends, accept invitations, or enter random matchmaking.
- Select shared course offerings for directed challenges, subject to enrollment and published question availability.
- Answer timed questions, see match outcomes, and revisit battle history.

Battles use server-controlled reading intervals and answer deadlines. Selected question versions and timing survive reconnection, while future questions and answer keys remain hidden from active player payloads. Social restrictions and enrollment checks are enforced on the server.

### Progress, XP, and leaderboards

- Earn XP through qualifying learning and battle activity.
- Track levels, badges, review opportunities, and reward history.
- Compare progress through global and faculty leaderboards, with profile visibility controls.

The reward system uses an append-only ledger and explicit award limits. XP is a progress signal; it is not spent or used to lock course content. See the [XP specification](docs/features/xp.md) for exact rules and migration-specific behavior.

### Notifications and installable access

The web application includes a manifest for installation, an offline route, notification settings, and browser push integration. Push delivery depends on browser support, permission, subscription state, and configured backend delivery services. Installation does not make all authenticated course workflows available offline.

## Lecturer and administrator tools

| Role | Capabilities |
| --- | --- |
| Approved lecturer | Access assigned courses, publish course announcements, manage course challenges, and inspect challenge participation and results. |
| Content administrator | Import material, review and publish content, and manage course offerings and academic catalogues. |
| Platform administrator | Manage users and lecturer applications, inspect reports, review battle operations, and access audit tools. |

**Course challenges** let assigned lecturers publish immediate or scheduled assessments using eligible bank questions, authored questions, or both. Publication freezes question content. Students receive one attempt, answers save during the attempt, and results remain hidden until the shared deadline. Challenge results do not award official grades or XP. See the [course challenge specification](docs/features/course-challenges.md).

Role-specific navigation is backed by authenticated database authorization. Hiding a control in the interface is not the access boundary.

## Engineering design

ScholaU uses a TypeScript monorepo with explicit boundaries between the interface, data access, shared domain contracts, and persistent business rules.

```text
Student / Lecturer / Administrator
                 |
        Next.js web application
                 |
      Shared API and domain packages
                 |
 Supabase Auth + PostgreSQL RPCs and RLS
                 |
 Academic content, attempts, progress,
 challenges, battles, rewards, and audit data

Private learning assets -> Supabase Storage or Cloudflare R2
Browser push delivery  -> Supabase functions and delivery jobs
```

Key implementation decisions include:

- **Academic access boundaries:** enrollment and academic scope govern eligible learning content and competition.
- **Database-enforced rules:** PostgreSQL functions and row-level security enforce protected reads, writes, grading, rewards, and role access.
- **Versioned content:** published assessment content and challenge snapshots preserve the question version used by a learner.
- **Server-controlled timing:** battle rounds and challenge deadlines do not depend on keeping a browser timer or tab open.
- **Private asset access:** lesson assets use an authorized storage path rather than unrestricted public object access.
- **Shared contracts:** `@scholau/api` and `@scholau/core` keep data access and domain validation reusable across the interface.
- **Operational separation:** deploying the web app, applying SQL migrations, and provisioning credentials are separate release steps.

## Scope and availability

This README describes functionality represented in the source. Availability in a running environment depends on its deployed commit, database migrations, published content, user permissions, and provider configuration. Sample development fixtures are not a claim of complete university curriculum coverage. Historical verification notes in the documentation describe the environment and date of those checks.

## Architecture

| Layer | Technology | Responsibility |
| --- | --- | --- |
| Application | Next.js, React, TypeScript | Routes, interface, sessions, and server endpoints. |
| Client data | TanStack Query, `@scholau/api` | Queries, mutations, data contracts, and realtime updates. |
| Domain | `@scholau/core`, Zod | Shared types, validation, and domain utilities. |
| Design | Tailwind CSS, `@scholau/tokens` | Visual tokens, styles, and design foundations. |
| Backend | Supabase, PostgreSQL | Authentication, RPCs, access policies, and persistence. |
| Assets | Supabase Storage, Cloudflare R2 | Private lesson assets, selected through server configuration. |
| Workspace | pnpm workspaces, Turborepo | Dependency management and cross-package tasks. |

The web application imports shared packages through `@scholau/*`. SQL migrations define the persistent schema and business rules. See [architecture and package boundaries](docs/architecture.md).

## Repository layout

```text
SCHOLAU/
├── .github/                 # CI workflows and collaboration templates
├── apps/
│   └── web/
│       ├── app/             # Next.js routes, layouts, and HTTP endpoints
│       ├── components/      # Components grouped by domain
│       ├── lib/             # Application integrations and utilities
│       ├── public/          # Browser-served static assets
│       └── scripts/         # Web-specific operational scripts
├── assets/
│   └── brand/               # Brand source assets and manifest
├── docs/
│   ├── features/            # Feature behavior and business rules
│   └── operations/          # Storage and operational guides
├── packages/
│   ├── api/                 # Data client, contracts, and hooks
│   ├── core/                # Domain types, schemas, and utilities
│   └── tokens/              # Shared visual tokens and styles
├── supabase/
│   ├── curricula/           # Academic catalogues, sources, and reconciliation
│   ├── migrations/          # Ordered SQL migration history
│   ├── tests/database/      # Database integration and security tests
│   ├── config.toml          # Local service configuration
│   └── seed.sql             # Development fixtures
├── CONTRIBUTING.md
├── SECURITY.md
├── pnpm-workspace.yaml
└── turbo.json
```

Runtime paths and migrations retain their existing locations. Historical guide paths in the root and `supabase/` link to their canonical documents in `docs/`.

## Getting started

### Prerequisites

- Git and Node.js. The manifest accepts Node.js 20.9 and newer; CI uses Node.js 24.
- pnpm **12.11.2**, as declared in `package.json`.
- A development Supabase project, or Supabase CLI and Docker for a local instance.

Use the pnpm version declared by the repository. If your installation includes Corepack, run `corepack enable` to respect the `packageManager` field. Alternatively, install that pnpm version directly.

### Installation

```bash
git clone https://github.com/JohnNwaokolo/scholau.git
cd scholau
pnpm install --frozen-lockfile
```

Copy `apps/web/.env.example` to `apps/web/.env.local`, then configure your development environment.

PowerShell:

```powershell
Copy-Item apps/web/.env.example apps/web/.env.local
```

macOS or Linux:

```bash
cp apps/web/.env.example apps/web/.env.local
```

Start development from the repository root:

```bash
pnpm dev
```

The web application normally runs at `http://localhost:3000`. Public navigation can render without a session. Authenticated workflows require a configured Supabase instance and an account with the appropriate permissions.

### Environment variables

| Variable | Purpose |
| --- | --- |
| `NEXT_PUBLIC_SUPABASE_URL` | Development Supabase instance URL. |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Public anon or publishable key for the same project. |
| `NEXT_PUBLIC_APP_BUILD` | Build number used for compatibility checks. |
| `SUPABASE_SECRET_KEY` | Server-only credential for administrative operations. `SUPABASE_SERVICE_ROLE_KEY` is also supported as a fallback. |
| `LESSON_ASSET_STORAGE` | Lesson image provider: `supabase` or `r2`. |
| `R2_ACCOUNT_ID`, `R2_BUCKET` | R2 account and bucket when using that provider. |
| `R2_ACCESS_KEY_ID`, `R2_SECRET_ACCESS_KEY` | Private R2 credentials, available only to the server. |

[`.env.example`](apps/web/.env.example) is the configuration reference. Variables prefixed with `NEXT_PUBLIC_` may reach the browser and must never contain privileged credentials. Keep environment files and private values out of Git.

### Local database

From the repository root, with Supabase CLI and Docker available:

```bash
supabase start
supabase status
```

Configure the application with the local instance URL and public key. To rebuild a disposable local database and load `seed.sql`:

```bash
supabase db reset --local
supabase test db
```

`db reset --local` deletes data in that local instance. Some migrations depend on scheduled jobs and extensions; confirm compatibility with your Supabase environment. Read the [backend guide](supabase/README.md) before preparing database changes.

## Development commands

Run workspace tasks from the repository root.

| Command | Purpose |
| --- | --- |
| `pnpm dev` | Start workspace development servers. |
| `pnpm lint` | Run web ESLint and the checks declared by shared packages. |
| `pnpm typecheck` | Generate route types and check TypeScript across workspaces. |
| `pnpm build` | Compile the application and run task dependencies. |
| `pnpm --filter @scholau/web start` | Serve an existing production web build. |
| `supabase test db` | Run SQL tests against a prepared local database. |

GitHub Actions installs dependencies with the frozen lockfile, then runs lint, type checking, and a production build. CI does not deploy the application or modify a database. SQL tests run separately and require a prepared instance.

## Technical documentation

| Guide | Contents |
| --- | --- |
| [Documentation index](docs/README.md) | Entry point for technical guides. |
| [Architecture](docs/architecture.md) | Dependencies, boundaries, and where new code belongs. |
| [Contributing](CONTRIBUTING.md) | Development, review, and validation workflow. |
| [Security](SECURITY.md) | Credential handling and vulnerability reporting. |
| [Backend](supabase/README.md) | Migrations, tests, and academic data. |
| [R2 storage](docs/operations/r2-storage.md) | Private access, asset migration, and rollback. |
| [Account erasure](docs/features/account-erasure.md) | Deletion scope and lifecycle. |
| [Lecturer registration](docs/features/lecturer-registration.md) | Applications and lecturer authorization. |
| [Social battles](docs/features/social-battles.md) | Competition rules and access. |
| [XP](docs/features/xp.md) | Progress and reward rules. |
| [Academic catalogues](supabase/curricula/README.md) | Sources, requirements, and programme mapping. |

## Changes and releases

Work on a branch and open a pull request describing the problem, resulting behavior, and validation. Add schema changes as new migrations; do not rewrite applied migrations to change production behavior.

Web deployment, database migrations, and secret provisioning are separate operations. Before releasing, review server variables, authentication redirect URLs, and application compatibility with the target schema. A successful build alone does not verify authenticated production workflows.
