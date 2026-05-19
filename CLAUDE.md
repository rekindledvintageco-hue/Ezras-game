# Dragon Tamer — Ezra's Game

## What this is
A 3D Pokémon-inspired dragon-catching adventure built for Ezra (age 10).
Single-file game: **everything lives in `index.html`** — HTML, CSS, and JS all inlined.
Three.js (3D library) is loaded from CDN only.

## File structure
- `index.html` — the entire game (one file, ~3500 lines)
  - Lines 1–145: HTML structure + overlay panels
  - Lines 146–580: CSS (style)
  - Lines 581–end of first `</script>`: Game logic — map, battle, save, hero, village, cabins
  - Second `<script>` block: 3D renderer — Three.js cel-shaded world

## How to test
- Visit the live GitHub Pages URL: `https://rekindledvintageco-hue.github.io/Ezras-game/`
- Or download `index.html` from GitHub and double-click it (needs internet for Three.js CDN)

## Git setup
- Repo: `rekindledvintageco-hue/Ezras-game`
- Working branch: `claude/setup-ezras-game-YaeSi` — all development goes here
- `gh-pages` branch — mirrors working branch for GitHub Pages hosting
- After changes: commit to working branch, then update gh-pages

## Deployment workflow (ALWAYS do this after changes)
1. Make changes → commit → push to `claude/setup-ezras-game-YaeSi`
2. Run: `git checkout gh-pages && git merge claude/setup-ezras-game-YaeSi && git push && git checkout claude/setup-ezras-game-YaeSi`
3. GitHub Pages auto-serves the updated game at the URL above

## Game features (all built — do NOT rebuild these)
- 3D cel-shaded overworld (80×45 tile map) — toon materials, clouds, fog
- **8 dragon species:** Ember Drake, Frost Whelp, Leaf Serpent, Storm Raptor, Sky Wyrm, Shadow Fang, **Thunderwing** (gold/purple, flies), **Rainbow Drake** (pink/cyan, flies)
- Battle system: attack, power move, pet help, weapon strike, tame, run
- 7 weapons with unique effects (dragon dagger, flame sword, ice spear, etc.)
- 6 mini-pets with battle bonuses (spark mouse, snow bunny, moss frog, etc.)
- **Mini-pet appears in 3D** floating beside the dragon when player is riding or flying — bobs and spins
- Hero customization (skin, hair, shirt, pants, hat) — reflected in 3D
- Flying (Sky Wyrm, Thunderwing, or Rainbow Drake) — press F
- Riding any dragon — press M. Camera distance increases when flying.
- Save/load via localStorage

## Village system (fully rebuilt)
- Village zone: far east of map, tile type 6 (x:62–74, y:15–29)
- **Wood resource** (`state.wood`) — chop trees with **T** key, get 5–10 wood per tree
- Tree mesh is removed from 3D scene instantly when chopped
- **Small Cabin** costs 50 wood (B key in village mode) — log cabin 3D mesh, door, windows, peaked roof
- **Big Cabin** costs 100 wood (N key in village mode) — two-story log cabin with sign
- **H key in village** dismantles nearest cabin (refunds 50% wood)
- **E key near cabin** enters it — opens overlay panel
- **Inside Small Cabin:** trade a dragon for 20 wood each (can't trade last dragon)
- **Inside Big Cabin:** dragon trading + Trophy Room upstairs (items unlock at 1/2/3/4/5 dragons)
- Cabin Trophy Room items: Dragon Rug, Fireplace, Dragon Throne, Dragon Banner, Crystal Den
- Village mode also allows normal player movement (tile collision)
- `state.cabins` array stores `{type, tx, ty}` for each placed cabin
- `syncVillageCabins()` in 3D renderer rebuilds cabin meshes from state

## Controls
- WASD / arrows: move
- [ ] : rotate camera
- **T**: chop nearby tree for wood 🪵
- **E**: enter house / enter nearby cabin / exit home
- **V**: toggle village mode (must be standing in village zone)
- **F**: fly (needs Sky Wyrm, Thunderwing, or Rainbow Drake)
- **M**: mount/dismount dragon
- **1–3**: switch active dragon
- **P**: assign mini-pet (at Pet Grove south, or after winning battles)
- **C**: customize hero | **I**: weapons | **B**: weapon shrine (north)
- _In village mode:_ **B**=small cabin | **N**=big cabin | **H**=dismantle | **E**=enter cabin

## Key state object fields
```js
state = {
  mode: "overworld" | "home" | "village" | "battle",
  player: { x, y, w:20, h:24, speed:3, dir },
  party: [...dragons],   activePet: 0,
  wild: null,            battle: null,
  wood: 0,               cabins: [{type, tx, ty}],
  cabinInterior: null,   homeDecor: [],
  weaponBag: [],         equippedWeaponId: null,
  hero: { skin, hair, shirt, pants, hairStyle, hat },
  mounted: false,        flying: false,
}
```

## Map tile types
- 0=grass, 1=path, 2=water, 3=tree (choppable), 4=tall grass (encounters)
- 5=house (player's home), 6=village zone, 7=pet grove, 8=weapon shrine

## Known architecture notes
- `window.GameCore` is the bridge between game-core logic and the 3D renderer
- `isPlaying()` returns true when the start screen IS hidden (game running)
- Battle UI uses 2D canvas sprites (not Three.js) overlaid on the 3D scene
- `onTreeChopped(tx,ty)` callback removes tree mesh from worldGroup
- `onCabinBuilt()` callback calls `syncVillageCabins()` to refresh 3D cabins
- E-key entry uses proper `if / else if` chain (no same-frame double-fire bug)
- Village mode in `updateSceneMode()`: worldGroup + playerGroup + cabinGroup all visible

## Bugs already fixed (don't re-introduce)
- `isPlaying()` was inverted — fixed: returns `startScreen.classList.contains("hidden")`
- V key instantly exited village — fixed: uses `else if` not second `if`
- E key bounced home/overworld same frame — fixed: uses `if / else if` chain
- Village blue screen — fixed: worldGroup visible when mode === "village"
- Player couldn't move in village mode — fixed: movement block includes "village"
