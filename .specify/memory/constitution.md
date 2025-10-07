<!--
Sync Impact Report
==================
Version Change: N/A (initial) → 1.0.0
Modified Principles: N/A (initial constitution)
Added Sections: 
  - Core Principles (5 principles)
  - Technical Constraints
  - Modernization Workflow
  - Governance
Removed Sections: N/A
Templates Status:
  ✅ plan-template.md - Updated with constitution references
  ✅ spec-template.md - Aligned with modernization requirements
  ✅ tasks-template.md - Updated with TDD and modernization task types
  ⚠ agent-file-template.md - Pending review for JavaScript/Node.js context
Follow-up TODOs: None
-->

# BrowserQuest Modernization Constitution

## Core Principles

### I. Fix Broken Code Only
Legacy code from 2012 MUST be modified ONLY when it fails to execute on modern Node.js 18+ LTS. If code runs without errors, it MUST be preserved as-is, regardless of style or patterns used. Every piece of code MUST be evaluated for execution compatibility: if it runs on Node.js 18+ LTS, it MUST remain unchanged. Only non-functional, deprecated, or error-producing code requires modification.

**Rationale**: Code last updated in 2012 (Node.js v0.4.7 era) may use older patterns but often still works correctly on modern runtimes. The goal is a runnable system with minimal changes, not a complete modernization. This "if it ain't broke, don't fix it" approach minimizes risk, preserves original behavior, and reduces implementation time.

### II. Modern Dependencies Only
All npm dependencies MUST use currently maintained packages with explicit semantic versions. Wildcards (`">0"`) are PROHIBITED. Dependencies without updates in the last 24 months MUST be replaced with maintained alternatives or implemented directly. Every dependency MUST be justified against the modern npm ecosystem.

**Rationale**: The original `package.json` uses wildcard versions and packages like `bison`, `websocket-server`, and `log` that are either deprecated or have superior modern alternatives. Modern dependency management requires explicit versioning for reproducible builds and security.

**Implementation**: Use `ws` (WebSocket library with 90M+ weekly downloads) instead of unmaintained `websocket-server`; use `pino` or `winston` instead of `log`; use native `JSON` instead of `bison`; use `node-cache` or `redis` instead of `memcache`.

### III. Preserve Codebase Patterns
The codebase MUST remain JavaScript throughout (no TypeScript conversion). Existing patterns (ES5 syntax, CommonJS, callbacks, var declarations, for loops, prototype inheritance) MUST be preserved if they execute without errors. Modernization to ES2020+ (async/await, ESM, optional chaining, const/let, classes) is OPTIONAL and should ONLY be done when it provides clear execution or maintainability benefits, not for style consistency alone.

**Rationale**: JavaScript has evolved significantly since 2012 (ES5 era), but older patterns still work correctly on modern runtimes. Forcing ES2020+ conversions creates unnecessary churn and risk. The codebase should remain stable unless specific language features cause execution failures or significantly impede necessary changes.

### IV. Node.js LTS Requirement
All code MUST run on Node.js 18.x LTS or later (current LTS as of 2025). Code MUST NOT use deprecated Node.js APIs. Built-in test runner (`node:test`) and native fetch API MUST be preferred over external alternatives when suitable.

**Rationale**: Node.js v0.4.7 (2012) to Node.js 18+ (2024+) represents a massive evolution. Many APIs were removed, security was hardened, and performance improved by orders of magnitude. Modern Node.js provides built-in capabilities that previously required libraries.

**Implementation**: Replace deprecated `Buffer()` constructor with `Buffer.from()`; replace `url.parse()` with WHATWG URL API; use native `crypto` module consistently; use native test runner for unit tests.

### V. Functional First, Features Second
The primary goal is a RUNNABLE, STABLE game server and client. New features are PROHIBITED until the existing game loop (player movement, combat, NPCs, items, chat) functions correctly on modern infrastructure. All modernization work MUST be validated by running the actual game.

**Rationale**: Users want to run BrowserQuest, not modernize an unusable codebase indefinitely. Every change must move toward a functional game. Adding features before achieving stability creates scope creep.

## Technical Constraints

### Stack Requirements
- **Language**: JavaScript (preserve existing patterns: ES5/CommonJS where working; ES2020+/ESM only where needed)
- **Runtime**: Node.js 18.x LTS minimum (prefer Node.js 20.x LTS)
- **Package Manager**: npm (with `package-lock.json` for reproducible builds)
- **WebSocket**: `ws` library (latest stable, currently 8.x) - only if replacing broken `websocket-server`
- **Logging**: `pino` or equivalent (only if replacing broken `log` package)
- **Testing**: Manual validation preferred; Node.js native test runner (`node:test`) optional for unit tests
- **Client Build**: Preserve existing build system if working; modern alternatives only if broken
- **Code Style**: Preserve existing style; formatters (Prettier, ESLint) optional

### Architecture Preservation
The original client/server separation MUST be maintained:
- **Client**: HTML5 canvas game rendering, served as static assets
- **Server**: Node.js WebSocket server handling game state, player sync, combat
- **Shared**: Common game types and constants accessible to both client and server

No database is required for the base game (in-memory state is acceptable). Memcache/Redis integration remains optional for metrics only.

### Performance Targets
- **Server**: Handle 200+ concurrent players per world instance (original design goal)
- **Client**: 60 FPS rendering on modern browsers (Chrome, Firefox, Safari, Edge)
- **Latency**: <100ms p95 for player action → server response → broadcast
- **Startup**: Server ready to accept connections within 2 seconds

### Compatibility
- **Browsers**: Modern evergreen browsers only (no IE11, no legacy polyfills)
- **Mobile**: Responsive design for tablets acceptable, phone optimization not required
- **Operating Systems**: Linux, macOS, Windows (via Node.js cross-platform support)

## Restoration Workflow

### Phase 1: Dependencies Audit & Replacement
1. Document every dependency in `package.json` and test installation on Node.js 18+
2. For each dependency that FAILS to install or causes runtime errors: (a) find modern maintained replacement, (b) use native Node.js alternative, or (c) implement directly
3. For dependencies that install and work: KEEP as-is with explicit semantic version
4. Update `package.json` with explicit semantic versions (no wildcards)
5. Test server startup and basic game loop after each dependency swap

### Phase 2: Fix Broken APIs Only
1. Test codebase for execution errors on Node.js 18+
2. Replace deprecated Node.js APIs ONLY if they cause errors (e.g., Buffer constructor throwing exceptions)
3. Ignore deprecation warnings that don't prevent execution
4. Test existing build system; only replace if broken
5. Validate client-server communication after each fix

### Phase 3: Preserve Working Code (SKIP if Phase 2 Successful)
This phase is OPTIONAL and should only be executed if there's a compelling reason beyond style:
1. Consider ES6 classes only if prototype patterns are causing maintenance issues
2. Consider `const`/`let` only if `var` scoping is causing bugs
3. Consider modern async patterns only if callback hell is blocking new features
4. Keep existing patterns unless they demonstrably impede progress

### Phase 4: Validation & Documentation
1. Run full game loop: player login → movement → combat → item collection → logout
2. Test multi-player synchronization with 10+ concurrent connections
3. Document new setup process in README (Node.js version, npm install, config files)
4. Create quickstart guide for running server and accessing client
5. Record known issues and migration TODOs

### Change Verification Process
After any code change:
1. Run linter (ESLint with modern JavaScript rules)
2. Run tests (at minimum: server startup test, WebSocket connection test)
3. Start server and connect with client to verify game loop
4. Check console for deprecation warnings or errors
5. Document any behavioral changes versus 2012 version

## Governance

### Amendment Process
This constitution governs all modernization work on BrowserQuest. Changes to principles require:
1. Documentation of why principle is blocking progress
2. Alternative approach that maintains "runnable modern game" goal
3. Update to this document with version bump and rationale
4. Validation that templates and workflow docs remain consistent

### Version Semantics
- **MAJOR** (X.0.0): Principle removed or fundamentally redefined (e.g., switching to TypeScript)
- **MINOR** (1.X.0): New principle added or existing principle materially expanded
- **PATCH** (1.0.X): Clarifications, wording improvements, non-semantic fixes

### Compliance Verification
All implementation work MUST verify:
- ✅ Runs on Node.js 18+ LTS without execution errors
- ✅ Dependencies explicitly versioned with maintained packages
- ✅ Existing code patterns preserved unless they cause errors
- ✅ Game functionality preserved (no feature regressions)
- ✅ Changes are validated by running the actual game
- ✅ Code modifications justified by specific error messages (not style preferences)

When constitutional principles conflict with pragmatic needs (e.g., a useful unmaintained library), document the exception explicitly with:
- Why the principle cannot be followed
- Risk assessment (security, maintenance, compatibility)
- Remediation plan (fork the library, implement alternative, accept risk with monitoring)

**Version**: 1.0.0 | **Ratified**: 2025-10-07 | **Last Amended**: 2025-10-07
