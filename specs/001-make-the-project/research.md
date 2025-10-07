# Research: BrowserQuest Node.js 20 Compatibility

**Date**: 2025-10-07  
**Objective**: Identify deprecated dependencies and APIs preventing execution on Node.js 20.x LTS

## Current State Analysis

### package.json Dependencies (2012 versions)

```json
{
  "underscore": ">0",       // Wildcard version
  "log": ">0",              // Deprecated logger
  "bison": ">0",            // Unmaintained (last update: 2013)
  "websocket": ">0",        // Ambiguous package name
  "websocket-server": ">0", // Unmaintained (last update: 2013)
  "sanitizer": ">0",        // HTML sanitizer
  "memcache": ">0"          // Optional (metrics only)
}
```

### Node.js Version Gap

- **Original**: Node.js v0.4.7 (2012)
- **Target**: Node.js 20.x LTS (2024)
- **Gap**: 12+ years, massive API evolution

---

## Dependency Research & Decisions

### 1. `underscore` - ✅ KEEP with explicit version

**Status**: Actively maintained  
**Latest**: 1.13.7 (2024)  
**Weekly Downloads**: 5M+  
**Decision**: **Keep with explicit version `^1.13.7`**

**Rationale**:
- Still works on Node.js 20
- Likely used extensively in 2012 codebase for utility functions
- Modern JavaScript has native alternatives (map, filter, reduce) BUT replacing would violate "fix broken code only"
- No execution-blocking issues

**Action**: Update `package.json` to `"underscore": "^1.13.7"`, make NO code changes

---

### 2. `log` - ⚠️ REPLACE with `pino`

**Status**: DEPRECATED (no longer maintained)  
**Last Update**: ~2014  
**Decision**: **Replace with `pino@^9.5.0`**

**Modern Alternatives**:
| Package | Pros | Cons |
|---------|------|------|
| **pino** | Fastest JSON logger, 7M+ weekly downloads, low overhead | Requires code changes to import |
| winston | Popular (11M+ downloads), feature-rich | Heavier, more complex |
| console | Native, zero deps | No log levels, no structured output |

**Selected**: **pino** - best performance for game server, minimal overhead

**Migration Pattern**:
```javascript
// OLD (log package):
var Log = require('log');
var log = new Log('debug');
log.info('Server starting');

// NEW (pino):
const pino = require('pino');
const log = pino({ level: 'debug' });
log.info('Server starting');
```

**Action**: Search codebase for `require('log')`, replace with pino initialization

---

### 3. `bison` - ⚠️ REPLACE with native JSON

**Status**: UNMAINTAINED (last commit 2013)  
**Purpose**: Binary JSON serialization (BJSON format)  
**Decision**: **Remove, use native `JSON.stringify/parse`**

**Rationale**:
- Bison was for performance (binary format smaller than JSON text)
- Modern V8 JSON parsing is extremely fast (unnecessary optimization)
- Native JSON eliminates dependency, zero breaking changes to data format
- WebSocket `ws` library handles string/binary transparently

**Migration Pattern**:
```javascript
// OLD (bison):
var BISON = require('bison');
var encoded = BISON.encode(data);
var decoded = BISON.decode(encoded);

// NEW (native JSON):
var encoded = JSON.stringify(data);
var decoded = JSON.parse(encoded);
```

**Action**: Search for `BISON` or `bison`, replace with native JSON

---

### 4. `websocket` & `websocket-server` - ⚠️ REPLACE with `ws`

**Status**: Both UNMAINTAINED  
**Confusion**: Two separate packages with similar names  
**Decision**: **Replace both with `ws@^8.18.0`**

**Modern WebSocket Libraries**:
| Package | Weekly Downloads | Last Update | Verdict |
|---------|------------------|-------------|---------|
| **ws** | 90M+ | Active (2024) | ✅ BEST |
| uWebSockets.js | 200K | Active | C++ bindings, overkill |
| socket.io | 6M+ | Active | Too heavy, adds protocol layer |

**Selected**: **ws** - industry standard, lightweight, WebSocket protocol only

**Migration Guide**:
```javascript
// OLD (websocket-server):
var ws = require('websocket-server');
var server = ws.createServer();
server.on('connection', function(conn) {
  conn.on('message', function(msg) { ... });
  conn.send('data');
});

// NEW (ws):
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 8000 });
wss.on('connection', function(ws) {
  ws.on('message', function(msg) { ... });
  ws.send('data');
});
```

**Key API Differences**:
- `websocket-server`: `conn.send()` method
- `ws`: `ws.send()` method (similar)
- Both: `on('connection')`, `on('message')` events (compatible)

**Action**: Locate WebSocket server initialization in `server/js/`, adapt to `ws` API

---

### 5. `sanitizer` - ⚠️ TEST, replace if broken

**Status**: Old package (last update 2016)  
**Purpose**: HTML sanitization (prevent XSS)  
**Decision**: **Test first, replace only if broken**

**Rationale**:
- Used for chat messages (prevent malicious HTML/scripts)
- If it still works on Node.js 20, KEEP (fix broken code only)
- If broken, replace with `validator@^13.12.0` or `dompurify@^3.x`

**Testing**:
```bash
npm install sanitizer@latest
node -e "const sanitizer = require('sanitizer'); console.log(sanitizer.sanitize('<script>alert(1)</script>'))"
```

**Action**: Test during implementation, replace only if import fails or throws errors

---

### 6. `memcache` - ❌ SKIP (optional)

**Status**: Unmaintained  
**Purpose**: Memcache client (metrics/stats only per server README)  
**Decision**: **Remove from package.json, make code optional**

**Rationale**:
- README says: "only if you want metrics"
- Optional feature, not core to game functionality
- If code checks for memcache, wrap in try/catch or conditional

**Action**: Remove from dependencies, ensure server starts without it

---

## Node.js API Compatibility Research

### Deprecated APIs to Fix

1. **Buffer Constructor** (deprecated since Node.js 6)
   ```javascript
   // DEPRECATED:
   new Buffer(string)
   new Buffer(size)
   
   // MODERN:
   Buffer.from(string)
   Buffer.alloc(size)
   ```
   **Search for**: `new Buffer(`

2. **url.parse()** (legacy API, not removed but discouraged)
   ```javascript
   // OLD:
   var url = require('url');
   var parsed = url.parse('http://example.com');
   
   // MODERN:
   const parsed = new URL('http://example.com');
   ```
   **Search for**: `url.parse(` (only fix if causing errors, per spec)

3. **crypto.createCipher()** (deprecated since Node.js 10)
   ```javascript
   // DEPRECATED:
   crypto.createCipher('aes-256-cbc', password)
   
   // MODERN:
   crypto.createCipheriv('aes-256-cbc', key, iv)
   ```
   **Search for**: `crypto.createCipher(` (unlikely in game code)

### Likely Safe (no action needed unless errors found)

- `fs` module: Basic file operations unchanged
- `http` module: HTTP server API stable
- `path` module: Path utilities unchanged
- `process` module: Process APIs stable

---

## Build System Analysis

### Current Build Process

Per `client/README.md`:
1. Configure WebSocket host/port in `client/config/config_build.json`
2. Run `bin/build.sh` (uses RequireJS optimizer `r.js`)
3. Generates `client-build/` directory (production-ready)

### Build System Decision

**Decision**: **SKIP build system, use development mode**

**Rationale**:
- Spec allows: "If build system broken, MAY be skipped; development mode sufficient"
- RequireJS optimizer (`r.js`) may have compatibility issues
- Development mode = serve `client/` directory directly via HTTP
- No client-side code changes needed (keep RequireJS module loader in `client/`)

**Development Mode Setup**:
```bash
# From client directory:
python -m http.server 8080
# OR
npx http-server -p 8080
```

**Action**: Document in README that build step is optional, provide dev mode instructions

---

## Codebase Patterns to Preserve

### Vanilla JavaScript Patterns (2012 era)

**DO NOT CHANGE** (per spec: "fix broken code only"):

1. **ES5 Syntax**:
   ```javascript
   var player = { x: 0, y: 0 };  // Keep var, don't change to const/let
   for (var i = 0; i < arr.length; i++) { ... }  // Keep for loops
   function Player(name) { this.name = name; }  // Keep function constructors
   ```

2. **Callback Patterns**:
   ```javascript
   fs.readFile('file.txt', function(err, data) {
     if (err) throw err;
     console.log(data);
   });
   // Do NOT convert to async/await or Promises
   ```

3. **Prototype Inheritance**:
   ```javascript
   Player.prototype.move = function(x, y) { ... };
   // Do NOT convert to ES6 classes
   ```

4. **Custom Class System** (`server/js/lib/class.js`):
   ```javascript
   var Player = Class.extend({
     init: function() { ... }
   });
   // Keep as-is, this pattern still works
   ```

---

## Summary of Changes

### Dependencies to Replace

| Old Package | New Package | Reason |
|-------------|-------------|--------|
| `log` | `pino@^9.5.0` | Deprecated, unmaintained |
| `bison` | Native `JSON` | Unmaintained since 2013 |
| `websocket` | `ws@^8.18.0` | Unmaintained, replace with modern |
| `websocket-server` | `ws@^8.18.0` | Unmaintained, replace with modern |
| `sanitizer` | Test, replace if broken | Old but may work |
| `memcache` | Remove (optional) | Not core functionality |

### Dependencies to Keep

| Package | Version | Reason |
|---------|---------|--------|
| `underscore` | `^1.13.7` | Works fine, widely used in codebase |

### Node.js APIs to Fix

1. `new Buffer(...)` → `Buffer.from(...)` or `Buffer.alloc(...)`
2. Other deprecated APIs: Fix only if causing errors

### Code to Preserve

- All vanilla JS (ES5 syntax, var, for loops, callbacks)
- All game logic (movement, combat, items, NPCs)
- All assets (images, CSS, sprites, audio, maps)

---

## Next Steps (Phase 1)

1. Create `quickstart.md` with manual validation steps
2. Document expected server startup behavior
3. Document client serving requirement (HTTP server needed)
4. Define acceptance criteria for each game mechanic

---

**Research Complete**: Ready for Phase 1 design artifacts

