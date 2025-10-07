# Quickstart: BrowserQuest Manual Validation

**Purpose**: Step-by-step manual testing guide to validate BrowserQuest restoration  
**Testing Approach**: Manual gameplay (no automated tests per spec clarification)  
**Target**: Node.js 20.x LTS on Linux/macOS/Windows

---

## Prerequisites

### Required Software

- **Node.js 20.x LTS** (verify: `node --version` should show `v20.x.x`)
- **npm** (comes with Node.js)
- **Modern web browser**: Chrome, Firefox, Safari, or Edge
- **HTTP server** for client (Python, Node.js http-server, or any)

### System Check

```bash
# Verify Node.js version
node --version   # Should output v20.x.x

# Verify npm
npm --version    # Should output 10.x.x or higher

# Clone repository (if not already)
git clone https://github.com/browserquest/BrowserQuest.git
cd BrowserQuest
```

---

## Phase 1: Installation

### Step 1.1: Install Dependencies

```bash
# From repository root
npm install
```

**Expected Outcome**: 
- ✅ All packages install without errors
- ✅ No "ENOENT" or "404" errors
- ⚠️ Deprecation warnings are acceptable (per spec clarification)

**If Errors Occur**:
- Check Node.js version is 20.x: `node --version`
- Clear npm cache: `npm cache clean --force`
- Delete `node_modules/` and retry: `rm -rf node_modules && npm install`

**Validation**:
```bash
# Check installed packages
ls node_modules/   # Should see: ws, pino, underscore, sanitizer

# Verify critical packages
node -e "console.log(require('ws').Server)"          # Should output [Function: WebSocketServer]
node -e "console.log(require('pino'))"               # Should output function
node -e "console.log(require('underscore').VERSION)" # Should output version like 1.13.7
```

---

## Phase 2: Server Startup

### Step 2.1: Configuration Setup

```bash
# Copy configuration template
cp server/config_local.json-dist server/config_local.json

# Optional: Edit config (default values work for local testing)
# Default: port 8000, 1 world, 200 max players per world
```

**Configuration Options** (`server/config_local.json`):
```json
{
  "host": "localhost",
  "port": 8000,
  "number_of_worlds": 1,
  "max_players_per_world": 200
}
```

### Step 2.2: Start Server

```bash
# From repository root
node server/js/main.js
```

**Expected Outcome**:
- ✅ Server starts without crashes
- ✅ Console logs: "Server is listening on port 8000" (or configured port)
- ✅ No uncaught exceptions or stack traces
- ⚠️ Deprecation warnings in console are acceptable

**Success Indicators**:
```
[timestamp] INFO: Server created
[timestamp] INFO: World 1 created
[timestamp] INFO: Server is listening on port 8000
```

**If Server Crashes**:
- Check for error messages mentioning deprecated APIs
- Verify all dependencies installed correctly
- Ensure port 8000 is not already in use: `lsof -i :8000` (macOS/Linux)

**Keep Server Running** for next steps (open new terminal for client setup)

---

## Phase 3: Client Setup

### Step 3.1: Configure Client WebSocket Connection (CRITICAL)

**⚠️ IMPORTANT**: This step is **REQUIRED** - the game will not work without these configuration files.

```bash
# From repository root, navigate to client config directory
cd client/config

# Copy configuration templates (REQUIRED for first-time setup)
cp config_build.json-dist config_build.json
cp config_local.json-dist config_local.json
```

**Edit both files** to set proper connection settings:

**`client/config/config_build.json`**:
```json
{
  "host": "localhost",
  "port": 8000
}
```

**`client/config/config_local.json`**:
```json
{
  "host": "localhost",
  "port": 8000,
  "dispatcher": false
}
```

### Step 3.2: Create Shared Resources Symlink (CRITICAL)

**⚠️ IMPORTANT**: This symlink is **REQUIRED** for the client to access game type definitions.

```bash
# From repository root, navigate to client directory
cd client

# Create symlink to shared resources
ln -s ../shared shared

# Verify symlink was created
ls -la shared  # Should show: shared -> ../shared
```

**Why this is needed**: The client code loads `/shared/js/gametypes.js`, but the HTTP server only serves files from the `client/` directory. This symlink makes the shared directory accessible.

### Step 3.3: Serve Client via HTTP

**CRITICAL**: Client MUST be served via HTTP (not `file://`) to avoid CORS issues with WebSocket.

**Option A: Python HTTP Server** (simplest):
```bash
# From client directory
cd client
python -m http.server 8080

# Keep running, open new terminal for testing
```

**Option B: Node.js http-server**:
```bash
# Install globally (once)
npm install -g http-server

# From client directory
cd client
http-server -p 8080
```

**Option C: Any other HTTP server** (nginx, Apache, etc.)

**Verify**:
- HTTP server running on port 8080 (or any available port)
- URL: `http://localhost:8080`

---

## Phase 4: Client Connection Test

### Step 4.1: Open Game in Browser

1. Open browser (Chrome/Firefox/Safari/Edge)
2. Navigate to: `http://localhost:8080`
3. **Perform Hard Refresh** (IMPORTANT):
   - Chrome/Firefox/Edge: `Ctrl+Shift+R` (Windows/Linux) or `Cmd+Shift+R` (macOS)
   - Safari: `Cmd+Option+R`
4. Wait for game to load

**⚠️ Why Hard Refresh?**: Browsers aggressively cache JavaScript and JSON files. After creating configuration files, a hard refresh ensures the browser loads the new config instead of cached data.

**Expected Outcome**:
- ✅ Game canvas renders (map, UI elements visible)
- ✅ Character selection screen appears
- ✅ No JavaScript errors in browser console (F12 → Console tab)
- ✅ WebSocket connection established (check Network tab → WS filter)
- ✅ No 404 errors for `/config/config_build.json` or `/shared/js/gametypes.js`

**Browser Console Check**:
```
F12 (open DevTools) → Console tab
```
- Should see game initialization messages
- NO red error messages about WebSocket connection failures
- WebSocket URL should show: `ws://localhost:8000`
- NO 404 errors in console

**Common Issue - 404 Error with URL like `/none?player-name=<username>`**:
- **Cause**: Missing client configuration files (config_build.json, config_local.json)
- **Solution**: Go back to Step 3.1 and create the config files
- **Then**: Hard refresh browser (Ctrl+Shift+R)

**If Client Doesn't Load**:
- Check browser console for errors
- Verify HTTP server is running on correct port
- Verify WebSocket server (Node.js) is running on port 8000
- Check `client/config/config_local.json` host/port settings
- Try clearing browser cache completely and retry

---

## Phase 5: Game Loop Validation

### Test 5.1: Character Creation ✅

**Steps**:
1. Enter a username in the text field (e.g., "TestPlayer")
2. Click "Play" button

**Expected Outcome**:
- ✅ Character creation screen disappears
- ✅ Game world loads with player sprite visible
- ✅ Player spawn point is valid (on walkable terrain, not in wall)

**Failure Indicators**:
- ❌ Blank screen
- ❌ Player sprite not visible
- ❌ Error in browser console about missing assets

---

### Test 5.2: Player Movement ✅

**Steps**:
1. Use arrow keys (↑↓←→) or WASD keys to move
2. Move in all four directions
3. Try moving into walls/obstacles

**Expected Outcome**:
- ✅ Player sprite moves smoothly across map
- ✅ Walking animation plays during movement
- ✅ Player cannot walk through walls/obstacles (collision detection works)
- ✅ Camera follows player (or map scrolls)

**Failure Indicators**:
- ❌ Player doesn't move
- ❌ Player can walk through walls
- ❌ No walking animation
- ❌ Input lag or frozen character

---

### Test 5.3: Combat System ✅

**Steps**:
1. Find a mob (rat, skeleton, crab, etc.) on the map
2. Click on the mob to attack
3. Observe combat interactions
4. Kill the mob

**Expected Outcome**:
- ✅ Player automatically approaches mob when clicked
- ✅ Attack animation plays
- ✅ Damage numbers appear above mob
- ✅ Mob's health decreases
- ✅ Mob dies after sufficient damage
- ✅ Death animation plays
- ✅ Loot drops (if mob drops items)

**Combat Mechanics to Verify**:
- Player takes damage from mob counter-attacks
- Player health decreases when hit
- Combat stops when mob dies
- Player can attack multiple mobs sequentially

**Failure Indicators**:
- ❌ Cannot click mobs
- ❌ No damage dealt
- ❌ Mob health doesn't decrease
- ❌ Mob doesn't die
- ❌ Player takes no damage

---

### Test 5.4: Item System ✅

**Steps**:
1. Kill a mob that drops loot
2. Click on the dropped loot
3. Open inventory (check UI)
4. Equip an item (weapon or armor)

**Expected Outcome**:
- ✅ Loot sparkles/animates on ground
- ✅ Loot can be clicked to collect
- ✅ Item appears in inventory
- ✅ Item can be equipped
- ✅ Player stats update (attack/defense increases)
- ✅ Equipped items show on character sprite (visual change)

**Items to Test**:
- Weapons (sword, axe, morningstar) → increase attack
- Armor (leather, mail, plate) → increase defense
- Potions (health flask) → restore health

**Failure Indicators**:
- ❌ Cannot pick up loot
- ❌ Items don't appear in inventory
- ❌ Cannot equip items
- ❌ Stats don't update

---

### Test 5.5: NPC Interaction ✅

**Steps**:
1. Find an NPC (villager, guard, king, priest, etc.)
2. Click on the NPC
3. Read dialogue/quest text

**Expected Outcome**:
- ✅ Dialogue bubble appears above NPC
- ✅ Quest text or flavor text is readable
- ✅ Bubble disappears after a few seconds or when player moves away

**NPCs to Test**:
- Villagers (quest givers)
- Guards (guards areas)
- King (main quest NPC)
- Priest (healer)

**Failure Indicators**:
- ❌ Cannot click NPCs
- ❌ No dialogue appears
- ❌ Dialogue text is garbled/missing

---

### Test 5.6: Chat System ✅

**Steps**:
1. Press Enter to open chat input
2. Type a message (e.g., "Hello world!")
3. Press Enter to send
4. Verify message appears in chat log

**Expected Outcome**:
- ✅ Chat input field appears when Enter pressed
- ✅ Can type message
- ✅ Message appears in chat log with username
- ✅ Chat log shows previous messages

**Failure Indicators**:
- ❌ Cannot open chat
- ❌ Cannot type
- ❌ Message doesn't send
- ❌ Chat log doesn't display

---

### Test 5.7: Multi-Player Synchronization ✅

**Steps**:
1. Keep first browser window open with logged-in character
2. Open second browser window/tab
3. Navigate to `http://localhost:8080` in second window
4. Create second character with different username
5. Move second character around
6. Observe both windows

**Expected Outcome**:
- ✅ Second player appears in first player's game
- ✅ First player appears in second player's game
- ✅ Movement is synchronized (both players see each other move in real-time)
- ✅ Combat actions visible to both players
- ✅ Chat messages from one player visible to other player

**Advanced Multi-Player Tests**:
- Both players attack same mob
- Both players collect loot
- Players see each other's equipped items
- Players see each other's health bars

**Failure Indicators**:
- ❌ Second player doesn't appear
- ❌ Movement not synchronized
- ❌ Lag exceeds 500ms
- ❌ Players invisible to each other

---

### Test 5.8: Health/Mana Regeneration ✅

**Steps**:
1. Take damage from mob (reduce health below max)
2. Stop combat and wait
3. Observe health bar

**Expected Outcome**:
- ✅ Health gradually regenerates over time
- ✅ Regeneration stops at max health
- ✅ Health bar updates visually

**Failure Indicators**:
- ❌ Health doesn't regenerate
- ❌ Health bar doesn't update

---

### Test 5.9: Achievement System ✅

**Steps**:
1. Perform achievement-triggering action (e.g., kill first mob, reach level 2)
2. Check for achievement notification

**Expected Outcome**:
- ✅ Achievement notification appears (popup or badge)
- ✅ Achievement sound plays (if audio enabled)

**Failure Indicators**:
- ❌ No achievement notification
- ❌ Achievements not tracked

---

## Phase 6: Console Validation

### Server Console Check

**Acceptable**:
- ⚠️ Deprecation warnings (e.g., `DEP0005` Buffer warnings)
- ℹ️ Info logs about player connections/disconnections
- ℹ️ Debug logs about game events

**NOT Acceptable** (indicates errors):
- ❌ Uncaught exceptions
- ❌ Stack traces
- ❌ "ERROR" level logs preventing functionality

### Client Browser Console Check

**Open DevTools** (F12) → Console tab

**Acceptable**:
- ⚠️ Warnings about deprecated APIs
- ℹ️ Game debug messages

**NOT Acceptable**:
- ❌ Red error messages about missing assets
- ❌ WebSocket connection errors
- ❌ 404 errors for JavaScript files
- ❌ CORS errors

---

## Phase 7: Asset Verification

### Verify No Asset Modifications

```bash
# Check git status - only code files should be modified
git status

# Verify NO changes to:
# - client/img/**/*.png
# - client/css/**/*.css
# - client/audio/**/*.{mp3,ogg}
# - client/sprites/**/*.json
# - client/maps/**/*.json
# - server/maps/**/*.json
# - client/fonts/**
```

**Expected**: Only `.js` files and `package.json` should be modified

---

## Phase 8: Code Diff Review

### Verify Minimal Changes

```bash
# View all modified files
git diff --stat

# Review code changes
git diff
```

**Validation Checklist**:
- ✅ Changes are in `server/js/` or `client/js/` only
- ✅ `package.json` has explicit versions (no wildcards)
- ✅ Deprecated API calls replaced (e.g., `Buffer.from()`)
- ✅ Dependency imports updated (e.g., `pino` instead of `log`)
- ❌ NO ES6+ syntax added (no const/let, no arrow functions, no template literals)
- ❌ NO refactoring of working code
- ❌ NO new features added

---

## Success Criteria Summary

**Validation Status**: ✅ **ALL CRITERIA PASSED** (2025-10-07)

| # | Criterion | Status |
|---|-----------|--------|
| 1 | `npm install` succeeds on Node.js 20.x | ✅ |
| 2 | Server starts without crashes | ✅ |
| 3 | Client connects via WebSocket | ✅ (after config setup) |
| 4 | Character creation works | ✅ |
| 5 | Player movement works | ✅ (click & keyboard) |
| 6 | Combat system works | ✅ |
| 7 | Item collection/equip works | ✅ |
| 8 | NPC interactions work | ✅ |
| 9 | Chat system works | ✅ |
| 10 | Multi-player sync works | ✅ |
| 11 | Health regeneration works | ✅ |
| 12 | Achievement system works (notifications display) | ✅ |
| 13 | No execution-blocking console errors | ✅ |
| 14 | All assets unchanged (verify git status) | ✅ |
| 15 | Code diff shows only fix broken code changes | ✅ |
| 16 | Vanilla JS patterns preserved | ✅ |

**Manual testing complete** - BrowserQuest fully functional on Node.js 20.x LTS!

**Key Learnings**:
- Client configuration files must be created from templates before first run
- Shared resources symlink is required for game type definitions
- Hard refresh needed after configuration changes to clear browser cache

---

## Troubleshooting

### Common Issues

**Issue**: `npm install` fails with "Cannot find module"  
**Solution**: Delete `node_modules/` and `package-lock.json`, retry

**Issue**: Server crashes with Buffer deprecation error  
**Solution**: Fix `new Buffer(` → `Buffer.from(` in server code

**Issue**: Client can't connect to WebSocket  
**Solution**: 
- Verify server running on port 8000
- Check client config has correct host/port
- Ensure client served via HTTP (not file://)

**Issue**: Assets (sprites, maps) don't load  
**Solution**:
- Check HTTP server serving from `client/` directory
- Verify paths in `client/index.html` are correct
- Check browser console for 404 errors

**Issue**: Game runs but movement laggy  
**Solution**:
- Check WebSocket connection quality
- Verify server not logging errors during gameplay
- Test with single player first

---

## Performance Benchmarks (Optional)

If testing performance:

**Server**:
- Should handle 200+ concurrent connections (per config)
- Memory usage should remain stable (no leaks)
- CPU usage should be reasonable (<50% on modern hardware)

**Client**:
- Should render at 60 FPS
- Movement should feel responsive (<100ms latency perceived)

**Network**:
- WebSocket latency <100ms p95 (local testing)
- Message size reasonable (<10KB per game update)

---

## Completion

Once all 14 success criteria are met:

1. Document any remaining warnings (acceptable per spec)
2. Update main README with Node.js 20 setup instructions
3. Commit changes with descriptive message
4. Tag as working version

**Validation complete**: BrowserQuest is now executable on Node.js 20.x LTS! 🎮


