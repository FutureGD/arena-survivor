# Arena Survivor

> A scalable 2D arena game built to demonstrate production-grade game engineering.
> Currently in active development — single-player foundation complete, multiplayer networking in progress.

---

## Demo
[▶ Watch the 1.5 minute demo video](https://www.youtube.com/watch?v=LsjyyDT1vKk)

---

## Current State
| Feature | Status |
|---|---|
| Tilemap arena with wall collision | ✅ Complete |
| Player movement (WASD) | ✅ Complete |
| Shooting with object pooling | ✅ Complete |
| Enemy chase AI | ✅ Complete |
| Health system with UI | ✅ Complete |
| 3-wave system with win/lose conditions | ✅ Complete |
| C++ A* pathfinding plugin | 🔄 Week 2 — In Progress |
| Multiplayer networking (NGO + UGS) | 📅 Month 2 |
| Firebase Auth + Leaderboard | 📅 Month 3 |

---

## Tech Stack
| Layer | Technology | Why |
|---|---|---|
| Engine | Unity 6 LTS + C# | Industry standard, cross-platform |
| Input | Unity New Input System | Future-proof, multiplayer-compatible |
| Physics | Rigidbody2D.MovePosition | Physics engine compliant — required for NGO sync |
| Object Pooling | Unity ObjectPool\<T\> | Zero GC allocations at runtime |
| Tilemaps | Unity Tilemap + Composite Collider 2D | Efficient 2D level building |
| UI | Unity UI (Canvas + Slider) | Health bar subscribed to C# events |
| Version Control | Git + GitHub | Public portfolio from day 1 |

---

## Architecture
```
arena-survivor-client/
├── Assets/
│   ├── Input/
│   │   ├── Input_Actions.inputactions  ← New Input System action map (WASD + mouse)
│   │   └── Input_Actions.cs            ← Auto-generated C# bindings
│   │
│   ├── Materials/
│   │   └── FrictionLess.physicsMaterial2D  ← Applied to Player and Enemy Rigidbody2D — prevents sticking to walls during movement
│   │
│   ├── Prefabs/
│   │   ├── Bullet.prefab               ← Bullet — used by ObjectPool<T>
│   │   ├── DeathParticle.prefab        ← Particle burst on enemy/player death
│   │   └── Enemy.prefab                ← Enemy — EnemyChase + Health + EnemyDamage
│   │
│   ├── Scenes/
│   │   └── Arena.unity                 ← Main scene
│   │
│   ├── Scripts/
│   │   ├── PlayerMovement.cs           ← WASD via New Input System + Rigidbody2D.MovePosition
│   │   ├── Shooting.cs                 ← Mouse aim + ObjectPool<T> bullet firing
│   │   ├── Bullet.cs                   ← OnTriggerEnter2D + return to pool
│   │   ├── EnemyChase.cs               ← Rigidbody2D.MovePosition toward player (A* replaces this Week 2)
│   │   ├── EnemyDamage.cs              ← OnTriggerStay2D → TakeDamage per second
│   │   ├── Health.cs                   ← Reusable component — float health + C# events
│   │   ├── HealthBarUI.cs              ← Subscribes to OnHealthChanged → updates UI slider
│   │   ├── Death.cs                    ← Handles death particle instantiation on OnDeath
│   │   ├── WaveData.cs                 ← [Serializable] struct — enemyCount, spawnInterval, enemySpeed
│   │   └── WaveManager.cs              ← State machine + coroutine spawner + win/lose logic
│   │
│   └── Sprites/
│       ├── Player.png                  ← Player sprite
│       ├── floor.png                   ← Floor tile
│       ├── wallMiddle.png              ← Wall tile (straight)
│       └── wallCorner.png              ← Wall tile (corner)
```

---

## How It Works

### Health System
`Health.cs` is a reusable component attached to both the player and enemies. It exposes two C# events:
```csharp
public event Action<float, float> OnHealthChanged;  // subscribed by HealthBarUI
public event Action OnDeath;                         // subscribed by WaveManager + Death.cs
```

Any script that needs to react to health changes subscribes to these events — the Health component itself does not know or care who is listening.

### Wave System
`WaveManager.cs` runs a coroutine that spawns enemies one at a time with a configurable delay between each. The state machine has four states:
```
BetweenWaves → WaveInProgress → BetweenWaves → ... → Victory
                                              ↘ GameOver (if player dies)
```

Each wave is configured via a `[System.Serializable]` WaveData struct — editable directly in the Unity Inspector without touching code.

### Object Pooling
Bullets use Unity's built-in `ObjectPool<T>`. The pool pre-allocates 20 bullets at scene start. No `Instantiate()` or `Destroy()` calls happen at runtime — bullets are fetched from and returned to the pool, producing zero heap allocations per shot.

---

## Performance
| System | Approach | Result |
|---|---|---|
| Bullet spawning | ObjectPool\<T\> | 0 GC allocations at runtime |
| Enemy movement | Rigidbody2D.MovePosition | Smooth collision, physics-compliant |
| Enemy pathfinding | Vector2.MoveTowards (current) | No wall avoidance — A* plugin replaces this in Week 2 |

---

## How to Run Locally

### Prerequisites
- Unity Hub
- Unity 6.3 LTS (download via Unity Hub)
- Git

### Steps
```bash
# 1. Clone the repository
git clone https://github.com/FutureGD/arena-survivor-client.git

# 2. Open Unity Hub → Add → select the cloned folder

# 3. Open the Arena scene
#    Assets → Scenes → Arena.unity

# 4. Press Play
```

---

## Repository Map
| Repo | Contents | Status |
|---|---|---|
| [arena-survivor](https://github.com/FutureGD/arena-survivor) | This overview + DEVLOG + architecture docs | ✅ Active |
| [arena-survivor-client](https://github.com/FutureGD/arena-survivor-client) | Unity 2D game client | ✅ Active |
| [arena-survivor-ai-plugin](https://github.com/FutureGD/arena-survivor-ai-plugin) | C++ A* native plugin (CMake) | 🔄 Week 2 |
| [arena-survivor-networking](https://github.com/FutureGD/arena-survivor-networking) | NGO + UGS networking code | 📅 Month 2 |
| [arena-survivor-backend](https://github.com/FutureGD/arena-survivor-backend) | Firebase scripts + Cloud Functions | 📅 Month 3 |
| [arena-survivor-dsa](https://github.com/FutureGD/arena-survivor-dsa) | Standalone A* + scaling benchmarks | 📅 Month 4 |

---

## DEVLOG
[Read the full engineering log →](DEVLOG.md)

Week 1 documents 7 bugs including a float-to-int truncation issue that silently zeroed all damage values, and a recurring Z-depth rendering problem that affected every GameObject in the project.

---

## Roadmap
- **Month 1** — Single-player foundation + C++ A* plugin ← *you are here*
- **Month 2** — Multiplayer via Netcode for GameObjects + Unity Gaming Services
- **Month 3** — Firebase Auth + persistent leaderboard
- **Month 4** — Scaling report: system design at 1,000 concurrent players
- **Month 5–8** — Polish, CI/CD, load testing, final portfolio packaging

---
