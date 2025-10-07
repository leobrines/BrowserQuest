BrowserQuest
============

BrowserQuest is a HTML5/JavaScript multiplayer game experiment.


Requirements
------------

- **Node.js 20.x LTS** or later
- Modern web browser (Chrome, Firefox, Safari, or Edge)
- Python 3 (for serving client files) OR Node.js http-server package


Quick Start
-----------

### 1. Install Dependencies

```bash
npm install
```

### 2. Configure Server

```bash
cp server/config_local.json-dist server/config_local.json
# Edit server/config_local.json if needed (default: localhost:8000)
```

### 3. Start Game Server

```bash
node server/js/main.js
```

You should see:
```
{"level":30,"time":...,"msg":"Starting BrowserQuest game server..."}
{"level":30,"time":...,"msg":"Server is listening on port 8000"}
```

### 4. Serve Client Files

**IMPORTANT**: The client MUST be served via HTTP (not opened as file://) for WebSocket connections to work.

**Option A - Python (simplest)**:
```bash
cd client
python -m http.server 8080
```

**Option B - Node.js http-server**:
```bash
npm install -g http-server
cd client
http-server -p 8080
```

**Option C - Any HTTP server** (nginx, Apache, etc.)

### 5. Play the Game

Open your browser and navigate to:
```
http://localhost:8080
```

Enter a username and click "Play" to start!


Multiplayer Testing
-------------------

To test multiplayer functionality:
1. Keep the first browser window open with a logged-in player
2. Open a second browser window/tab
3. Navigate to `http://localhost:8080`
4. Create a second character with a different username
5. Both players should see each other and interact in real-time


Troubleshooting
---------------

**Server won't start**:
- Verify Node.js 20+ is installed: `node --version`
- Check that port 8000 is not already in use
- Ensure `npm install` completed successfully

**Client won't connect**:
- Ensure the game server is running on port 8000
- Verify the client is served via HTTP (not file://)
- Check browser console (F12) for WebSocket connection errors
- Verify `client/config/config_local.json` points to correct host/port

**Assets not loading**:
- Ensure HTTP server is running from the `client/` directory
- Check browser console for 404 errors
- Verify file permissions allow reading client files


Documentation
-------------

Additional documentation is located in client and server directories.


License
-------

Code is licensed under MPL 2.0. Content is licensed under CC-BY-SA 3.0.
See the LICENSE file for details.


Credits
-------
Created by [Little Workshop](http://www.littleworkshop.fr):

* Franck Lecollinet - [@whatthefranck](http://twitter.com/whatthefranck)
* Guillaume Lecollinet - [@glecollinet](http://twitter.com/glecollinet)