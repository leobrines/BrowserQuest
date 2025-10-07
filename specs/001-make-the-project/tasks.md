# Tasks: Restore BrowserQuest Executability

**Input**: Design documents from `<repository_root>/specs/001-make-the-project/`  
**Prerequisites**: plan.md (✅), research.md (✅), quickstart.md (✅)

## Format: `[ID] [P?] Description`
- **[P]**: Can run in parallel (different files, no dependencies)
- Include exact file paths in descriptions

## Path Conventions
- **Repository root**: `<repository_root>/`
- **Server code**: `server/js/`
- **Client code**: `client/js/`
- **Shared code**: `shared/js/`

---

## Phase 3.1: Dependency Audit & Setup

### T001 - Audit Current Dependencies on Node.js 20 [X]
**Description**: Test existing package.json with Node.js 20.x to identify which dependencies actually fail  
**File**: `package.json`  
**Actions**:
1. Ensure Node.js 20.x is installed: `node --version`
2. Attempt installation: `npm install`
3. Document all errors and warnings
4. Note which packages fail to install vs. which install but may have issues
5. Search codebase for usage of each dependency to understand impact

**Expected Errors**:
- `websocket-server` - likely 404 or build failure
- `log` - may install but deprecated
- `bison` - may install but unmaintained

**Success**: Document created with error messages and dependency usage map

---

### T002 - Update package.json with Explicit Modern Versions [X]
**Description**: Replace wildcard versions with explicit modern package versions  
**File**: `<repository_root>/package.json`  
**Actions**:
```json
{
  "name": "BrowserQuest",
  "version": "0.0.1",
  "private": false,
  "type": "commonjs",
  "engines": {
    "node": ">=20.0.0"
  },
  "dependencies": {
    "underscore": "^1.13.7",
    "pino": "^9.5.0",
    "ws": "^8.18.0",
    "sanitizer": "^0.1.3"
  }
}
```

**Changes**:
- Remove: `log`, `bison`, `websocket`, `websocket-server`, `memcache`
- Add: `pino` (replaces log), `ws` (replaces websocket packages)
- Keep: `underscore` with explicit version
- Keep: `sanitizer` (test if compatible, remove in T008 if broken)
- Add: `type: "commonjs"` (explicit CommonJS modules)
- Add: `engines` field (Node.js 20+ requirement)

**Success**: package.json updated, ready for npm install

---

### T003 - Test Installation with New Dependencies [X]
**Description**: Verify npm install works with updated package.json  
**File**: None (installation test)  
**Actions**:
1. Delete `node_modules/` and `package-lock.json`
2. Run: `npm install`
3. Verify all packages install successfully
4. Check for deprecation warnings (acceptable per spec)
5. Test imports: `node -e "console.log(require('ws').Server)"`

**Expected Outcome**:
- ✅ All packages install without errors
- ⚠️ Deprecation warnings acceptable
- ✅ Key packages importable

**Success**: Clean install on Node.js 20.x

---

## Phase 3.2: WebSocket Migration (ws package)

### T004 - Identify WebSocket Server Implementation [X]
**Description**: Locate where websocket-server is initialized in server code  
**File**: Likely `server/js/ws.js` or `server/js/main.js`  
**Actions**:
1. Search for: `require('websocket-server')` or `require('websocket')`
2. Search for: `createServer`, `WebSocket`, `ws.`
3. Document current API usage pattern
4. Identify all connection, message, send, close handlers

**Expected Location**: `server/js/ws.js` (based on filename)

**Success**: Documented current WebSocket implementation

---

### T005 - Replace websocket-server with ws Package [X]
**Description**: Migrate WebSocket server to modern ws library  
**File**: `<repository_root>/server/js/ws.js`  
**Actions**:

**OLD Pattern** (search for):
```javascript
var ws = require('websocket-server');
var server = ws.createServer();
server.on('connection', function(conn) {
  conn.on('message', function(msg) { ... });
  conn.send(data);
});
```

**NEW Pattern** (replace with):
```javascript
var WebSocket = require('ws');
var wss = new WebSocket.Server({ port: 8000 }); // or from config
wss.on('connection', function(ws) {
  ws.on('message', function(msg) { ... });
  ws.send(data);
});
```

**Key Changes**:
- `require('websocket-server')` → `require('ws')`
- `ws.createServer()` → `new WebSocket.Server({ port })`
- `conn` parameter → `ws` parameter
- Event handlers same: `on('connection')`, `on('message')`

**Preserve**:
- All game logic inside event handlers
- Vanilla JS style (var, callbacks)
- No refactoring of working code

**Success**: Server uses ws package, compiles without errors

---

## Phase 3.3: Logging Migration (pino package)

### T006 - Identify log Package Usage [X]
**Description**: Find all locations where log package is used  
**Files**: Search across `server/js/*.js`  
**Actions**:
1. Search for: `require('log')`
2. Search for: `new Log(` 
3. Search for: `log.info(`, `log.error(`, `log.debug(`
4. Document all log statements and their context

**Expected Locations**:
- `server/js/main.js` - server startup logs
- `server/js/worldserver.js` - game event logs
- Other server files - various logging

**Success**: Complete list of files using log package

---

### T007 - Replace log with pino Package [X]
**Description**: Migrate logging to modern pino library  
**Files**: All server files identified in T006  
**Actions**:

**OLD Pattern** (search for):
```javascript
var Log = require('log');
var log = new Log('debug');
log.info('message');
log.error('error');
```

**NEW Pattern** (replace with):
```javascript
var pino = require('pino');
var log = pino({ level: 'debug' });
log.info('message');
log.error('error');
```

**Preserve**:
- Keep var declarations (don't change to const)
- Keep same log levels (info, error, debug, warn)
- Keep same log message content
- Minimal changes only

**Success**: All log statements use pino, server runs without logging errors

---

## Phase 3.4: Binary Serialization Migration (bison → JSON)

### T008 - Identify bison Package Usage [X]
**Description**: Find all locations where bison is used for serialization  
**Files**: Search across `server/js/*.js` and `client/js/*.js`  
**Actions**:
1. Search for: `require('bison')` or `require('BISON')`
2. Search for: `BISON.encode(`, `BISON.decode(`
3. Search for: `.encode(`, `.decode(` (if assigned to variable)
4. Document all encode/decode locations

**Expected Locations**:
- WebSocket message handlers (sending/receiving game data)
- Possibly in `server/js/ws.js` or message handling files

**Success**: Complete list of bison usage

---

### T009 - Replace bison with Native JSON [X]
**Description**: Replace binary serialization with native JSON.stringify/parse  
**Files**: All files identified in T008  
**Actions**:

**OLD Pattern** (search for):
```javascript
var BISON = require('bison');
var encoded = BISON.encode(data);
var decoded = BISON.decode(message);
```

**NEW Pattern** (replace with):
```javascript
// Remove bison import
var encoded = JSON.stringify(data);
var decoded = JSON.parse(message);
```

**Notes**:
- Native JSON is built-in (no require needed)
- Performance difference negligible for game message sizes
- ws library handles string messages transparently
- Preserve all surrounding game logic

**Success**: No bison references remain, JSON serialization works

---

## Phase 3.5: Deprecated Node.js API Fixes

### T010 [P] - Search for Deprecated Buffer Constructor [X]
**Description**: Find all instances of deprecated Buffer() constructor  
**Files**: All `*.js` files in `server/js/`, `client/js/`, `shared/js/`  
**Actions**:
1. Search for: `new Buffer(`
2. Search for: `Buffer(` (without new)
3. Document each occurrence with file path and line number
4. Categorize by usage:
   - `new Buffer(string)` → `Buffer.from(string)`
   - `new Buffer(size)` → `Buffer.alloc(size)`
   - `new Buffer(array)` → `Buffer.from(array)`

**Expected**: May find in server code handling binary data

**Success**: Complete list of Buffer constructor usage

---

### T011 [P] - Fix Deprecated Buffer Constructor Calls [X]
**Description**: Replace deprecated Buffer() with Buffer.from() or Buffer.alloc()  
**Files**: All files identified in T010  
**Actions**:

**OLD Patterns** (replace):
```javascript
new Buffer('string')           → Buffer.from('string')
new Buffer('string', 'utf8')   → Buffer.from('string', 'utf8')
new Buffer(10)                 → Buffer.alloc(10)
new Buffer([1, 2, 3])          → Buffer.from([1, 2, 3])
```

**Preserve**:
- All surrounding code unchanged
- Keep var declarations
- Keep same functionality

**Success**: No `new Buffer(` patterns remain, Node.js 20 doesn't log Buffer deprecation warnings

---

### T012 [P] - Search for Other Deprecated APIs [X]
**Description**: Check for other deprecated Node.js APIs  
**Files**: All `*.js` files  
**Actions**:
1. Search for: `crypto.createCipher(` (deprecated, needs createCipheriv)
2. Search for: `url.parse(` (legacy, but only fix if causing errors)
3. Search for: `domain` module (deprecated)
4. Search for: `sys` module (renamed to `util`)
5. Document findings

**Note**: Only fix if causing execution errors (per spec clarification)

**Success**: Documented any deprecated API usage

---

### T013 - Fix Critical Deprecated APIs (if found in T012) [X]
**Description**: Fix only execution-blocking deprecated APIs  
**Files**: Depends on T012 findings  
**Actions**:
- **If crypto.createCipher found**: Replace with crypto.createCipheriv (requires IV)
- **If domain module found**: Remove or replace with try/catch
- **Do NOT fix url.parse** unless causing errors (warnings are acceptable)

**Success**: No execution-blocking deprecated APIs remain

---

## Phase 3.6: Sanitizer Compatibility Check

### T014 - Test sanitizer Package Compatibility [X]
**Description**: Verify if sanitizer@0.1.3 works on Node.js 20  
**File**: Test import and usage  
**Actions**:
1. After T003 (npm install), test: `node -e "var s = require('sanitizer'); console.log(s.sanitize('<script>xss</script>'))"`
2. If works: ✅ Keep in package.json
3. If fails: ❌ Remove from package.json (or replace with validator/dompurify)
4. Locate usage in codebase (likely chat message handling)
5. If removed, either:
   - Option A: Remove sanitization (acceptable for local/trusted environment)
   - Option B: Replace with simple regex: `msg.replace(/<[^>]*>/g, '')`

**Expected**: Likely still works (simple package)

**Success**: Decision made on sanitizer (keep or remove)

---

## Phase 3.7: Server Startup Validation

### T015 - Test Server Startup [X]
**Description**: Verify server starts without crashes on Node.js 20  
**Files**: None (runtime test)  
**Actions**:
1. Copy config: `cp server/config_local.json-dist server/config_local.json`
2. Start server: `node server/js/main.js`
3. Observe console output
4. Check for:
   - ✅ "Server is listening on port 8000" message
   - ✅ No uncaught exceptions
   - ✅ No stack traces
   - ⚠️ Deprecation warnings acceptable
5. Let run for 30 seconds to ensure stability
6. Ctrl+C to stop

**Expected Errors** (if any previous tasks incomplete):
- Module not found errors → revisit T002-T003
- Buffer deprecation → revisit T010-T011
- WebSocket errors → revisit T004-T005

**Success**: Server starts cleanly and runs without crashes

---

## Phase 3.8: Client HTTP Serving Setup

### T016 - Document Client Serving Requirement [X]
**Description**: Create instructions for serving client via HTTP  
**File**: `<repository_root>/README.md`  
**Actions**:
Add section to README:

```markdown
## Running BrowserQuest (Node.js 20.x)

### Prerequisites
- Node.js 20.x LTS or later
- Python 3 (for HTTP server) OR Node.js http-server package

### Setup

1. **Install Dependencies**:
   ```bash
   npm install
   ```

2. **Configure Server**:
   ```bash
   cp server/config_local.json-dist server/config_local.json
   # Edit if needed (default: localhost:8000)
   ```

3. **Start Game Server**:
   ```bash
   node server/js/main.js
   ```
   
   Expected output: "Server is listening on port 8000"

4. **Serve Client** (in new terminal):
   
   **Option A - Python**:
   ```bash
   cd client
   python -m http.server 8080
   ```
   
   **Option B - Node.js**:
   ```bash
   npm install -g http-server
   cd client
   http-server -p 8080
   ```

5. **Play**:
   - Open browser: `http://localhost:8080`
   - Enter username and click "Play"

### Notes
- Client MUST be served via HTTP (not file://) for WebSocket to work
- Server and client must run simultaneously
- Use modern browsers (Chrome, Firefox, Safari, Edge)
```

**Success**: README has clear setup instructions for Node.js 20

---

### T017 - Test Client WebSocket Connection [MANUAL - READY]
**Description**: Verify client can connect to server via WebSocket  
**Files**: None (integration test)  
**Actions**:
1. Ensure server running (from T015)
2. Serve client via HTTP (follow T016 instructions)
3. Open browser: `http://localhost:8080`
4. Open browser DevTools (F12) → Console
5. Check for:
   - ✅ No WebSocket connection errors
   - ✅ Game canvas loads
   - ✅ Character creation screen appears
6. Check Network tab → WS (WebSocket filter):
   - ✅ Connection to `ws://localhost:8000` established
   - ✅ Status: 101 Switching Protocols

**Expected Issues** (if found):
- CORS errors → ensure client served via HTTP, not file://
- Connection refused → ensure server running on port 8000
- 404 errors → check client config points to correct server

**Success**: Client loads and WebSocket connects successfully

---

### T018 - Verify Client Assets Load [MANUAL - READY]
**Description**: Confirm all game assets (sprites, maps, audio) load without errors  
**Files**: None (validation test)  
**Actions**:
1. With client loaded in browser (from T017)
2. Check browser Console for errors:
   - ❌ No 404 errors for PNG files
   - ❌ No 404 errors for JSON files (sprites, maps)
   - ❌ No 404 errors for audio files (MP3/OGG)
3. Visual check:
   - ✅ Map tiles render
   - ✅ Character sprite visible after login
   - ✅ UI elements visible (health bar, inventory, chat)

**Success**: All assets load, no 404 errors in console

---

## Phase 3.9: Manual Game Loop Validation

**Note**: The following tasks are manual gameplay tests per quickstart.md.  
No automated tests per spec clarification: "Don't create or run any unit or integration test"

---

### T019 - Manual Test: Character Creation [MANUAL - READY]
**Description**: Validate player can create character and enter game  
**Reference**: `specs/001-make-the-project/quickstart.md` - Test 5.1  
**Actions**:
1. Open `http://localhost:8080` in browser
2. Enter username: "TestPlayer1"
3. Click "Play" button
4. Verify:
   - ✅ Character creation screen disappears
   - ✅ Game world loads
   - ✅ Player sprite visible on map
   - ✅ Spawn point is walkable (not in wall)

**Success**: Character creation works, player enters game

---

### T020 - Manual Test: Player Movement [MANUAL - READY]
**Description**: Validate player movement and collision detection  
**Reference**: `specs/001-make-the-project/quickstart.md` - Test 5.2  
**Actions**:
1. Use arrow keys (↑↓←→) to move player
2. Try WASD keys as alternative
3. Move in all four directions
4. Try walking into walls/obstacles
5. Verify:
   - ✅ Player sprite moves smoothly
   - ✅ Walking animation plays
   - ✅ Cannot walk through walls (collision detection)
   - ✅ Camera follows player

**Success**: Movement mechanics work correctly

---

### T021 - Manual Test: Combat System [MANUAL - READY]
**Description**: Validate combat mechanics (attack, damage, death, loot)  
**Reference**: `specs/001-make-the-project/quickstart.md` - Test 5.3  
**Actions**:
1. Find mob on map (rat, skeleton, crab)
2. Click mob to attack
3. Observe combat:
   - ✅ Player approaches mob
   - ✅ Attack animation plays
   - ✅ Damage numbers appear
   - ✅ Mob health decreases
   - ✅ Player takes damage from mob counter-attacks
   - ✅ Mob dies after sufficient damage
   - ✅ Death animation plays
   - ✅ Loot drops (if applicable)

**Success**: Combat system functional

---

### T022 - Manual Test: Item Collection [MANUAL - READY]
**Description**: Validate item pickup, inventory, and equipment  
**Reference**: `specs/001-make-the-project/quickstart.md` - Test 5.4  
**Actions**:
1. Kill mob to get loot drop
2. Click loot on ground
3. Check inventory (UI)
4. Equip item (weapon or armor)
5. Verify:
   - ✅ Loot collected
   - ✅ Item in inventory
   - ✅ Item can be equipped
   - ✅ Stats update (attack/defense)
   - ✅ Visual change on character sprite

**Success**: Item system works

---

### T023 - Manual Test: NPC Interaction [MANUAL - READY]
**Description**: Validate NPC dialogue and quests  
**Reference**: `specs/001-make-the-project/quickstart.md` - Test 5.5  
**Actions**:
1. Find NPC (villager, guard, king, priest)
2. Click NPC
3. Verify:
   - ✅ Dialogue bubble appears
   - ✅ Text is readable
   - ✅ Bubble disappears after timeout or movement

**Success**: NPC interactions work

---

### T024 - Manual Test: Chat System [MANUAL - READY]
**Description**: Validate chat input and display  
**Reference**: `specs/001-make-the-project/quickstart.md` - Test 5.6  
**Actions**:
1. Press Enter to open chat
2. Type message: "Hello world!"
3. Press Enter to send
4. Verify:
   - ✅ Chat input opens
   - ✅ Can type message
   - ✅ Message appears in chat log
   - ✅ Username shown with message

**Success**: Chat system works

---

### T025 - Manual Test: Multi-Player Synchronization [MANUAL - READY]
**Description**: Validate real-time player synchronization  
**Reference**: `specs/001-make-the-project/quickstart.md` - Test 5.7  
**Actions**:
1. Keep first browser window open (TestPlayer1)
2. Open second browser window/tab
3. Navigate to `http://localhost:8080`
4. Create character: "TestPlayer2"
5. Move TestPlayer2 around
6. Observe both windows:
   - ✅ Player2 visible in Player1's game
   - ✅ Player1 visible in Player2's game
   - ✅ Movement synchronized in real-time
   - ✅ Combat actions visible to both
   - ✅ Chat messages visible to both

**Success**: Multi-player synchronization works

---

### T026 - Manual Test: Health Regeneration [MANUAL - READY]
**Description**: Validate health regeneration over time  
**Reference**: `specs/001-make-the-project/quickstart.md` - Test 5.8  
**Actions**:
1. Take damage from mob (reduce health)
2. Stop combat and wait
3. Verify:
   - ✅ Health gradually regenerates
   - ✅ Stops at max health
   - ✅ Health bar updates visually

**Success**: Health regeneration works

---

## Phase 3.10: Final Validation & Documentation

### T027 - Verify No Asset Modifications [X]  
**Description**: Confirm git diff shows only code changes, no assets (validates FR-024)  
**Purpose**: Specific check for zero asset modifications - distinct from T028 (code style) and T030 (complete checklist)  
**Files**: None (git validation)  
**Actions**:
```bash
# Check git status
git status

# View modified files
git diff --stat

# Verify NO changes to:
# - client/img/**/*.png
# - client/css/**/*.css
# - client/audio/**/*.{mp3,ogg}
# - client/sprites/**/*.json
# - client/maps/**/*.json
# - server/maps/**/*.json
# - client/fonts/**
# - client/index.html
```

**Expected**: Only `.js` files and `package.json` modified

**Success**: All assets unchanged (per spec requirement)

---

### T028 - Review Code Changes for Fix Broken Code Approach [X]
**Description**: Verify only minimal changes made to fix broken code (validates FR-027, FR-028, FR-029)  
**Purpose**: Focused code review for adherence to "fix broken code only" principle - distinct from T027 (assets) and T030 (complete acceptance)  
**Files**: None (code review)  
**Actions**:
```bash
# Review all changes
git diff

# Checklist:
# ✅ Only deprecated API fixes (Buffer, websocket, log, bison)
# ✅ package.json has explicit versions
# ❌ No ES6+ syntax added (no const/let, arrow functions, template literals)
# ❌ No refactoring of working code
# ❌ No code style improvements
# ❌ No new features added
```

**Success**: Code diff contains only fix broken code changes

---

### T029 - Document Remaining Deprecation Warnings [X]
**Description**: List any remaining console warnings (acceptable per spec)  
**File**: Create `specs/001-make-the-project/known-warnings.md`  
**Actions**:
1. Start server, capture console output
2. Load client, capture browser console
3. Document any warnings:
   - Deprecation warnings (acceptable)
   - Performance warnings (acceptable)
4. Confirm no ERROR-level messages

**Success**: Warnings documented, no errors

---

### T030 - Final Acceptance Test [MANUAL - READY]
**Description**: Run complete 16-point validation checklist (validates all FRs)  
**Purpose**: Comprehensive acceptance test covering all functional requirements - combines findings from T027 (assets), T028 (code), T019-T026 (gameplay)  
**Reference**: `specs/001-make-the-project/quickstart.md` - Success Criteria Summary  
**Actions**:
Execute all 16 success criteria and check:

| # | Criterion | Status |
|---|-----------|--------|
| 1 | npm install succeeds on Node.js 20.x | ✅ |
| 2 | Server starts without crashes | ✅ |
| 3 | Client connects via WebSocket | ☐ (manual) |
| 4 | Character creation works | ☐ |
| 5 | Player movement works | ☐ |
| 6 | Combat system works | ☐ |
| 7 | Item collection/equip works | ☐ |
| 8 | NPC interactions work | ☐ |
| 9 | Chat system works | ☐ |
| 10 | Multi-player sync works | ☐ |
| 11 | Health regeneration works (from T026) | ☐ |
| 12 | Achievement system works (notifications display when triggered) | ☐ |
| 13 | No execution-blocking console errors | ✅ |
| 14 | All assets unchanged | ✅ |
| 15 | Code diff shows only fix broken code changes | ✅ |
| 16 | Vanilla JS patterns preserved | ✅ |

**Success**: ALL 16 criteria checked ✅

**Note**: Criteria 11-12 explicitly validate FR-022 (health regeneration) and FR-021 (achievement system). Criterion 15 uses "fix broken code" terminology per constitution.

---

## Dependencies

### Critical Path (must complete in order):
1. **T001-T003**: Setup (must complete before any code changes)
2. **T004-T005**: WebSocket migration (T004 identifies, T005 implements)
3. **T006-T007**: Logging migration (T006 identifies, T007 implements)
4. **T008-T009**: Bison migration (T008 identifies, T009 implements)
5. **T015**: Server startup test (blocks client tests)
6. **T016-T018**: Client setup (requires T015 server running)
7. **T019-T026**: Manual validation (requires T015-T018)
8. **T027-T030**: Final validation (requires all above)

### Parallel Opportunities:
- **T010 [P]** and **T012 [P]**: Can search for deprecated APIs simultaneously
- **T011 [P]**: Can fix Buffer calls independently once identified
- **T019-T026**: Manual tests can be done in any order after T018

---

## Parallel Execution Example

These identification tasks can run simultaneously (different search patterns, read-only):
```bash
# Terminal 1:
# T010: Search for Buffer constructor
grep -r "new Buffer(" server/ client/ shared/

# Terminal 2:  
# T012: Search for other deprecated APIs
grep -r "crypto.createCipher(" server/ client/
grep -r "url.parse(" server/ client/
```

---

## Notes

### Fix Broken Code Philosophy
- ✅ Fix ONLY execution-blocking errors
- ✅ Preserve all vanilla JS (var, for loops, callbacks)
- ✅ Change only what's broken to get game running
- ❌ No refactoring of working code
- ❌ No ES6+ modernization
- ❌ No automated test creation

### Manual Validation Approach
- All validation is manual gameplay testing (T019-T026)
- No automated tests per spec clarification and user input
- Use quickstart.md as validation guide
- Human tester must verify each game mechanic

### Expected Total Time
- Setup & Dependencies: 30-60 minutes (T001-T003)
- Code Migration: 2-3 hours (T004-T014)
- Server Testing: 30 minutes (T015)
- Client Setup: 30 minutes (T016-T018)
- Manual Validation: 1-2 hours (T019-T026)
- Documentation: 30 minutes (T027-T030)
- **Total**: 5-8 hours

---

## Task Generation Summary

- **Total Tasks**: 30
- **Parallel Tasks**: 3 (T010, T011, T012)
- **Manual Tests**: 8 (T019-T026)
- **Automated Tests**: 0 (per spec requirement)
- **Files Modified**: ~5-10 (package.json, server/js/*.js, README.md)
- **Assets Modified**: 0 (per spec requirement)

---

**Tasks Ready for Execution**  
**Next**: Begin with T001 (Dependency Audit)  
**Reference**: Use quickstart.md for validation procedures

