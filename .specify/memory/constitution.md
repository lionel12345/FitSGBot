<!--
Sync Impact Report:
- Version change: 1.1.1 -> 1.2.0
- Added Principles: None
- Modified Principles:
  - I. Exclusive Webhook Architecture (Zero Polling) (Incorporated Telegram Bot API webhook specifics, secret_token validation, and grammY webhook adapters)
  - II. Strict TypeScript & Node.js Standards (Expanded to mandate grammY typed Context, custom Context flavors, and safe Telegram entity handling)
- Modified Sections:
  - Preamble (Explicitly defined FitSGBot as a Telegram Bot powered by grammY)
  - Technology Stack & Architectural Constraints (Detailed grammY Telegram Bot framework mechanics, middleware pipeline, and Telegram Bot API constraints)
- Removed Sections: None
- Follow-up TODOs: None
-->

# FitSGBot Constitution

FitSGBot is a dedicated Telegram Bot application serving fitness tracking, community workouts, and user interactions via the Telegram Bot API. It is architected on Node.js and TypeScript utilizing the [grammY](https://grammy.dev) bot framework as its foundational core.

## Core Principles

### I. Exclusive Webhook Architecture (Zero Polling)
All Telegram updates MUST be delivered to the application exclusively via Telegram Bot API HTTP webhooks configured with HTTPS and a verification secret token (`secret_token`). Polling mechanisms (including long polling and `bot.start()` / `getUpdates` loops) MUST NEVER be used or committed anywhere in the codebase across any environment (development, test, staging, or production).
- Local development workflows MUST use webhook tunneling (e.g., ngrok, Cloudflare Tunnel) or local mock update dispatchers.
- Webhook endpoints MUST immediately acknowledge incoming Telegram update payloads with an HTTP 200 OK response within Telegram's timeout window (< 5 seconds), offloading any heavy computation, database writes, or outbound network calls to asynchronous background tasks to prevent Telegram retry storms.

### II. Strict TypeScript, Node.js & grammY Context Standards
The application MUST be built using Node.js (Active LTS) and TypeScript with strict compiler settings enabled (`strict: true`, no implicit `any`, strict null checks).
- Telegram context objects MUST use typed grammY `Context` flavors (e.g., `CustomContext` extending `Context` with session, user, and database dependencies).
- All incoming updates, callback query data, and external inputs MUST be strictly validated before domain logic execution.
- Direct reliance on untyped message objects is prohibited; developers must use grammY filters (e.g., `bot.on("message:text")`, `bot.command("start")`) to narrow types safely.

### III. Relational Persistence & Data Integrity
Telegram user accounts, session state, workout/fitness logs, subscriptions, and interaction history MUST be persisted in PostgreSQL using a type-safe Object-Relational Mapping (ORM) layer (such as Prisma or Drizzle ORM). Direct unparameterized queries are forbidden. All database schema modifications MUST be managed through version-controlled, repeatable migration scripts.

### IV. Automated Testing & Webhook Contract Verification
Code quality MUST be verified using Vitest for automated unit and integration testing. Every webhook handler, bot command route, conversation flow, and business service MUST have corresponding automated test cases covering nominal flows, invalid payloads, error conditions, and edge cases with mocked Telegram update payloads. Pull requests MUST NOT be merged if any automated test fails.

### V. Security, Secret Management & Endpoint Validation
All sensitive credentials—including the Telegram Bot Token obtained from [@BotFather](https://t.me/BotFather), webhook secret tokens, and database connection strings—MUST be loaded exclusively via environment variables and MUST NEVER be committed to the repository. The webhook endpoint MUST verify incoming requests against the `X-Telegram-Bot-Api-Secret-Token` header to guarantee that payloads originate genuinely from Telegram.

### VI. Feature-Based Development & Strict Traceability (NON-NEGOTIABLE)
All development and codebase changes MUST strictly follow a feature-based development model. No code modifications, refactoring, or infrastructure updates can be performed without explicitly identifying and indicating the specific feature being modified. If a change introduces new bot commands, handlers, or capabilities, it MUST first be formally documented and defined in the feature specifications (`specs/`) before implementation. Changes lacking clear feature attribution or specification linkage MUST be rejected.

## Technology Stack & Architectural Constraints

- **Platform**: Telegram Bot API (v7.0+ compliant).
- **Core Bot Framework**: [grammY](https://grammy.dev) (handling bot instance, middleware routing, and Telegram API interaction).
- **Runtime**: Node.js (v20+ LTS).
- **Language**: TypeScript (strict mode, target ES2022+).
- **Server Framework**: Fastify or Express configured with grammY's `webhookCallback(bot, "fastify")` or `webhookCallback(bot, "express")`.
- **Database & ORM**: PostgreSQL managed via Prisma or Drizzle ORM.
- **Testing Framework**: Vitest with mock fixtures for simulated Telegram Update objects.
- **Linter & Formatter**: ESLint with TypeScript-ESLint and Prettier.

## Development Workflow & Quality Gates

1. **Feature-Driven Lifecycle**: Every code change must map directly to a defined feature. For new bot features or commands, specifications must be created via Spec Kit (`/speckit-specify`) before implementation. For existing features, modifications must explicitly reference the target feature in the branch name, commit message, and PR description.
2. **Branching & PRs**: Feature branches created from and merged into `main` via pull requests (e.g., `feat/<feature-id>-<name>` or `fix/<feature-id>-<name>`). Direct commits to production branches are prohibited.
3. **Quality Gates**: Every pull request must pass the automated CI pipeline:
   - Type verification (`tsc --noEmit`).
   - Linting check (`eslint .`).
   - Automated test suite execution (`vitest run`).
   - Feature traceability check (verifying the change maps to an approved feature specification).
4. **Telegram Contract Testing**: Changes to bot commands, middleware, or webhook adapters must include corresponding Telegram `Update` mock fixtures and integration test suites.

## Governance

This constitution defines the non-negotiable architectural and engineering standards for FitSGBot. All contributors, AI assistants, and automated tooling MUST abide by these principles.

- **Precedence**: This constitution supersedes informal conventions or ad-hoc architectural preferences.
- **Amendment Policy**: Amendments require formal review and a semantic version bump:
  - **MAJOR**: Breaking changes to core principles or foundational architectural redesigns.
  - **MINOR**: Addition of new principles, tech stack expansions, or structural requirements.
  - **PATCH**: Clarifications, wording improvements, or non-functional corrections.
- **Review Cycle**: Architecture reviews and compliance checks must accompany feature development.

**Version**: 1.2.0 | **Ratified**: 2026-09-24 | **Last Amended**: 2026-09-24
