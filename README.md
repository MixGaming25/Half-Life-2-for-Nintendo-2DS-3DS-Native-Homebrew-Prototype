# Half-Life 2 / 2DS experiment

![Target](https://img.shields.io/badge/Target-Nintendo%202DS%20%7C%203DS-E60012?style=flat-square)
![Language](https://img.shields.io/badge/Language-C%20%2B%20Python%203-00599C?style=flat-square)
![Graphics](https://img.shields.io/badge/Graphics-Citro3D%20%2B%20Citro2D-1E607A?style=flat-square)
![SDK](https://img.shields.io/badge/SDK-devkitARM%20%2F%20libctru-2EA44F?style=flat-square)
![Stage](https://img.shields.io/badge/Stage-Milestone%2005%20(Prototype)-E87A24?style=flat-square)

Milestone 05 adds chapter loading, an HL2-style menu, original G-Man voice cues,
and moving trains to the native 2DS prototype. **The full Point Insertion chapter
is not finished or verified playable end to end.** Combat, original animation
playback, several scripted interactions, and cinematic effects still need work.
See [the chapter status](docs/POINT_INSERTION_STATUS.md) for exact limits.
The [arrival fix](docs/ARRIVAL_FIX.md) repairs early stops from older saves and
keeps the train doors open. The station turnstile now rotates when pushed,
and NPC-only clip brushes no longer block the player. Existing station saves
are migrated to the added hinge body and updated collision indices.
The [customs queue fix](docs/CUSTOMS_QUEUE_FIX.md) adds advancing citizens,
scripted walking and the original NPC trigger that releases the checkpoint.

## Run

```sh
./tools/run-azahar.sh
```

The launcher uses the project-local Azahar 2127.0-rc2 AppImage when available.
It uses a separate settings directory under `.tools/azahar-config`, initialized
from the installed emulator's settings, with original-3DS mode selected for the
2DS target. Later edits to that project profile are preserved. `AZAHAR_BIN`
overrides the executable.
Your installed emulator is not replaced.

Azahar 2126.2 has a stubbed CSND playback implementation. The newer release
candidate implements CSND; this matters when DSP firmware is unavailable and the
application falls back from NDSP to CSND. The pinned official AppImage used here
has SHA-256:

```text
f06ffe57ae4bf0a90040146ecd727fcbae3229310cdbb0adb06f703bdf1700b0
```

[Official emulator release](https://github.com/azahar-emu/azahar/releases/tag/2127.0-rc2).
On real Nintendo hardware, NDSP requires the console's DSP firmware; CSND is the
fallback. Real hardware remains untested. Emulator frame rates do not establish
original-2DS performance.

## Controls

| Action | Console | Current keyboard mapping |
| --- | --- | --- |
| Move / menu selection | Circle Pad | Arrow keys |
| Look / menu selection | D-pad | F/H/T/G |
| Look | Bottom touchscreen drag | Mouse drag |
| Select / jump | A | A |
| Back / grab / drop | B | S |
| Use / push / throw held object | R | W |
| Run | L | Q |
| Pause / resume menu | START | M |
| Skip introduction | B during intro | S |
| Noclip, with Debug HUD enabled | Y | X |

The main menu has New Game, Load Game, Options and Quit. Pause adds Resume and
Save Game. Options change volume, look speed and the debug display. The debug
HUD reports missing inputs/targets and render/event budget limits.

Checkpoints are written to `sdmc:/3ds/hl2-2ds/checkpoint.dat`. Player position,
entity state, delayed events and mutable entity flags are restored. A checkpoint
is also written when arriving on the train and crossing map transitions. Active
speech is not seek-restored when loading a mid-scene save.

## Build from the supplied dump

The SDK is installed at `~/.local/share/hl2-2ds-sdk/opt/devkitpro`. Python 3,
Pillow, FFmpeg, Make, a host C compiler, devkitARM, libctru, Citro3D and Citro2D are required.

```sh
python3 tools/chapter_assets.py
make
./tools/run-azahar.sh
```

The converter reads `Half-Life 2 dump/halflife2/hl2`. It prepares five packages:
a G-Man room package and the four `d1_trainstation_01`–`04` chapter maps. It uses
shared model instances for static props, 64×64 textures, brush collision, entity
models, compiled VCD timing, sounds and parent metadata. Files in the dump are
read, not executed or modified.

Prepared packages are stored under `romfs/chapter/`; converted mono PCM audio is
under `romfs/audio/`; the menu background is derived from the original game
texture. These are embedded in `dist/source-2ds-lab.3dsx`. The dump, generated
game assets, build output and local tools are ignored by Git. Generated builds
contain local game assets.

To regenerate only selected map packages:

```sh
python3 tools/chapter_assets.py --only station02 station04
make
```

To refresh metadata/audio without regenerating maps, pass `--only` with no map
names. Existing PCM conversions are cached. To refresh a changed sound, delete
that generated PCM file first. The lower-level `tools/convert_bsp.py` remains
available for individual map experiments, but the menu build uses chapter folders.

The SDK bootstrap tool can install a user-local SDK without root or Docker:

```sh
python3 tools/bootstrap-toolchain.py
```

`DEVKITPRO` and `DEVKITARM` override SDK paths. Builds support spaces in paths,
compile with warnings as errors and produce `.3dsx`, `.smdh`, debug ELF and map
files in `dist/`. `make clean` removes build output, including the preserved
milestone-04 binary; it leaves the dump and converted assets intact.

## Verification

```sh
python3 -m unittest discover -s tests -v
```

The 34 tests cover binary asset readers, compiled scene timing, duplicate event
connections, model fixups and bind-pose orientation, movement, entity logic,
physics, trains, passengers, parenting, checkpoint round trips and malformed
packages. Native movement/runtime harnesses use AddressSanitizer and
UndefinedBehaviorSanitizer.

The actual train-map smoke test ran for 60 simulated seconds: all carriages
stopped at the platform, the passenger stayed aboard, and the middle carriage
finished near its authored station position. That test also reported unsupported
map events; it is **not** an end-to-end chapter playthrough. Remaining map
packages are load-tested separately. All chapter packages currently convert
without missing model or material errors. Refractive glass and eyes use authored
lower-end texture fallbacks; WindowImposter reflections are omitted. Full shader
effects and facial animation are not reproduced.

## Technical references

- [Valve BSP structures](https://github.com/ValveSoftware/source-sdk-2013/blob/master/src/public/bspfile.h)
- [Valve static prop layouts](https://github.com/ValveSoftware/source-sdk-2013/blob/master/src/public/gamebspfile.h)
- [Valve model layouts](https://github.com/ValveSoftware/source-sdk-2013/blob/master/src/public/studio.h)
- [Valve optimized mesh layouts](https://github.com/ValveSoftware/source-sdk-2013/blob/master/src/public/optimize.h)
- [Valve compiled scene image](https://github.com/ValveSoftware/source-sdk-2013/blob/master/src/public/scenefilecache/SceneImageFile.h)
- [Valve choreography decoding](https://github.com/ValveSoftware/source-sdk-2013/blob/master/src/game/shared/choreoscene.cpp)
- [Official 3DS examples](https://github.com/devkitPro/3ds-examples)

The application is custom C/Citro3D code and offline Python conversion. It does
not run the Source engine/game DLL. Layout references informed the original
readers; game content comes from the supplied dump.
