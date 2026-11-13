# Drill Dozer (Screw Breaker: Goushin DoriRureRo in Japan)

| | |
|---|---|
| Releases | `V49E` USA (Europe's Virtual Console release and Japan's Screw Breaker not yet supported) |
| Dumps | `V49E` cartridge `e60ec183` (tested) |
| Profile | `src/profiles.c`: the `V49E` entry (`dd_virt`, `logoMaskDD`) |
| Tests | `tests/regress/drill-dozer.json` (`tools/regress.py drill-dozer`) |
| Status | supported |

What GBA DS does with this game. Each rule is a promise the profile keeps; the regression tests check them (the cases citing a rule are listed by `tools/regress.py --list drill-dozer`). Change a rule here first when the behaviour is meant to change, then the profile, then the tests and goldens. Rule numbers are never reused: a dropped rule stays, marked *retired*.

Every game also follows the [general rules](README.md#general-rules) (GEN-1 to GEN-7).

Modelled on Super Mario Advance 3 ([SM3 rules](super-mario-advance-3.md)) and Wario Land 4 ([WL4 rules](wario-land-4.md)): Drill Dozer has no area map, so while Jill is in a stage the second screen shows the Equipment screen (`SELECT`), opened on the hidden copy and kept up to date with her Drill Bit, Drill Type, Gear, Energy, Chips and the current room's name, its button prompts left out. Because the Equipment screen fills the whole 240×160 screen and the game's live gear indicator and RPM gauge react as you drill and shift, the game screen stays whole.

## Rules

**DD-1. Logo.** The logo (GEN-1) is the title screen's `DRILL DOZER` badge with the red drill bit inside its `O`, cut out to the outside of the badge's cyan-and-white rim (`logoMaskDD`), without the black buffer around it on BG0, the brick rubble sprites, the ™ mark, `PRESS START` or the copyright line.

**DD-2. In a stage: the Equipment screen.** While Jill is in a stage (walking, jumping, drilling, underwater, flying or in a boss fight), the second screen shows the game's Equipment screen (`SELECT`), opened on the hidden copy and taken again about once a second (`map_interval`), whole at 240×160 once its fade-in is done: Jill and the Red Dozer on the left (matching her current drill bit and outfit), Drill Bit (`Normal`, `Super Bit` or `Mega Bit`), Drill Type (`Normal`, `Propeller` or `Jet`), Gear (`1st Gear`, `2nd Gear` or `3rd Gear`, the highest gear found in the stage so far), Energy (current and maximum, with an icon for each extra Energy Tank owned), Chips, and the current room's name in the plate at the bottom left. The `(A) Help` and `(B) Back` prompts at the bottom right are left out: the `(A)` and `(B)` button sprites are hidden, and the `Help` and `Back` words are replaced by the orange striped background the game draws behind them, leaving the corner screw intact. With `map_crop = 0` (`gba-ds.cfg`) the whole Equipment screen is shown, the prompts included.

**DD-3. Room changes and upgrades show at once.** Going through a door into another room, finding the 2nd or 3rd Gear in a stage, changing Drill Type (`Normal`, `Propeller` or `Jet`), upgrading the Drill Bit or Energy Tanks, or changing Jill's outfit starts a new capture as soon as the Equipment screen can be opened (GEN-6), without waiting for the next second.

**DD-4. The game screen stays whole.** Nothing is taken off the game screen: the energy bar and 1-2-3 gear indicator at the top left and the translucent RPM / `SHIFT UP!!` gauge spin and flash in real time as Jill drills and shifts gears, and stay where the action is.

**DD-5. In-stage menus, conversations and doors.** Opening `START`'s menu (`Save` / `Sleep` / `Return to Trailer`) in a stage, reading the `Help` manual (`A` inside Equipment), talking in an in-stage conversation or tutorial, or going through a door keeps the last Equipment picture on the second screen. While the Equipment screen itself (`SELECT`) is open on the game screen, the second screen shows the logo instead, so the same screen isn't on both; the Equipment picture comes back after it closes (or when `Help` opens over it).

**DD-6. Logo elsewhere.** The second screen shows the logo at half strength on dark grey (GEN-1) before the first capture and whenever Jill isn't in a stage, forgetting the last stage's Equipment picture: the title screen, the opening, the file select, the Red Dozer Trailer hub (and its Map, Equipment, Treasures, Save and Shop screens), the world map / stage select, stage intro and outro scenes, and from the moment Jill runs out of energy in a stage through her explosion and the `Pay 50 chips to continue?` / Game Over screen.

## How it was checked

Checked on the USA cartridge dump (`V49E`, CRC32 `e60ec183`) with `build/host/harness` and `tools/regress.py` across boot, the Health & Safety screen, the title screen, the file select, the opening cutscene, the Red Dozer Trailer hub (and its Equipment and Shop screens), the world map, Stage 1-1 (`Dark Pit`, including the opening dialogue, green snail tutorials, `START`'s Save screen, `SELECT`'s Equipment screen, `A`'s 13-page Help manual, and running out of energy into `Pay 50 chips to continue?`), Stage 2-2 (`Giant Vault Door`), Stage 3-2 (`Stone Face Plaza`, walking through the door into `Watering Hole`, and underwater in `Bubble Room`), Stage 4-2 (`Metal City Exit` and `90 Ft. Above`), and the final boss fight (`Showdown`, room `0xB1`, with 3rd Gear, Mega Bit and five Energy Tanks). Room warps were driven from the room header table at ROM `0x0821B314` (stride 36 bytes) and the room name table at `0x0821F214`; the Equipment screen's fields were verified by disassembling its initializer at `0x0805CE10` and `0x0800C1E0`.

## Not covered yet

- Europe's Virtual Console release (`Drill Dozer (Europe) (En,Fr,De,Es,It) (Virtual Console)`, `4c09614e`) and Japan's cartridge release (`Screw Breaker - Goushin DoriRureRo`, `b17532ee`): no ROMs tested yet.
- Post-game bonus stages (the unlocked secret rooms after the credits), the Treasures gallery, the Sound Room, and the ending / credits roll.

## Facts found so far (for sessions)

| What | Where / value (`V49E`) |
|---|---|
| Global game struct | `0x020345C8` in EWRAM |
| Main mode (`+0x04`) | `0x020345CC`: `0x00` title / file select / room-load fade-in, `0x01` live stage play, `0x02` door fade-out, `0x03` dying / `Pay 50 chips to continue?`, `0x06` menus / Trailer hub / world map / stage scenes, `0x07` stage intro scene |
| Dialogue / tutorial flag (`+0x09`) | `0x020345D1`: `0x00` normal play, `0x01` in-stage dialogue box or snail tutorial |
| Menu sub-screen (`+0x0A`) | `0x020345D2`: when `0x020345CC == 0x06`, `0x02` for in-stage `SELECT` Equipment (and `Help` opened from it; Equipment is GBA video Mode 1, `DISPCNT & 7 == 1`, while `Help` switches to Mode 0), `0x03` for `START` Save, Trailer hub and world map |
| Area / Stage / Drill Type / Room (`+0x0F..+0x12`) | `0x020345D7` Area (`1..6`, `0` on title / file select), `0x020345D8` Stage (`1..0x11`), `0x020345D9` Drill Type (`0` Normal, `1` Propeller, `2` Jet), `0x020345DA` Room ID (up to `0xCF`; names at ROM `0x0821F214`) |
| Control lock (`+0x1E`) | `0x020345E6`: `0x00` only when Jill has live control and `SELECT` opens Equipment; non-zero in dialogues (`0x7D`), tutorials (`0x79`), doors (`0x59` / `0x01`) and menus (`0x7F`) |
| Stage room pointer (`+0x38`) | `0x02034600` (word): `0x0202xxxx` whenever a stage room is loaded (including in-stage `START` / `SELECT` / `Help`), `0` on the title, file select, Trailer hub, world map, stage scenes and Game Over (and briefly during frames `145..159` of a room door load while `0x020345CC == 0`) |
| Upgrades & Chips | `0x0203463B` Drill Bit (`0` Normal, `1` Super Bit, `2` Mega Bit), `0x0203463C` Energy Tanks (`0..5`), `0x0203465D` outfit, `0x02034670` (halfword) Chips (`0..99999`) |
| Stage gears & Energy | `0x020346B7` 2nd Gear found, `0x020346B8` 3rd Gear found, `0x020346BC` (halfword) current Energy (`100..600`, `0` when dead) |
| Equipment screen | `SELECT` fades in over ~62 frames and resets its local animation counter to 0 each time it opens, so captures settle identically with `animFrames = 0`. `(B)` (`153,144`) and `(A)` (`166,134`) are 16×16 sprites in palette 12 (no other Equipment sprite uses palette 12); `Help` and `Back` are on `BG0` at `174..212, 136..159` over `BG1` / `BG2`'s orange stripes |
| Title logo | `wait = 360`, `pressStart = true` (skips Health & Safety), `wait2 = 1100`. `BG0` holds `DRILL DOZER` + ™ (`9..230, 35..108`) with an uneven black buffer around the cyan rim against `L4` rubble sprites; the red drill bit inside `O` is an `L4` 32×32 sprite at `140,66` |

## For the merge

- **CHANGELOG.md** (under `### New games`):
  - **Drill Dozer** (USA): in a stage (on foot, underwater, flying and in boss fights), the second screen shows the Equipment screen (the one SELECT opens), kept up to date about once a second and straight away when you go into another room or pick up a gear or upgrade: Jill and the Red Dozer, your Drill Bit, Drill Type, Gear, Energy and Energy Tanks, Chips, and the room's name, without the Help and Back button prompts. The energy bar, gear indicator and shift gauge stay on the game screen; conversations, tutorials, doors and START's Save screen keep the last picture; opening Equipment on the game screen itself, running out of energy, the Trailer hub, the world map and the title show the logo. Not yet tried: the European Virtual Console and Japanese (Screw Breaker) releases, the post-game bonus stages and the ending.
- **wiki/Supported-games.md** (table row):
  - `| Drill Dozer (Screw Breaker: Goushin DoriRureRo in Japan) | \`V49E\` (Europe's Virtual Console and Japan's \`V49J\` not yet supported) | in stages: the Equipment screen (Drill Bit, Drill Type, Gear, Energy, Chips and the room's name) | |`
- **wiki/Supported-games.md** (section, if desired):
  - `## Drill Dozer`
  - `- **In a stage,** the second screen shows the **Equipment** screen (\`SELECT\`): Jill and the Red Dozer, your Drill Bit, Drill Type, Gear, Energy, Chips and the current room's name, without its Help and Back prompts. Opening Equipment on the game screen itself shows the logo on the second screen meanwhile; \`START\`'s Save menu, the Help manual, conversations and doors keep the last picture.`
  - `- Only the USA release (\`V49E\`) is supported so far.`
- **docs/games/README.md** (table row):
  - `| [Drill Dozer (Screw Breaker in Japan)](drill-dozer.md) | DD | \`V49E\` |`
