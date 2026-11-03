# STORMFALL — build prompt

Paste everything below the line into an AI coding tool.

---

Build a complete, playable third-person battle royale in **one single `index.html` file** (HTML + CSS + one ES module script, no build step, no bundler, no servers). It must run by opening the file, on desktop and on a phone. Target roughly 8,000–9,000 lines. Do not ask me questions — make sensible choices and ship a finished, self-consistent game.

## Tech setup

- three.js r0.160.0 loaded through an import map from jsDelivr, e.g. `{"imports":{"three":"https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js"}}`.
- Everything else procedural: no external models, textures or audio files. Build all meshes from primitives and generate textures as canvas data URLs at boot.
- Fonts from Google Fonts: `Anton` (display), `Barlow Condensed` 600/700 (labels), `Barlow Semi Condensed` 500/600/700 (body).
- All sound is synthesized with the Web Audio API (oscillators + a noise buffer), no samples.
- Deck the whole thing in CSS custom properties for colours so it reads like a designed product, not a template.
- A boot watchdog: if the module has not started in ~9 seconds, show a "cannot connect" overlay.

## Visual identity

Dark navy/military UI, cinematic and angular. Absolutely no rounded cards, no purple-gradient "AI slop" look.

- Palette: background `#050b14`, panels `#0b1626`, ink `#eaf3ff`, accent `#4fc3ff`, gold `#ffc844`, danger `#ff4d5e`.
- Panels are cut with `clip-path` (angled corners), 1px hairline borders, subtle inner highlight, hard drop shadows.
- Labels: uppercase, small, heavy letter-spacing, `Barlow Condensed`. Headings: `Anton`, italic, tight tracking, gradient fills for the big ones.
- Every interactive element animates on hover/press; HUD elements bump and pop when their values change. Transitions are short (100–250 ms).
- Everything is tabular-numeric.

## World

- Analytic heightmap island, ~660 m across, plateau/beach/water falloff, vertex-coloured (wet sand, sand, three greens, rock, snow). One function `terrainHeight(x,z)` is the single source of truth for the ground, used by physics, placement, rays and the minimap.
- Water plane, seabed, fog, sky dome with a shader gradient, one directional sun with shadows that follows the player, hemisphere fill light.
- Props: ~200 pines, 80 rocks, 9 POI clusters with houses and shipping containers, ~12 lone houses. **Every house must be enterable** — doorways on two opposite walls, walls rotated with the building, colliders rotated to match.
- Chests with gold beams, scattered outdoor loot, and huge render-distance culling (only props within ~250 m are drawn; fog hides the rest).

## Movement & camera

- WASD/arrow movement, shift sprint, space jump with coyote time, step-up over low geometry, AABB collision resolution against a spatially hashed collider grid plus separate ramp surfaces.
- Spawn with a skydive drop: glide phase, fast fall, land with a camera dip and a "LANDED" toast.
- Third-person over-the-shoulder camera that pins the character to a **fixed point on screen** (~37% across, 63.5% down) by deriving shoulder/up offsets from FOV and aspect. Camera ray-clips against geometry so it never enters walls. Subtle head bob, breathing, landing dip and recoil.
- Full mouse-look via pointer lock with a fallback; touch: left thumbstick, drag-to-look on the canvas, and on-screen buttons for fire/jump/reload/build/use/pocket/inventory.

## Combat

- Hitscan. Bullets originate from the camera, apply spread as a cone, trace against world boxes, terrain (marching), shield bubbles and **capsule hitboxes**.
- Hitboxes: a vertical capsule (4 stacked spheres) covering legs + torso, plus a separate head sphere that takes priority when it is within ~0.18 m of the body hit — so headshots register reliably instead of being swallowed by the body. Same setup for the player.
- Headshot multipliers per weapon, rarity damage multipliers, shotgun pellet falloff that is gentle (never below 50%).
- **Siphon**: in ranked and tournament matches, every elimination restores 100 points of health, spilling into shield when health is full.
- Damage numbers: **cumulative per target** — repeated hits merge into one number that counts up, it re-pops on each hit, and it is **pinned in world space** just to the right of where the hit landed. Camera movement keeps it on the enemy; the enemy moving does not drag it along. Fades after ~1 s, max 8 at once.
- Taking damage: a metallic clang when shield absorbs, a dull thud when health takes it, a red/blue edge vignette, and up to 4 arc markers around the crosshair pointing to exactly where the shot came from (blue = shield, red = health).

## Weapons (use these numbers)

| key | name | ammo | dmg | rpm | mag | reload | spread | ads | pellets | hs | range | auto | notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| pistol | Pistol | light | 25 | 380 | 16 | 1.6 | 0.016 | 0.006 | 1 | 1.9 | 160 | yes | |
| ar | Assault Rifle | medium | 31 | 360 | 30 | 2.4 | 0.046 | 0.016 | 1 | 1.6 | 240 | yes | |
| scopedar | Scoped Assault Rifle | medium | 29 | 300 | 20 | 2.6 | 0.030 | 0.0001 | 1 | 1.7 | 320 | yes | 4x |
| redeye | Red-Eye Rifle | medium | 34 | 420 | 25 | 2.5 | 0.040 | 0.0001 | 1 | 1.7 | 260 | yes | 2x |
| tactical | Tactical Scoped Rifle | medium | 30 | 220 | 20 | 2.8 | 0.026 | 0.0001 | 1 | 1.8 | 330 | no | 4x |
| marksman | Marksman Rifle | medium | 46 | 150 | 12 | 3.0 | 0.030 | 0.0001 | 1 | 2.0 | 380 | no | 6x |
| revolver | Scoped Revolver | light | 62 | 110 | 6 | 2.6 | 0.016 | 0.0001 | 1 | 1.9 | 220 | yes | 2x |
| smg | SMG | light | 17 | 750 | 30 | 2.0 | 0.058 | 0.030 | 1 | 1.5 | 120 | yes | |
| shotgun | Pump Shotgun | shells | 13 | 80 | 5 | 3.1 | 0.062 | 0.048 | 10 | 1.75 | 62 | no | |
| sniper | Bolt Sniper | heavy | 105 | 40 | 1 | 2.8 | 0.052 | 0.0001 | 1 | 2.5 | 400 | no | 8x |
| heavyar | Heavy Assault Rifle | medium | 38 | 260 | 25 | 2.9 | 0.050 | 0.018 | 1 | 1.7 | 270 | yes | |
| doublebarrel | Double Barrel | shells | 12 | 150 | 2 | 2.6 | 0.070 | 0.055 | 8 | 1.7 | 48 | no | |
| railgun | Railgun | heavy | 150 | 22 | 1 | 3.6 | 0.030 | 0.0001 | 1 | 1.5 | 520 | no | 8x, rare |

- Scoped weapons (`scoped: true`): aiming pulls the camera into the optic, hides the player rig, fades in a real DOM scope overlay (lens mask, ring, crosshair reticle, zoom label), forces spread to near zero and cuts recoil to 15%.
- Every gun needs a hand-built blocky 3D model (body, barrel, mag, stock, optic where relevant) shown in the character's hands, with a muzzle flash and a weapon-switch dip animation.
- Loot pool weighted so scoped rifles are roughly 60% of gun drops.

## Rarity & loot

- 5 tiers: Common 1.00, Uncommon 1.05, Rare 1.10, Epic 1.17, Legendary 1.25 damage multipliers, each with its own colour (grey, green, blue, purple, gold).
- Floor loot is a **3D model** lying on the ground (guns, medkits, bandage rolls, shockwave balls, ammo boxes) with a rarity ring and a faint light beam.
- Ammo must be **plentiful**: ~26% of drops, 30–72 rounds each; chests give 45–90 rounds of the type the player's carried guns use; bot drops give 40–80; picking up a gun with under 24 reserve tops it up by 30; ammo auto-picks up within 3.2 m.
- Heals: shield potion (+50 shield, 2.6 s), medkit (+100 hp, 3.4 s), bandages (+15 hp capped at 75, 1.8 s). Quick-heal on `H` picks the right one.
- Shockwave grenade: launch impulse.
- Chests burst open with loot, ammo and wood.

## Pocket items (chosen in the lobby)

One slot, one charge, 15–24 s cooldown, bound to `F`, used by bots too:
1. **Pocket Shockwave** — launches you and everyone near the impact.
2. **Shield Bubble Jr** — a dome you can walk through but bullets cannot.
3. **Med Mist** — healing smoke that restores health and shield while you stand in it.

## Building

- `Q` toggles build mode; walls/floors/ramps on a 4 m grid, placed in front of the player, 10 wood each, wood from trees/chests, ghost preview turns red when unaffordable.
- Buildings are destructible colliders; ramps are walkable sloped surfaces.

## Storm

7 phases, shrinking to nothing, with a shader cylinder + ground ring:

`wait/shrink/radius-multiplier/dps` = 34/32/0.62/1, 26/28/0.58/2, 22/24/0.55/3, 20/21/0.5/5, 18/18/0.45/8, 14/15/0.42/10, 12/14/0.0/15.

Damage ticks once per second outside the circle, for players and bots.

## Bots

10–40 of them, names from a pool, random outfits, drop from the sky, then:
- three-state AI: acquire target (team-aware), fight with strafing, advance/retreat by distance, strafe direction flips on a timer;
- aim error scales with skill and distance, headshot bias, burst pauses;
- build defensive walls when hurt, open chests to upgrade loot, use their pocket item (throw/escape/heal), panic-run into the circle when the storm closes;
- detect when they are pressed against a wall (wanted to move but barely moved), probe which side has room and slide along the obstacle until the path clears, instead of running into it;
- box up: when shot they throw a three-wall U-shape around themselves (skilled bots) or a single wall (rookie bots); the Duos partner breaks off a revive or a reboot-van run to fight back when it is attacked, and builds a box at the van before rebooting;
- a difficulty tier drives skill, reaction time and damage.

## Modes (8)

Every mode shows on a compact 2-column lobby grid of cards with a kicker, name and foot line; the description shows next to PLAY.

1. **Battle Royale** — solo, 10–40 opponents, storm, full loot.
2. **Ranked** — same island, bots scale with your rank, RP is earned and shown live, ends in a ranked recap.
3. **Duos** — you + one bot partner vs 8 rival pairs; knock & reboot everywhere.
4. **Reload** — 25 players, 3 lives each, 5-second respawn countdown.
5. **Gun Game** — no loot, start on a pistol, upgrade one weapon per elimination through 10 guns, win at the top.
6. **Blitz** — no drop phase, storm already closing, ~7 minute matches, extra loot.
7. **Team Rumble** — 6 blue (you + 5 bots) vs 12 red, endless respawns, first team to 30 eliminations, live scoreboard, no storm.
8. **Sandbox** — infinite mats, no storm, no bots, all guns available.

**Modifiers** (toggle chips in the lobby, apply to any mode): low gravity, shotguns only, snipers only, fast storm, no building, glass cannon (everyone at ~55 hp).

## Knock, revive, reboot

- In team modes every lethal hit **knocks** instead of killing when a living team-mate exists.
- Knocked players/bots crawl at 1.5 m/s, bleed out over 42 s, and are revived automatically after 3.4 s of a living team-mate standing within 3.6 m. Revived at 35 hp / 25 overshield.
- When you are knocked: full-screen red-tinted overlay with a bleed countdown and "crawling/reviving" hint.
- **Reboot vans** (three, only in Duos): if your partner is fully eliminated, stand at a van and hold `E` for 8 s to bring them back with a pistol. The bot partner does the same for you automatically.

## Meta progression

- Profile saved in `localStorage` (`stormfall.profile.v1`): V-Bucks, cash, owned/equipped skins, battle pass state, pocket choice, rank RP, modifiers, settings.
- **V-Bucks**: +50 per elimination, +300 per win (400 in team modes). Battle Pass costs 1,000 and is per-season (must be repurchased each season). 14 skins, item shop, locker with equip, battle pass track with tiers that unlock skins and V-Bucks with a full unlock-reveal animation (spinning rays, sparks, card).
- **Ranked**: 8 ranks (Bronze → Unreal) × 3 divisions, 220 RP per rank. +12 RP per elimination, +20 for top 5, +75 for a win, −18 for finishing outside the top 5, floored at 0 with rank protection. A **ranked recap** at the end of every ranked match animates the RP swing, the percentage progress of your rank, and a gold RANK UP / red RANK DOWN banner when you cross a rank. Ranks reset to Bronze automatically when a new season begins (never via a manual button), and there is a Ranked Duos queue that adds the bot partner, knock/revive/reboot and vans on top of ranked scoring.
- **Competitive tab — tournaments**: 7 three-game tournaments with fake cash prizes, each gated behind a rank (Bronze $750 → Unreal $20,000; lower requirement, lower prize). Every event lets you pick between two or three game modes, and the prize scales with difficulty: Rumble 60%, Blitz 70%, Gun Game 80%, Reload 85%, Battle Royale 100%, Duos 115%. Entering generates a rival leaderboard; each of your three games scores points (**+3 per elimination**, placement points 25/20/17/12/8 for 1st/2nd/3rd/top 5/top 10) and the rivals score too. Finish inside the payout places and the cash prize is added. Cash is shown top-right next to V-Bucks and can be exchanged for V-Bucks ($750→300, $2,000→900, $5,000→2,500). Live scoring: points tick up during the match, each award **slides a card in on the right** ("+3 / ELIMINATION / RAVAGER"), and the total sits beside the health bar.

## HUD

- Top left: mode badge, leave-match button, kill feed.
- Top centre: storm panel (label, bar, countdown) or the team scoreboard in Rumble.
- Top right: minimap in an angled frame, compass letters, alive/elim counters.
- Centre: crosshair with dynamic bloom that opens when moving/firing and tightens when aiming; hitmarker that turns blue on shield hits and red on kills.
- Bottom right: hotbar of 5 slots (icons, key numbers, ammo/count, rarity underline) with pop animation on switch, ammo readout, pocket item slot with radial cooldown fill.
- Bottom centre-right: shield and health bars (shield bar shows overshield with a bright border), lives pips in Reload.
- Bottom left: wood counter, build piece selector.
- Left of the health bars: tournament points / rank RP panel during those matches. In team modes a teammate panel lists each mate with health/shield bars, KNOCKED plus a bleed countdown and revive progress, or OUT — and in Duos you spectate your mate after dying, with a banner saying what they are doing (heading to a van, at a van, rebooting you).
- Prompts and a progress bar for interactions (open chest, pick up, reboot).

## Multiplayer (optional but included)

- PeerJS 6-digit room codes over the public broker, plus a manual direct WebRTC offer/answer path with no ICE servers for same-Wi-Fi play, in HOST / JOIN tabs.
- Host authoritative: streams bot + storm snapshots at 10 Hz; guests send position at ≤10 Hz and ask the host to resolve their hitscan.
- On connect, drop shadows and pixel ratio for performance and skip per-bot weapon models on the guest.
- If the peer library fails to load from one CDN, try fallbacks and show a clear status message.

## Performance & polish

- Distance culling for props, bots, loot and corpses.
- Asset preload with a loading screen (fonts + generated textures, minimum ~0.6 s) so pressing PLAY before it finishes still starts the match when ready.
- Death animation: bodies topple and sink instead of vanishing.
- Screen shake, muzzle flashes, tracers, impact sparks, shockwave rings, revive and wipe banners.

## Versioning

- `VERSION = {major, minor, patch}` shown in the lobby and on a "What's New" patch-notes card that appears at boot and cannot be dismissed for 5 seconds.
- Second number = major updates, third = small updates.

## Acceptance checklist

- Opens with no console errors; loading screen → lobby.
- All 8 mode cards start a match with the right bot count, storm behaviour and win condition.
- No way to soft-lock: leaving a match returns to the menu, and every mode can end.
- Shots hit what the crosshair covers; headshots register; shotgun feels strong up close and falls off gently.
- Damage numbers stack per target, track the camera, and stay behind when the enemy runs.
- Ranked match ends with the recap; tournament match ends with points banked into the leaderboard; cash can be exchanged for V-Bucks.
- A phone in portrait can play, reach every menu tab, and see the whole HUD without overlap.
