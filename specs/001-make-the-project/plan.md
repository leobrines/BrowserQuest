# Implementation Plan: Restore BrowserQuest Executability

**Branch**: `001-make-the-project` | **Date**: 2025-10-07 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/home/leit0/Desktop/github/BrowserQuest/specs/001-make-the-project/spec.md`

## Execution Flow (/plan command scope)
```
1. Load feature spec from Input path
   → ✅ Spec loaded successfully
2. Fill Technical Context (scan for NEEDS CLARIFICATION)
   → ✅ No NEEDS CLARIFICATION markers
   → ✅ Project Type: Web (client + server)
3. Fill Constitution Check section
   → ⚠️ IMPORTANT: Spec overrides constitution modernization principles
   → ✅ Spec explicitly requires "fix broken code only" approach
4. Evaluate Constitution Check section
   → ✅ Approach justified: restore functionality with minimal changes
   → ✅ Progress Tracking: Initial Constitution Check PASS
5. Execute Phase 0 → research.md
   → ✅ Research dependencies and deprecated APIs
6. Execute Phase 1 → quickstart.md (no contracts/data-model for infrastructure restoration)
   → ✅ Create validation quickstart
7. Re-evaluate Constitution Check
   → ✅ Post-Design Constitution Check: PASS
8. Plan Phase 2 → Task generation approach described
9. ✅ STOP - Ready for /tasks command
```

**IMPORTANT**: The /plan command STOPS at step 9. Phases 2-4 are executed by other commands:
- Phase 2: /tasks command creates tasks.md
- Phase 3-4: Implementation execution (manual or via tools)

## Summary

**Primary Requirement**: Make BrowserQuest (last updated 2012, Node.js v0.4.7) executable on modern Node.js 20.x LTS with minimal code modifications.

**Technical Approach**: 
1. Replace unmaintained/broken npm packages with modern equivalents (ws, pino, sanitizer)
2. Fix deprecated Node.js APIs that cause errors (not warnings)
3. Preserve all 2012-era vanilla JavaScript code that still runs
4. Validate via manual gameplay testing (no automated test suite)

**Philosophy**: "If it ain't broke, don't fix it" - fix broken code only for execution-blocking issues.

## Technical Context

**Language/Version**: JavaScript ES5 (preserved where working) / Node.js 20.x LTS  
**Primary Dependencies**: 
- `ws@^8.18.0` (replaces websocket/websocket-server)
- `pino@^9.5.0` (replaces log)  
- `sanitizer@^0.1.3` (keep if compatible, else remove)
- `underscore@^1.13.7` (keep - widely used in codebase)
- Native JSON (replaces bison)
- memcache optional (skip if not needed)

**Storage**: In-memory game state (no database)  
**Testing**: Manual gameplay validation only (no unit/integration tests per clarification)  
**Target Platform**: Node.js 20.x LTS server, Modern browsers (Chrome/Firefox/Safari/Edge), Linux/macOS/Windows  
**Project Type**: Web (client + server architecture)  
**Performance Goals**: 200+ concurrent players per world, 60 FPS client rendering, <100ms p95 latency  
**Constraints**: No asset modifications (images, CSS, audio), no new features, no gameplay bug fixes, preserve vanilla JS  
**Scale/Scope**: 2012 codebase (~50 JS files), restore to runnable state

## Constitution Check
*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### ⚠️ CRITICAL NOTE: Spec Overrides Constitution

The specification explicitly requires **"fix broken code only"** approach, which differs from constitution's original "Destructive Refactoring" principle. This is justified because:

1. **User explicitly requested minimal changes**: "Keep extremely minimal code modifications if code is runnable"
2. **Spec clarifications confirm approach**: Ignore deprecation warnings, manual testing only, preserve vanilla JS
3. **Risk mitigation**: Full modernization would violate "no core game changes" requirement

**Constitution principles adapted for this feature**:

### Principle I: Destructive Refactoring → FIX BROKEN CODE ONLY
- ❌ Constitution says: "Legacy code MUST be completely rewritten"
- ✅ Spec says: "Vanilla JavaScript MUST be preserved if it runs without errors"
- **Resolution**: Follow spec - replace only broken/incompatible code

### Principle II: Modern Dependencies Only → COMPATIBLE DEPENDENCIES
- ✅ Constitution: "Use maintained packages with explicit versions"
- ✅ Spec: "Dependencies updated ONLY when they fail to install"
- **Resolution**: Aligned - replace unmaintained packages (websocket-server, log, bison)

### Principle III: Language Consistency → PRESERVE VANILLA JS
- ❌ Constitution says: "MUST use ES2020+, ESM, async/await"
- ✅ Spec says: "NO refactoring, NO modernization, NO ES6+ conversions"
- **Resolution**: Follow spec - keep ES5, callbacks, var, for loops where they work

### Principle IV: Node.js LTS Requirement → NODE 20.x TARGET
- ✅ Constitution: "Run on Node.js 18.x LTS or later"
- ✅ Spec: "System MUST run on Node.js 18.x LTS minimum (20.x preferred)"
- ✅ User input: "run on node v20"
- **Resolution**: Aligned - target Node.js 20.x LTS

### Principle V: Functional First, Features Second → RUNNABLE GAME ONLY
- ✅ Constitution: "RUNNABLE, STABLE game server and client"
- ✅ Spec: "No new features, no bug fixing, functional restoration only"
- **Resolution**: Aligned - validate game loop works

**Justification for Deviation**: This is infrastructure restoration, not feature development. The goal is minimum viable changes to achieve execution on modern Node.js, not modernization.

## Project Structure

### Documentation (this feature)
```
specs/001-make-the-project/
├── plan.md              # This file (/plan command output)
├── research.md          # Phase 0 output (/plan command)
├── quickstart.md        # Phase 1 output (/plan command)
└── tasks.md             # Phase 2 output (/tasks command - NOT created by /plan)
```

### Source Code (repository root)
```
# Existing structure (preserved):
/home/leit0/Desktop/github/BrowserQuest/
├── client/
│   ├── js/              # Client-side game code (preserve vanilla JS)
│   ├── css/             # Styling (no modifications)
│   ├── img/             # Images (no modifications)
│   ├── audio/           # Audio files (no modifications)
│   ├── sprites/         # Sprite definitions (no modifications)
│   ├── maps/            # Client map data (no modifications)
│   └── index.html       # Entry point (minimal changes only if needed)
│
├── server/
│   ├── js/              # Server-side game logic (fix deprecated APIs only)
│   │   ├── main.js      # Server entry point
│   │   ├── worldserver.js  # Core server logic
│   │   └── lib/         # Utility libraries
│   ├── maps/            # Server map data (no modifications)
│   └── config*.json     # Configuration files (may need updates)
│
├── shared/
│   └── js/
│       └── gametypes.js # Shared constants (preserve)
│
├── bin/
│   └── build.sh         # Build script (skip if broken)
│
└── package.json         # WILL BE UPDATED with explicit versions
```

**Structure Decision**: Preserve existing client/server/shared structure. No reorganization. Changes limited to:
- `package.json` - update dependencies
- `server/js/*.js` - fix deprecated Node.js APIs if needed
- `client/js/*.js` - only if execution-blocking issues found
- Documentation: Update README with Node.js 20 setup instructions

## Phase 0: Outline & Research

**Objective**: Identify which dependencies/APIs are actually broken on Node.js 20.x

### Research Tasks

1. **Dependency Audit**:
   - `underscore: ">0"` → Check if latest (1.13.x) works with Node.js 20
   - `log: ">0"` → DEPRECATED, replace with `pino` or `winston`
   - `bison: ">0"` → UNMAINTAINED (last update 2013), replace with native JSON
   - `websocket: ">0"` → CONFUSING name, likely meant `ws`
   - `websocket-server: ">0"` → UNMAINTAINED, replace with `ws`
   - `sanitizer: ">0"` → Check compatibility, may need `validator` or `dompurify`
   - `memcache: ">0"` → Optional (metrics only), skip unless required

2. **Node.js API Compatibility**:
   - Search codebase for `new Buffer(` → Replace with `Buffer.from()` or `Buffer.alloc()`
   - Search for `url.parse(` → May need WHATWG URL (but only if causing errors)
   - Search for `crypto.createCipher` → Replace with `crypto.createCipheriv` if found
   - Check for other deprecated APIs via Node.js 20 deprecation list

3. **WebSocket Implementation Research**:
   - Original uses `websocket` or `websocket-server` package
   - Modern equivalent: `ws@8.x` (90M+ weekly downloads, actively maintained)
   - API differences: document minimal changes needed to adapt

4. **Build System Analysis**:
   - Check if `bin/build.sh` uses RequireJS optimizer (r.js)
   - Determine if build step is required or can be skipped (dev mode)
   - Per spec: "If build system broken, MAY be skipped"

**Output**: `/home/leit0/Desktop/github/BrowserQuest/specs/001-make-the-project/research.md`

## Phase 1: Design & Contracts

*Note: For infrastructure restoration, traditional contracts/data-model don't apply. Focus on quickstart validation.*

### Quickstart Guide (Manual Validation)

Create `/home/leit0/Desktop/github/BrowserQuest/specs/001-make-the-project/quickstart.md` with step-by-step validation:

1. **Setup**: Install Node.js 20.x, clone repo, run `npm install`
2. **Server Start**: Copy config, start server, verify no crashes
3. **Client Access**: Serve client via HTTP, open browser, verify connection
4. **Gameplay Validation**: 
   - Create character
   - Move around map
   - Attack mob, verify combat
   - Collect item, verify inventory
   - Click NPC, verify dialogue
   - Send chat message
   - Open second browser window, verify multi-player sync

This serves as the acceptance test (manual execution required per clarification).

### Agent-Specific File (Cursor)

Since using Cursor, update `.cursor/rules` or root-level instructions with:
- Target: Node.js 20.x LTS
- Preserve: All vanilla JS patterns, ES5 syntax
- Fix only: Execution-blocking errors
- No tests: Manual validation only

**Output**: `/home/leit0/Desktop/github/BrowserQuest/specs/001-make-the-project/quickstart.md`

## Phase 2: Task Planning Approach

*This section describes what the /tasks command will do - DO NOT execute during /plan*

**Task Generation Strategy**:

1. **Setup Tasks** (T001-T004):
   - T001: Audit current dependencies on Node.js 20 (run `npm install` and document errors)
   - T002: Update `package.json` with explicit modern versions
   - T003: Test installation after package.json updates
   - T004: Document setup instructions for README

2. **Dependency Replacement Tasks** (T005-T010):
   - T005: Replace `log` with `pino` in server code
   - T006: Replace `websocket-server` with `ws` in server code
   - T007: Replace `bison` usage with native JSON
   - T008: Handle `sanitizer` - test or replace
   - T009: Update `underscore` to explicit version (if working, no code changes)
   - T010: Remove or skip `memcache` dependency

3. **API Modernization Tasks** (T011-T015):
   - T011: Search and replace deprecated `Buffer()` constructor
   - T012: Fix any deprecated crypto APIs (if found)
   - T013: Fix any deprecated url APIs (if found)
   - T014: Fix any other Node.js 20 incompatibilities (discovered in Phase 0)
   - T015: Test server startup

4. **Client Validation Tasks** (T016-T018):
   - T016: Set up HTTP server for client (document command)
   - T017: Test WebSocket connection from client to server
   - T018: Verify client assets load (sprites, maps, audio)

5. **Game Loop Validation Tasks** (T019-T026):
   - T019: Manual test: Character creation
   - T020: Manual test: Movement
   - T021: Manual test: Combat
   - T022: Manual test: Item collection
   - T023: Manual test: NPC interaction
   - T024: Manual test: Chat
   - T025: Manual test: Multi-player synchronization
   - T026: Document any remaining issues

6. **Documentation Tasks** (T027-T028):
   - T027: Update README with Node.js 20 setup instructions
   - T028: Document client serving requirement (HTTP server)

**Ordering Strategy**:
- Sequential: Setup → Dependencies → APIs → Validation
- No parallelization needed (small codebase, fix broken code approach)
- No TDD: All validation is manual (per spec clarification)

**Estimated Output**: 30 numbered, ordered tasks in tasks.md (T001-T030)

**IMPORTANT**: This phase is executed by the /tasks command, NOT by /plan

## Phase 3+: Future Implementation

*These phases are beyond the scope of the /plan command*

**Phase 3**: Task execution (/tasks command creates tasks.md)  
**Phase 4**: Implementation (execute tasks.md following fix broken code principle)  
**Phase 5**: Validation (manual gameplay testing per quickstart.md)

## Complexity Tracking

*No violations - approach is minimal changes, no new architecture*

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| None | Fix broken code only | N/A |

## Progress Tracking

**Phase Status**:
- [x] Phase 0: Research complete (/plan command) - research.md generated
- [x] Phase 1: Design complete (/plan command) - quickstart.md generated
- [x] Phase 2: Task planning approach described (/plan command) - ready for /tasks
- [ ] Phase 3: Tasks generated (/tasks command) - NOT YET EXECUTED
- [ ] Phase 4: Implementation complete
- [ ] Phase 5: Validation passed

**Gate Status**:
- [x] Initial Constitution Check: PASS (with spec override justification)
- [x] Post-Design Constitution Check: PASS
- [x] All NEEDS CLARIFICATION resolved (via /clarify session)
- [x] Complexity deviations documented (none)

---

*Plan generated: 2025-10-07*
*Ready for: `/tasks` command to generate implementation tasks*
*Spec version: Per spec.md with 3 clarifications from 2025-10-07 session*
