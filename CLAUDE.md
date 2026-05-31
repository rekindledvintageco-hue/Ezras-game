# Dragon Tamer — Ezra's Game

A 3D Pokémon-inspired dragon-catching adventure built for Ezra (age 10).
**Single-file game: everything lives in `index.html`** — HTML, CSS, and JS all inlined.
Three.js (3D library) is loaded from CDN only.

## Live site
https://rekindledvintageco-hue.github.io/Ezras-game/

Hard-refresh with **Cmd/Ctrl + Shift + R** to bypass cache after a deploy.

## Repo / branches
- Repo: `rekindledvintageco-hue/Ezras-game`
- Working branch: `claude/setup-ezras-game-YaeSi` — all development goes here
- `gh-pages` branch — mirrors working branch for GitHub Pages hosting

## Deployment workflow (ALWAYS do this after committing)
```bash
git push -u origin claude/setup-ezras-game-YaeSi
git checkout gh-pages && git merge claude/setup-ezras-game-YaeSi && git push
git checkout claude/setup-ezras-game-YaeSi
```

## File structure
- `index.html` — the entire game (~4400 lines)
  - Lines 1–425: HTML structure + overlay panels
  - Lines 7–425: Inlined CSS
  - Lines ~570–2740: Game-core JS (constants, state, battle, save, input, hero)
  - Lines ~2745–end: 3D renderer JS (Three.js cel-shaded world)
- `CLAUDE.md` — this doc

---

## Visual style
- **Cel-shaded toon look:** `THREE.MeshToonMaterial` with a 4-band black-to-white gradient map (`createGradientTexture`) for that Pokémon-anime feel.
- **Sky:** `0x87ceeb` background, fog 40–120, 22 floating cloud puffs.
- **Lighting:** Hemisphere light (warm sky / green ground) + a strong directional sun from above-right.
- **Color palette uses dragon `color` + `accent` per species.**
  - `lightenHex(hex, amount)` helper exists for belly/trim shades (positive = lighter, negative = darker).
- **2D UI** (HUD, panels) uses purple-on-cream theming with orange/gold accents.

---

## 9 dragon species
All dragons can fly (`canFly: true`) and be ridden (`canRide: true`). Press **F** to fly an active dragon, **M** to mount.

| ID | Name | Type | Color / Accent | HP | ATK | Power | Notes |
|---|---|---|---|---|---|---|---|
| `ember` | Ember Drake | fire | orange / yellow | 35 | 12 | **Inferno Blast** (1.85x, fire) | 🔥 Fire-breather |
| `frost` | Frost Whelp | ice | sky-blue / pale | 40 | 10 | Blizzard Shard (1.55x, may freeze) | |
| `leaf` | Leaf Serpent | nature | green / pale-green | 42 | 9 | Vine Renewal (heals 35%) | |
| `storm` | Storm Raptor | storm | purple / lavender | 32 | 14 | **Fire Breath** (1.7x) | 🔥 Fire-breather |
| `sky` | Sky Wyrm | sky | cyan / white | 38 | 11 | Gale Force (1.65x pierce) | |
| `shadow` | Shadow Fang | dark | indigo / purple | 36 | 13 | Night Pulse (1.6x drain) | |
| `thunderWing` | Thunderwing | storm | gold / purple | 34 | 14 | **Fire Breath** (1.7x) | 🔥 Glows gold. Fire-breather |
| `rainbow` | Rainbow Drake | sky | pink / cyan | 36 | 10 | **Fire Breath** (1.7x) | 🔥 Glows purple. Fire-breather |
| `fininator` | **Fininator** | ice | red / blue | 40 | 13 | Fin Slash (1.75x) | Red+blue Mega-Charizard-X style with razor side-fins and **blue flames flickering from mouth corners** |

**Powers** (`POWERS` dict): `inferno`, `blizzard`, `vineHeal`, `thunder` (kept for archive), `gale`, `nightPulse`, `boltDive` (archive), `prismBeam` (archive), `fireBreath`, `finSlash`. Powers have `mult`, `effect` ("damage" | "heal" | "drain"), and optional `pierce`, `recoil`, `status: "freeze"`, `statusChance`, `healRatio`, `drainRatio`.

**Type chart** (`typeMultiplier`):
- fire → ice 1.5, nature 1.5, water 0.5
- ice → nature 1.5, fire 0.5
- nature → storm 1.5, fire 0.5
- storm → sky 1.5, nature 0.5
- sky → nature 1.5, storm 0.5
- dark → sky 1.5, nature 1.5

### Dragon 3D model (`buildDragonMesh`)
~70 meshes per dragon (Pokémon-mega-Charizard-style):
- Body (egg) + lighter belly underside
- Head with snout, jaw, two **backward-pointing horns**, brow ridges, white eyes + black pupils, **two fangs**
- **Two long curving whisker feelers** off each cheek (3-segment cylinders sweeping back)
- **Cream mane** (5 fluffy cone tufts down the spine) + 3 small dark spikes
- **Angular bat wings** per side: wing-arm bone + 3 finger struts + 3 membrane panels. Bigger/darker for flyers.
- Two arms (cylinders) with 3 white hand-claws each
- Two legs (cylinders) + feet (boxes) with 3 white toe-claws each
- 4-segment tapering tail ending in a **4-cone fluffy flame tuft** (orange for fire-type, cream otherwise)
- **Species-specific extras:**
  - `thunderWing`: gold glow
  - `rainbow`: purple glow
  - `fininator`: teal/red glow + **razor side-fins + tail-fins** on both flanks + **animated blue mouth-corner flames** (`userData.isFinFire`, flickered every frame in the animate loop)

---

## Hero (player character)
Customizable via **C key** or "🎨 Customize character" on start screen.

### Fields (`state.hero`)
`skin, hair, shirt, pants, hairStyle ("short" | "spiky" | "long"), hat ("none" | "cap" | "crown")`.

Pokémon-trainer-style 3D model (`buildHeroMesh`, ~26–38 meshes):
- 2 visible legs (cylinders) + shoes (boxes, darker than pants)
- Belt + gold buckle
- Torso with V-neck collar trim
- 2 arms angled out + darker cuffs
- Hands (skin spheres)
- Neck + head with small ears
- Per-hair-style detail: short = front fringe; spiky = 5 radiating cone tufts; long = back-hair box + side bangs
- Cap = box + brim + white button; crown = circlet + 6 spikes + center red jewel
- White-of-eye + pupils + small smile mouth

`updateHeroColors` rebuilds the hero from `state.hero` on every customize change.
`drawHero` (2D canvas) for the customize-panel preview is unchanged.

---

## World map
- **Size: 95 × 55 tiles** (was 80×45 originally). `MAP_W` and `MAP_H` constants.
- `TILE = 32` pixels per tile, `UNIT = 2.5` Three.js units per tile.
- Tile types: `0=grass, 1=path, 2=water, 3=tree (choppable), 4=tall grass (encounter zone), 5=house, 6=village, 7=pet grove, 8=weapon shrine`.

### Features
- Player **home** at tiles (7–8, 21–22) — enter with E.
- **Village zone** (62–74, 15–29) — far east. Press V here to enter village build mode.
- **Pet grove** (33–47, 35–40) — press P near it to befriend a mini-pet for active dragon.
- **Weapon shrine** (36–44, 5–9) — press B near it for a random weapon.
- **Central lake** (radius around 42,22), smaller southwest pond, bridge across (38–45, 22–23).
- **Forests** (`tile 3` = choppable trees):
  - Northern forest (8–30, 4–13) — denser (75% kept)
  - Southern deep woods (12–58, 42–50)
  - South-east forest (76–92, 38–52)
  - Far-east woods (78–92, 5–14)
  - Western birch grove (4–11, 25–40)
  - Random scattered (8% per tile)
- **Tall grass** patches for wild battles in multiple spots — see `setRect(..., 4, false)` calls in `generateMap`.

### Wild dragons
- `MAX_WILD_DRAGONS = 65`. Stored in `wildOverworld` array.
- Spawn on tile 4 (tall grass) and rarely on tile 0 (grass) at game-start.
- Wander randomly; bob via `d.bob`.
- Walking near one in **overworld** mode triggers a battle (`checkWildDragonTouch`, range 30px).
- Defeated/tamed dragons are removed; respawn timer (~8s) refills empty tall-grass tiles up to MAX.
- 3D meshes synced via `syncWildDragons` (creates/removes dragon meshes from `wildGroup`).

---

## Wood, cabins, and the village
- `state.wood` (number), `state.cabins` array of `{type, tx, ty}`.
- Press **T** near a tree to chop it: 5–10 wood per tree, the 3D tree mesh is removed instantly via `onTreeChopped` callback.

### Village mode
- Press **V** while standing on village zone (tile type 6) to enter `state.mode = "village"`.
- Auto-exits village mode when you walk out of the zone (so you can battle dragons again).
- In village mode:
  - **B** = build Small Cabin (50 wood)
  - **N** = build Big Cabin (100 wood)
  - **H** = dismantle nearest cabin (50% wood refund)
  - **E** near a cabin = enter it
- Cabins are **rebuilt in 3D** via `syncVillageCabins()` triggered by `onCabinBuilt` callback. Small = single-story log cabin with peaked roof, door, windows. Big = two-story log cabin with a gold sign.

### Cabin interior (walkable!)
- `state.mode = "cabin"`, `state.cabinInterior` in `"small" | "big" | "upstairs"`.
- Player can walk around inside (12×8 small, 14×10 big, 12×8 upstairs).
- 3D scene: floor planks, wooden walls, ceiling beams, exit door, small chair.
- **Villager NPC** (green robe + wizard hat + white beard) with a floating gold sparkle above (animated `userData.isSparkle`). Stand near him and press **E** to open the trade overlay.
- **Stairs** (big cabin only) with an animated **gold "UP" arrow** floating + spinning above (`userData.isStairsMarker`). Walking onto the staircase auto-triggers floor switch — `state.cabinStairsCooldown` (35 frames) blocks instant re-trigger bouncing.
- Upstairs has a flipped "DOWN" arrow and visible **trophies** that spawn based on dragon count:
  - 1 dragon → Dragon Rug (red+gold floor mat)
  - 2 → Fireplace (stone with cone flame)
  - 3 → Dragon Throne (gold seat with 2 horns)
  - 4 → Dragon Banner (flagpole + flag)
  - 5 → Crystal Den (4 cyan crystal cones + purple orb)

### Cabin trade overlay (`renderCabinPanel`)
Opens when E is pressed near the villager. **Pressing "Done Talking" closes the panel but you stay in cabin mode** so you can keep walking around.
1. **Trade Dragon → 20 Wood** — can't trade your last dragon.
2. **Clothes Shop**: 5 outfits, each gives a battle bonus.
3. **Trophy summary** (big cabin only): list of unlocked decorations.

### Outfits (`OUTFITS`)
| ID | Name | Cost | Shirt/Pants/Hat | Bonus |
|---|---|---|---|---|
| `adventurer` | Adventurer's Vest | 30 🪵 | green / brown / none | +2 ATK |
| `wizard` | Wizard Robes | 50 🪵 | dark-blue / black / crown | +2 ATK, +10% tame |
| `knight` | Knight Armor | 60 🪵 | silver / gray / cap | +3 ATK |
| `royal` | Royal Robes | 80 🪵 | purple / dark-purple / crown | +4 ATK, +8% tame |
| `dragonRider` | Dragon Rider Suit | 120 🪵 | red / dark-red / cap | +6 ATK, +20% tame |

`state.outfit` (id), `state.outfitBag` (owned ids). `applyOutfit()` updates `state.hero.shirt/pants/hat` and re-renders the 3D model via `onHeroChanged`. `getOutfitBonus()` adds `atkBonus` to `calcDamage` and `tameBonus` to tame chance.

---

## Weapons (`WEAPONS`)
7 weapons + the starter `dragonDagger`. Active dragon uses it in battle via the **Weapon** action button.

| ID | Name | Flat | Mult | Active mult | Special |
|---|---|---|---|---|---|
| `dragonDagger` | Dragon Dagger 🗡️ | +4 | 1.0 | 1.45 | +3% tame |
| `flameSword` | Flame Sword 🔥 | +6 | 1.08 | 1.75 | +35% vs ice |
| `iceSpear` | Ice Spear 🧊 | +5 | 1.05 | 1.5 | 22% freeze chance |
| `natureStaff` | Nature Staff 🌿 | +3 | 1.0 | 1.2 | Heals +6 on attack |
| `stormHammer` | Storm Hammer ⚡ | +8 | 1.1 | 2.0 | Power CD −1 |
| `skyBow` | Sky Bow 🏹 | +7 | 1.12 | 1.65 | +20% with sky dragons |
| `shadowBlade` | Shadow Blade 🌑 | +6 | 1.05 | 1.6 | +12% tame |

`state.weaponBag`, `state.equippedWeaponId`. Open inventory with **I**. Weapons drop with 20% chance after battle wins, or visit the Weapon Shrine.

---

## Mini-pets (`MINI_PETS`)
Each dragon can have one. They appear:
- As a 3D model floating beside you when riding/flying (bobs and spins — `mountPetMesh`).
- In battles as a "Pet Help" action button.

| ID | Name | Emoji | Bonus | Value |
|---|---|---|---|---|
| `sparkMouse` | Spark Mouse | 🐭 | damage | +8 |
| `snowBunny` | Snow Bunny | 🐰 | shield | 20% |
| `mossFrog` | Moss Frog | 🐸 | heal | +14 HP |
| `buzzBeetle` | Buzz Beetle | 🪲 | powerBoost | 1.25x |
| `cloudMoth` | Cloud Moth | 🦋 | damage | +10 |
| `shadeBat` | Shade Bat | 🦇 | lifesteal | 25% |

`DRAGON_PET_AFFINITY` maps each species to its "natural" pet (revealed at the Pet Grove). 22% chance per battle win to find a random mini-pet, stored in `state.miniPetBag` until assigned with **P**.

---

## Battle system
- `state.mode = "battle"`, `state.battle = { busy, powerCd, petCd, weaponCd, playerShield, wildFrozen }`, `state.wild` (the enemy dragon object).
- Triggered by `checkWildDragonTouch` (overworld mode only). On end, `state.encounterCooldown` is set so dragons don't instantly re-engage.
- **3D battle scene** (`battleGroup`): green circular floor + gold ring, player dragon at (-3.5, 0, 0) facing right, wild dragon at (3.5, 0, 0) facing left. Camera at (0, 5, 12) looking at (0, 1, 0). Dragons bob up and down in `animate`.
- **Battle UI** (now a bottom-bar overlay, transparent background, max-width 720px) with the 3D dragons visible above. 2D dragon canvas sprites are hidden via CSS. Shows: panel header, HP bars (player left, wild right), power/pet info, battle log, action buttons.

### Action buttons
- **Attack** (basic damage)
- **Power** ⚡ (dragon's signature move; cooldown 3 - weapon powerCdReduce)
- **Pet Help** 🐾 (uses mini-pet's bonus: damage, shield, heal, lifesteal, powerBoost — cd 3)
- **Weapon** 🗡️ (active strike, cd 2; freeze chance applies)
- **Tame** 🪢 (chance = (1−hpRatio)×0.55 + 0.15 + petBonus + weapon.tameBonus + outfit.tameBonus)
- **Run** 🏃 (85% chance to escape)

### Damage calc (`calcDamage`)
```
mult = typeMultiplier(attacker.type, defender.type)  // pierced if pierce
base = attacker.atk * (0.85 + Math.random()*0.3) * powerMult
if player attacking: base = base * weapon.atkMult + weapon.atkFlat + outfit.atkBonus
damage = floor(base * mult), min 1
```

### Switching dragons
- Active dragon switch via **1/2/3** keys (overworld only).
- On faint, auto-switches to next healthy party member. If none, party heals and battle ends.

---

## State object
```js
state = {
  mode: "overworld" | "home" | "village" | "battle" | "cabin",
  player: { x, y, w:20, h:24, speed:3, dir:"down" },
  mounted: false,           flying: false,            flyTimer: 0,
  party: [...dragons],      activePet: 0,
  wild: null,               battle: null,
  decorTool: "bed",         hammerMode: false,
  villageGrid: [],          cabins: [{type, tx, ty}],
  wood: 0,
  cabinInterior: null,      // "small" | "big" | "upstairs" when inside
  currentCabin: null,       // {type,tx,ty} of the cabin we entered
  cabinStairsCooldown: 0,
  outfit: null,             outfitBag: [],
  homeDecor: [],            miniPetBag: [],
  hero: { skin, hair, shirt, pants, hairStyle, hat },
  weaponBag: [],            equippedWeaponId: null,
  hasStarterWeapon: false,
  uiLock: false,            encounterCooldown: 0,    encounterDragonId: null,
  camera: { x:0, y:0 },
}
```

Saved to localStorage under key `"dragonTamerSave"` on every change.

**`normalizeDragon` refreshes species-level fields** (canFly, canRide, type, color, accent, powerId, name) from the current DRAGONS table on every save load — so old saves automatically pick up new abilities (e.g., the canFly-everyone update or the Fire Breath power swap).

---

## Controls reference
| Key | Action |
|---|---|
| WASD / arrows | Move |
| `[` `]` | Rotate camera |
| **T** | Chop nearby tree for 5–10 🪵 wood |
| **E** | Enter home / enter cabin / talk to villager / exit |
| **V** | Toggle village mode (must be in village zone) |
| **F** | Fly (any dragon — they all fly now) |
| **M** | Mount / dismount active dragon |
| **1 / 2 / 3** | Switch active dragon |
| **P** | Assign mini-pet (at Pet Grove or after wins) |
| **C** | Customize hero |
| **I** | Weapons inventory |
| **B** | Weapon Shrine (in overworld near shrine), build Small Cabin (in village) |
| **N** | Build Big Cabin (in village) |
| **H** | Dismantle nearest cabin (in village) |

---

## Architecture notes
- **`window.GameCore` is the bridge** between game-core logic and the 3D renderer. Renderer reads from it and hooks back via callbacks.
- **3D <-> game-core callbacks:**
  - `onGameStart` — fires when start screen dismissed
  - `onHeroChanged` — rebuilds hero mesh
  - `onTreeChopped(tx, ty)` — removes tree mesh from `worldGroup`
  - `onCabinBuilt` — runs `syncVillageCabins()`
  - `onCabinEnter` — builds cabin interior, switches scene
  - `onCabinExit` — clears interior, returns to outdoor scene
  - `onBattleStart` — switches to battle scene
  - `onBattleEnd` — returns to world scene
- **`isPlaying()`** returns true when start screen IS hidden (game running). Animate loop early-returns if not playing.
- **`updateSceneMode()`** centralizes which Three.js groups are visible per `state.mode`. Worldgroup + wildGroup + cabinGroup visible only outdoors.
- **Animation tags** (`userData.{isSparkle, isStairsMarker, isFinFire, isWing, ...}`) — the animate loop sweeps these to apply per-frame transforms.

---

## Bugs already fixed (don't re-introduce)
- `isPlaying()` was inverted — fixed: returns `startScreen.classList.contains("hidden")`.
- V key instantly exited village mode — fixed: uses `else if` not a second `if`.
- E key bounced home/overworld same frame — fixed: `if / else if` chain in input handler.
- Village blue screen — fixed: `worldGroup` is visible when `mode === "village"`.
- Player couldn't move in village mode — fixed: movement block includes "village".
- Exiting cabin left mode in "village", so wild-dragon battles wouldn't trigger — fixed: `exitCabin()` sets mode to "overworld" + small encounterCooldown.
- Walking out of village zone left mode stuck in village — fixed: auto-exit when out of zone.
- Stairs interaction radius too tight, players walked through stairs — fixed: `nearCabinStairs` uses maxDist=2, **walking onto stairs auto-triggers floor switch** (no E required), with `cabinStairsCooldown` to block bouncing.
- Stale localStorage saves had `canFly: false` baked in — fixed: `normalizeDragon` refreshes species-level fields from DRAGONS table.

---

## Testing
- Local dev: `python3 -m http.server 8765` from `/home/user/Ezras-game`, open `http://localhost:8765/index.html`.
- For headless verification with Playwright, the Three.js CDN may be blocked in sandboxed envs — stub `window.THREE` via `addInitScript` so game-core can run. Game logic (input, save load, mode transitions, mesh construction) is testable; actual rendering requires a real browser.

---

## Conventions in this codebase
- Single-file. **Always edit `index.html` only.** No new files unless explicitly requested.
- Commit messages: subject line first, blank line, then bulleted body. Include `https://claude.ai/code/...` URL at the bottom (the harness fills it in).
- Be concise. No comments unless the *why* is non-obvious. No "Added on YYYY-MM-DD" or "Used by X" comments.
- Don't add backwards-compat shims when you can just change the code (e.g., delete unused powers from `POWERS` dict if needed).
- When user sends references / screenshots, match them: the dragon model came directly from Mega Charizard X/Y refs; the hero came from "Pokémon Sword/Shield trainer" vibes.
