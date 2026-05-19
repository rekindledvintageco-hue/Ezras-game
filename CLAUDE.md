# Dragon Tamer — Ezra's Game

## What this is
A 3D Pokémon-inspired dragon-catching adventure built for Ezra (age 10).
Single-file game: **everything lives in `index.html`** — HTML, CSS, and JS all inlined.
Three.js (3D library) is loaded from CDN only.

## File structure
- `index.html` — the entire game (one file, ~3000 lines)
  - Lines 1–145: HTML structure
  - Lines 146–580: CSS (style)
  - Lines 581–2420: Game logic (game-core.js) — map, battle, save, hero
  - Lines 2421–3000: 3D renderer (game3d.js) — Three.js cel-shaded world

## How to test
- Download `index.html` from GitHub and double-click it (needs internet for Three.js CDN)
- Or visit the GitHub Pages URL once enabled (see below)

## Git setup
- Repo: `rekindledvintageco-hue/Ezras-game`
- Working branch: `claude/setup-ezras-game-YaeSi` — all development goes here
- `gh-pages` branch — mirrors working branch for GitHub Pages hosting
- After changes: commit to working branch, then update gh-pages

## Deployment workflow
1. Make changes → commit → push to `claude/setup-ezras-game-YaeSi`
2. Run: `git checkout gh-pages && git merge claude/setup-ezras-game-YaeSi && git push && git checkout claude/setup-ezras-game-YaeSi`
3. GitHub Pages auto-serves the updated game

## GitHub Pages setup (one-time, done by user)
1. Go to github.com/rekindledvintageco-hue/Ezras-game
2. Settings → Pages → Source: Deploy from branch
3. Branch: `gh-pages` / folder: `/ (root)` → Save
4. URL will be: `https://rekindledvintageco-hue.github.io/Ezras-game/`

## Game features (already built)
- 3D cel-shaded overworld (80×45 tile map)
- 6 dragon types: Ember Drake, Frost Whelp, Leaf Serpent, Storm Raptor, Sky Wyrm, Shadow Fang
- Battle system: attack, power move, pet help, weapon strike, tame, run
- 7 weapons with unique effects
- 6 mini-pets with battle bonuses
- Home decorating, village building
- Hero customization (skin, hair, shirt, pants, hat)
- Flying (needs Sky Wyrm), riding, camera rotation
- Save/load via localStorage

## Controls
- WASD / arrows: move
- [ ] : rotate camera
- E: enter house | V: village | F: fly | M: mount
- 1–3: switch dragon | P: assign pet | C: customize | I: weapons | B: shrine

## Known architecture notes
- `window.GameCore` is the bridge between game-core logic and the 3D renderer
- `isPlaying()` in game3d section returns true when start screen IS hidden (game running)
- Battle UI uses 2D canvas sprites (not Three.js) overlaid on top of the 3D scene
