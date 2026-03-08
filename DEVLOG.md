# DEVLOG - Arena Survivor

---

## Entry 0: 6th March 2026

- **Goal:** To build a live scalable multiplayer arena game with AI Bots, and get placed on a good company by November 2026.

- **Current Status:** I know basic C++ and basic Unity, I've never build a multiplayer game, never used networking, never deployed anything to the cloud. I've made 3-4 small games in Unity over the span of last 4 years.

- **My Fear:** Like always I fear my momentum getting lost after a hard start. I fear this project would get too complex and I'll abonden it. I fear that I've already wasted so much time and have very little time to catch up.

- **My Resolve** I've been deciding since 6th standard about my future, now the deciding is done. Starting today I am someone who doesn't wait to feel ready, every single day I'll open the editor, one problem, one commit, one devlog. I've already wasted a decade wanting to feel ready now the resolve is made. See myself tomorrow!

---

## Day 3 — Scene + Tilemap Setup

### What I did
- Installed 2D Tilemap package from Package Manager
- Created Floor tilemap — filled 20x20 grid with floor tile
- Created Walls tilemap — drew walls around arena perimeter
- Set up camera to frame the entire arena
- Watched Unity Learn 2D Game Kit first section

### What I learned
- Tilemaps use a Grid parent with Tilemap children
- PPU (Pixels Per Unit) must match sprite size or tiles won't fit the grid
- Need to switch active tilemap in Tile Palette when painting different layers
- Walls Tilemap needs a Tilemap Collider 2D for future collision

### Issues I ran into
- Floor sprite was not filling the tilemap box — fixed by matching
  PPU to sprite pixel dimensions and setting Filter Mode to Point

### Commit
feat: arena tilemap floor and walls

### Next up
- Day 4: adding player character and movement to the arena.

---

## Day 4 — Player Movement

### What I did
- Watched Brackeys 2D Movement tutorial
- Created Player GameObject with Sprite Renderer (added a top-down blue man sprite)
- Added Rigidbody2D to Player
- Installed new Input System package from Package Manager
- Wrote characterCtrl.cs script — WASD movement using Rigidbody2D.MovePosition
- Tested player movement and wall collision

### What I learned
- Rigidbody2D.MovePosition works for movement but not 100% sure if it's
  the best approach — need to look into this more
- Gravity Scale set to 0 because the game is top-down, player shouldn't fall
- Freezing Z rotation stops the player from spinning unnecessarily on collision
- Continuous Collision Detection prevents clipping through walls
- Movement logic goes in FixedUpdate (runs at fixed time intervals),
  input reading goes in Update (catches input as fast as possible)
- Input.GetAxisRaw returns -1, 0 or 1 with no smoothing — better for
  responsive movement
- Normalizing the input vector prevents diagonal movement being faster

### Issues I ran into
- Forgot to add BoxCollider2D before pushing to repo — wall collision
  not working because of this, need to add it and push a fix commit

### Commit
feat: player WASD movement with Rigidbody2D

### Next up
- Fix: add BoxCollider2D to Player and verify wall collision works
```
