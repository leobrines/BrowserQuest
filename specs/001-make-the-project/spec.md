# Feature Specification: Restore BrowserQuest Executability

**Feature Branch**: `001-make-the-project`  
**Created**: 2025-10-07  
**Status**: Draft  
**Input**: User description: "make the project runnable and stable. server and client code must be executable. No new features. No bug fixing. No modernization. Only make BrowserQuest runnable again in current LTS versions of language. If needed, use libraries. No images, videos, css or assets updates. Keep extremely minimal code modifications if code is runnable, ok or not outdated (ex vanilla js, raw js, etc). If its using deprecated libraries, functions or classes, you can recreate it at all. No core game changes, at least is really necessary."

## Execution Flow (main)
```
1. Parse user description from Input
   → ✅ Feature description provided
2. Extract key concepts from description
   → ✅ Actors: Developers deploying BrowserQuest
   → ✅ Actions: Install dependencies, start server, load client
   → ✅ Data: Game state (players, mobs, items)
   → ✅ Constraints: EXTREMELY minimal changes, fix broken code only for deprecated/broken code
   → ✅ Preservation: Vanilla JS stays as-is, NO asset modifications, NO core game logic changes
3. For each unclear aspect:
   → No ambiguities - requirements are explicit
4. Fill User Scenarios & Testing section
   → ✅ Clear user flow: install → configure → run → play
5. Generate Functional Requirements
   → ✅ All requirements testable
6. Identify Key Entities
   → Not applicable (infrastructure restoration, not data modeling)
7. Run Review Checklist
   → ✅ No [NEEDS CLARIFICATION] markers
   → ✅ Uses Node.js LTS requirement (allowed per constitution)
8. Return: SUCCESS (spec ready for planning)
```

---

## ⚡ Quick Guidelines
- ✅ Focus on WHAT users need: A working game server and client
- ✅ WHY: BrowserQuest (last updated 2012) doesn't run on modern Node.js
- ⚠️ Implementation details justified: Node.js LTS requirement is the core constraint

---

## Clarifications

### Session 2025-10-07
- Q: How should the client be accessed during development/testing? → A: Serve client via HTTP server (e.g., `python -m http.server` or similar) on a port, then access via http://localhost
- Q: How should deprecation warnings (non-blocking warnings in console) be handled? → A: Ignore deprecation warnings - only fix actual errors that prevent execution
- Q: What level of testing is required to validate the restoration? → A: Manual testing only - developer plays through game scenarios and validates visually

### Runtime Configuration Requirements (Discovered During Validation)
- **Client Configuration Files**: The repository only includes `-dist` template files. Users must create `client/config/config_build.json` and `client/config/config_local.json` from templates before first run
- **Shared Resources Access**: Client code references `/shared/js/gametypes.js`, but HTTP server only serves from `client/` directory. A symlink (`client/shared → ../shared`) is required
- **Browser Caching**: After configuration changes, browsers may cache broken state. Hard refresh (Ctrl+Shift+R) is required to load new configuration

---

## User Scenarios & Testing *(mandatory)*

### Primary User Story
A developer wants to run BrowserQuest locally to experience the game or deploy it for others. They clone the repository, follow setup instructions, start the server, open the client in a browser, and successfully play the game (move around, fight enemies, collect items, chat).

### Acceptance Scenarios

1. **Given** a fresh clone of BrowserQuest repository, **When** developer runs `npm install` using Node.js 18.x or 20.x LTS, **Then** all dependencies install successfully without errors (deprecation warnings are acceptable)

2. **Given** dependencies are installed, **When** developer copies `server/config_local.json-dist` to `server/config_local.json` and runs `node server/js/main.js`, **Then** server starts and logs "Server is listening on port 8000" (or configured port) without crashes

3. **Given** server is running, **When** developer creates client configuration files (`cd client/config && cp config_build.json-dist config_build.json && cp config_local.json-dist config_local.json`), creates shared resources symlink (`cd client && ln -s ../shared shared`), serves client directory via HTTP server (e.g., `python -m http.server 8080` from client directory), and opens `http://localhost:8080` in Chrome/Firefox/Safari with hard refresh (Ctrl+Shift+R), **Then** game client loads, establishes WebSocket connection to server, and displays character selection screen

4. **Given** client is connected, **When** player creates character and enters game world, **Then** player can move using arrow keys or WASD, character sprite renders correctly, and map displays properly

5. **Given** player is in game world, **When** player encounters and attacks a mob (rat, skeleton, etc.), **Then** combat mechanics work (damage dealt, health reduced, mob dies, loot drops)

6. **Given** multiple clients connected, **When** players move in same area, **Then** each player sees other players' movements in real-time via WebSocket synchronization

7. **Given** player collects item (weapon, armor, potion), **When** item is equipped or used, **Then** player stats update correctly (attack, defense, health)

8. **Given** player interacts with NPC, **When** clicking NPC sprite, **Then** dialogue bubble appears with quest text or vendor interface

### Edge Cases
- What happens when server crashes during gameplay? (Out of scope: no bug fixing - server must start, crashes are existing bugs)
- How does system handle WebSocket connection loss? (Out of scope: existing reconnection logic preserved as-is)
- What if multiple world instances are configured? (Must work: configuration file specifies world count, each must initialize)
- What if outdated browser is used? (Out of scope: target modern evergreen browsers only, per constitution)
- What if developer skips client configuration setup? (System fails gracefully: browser shows 404 error with URL `/none?player-name=<username>`, documented in troubleshooting)
- What if browser caches old configuration? (Hard refresh required: Ctrl+Shift+R to clear cache and load new config)

---

## Requirements *(mandatory)*

### ⚠️ Critical Constraint: Fix Broken Code Only

**ABSOLUTE RULES**: 
1. **Zero Asset Modifications** - No visual, audio, or styling changes
2. **Fix Broken Code Only** - Only fix what's actually broken or deprecated
3. **Preserve Vanilla JS** - Working vanilla/raw JavaScript stays unchanged

**Forbidden Changes**:
- ❌ Images: PNG, JPG, GIF files in `client/img/`
- ❌ CSS: All `.css` files in `client/css/`
- ❌ Audio: MP3, OGG files in `client/audio/`
- ❌ Sprites: JSON sprite definitions in `client/sprites/`
- ❌ Maps: JSON map data in `client/maps/` and `server/maps/`
- ❌ Fonts: Web font files in `client/fonts/`
- ❌ Videos: Any video files (if present)
- ❌ HTML structure: `client/index.html` (except WebSocket config if absolutely necessary)
- ❌ Working vanilla JS code (no refactoring, no style improvements, no ES6+ conversions)
- ❌ Core game logic (movement algorithms, combat formulas, pathfinding, unless execution-blocking)

**Only Fix When**:
- ⚠️ Deprecated npm library that won't install on Node.js 18+
- ⚠️ Deprecated Node.js API that throws **errors** (not warnings) preventing execution
- ⚠️ Syntax/API that causes crashes or prevents server/client from running
- ⚠️ Build tools that are incompatible with modern Node.js
- ✅ Deprecation warnings (console warnings) are acceptable - ignore them unless they block execution

**Allowed Changes** (only when necessary):
- ✅ Replace deprecated npm packages in `package.json`
- ✅ Fix deprecated Node.js API calls (e.g., `Buffer()` → `Buffer.from()`)
- ✅ Update configuration: `package.json`, `config*.json` files
- ✅ Fix critical syntax errors preventing execution
- ✅ Documentation: README.md updates for setup instructions

**Philosophy**: "If it ain't broke, don't fix it" - Preserve the 2012 vanilla JS wherever possible

**Decision Tree - Fix or Keep?**:
```
Does code execute without errors? 
├─ YES → KEEP AS-IS (even if "ugly", uses var, old patterns)
└─ NO → Is it deprecated library/API?
    ├─ YES → FIX with minimal replacement
    └─ NO → Is it critical for game to run?
        ├─ YES → FIX with minimal change
        └─ NO → SKIP (out of scope)
```

**Examples**:
- ✅ KEEP: `var player = new Player()` (works fine, don't change to `const`)
- ✅ KEEP: jQuery-style DOM manipulation (if client uses it and works)
- ✅ KEEP: Callback-based async (if not causing issues)
- ✅ KEEP: `Class.extend()` pattern (custom inheritance, works fine)
- ⚠️ FIX: `new Buffer('data')` → `Buffer.from('data')` (only if throws error, not just warning)
- ⚠️ FIX: `require('websocket-server')` → replace with `ws` (package doesn't install)
- ⚠️ FIX: Missing semicolons that cause syntax errors in strict mode
- ✅ KEEP: Code that logs deprecation warnings but still executes correctly
- ❌ DON'T: Convert `var` to `const/let` (no execution benefit)
- ❌ DON'T: Add async/await where callbacks work
- ❌ DON'T: Replace `for` loops with `.map()` or modern array methods
- ❌ DON'T: Add ES6 classes, arrow functions, or template literals

### Functional Requirements

#### Server Execution
- **FR-001**: System MUST install all npm dependencies when running `npm install` on Node.js 18.x LTS or 20.x LTS without errors
- **FR-002**: System MUST start the game server by running `node server/js/main.js` without crashes or unhandled exceptions
- **FR-003**: Server MUST listen for WebSocket connections on configured port (default 8000)
- **FR-004**: Server MUST accept client connections and handle player authentication (username entry)
- **FR-005**: Server MUST maintain game state for all connected players, mobs, items, and NPCs
- **FR-006**: Server MUST broadcast player movement, combat events, and chat messages to relevant clients
- **FR-007**: Server MUST load map data from `server/maps/world_server.json` without errors
- **FR-008**: Server MUST support multiple world instances as configured in `config_local.json`

#### Client Execution
- **FR-009**: Client MUST load when served via HTTP server (e.g., `python -m http.server`, `npx http-server`, or similar) and accessed via `http://localhost:[port]` in modern browsers (Chrome, Firefox, Safari, Edge)
- **FR-009a**: Client MUST be able to load configuration from `client/config/config_build.json` and `client/config/config_local.json` (which must be created from `-dist` templates before first run)
- **FR-009b**: Client MUST be able to access shared game types from `/shared/js/gametypes.js` (requires symlink `client/shared → ../shared`)
- **FR-010**: Client MUST establish WebSocket connection to configured server host/port when served via HTTP (not file:// protocol)
- **FR-011**: Client MUST render game canvas with map, character sprites, and entities
- **FR-012**: Client MUST accept keyboard input for player movement (arrow keys, WASD)
- **FR-013**: Client MUST accept mouse clicks for combat targeting and NPC interaction
- **FR-014**: Client MUST play audio for game events (combat, loot, chat) when audio enabled
- **FR-015**: Client MUST display chat interface for player communication
- **FR-016**: Client MUST load sprite data from `client/sprites/*.json` and render correctly

#### Game Loop Preservation
- **FR-017**: Player movement mechanics MUST function as originally designed (grid-based pathfinding)
- **FR-018**: Combat mechanics MUST function (attack, damage calculation, death, respawn)
- **FR-019**: Item system MUST function (loot drops, inventory, equipment, stat bonuses)
- **FR-020**: NPC interactions MUST function (dialogue, quests, vendors)
- **FR-021**: Achievement system MUST function (tracking, notifications)
- **FR-022**: Health/mana regeneration MUST function over time

#### Compatibility Constraints
- **FR-023**: System MUST run on Node.js 18.x LTS minimum (Node.js 20.x LTS preferred)
- **FR-024**: System MUST NOT modify any game assets including: images (PNG, JPG), sprites, CSS files, maps (JSON), audio (MP3, OGG), videos, fonts, or any files in `client/img/`, `client/css/`, `client/audio/`, `client/fonts/`, `client/sprites/`, `client/maps/`, `server/maps/`
- **FR-025**: System MUST NOT introduce new game features or change game mechanics
- **FR-026**: System MUST NOT fix existing gameplay bugs (only fix execution-blocking issues)
- **FR-027**: Dependencies MUST be updated ONLY when they fail to install or cause runtime **errors** (not warnings) on Node.js 18.x+ LTS. Deprecation warnings (console warnings that don't prevent execution) MUST be ignored. Code changes MUST be justified by specific **error messages** that prevent execution.
- **FR-028**: Vanilla JavaScript code (ES5, raw DOM manipulation, etc.) MUST be preserved if it runs without errors - NO refactoring, NO modernization, NO style improvements
- **FR-029**: Core game logic (pathfinding, combat calculations, entity management) MUST remain unchanged unless code literally cannot execute

#### Client Serving
- **FR-030**: Client MUST be served via HTTP protocol (not file://) to avoid CORS restrictions with WebSocket connections
- **FR-031**: Any standard HTTP server (Python http.server, Node.js http-server, nginx, Apache, etc.) MAY be used to serve client files during development

#### Build System (Optional)
- **FR-032**: If client build system (`bin/build.sh`) is broken, it MAY be skipped; development mode (HTTP-served client directory) is sufficient for this phase
- **FR-033**: If RequireJS optimization is incompatible, modern alternative MAY be used, but development mode is acceptable

#### Testing & Validation
- **FR-034**: Validation MUST use manual testing (playing the game) - no automated test suite required
- **FR-035**: Adding automated tests is out of scope (would violate minimal changes principle)
- **FR-036**: Validation MUST verify all 8 acceptance scenarios by manual gameplay (✅ Completed 2025-10-07)

### Success Criteria
The feature is complete when a developer can manually validate all of the following:

1. **Installation**: Developer can install dependencies with `npm install` on Node.js 18.x+ without errors
2. **Server Configuration**: Developer can copy server config template and start server successfully
3. **Client Configuration**: Developer can create client configuration files from templates (`config_build.json`, `config_local.json`)
4. **Shared Resources Setup**: Developer can create symlink for shared resources (`client/shared → ../shared`)
5. **Server Startup**: Server starts with `node server/js/main.js` and runs continuously without crashes
6. **Client Connection**: Client loads in browser (via HTTP server with hard refresh) and establishes WebSocket connection to server
7. **Character Creation**: Developer can enter username and create a character
8. **Movement**: Player can move around the map using arrow keys/WASD or click-to-move, sprite animates correctly
9. **Combat**: Player can attack mobs (rats, skeletons, etc.), damage is dealt, mobs die, loot drops
10. **Items**: Player can collect items (weapons, armor, potions), equip them, and stats update
11. **NPCs**: Player can click NPCs and see dialogue/quest text
12. **Chat**: Player can send chat messages visible to other connected players
13. **Multi-player**: Multiple browser windows can connect simultaneously, players see each other's movements in real-time
14. **Console Clean**: No console errors that prevent gameplay (warnings are acceptable)
15. **Assets Unchanged**: All assets (images, CSS, sprites, audio, maps) remain completely unchanged from original repository
16. **Minimal Changes**: Code diff shows ONLY changes to fix deprecated/broken functionality - no gratuitous refactoring
17. **Vanilla Preservation**: Vanilla JavaScript patterns (2012-era) are preserved wherever they still work

**Validation Method**: Manual gameplay testing - no automated test suite required (adding tests would violate minimal changes principle)

---

## Review & Acceptance Checklist
*GATE: Automated checks run during main() execution*

### Content Quality
- [x] No implementation details (languages, frameworks, APIs) - EXCEPTION: Node.js LTS requirement specified per constitution allowance
- [x] Focused on user value and business needs (runnable game is the user value)
- [x] Written for non-technical stakeholders where applicable
- [x] All mandatory sections completed

### Requirement Completeness
- [x] No [NEEDS CLARIFICATION] markers remain
- [x] Requirements are testable and unambiguous (each FR has clear pass/fail)
- [x] Success criteria are measurable (16 explicit validation points)
- [x] Scope is clearly bounded (fix broken code only, preserve vanilla JS, NO asset modifications, NO gratuitous refactoring)
- [x] Dependencies and assumptions identified (Node.js LTS, modern browsers, all assets/vanilla code preserved)

---

## Execution Status
*Updated by main() during processing*

- [x] User description parsed
- [x] Key concepts extracted (actors, actions, constraints)
- [x] Ambiguities marked (none - requirements explicit)
- [x] User scenarios defined (8 acceptance scenarios, 4 edge cases)
- [x] Requirements generated (38 functional requirements across 7 categories - FR-009a, FR-009b added for configuration)
- [x] Entities identified (N/A - infrastructure restoration)
- [x] Review checklist passed
- [x] Implementation completed and validated
- [x] Runtime configuration requirements discovered and documented
- [x] All 17 success criteria validated through manual testing (2025-10-07)

---

**Status**: ✅ COMPLETE AND VALIDATED - All requirements met, game fully functional on Node.js 20.x LTS

**Validation Notes**: 
- Manual testing completed with all 17 success criteria passed
- Critical runtime configuration requirements discovered: client config files and shared symlink required before first run
- Browser cache hard refresh needed after configuration changes
- See `IMPLEMENTATION-SUMMARY.md` for complete validation results and `quickstart.md` for setup guide
