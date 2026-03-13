# DEVLOG - Arena Survivor

---

## Entry 0: 6th March 2026

- **Goal:** To build a live scalable multiplayer arena game with AI Bots, and get placed on a good company by November 2026.

- **Current Status:** I know basic C++ and basic Unity, I've never build a multiplayer game, never used networking, never deployed anything to the cloud. I've made 3-4 small games in Unity over the span of last 4 years.

- **My Fear:** Like always I fear my momentum getting lost after a hard start. I fear this project would get too complex and I'll abonden it. I fear that I've already wasted so much time and have very little time to catch up.

- **My Resolve** I've been deciding since 6th standard about my future, now the deciding is done. Starting today I am someone who doesn't wait to feel ready, every single day I'll open the editor, one problem, one commit, one devlog. I've already wasted a decade wanting to feel ready now the resolve is made. See myself tomorrow!

---

## DEVLOG ENTRY 1 — Week 1: Unity 2D Arena Core
- **Date:** [13/03/2026]
- **Status:** ✅ Complete

---

### What I Built
- Tilemap arena: 20×20 floor with perimeter walls and Composite Collider 2D
- Player movement: New Input System + Rigidbody2D.MovePosition in FixedUpdate
- Shooting: ObjectPool — 0 GC allocations at runtime
- Enemy chase: Vector2.MoveTowards toward player per FixedUpdate
- Health system: reusable Health.cs with C# float health, events subscribed by UI slider and WaveManager
- Wave system: 3 waves with increasing enemy count and speed, win/lose conditions

---

### What Broke

**Bug 1 — Tile sprites wrong size on Tilemap**
Placed tile sprites on the Tilemap but they didn't fit the grid cells — appeared too small or overflowing.
- **Cause:** Pixels Per Unit value on the sprite didn't match the tile size Unity expected.
- **Fix:** Selected sprite in Project panel → Inspector → adjusted Pixels Per Unit until it fit correctly.

---

**Bug 2 — Old Input System deprecation warning**
Console kept showing warnings about the old Input System being discontinued mid-build on Day 2.
- **Fix:** Switched to the New Input System via Project Settings → enabled the package → editor restarted automatically → rewrote movement input for the new system. No major issues during the switch.

---

**Bug 3 — GameObjects invisible, rendering behind Tilemap (recurring)**
Affected player, enemies, bullets, and spawn points across multiple days. Objects existed in the scene but were invisible in the Game view.
- **Cause:** All GameObjects had Z = 0. The Tilemap also sits at Z = 0 and renders on top of everything at the same depth.
- **Fix:** Set Z = -1 on every GameObject and all 4 spawn points. Enemies instantiated at spawn points automatically inherit Z = -1.
- **Note:** I was aware of Sorting Layers before this but had never used them. This was the first time I understood why they exist — they are the proper long-term solution to this problem.

---

**Bug 4 — Enemy movement jitter on player collision**
Enemies vibrated rapidly when touching the player instead of pushing smoothly.
- **Cause:** EnemyChase was using Transform.position which teleports the object directly, bypassing the physics engine. Every frame the physics engine pushed the objects apart, then Transform.position immediately teleported the enemy back into the collision.
- **Fix:** Switched to Rigidbody2D.MovePosition which moves through the physics engine and respects collision resolution properly.

---

**Bug 5 — Health bar not updating when player takes damage**
*(Day 5 — hardest bug of the week)*
Health bar stayed full even while enemies were continuously colliding with the player.

Debugging process:
1. Suspected collider first — added `Debug.Log` inside `OnTriggerStay2D`, confirmed it was firing correctly. Collision was not the problem.
2. Suspected damage logic — traced through code, found damage value was `0.2f` per frame.
3. Found root cause: Health was stored as `int`. Casting `0.2f` to `int` truncates to `0`, so every damage call was `health -= 0`. Nothing was changing.
4. First attempted fix: damage accumulator variable — collect fractional damage until it reached `1f` then apply as whole number. This worked but introduced a second bug: forgot to reset the accumulator to `0` after applying, causing damage to compound incorrectly.
5. Final fix: changed health field from `int` to `float`. Original damage code worked perfectly with no other changes needed.

---

**Bug 6 — Wave system running but enemies invisible on spawn**
*(Day 6)*
Wave system appeared to not be working — no enemies visible after pressing Play.

Debugging process:
1. No Console errors — script failure ruled out.
2. Read through WaveManager code and traced execution flow manually several times — logic was correct.
3. Eventually noticed enemies were spawning but invisible.
- **Cause:** All 4 spawn points had Z = 0 — same Z depth issue as Bug 3. Enemies were spawning behind the Tilemap.
- **Fix:** Set Z = -1 on all 4 spawn point Transforms. Enemies now spawn at the correct depth automatically.

---

**Bug 7 — Player death not triggering lose condition**
Player dying did not show the lose panel or change game state.
- **Cause:** `OnPlayerDied` method existed in WaveManager with correct logic, but nothing had subscribed it to the player's `OnDeath` event. The method was never called.
- **Fix:** Added one line to Player's `Start()`:
```csharp
GetComponent().OnDeath += FindObjectOfType().OnPlayerDied;
```
- **Scene restart:** Added a 2-second delay coroutine after both win and lose conditions. `SceneManager.LoadScene` reloads the active scene after the panel is visible for 2 seconds. Works correctly for both outcomes.

---

### Key Technical Insight

Float-to-int casting silently truncates — it does not round. `0.9f` becomes `0`, not `1`. When a system is not responding to changes, check your data types before assuming the logic is wrong. Changing one field from `int` to `float` eliminated a bug that had spawned two incorrect fix attempts before the root cause was found.

Secondary insight: In Unity 2D, Z coordinate controls render order when Sorting Layers are not configured. The Tilemap at Z = 0 renders on top of everything else also at Z = 0. This affected every single GameObject in the project and had to be fixed individually until a proper Sorting Layer setup replaces it.

---

### Next Week
Build the C++ A* pathfinding plugin. Move enemy navigation off `Vector2.MoveTowards` entirely. Expected outcome: enemies navigate around walls instead of walking through them.
