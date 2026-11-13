# The Legend of Zelda: The Minish Cap

| | |
|---|---|
| Releases | `BZME` USA (Europe's `BZMP` and Japan's `BZMJ` not yet supported) |
| Dumps | `BZME` cartridge `abcebbb1` (tested) · `BZMP` cartridge `e8637292` · `BZMJ` cartridge `6ce771a5` |
| Profile | `src/profiles.c`: the `BZME` entry (`tmc_compose`, `tmc_virt`) |
| Tests | `tests/regress/zelda-the-minish-cap.json` (`tools/regress.py zelda-the-minish-cap`) |
| Status | supported |

What GBA DS does with this game. Each rule is a promise the profile keeps; the regression tests check them (the cases citing a rule are listed by `tools/regress.py --list zelda-the-minish-cap`). Change a rule here first when the behaviour is meant to change, then the profile, then the tests and goldens. Rule numbers are never reused: a dropped rule stays, marked *retired*.

Every game also follows the [general rules](README.md#general-rules) (GEN-1 to GEN-7).

Modelled on [A Link to the Past](zelda-a-link-to-the-past.md): outdoors in Hyrule, the second screen shows the detailed **regional map** (`PauseMenu_Screen_6`, scrolled vertically to centre on Link), and in dungeons it shows the **Dungeon Map** (`PauseMenu_Screen_5`) on Link's floor, both opened on the hidden copy in the background so gameplay never pauses; indoors where the game has no map, the outdoor map stays up dimmed. Unlike A Link to the Past, nothing leaves the live game screen because the full-screen framed map leaves no room for a split HUD.

## Rules

**TMC-1. Logo.** On the title screen, the file select, during the Picori Festival prologue before Link receives the World Map, on Game Over and during the ending credits, the second screen shows the logo (GEN-1): `THE LEGEND OF ZELDA` and `The Minish Cap` over the Four Sword, its golden pommel gem and flowing ribbon (`BG2` and `OBJ` above row 122), without the forest background, light rays, `PRESS START` or copyright line.

**TMC-2. Overworld: the regional map.** Outdoors in Hyrule (once Link has the World Map), the second screen shows the detailed map of the region Link is in (`Hyrule Town`, `Minish Woods`, `Eastern Hills`, `Mt. Crenel` and the rest of the 17 regions) inside its carved stone frame with the region's name below, scrolled vertically to centre on Link (`gMenu.field_0xa`) and refreshed in the background about once a second (`map_interval`) and as soon as Link enters a new room or area or gains a map hint (GEN-6).

**TMC-3. Dungeons: the floor map.** In a dungeon with a map (Deepwood Shrine, Cave of Flames, Fortress of Winds, Temple of Droplets, Palace of Winds and Dark Hyrule Castle), the second screen shows the Dungeon Map on Link's current floor: the floor list and current floor bracket, Link's floor head and room marker, visited rooms (and unvisited rooms once the Dungeon Map is found), chests and the boss skull once the Compass is found, the Dungeon Map / Big Key / Compass icons owned, and the dungeon's name below, refreshed as Link enters a new room or picks up a dungeon item (GEN-6).

**TMC-4. Steady markers and no button prompts.** Markers that alternate every 32 frames on the regional map (Link's head against the red story objective checkmarks and Kinstone Fusion icons) are composited so all of them stay visible and steady at once, and the dungeon map is taken at a fixed tick phase so its floor bracket, Link markers and boss skull hold still without flashing (as A Link to the Past's LTTP-6). Because the second screen isn't paged or scrolled directly, the `[MAP]` tab badge, `[L]` and `[R]` tab buttons, `(B) Return` prompt and vertical scroll arrows are left out, showing all four Triforce corners of the stone frame.

**TMC-5. Indoors without a map.** Inside houses, shops, caves, Minish Village, the Elemental Sanctuary and the Royal Crypt (where the game has no indoor floor map), the outdoor regional map from outside stays on the second screen, dimmed (`dimWhenUnavailable`), and brightens again once Link steps back outside (or switches to the dungeon map on entering a dungeon); before the first map of the session, the logo stays.

**TMC-6. Menus, conversations and transitions.** In conversations, cutscenes, room transitions, and while the game's own pause screens (Items, Quest Status, World Map, Kinstone Bag, Figurines, Sleep / Save) are open on the game screen, the second screen keeps its last map picture (or stays dimmed if opened indoors); returning to the title, file select, Game Over or staff roll forgets the session's map and brings back the logo (GEN-6).

**TMC-7. The HUD stays.** Hearts, Rupees, Keys and the `A` / `B` / `R` action buttons (whose `R` label changes live with context) stay on the game screen.

**TMC-8. One screen.** With one screen (`layout = single`), the game plays as it is.

## How it was checked

Checked on the USA cartridge dump (`BZME`, CRC32 `abcebbb1`) with `build/host/harness`, `tools/regress.py` and the `zeldaret/tmc` decompilation across the title screen, the file select, the prologue in Smith's House before obtaining the World Map, Minish Woods (`Area 0x00`), Hyrule Town (`Area 0x02`) including vertical scrolling and `START`'s pause menu, inside Mayor Hagen's House (`Area 0x20`) and stepping back outdoors, Deepwood Shrine (`Area 0x48`) before and after collecting the Dungeon Map / Big Key / Compass, and Game Over (`TASK_GAMEOVER`). EWRAM/IWRAM addresses and screen state transitions were verified directly against `zeldaret/tmc` `linker.ld`, `pauseMenu.c`, `pauseMenuScreen6.c` and `areaMetadata.c`.

## Not covered yet

- Europe (`Legend of Zelda, The - The Minish Cap (Europe) (En,Fr,De,Es,It)`, `BZMP`, `e8637292`) and Japan (`Zelda no Densetsu - Fushigi no Boushi (Japan)`, `BZMJ`, `6ce771a5`) dumps: `zeldaret/tmc`'s `linker.ld` places the EWRAM and `0x03000BF0..0x03004050` IWRAM structures at the same addresses, with `BZMP`'s language menu shifting title logo timing, but neither ROM has been tested yet.
- Late-game dungeons past Deepwood Shrine, Ezlo's barrel/minish portal states (`GAMEMAIN_BARRELUPDATE`), Kinstone Fusion map icons in live saves, final Vaati phases and the ending staff roll (`TASK_STAFFROLL`): all share the same state machine and metadata flags (`AR_IS_OVERWORLD`, `AR_HAS_MAP`), untried.

## Facts found so far (for sessions)

| What | Where / value (`BZME`, shared across USA/EU/JP in `zeldaret/tmc` `linker.ld`) |
|---|---|
| `gMain` (`0x03001000`) | `task` at `0x03001002` (`0` Title, `1` File Select, `2` Game, `3` Game Over, `4` Staff Roll); `state` at `0x03001003` (`2` `GAMETASK_MAIN`); `substate` at `0x03001004` (`2` `GAMEMAIN_UPDATE`, `5` `GAMEMAIN_BARRELUPDATE`, `7` `GAMEMAIN_SUBTASK`); `ticks` at `0x0300100C` (halfword) |
| `gRoomControls` / `gFadeControl` / `gRoomTransition` | `area` `0x03000BF4`, `room` `0x03000BF5`, `scroll_flags` `0x03000BFF` (`& 4` scrolling); `gFadeControl.active` `0x03000FD0`; `transitioningOut` `0x030010A8` |
| `gArea` (`0x02033A90`) | `areaMetadata` at `0x02033A90` (`0x01` `AR_IS_OVERWORLD`, `0x04` `AR_IS_DUNGEON`, `0x08` `AR_HAS_MAP`), `dungeon_idx` at `0x02033A93` |
| Pause check guards | `gPauseMenuOptions.disabled` `0x02034490`, `gMessage.state` `0x02000050` (`& 0x7F`), `gSave.stats.health` `0x02002AEA`, `gSave.inventory[0]` `0x02002B32`, `gPlayerState.controlMode` `0x0300400B`, `gPriorityHandler.priority_timer` `0x03003DC8`, `gPlayerState.framestate` `0x03004028` |
| World Map & dungeon items | `ITEM_MAP` (`0x47`) at `(gSave + 0xF2)[17]` = `0x02002B43` (`>> 6 & 3`); `gSave.map_hints` at `0x02002A60` (halfword); `gSave.windcrests` at `0x02002A80` (word, bits `0..16`); `gSave.dungeonItems[0..15]` at `0x02002EAC` |
| `gPauseMenuOptions` (`0x02034490`) | `screen` `0x02034491` (poking `4` before `START` auto-selects `Screen 5` when `AreaHasMap()`, else `Screen 4` World Map); `unk11` state `0x020344A1` (`2` settled); `unk16` blend `0x020344A6` (`0` settled) |
| `gMenu` (`0x02000080`) | `menuType` `0x02000085` (`1` init done), `field_0xa` `0x0200008A` (regional vertical scroll, initialized to `maxScroll >> 1`), `gGenericMenu.unk2c` `0x020000AC` (bit 5 alternates Link's head vs hints/fusions on `Screen 6`; bit 4 draws scroll arrows); `gMapDataBottomSpecial` `0x02019EE0` (Link's regional entry at index `0` when `+0x00 == 1`, local `y` at `+0x06`, list end `+0x02 == 0`) |
| `Screen 5` / `Screen 6` OAM chrome | `[MAP]` + `[L]` / `[R]` buttons: tiles `16..127` at priority `1`; `(B) Return` on `Screen 6`: tiles `256..511` at priority `0` (`x >= 140, y <= 32`) |
| Title logo | `{ { 0, 140 }, { K_START, 4 }, { 0, 260 } }` (frame 404, before `PRESS START` appears), `layers = 0x14` (`BG2` Four Sword + ribbon, `OBJ` wordmark + pommel gem), `maxY = 122` |

## For the merge

- **CHANGELOG.md** (under `### New games`):
  - **The Legend of Zelda: The Minish Cap** (USA): outdoors in Hyrule (once Link has the World Map), the second screen shows the detailed regional map (`PauseMenu_Screen_6`) scrolled vertically to centre on Link; in dungeons with a map, it shows the floor map (`PauseMenu_Screen_5`) on Link's floor with chests, boss skull and dungeon items. Alternating markers hold steady at once, and the `[MAP]`, `[L]` / `[R]`, `(B) Return` and scroll arrow prompts are left out; indoors without a map, the outdoor map stays dimmed, while pause menus and conversations keep the last picture and the HUD stays on the game screen. Not yet tried: Europe and Japan.
- **wiki/Supported-games.md** (table row):
  - `| The Legend of Zelda: The Minish Cap | `BZME` (`BZMP` and `BZMJ` not yet supported) | outdoors: regional map; in dungeons: floor map; indoors: outdoor map dimmed | |`
- **docs/games/README.md** (table row):
  - `| [The Legend of Zelda: The Minish Cap](zelda-the-minish-cap.md) | TMC | `BZME` |`
