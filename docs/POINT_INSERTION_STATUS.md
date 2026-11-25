# Point Insertion implementation status

The requested result is the complete first chapter, including the main menu,
G-Man introduction, train arrival and all four maps through Alyx's rescue.
**This result has not yet been achieved.** The current milestone is a chapter
integration build, with the following concrete additions and remaining gaps.

| System | Current state |
| --- | --- |
| Menu | New Game, pause/resume, options, save/load, quit; original background texture and HL2-style typography |
| Materials | Original base textures; authored DX60/DX8 texture fallbacks for refractive glass and eyes; WindowImposter reflections omitted; no full Source shader reproduction |
| Four chapter maps | Converted as separate packages; shared static model instances keep memory lower than baking every prop |
| Original audio | Original sound scripts resolved from the dump; PCM preparation; NDSP streaming with CSND fallback; current speech is not restored by save/load |
| G-Man introduction | Original model, original speech timing/voices and a 60.168-second scene clock; fixed camera; original facial animation, animation clips, scene blends and full visual choreography are missing |
| Train | Three track-following coaches, authored speed/stop outputs, parented passengers/doors, hollow brush collision, passenger translation/rotation; real-map 60-second simulation reaches the station |
| Map changes | `trigger_changelevel`, landmarks and player offsets; loading `d1_trainstation_05` marks this chapter complete instead of loading chapter two |
| Scene movement | Bound MOVE_TO actors/targets dispatch walking; completion waits for active walks; NPC-only filtered triggers release the original customs gate |
| Scene timing | Compiled scene-image v2 and binary VCD timing are decoded; speech and `FIRETRIGGER` events are scheduled at their original timestamps |
| Scene animation | Most requested animation clips, gestures, facial flexes, look/move channels and blending remain unimplemented; procedural poses do not replace these faithfully |
| NPCs | Ordered act-busy queues; actor-preserving schedule selection; collision-tested walking links, bounded routing and local yielding; separate NPC hull/clip rules; no full combat, relationships or Source AI schedules |
| Dynamic objects | Turnstile supports an anchored vertical hinge with push-driven rotation and matching convex collision; other props use simplified fixed-orientation physics; use, grab, throw, sleep/wake and impulse transfer; not VPhysics |
| Doors | Sliding and basic rotating doors; brush collision and parent transforms; incomplete obstruction/crush, locks and animation semantics |
| Templates | Activate prepared named template members; repeated spawning of independent copies is not implemented |
| Player/campaign | Walking, jumping and stepping; no complete health/damage, death, rescue or chase logic; no certified end-to-end route |

## Validation and unresolved blockers

The real train test catches initial path-node events, speed changes and stopping
all coaches and asserts their authored stop positions and the exit door remaining open. See
[the arrival regression report](ARRIVAL_FIX.md) for the old-checkpoint repair. Its final carriage position was approximately (-20.70, -0.12, 20.94)
in the station package's coordinates, and the passenger remained inside the
coach. The corrected fresh-start test reports 55 unsupported startup/arrival input or target
requests. Some are effects/sound controls; others still affect gameplay. A
missing-event count must not be interpreted as a successful original sequence.

The four-map packages now contain the necessary geometry and assets, but merely
loading and walking around those packages is not a faithful playable chapter.
The chapter still needs original animation playback, additional input handlers,
broader NPC navigation and schedule coverage, health/combat behavior, and the scripted rooftop/rescue
ending. These remain the completion criteria for the user's full request.

## Emulator audio

Azahar 2126.2 stubs CSND command execution. Without a DSP firmware file, NDSP
initialization fails, so successful CSND API calls in that older emulator do not
prove audible playback. The official 2127.0-rc2 source implements those PCM
commands. A checksum-verified project-local copy is used by the launch script;
the installed emulator is not overwritten.

- [2126.2 CSND service](https://github.com/azahar-emu/azahar/blob/2126.2/src/core/hle/service/csnd/csnd_snd.cpp)
- [2127.0-rc2 CSND service](https://github.com/azahar-emu/azahar/blob/2127.0-rc2/src/core/hle/service/csnd/csnd_snd.cpp)

## Reproduce the chapter smoke test

```sh
cc -std=c11 -Wall -Wextra -Werror -fsanitize=address,undefined -g -Isource \
  tests/chapter_test.c source/runtime.c source/runtime_assets.c \
  source/collision.c source/game_save.c -lm -o build/chapter-test
./build/chapter-test romfs/chapter/station01
```

Passing the native unit and load tests does not certify game accuracy or real
2DS performance. Those checks remain separate from a full, normal-controls
chapter playthrough.

## Station turnstile regression

The gate's `func_physbox` and vertical `phys_hinge` are converted together. The
rotor uses its individual convex brushes rather than an enclosing solid box;
walking into it applies angular motion, with bounded player contact resolution
against the rotated planes. The anchor remains fixed. Use/push also turns it;
it cannot be picked up as a loose prop. This is a limited hinge solver, not full
VPhysics or arbitrary constraint support.

An NPC-only clip brush at the gate had no player-solid planes. Its old fallback
box incorrectly blocked the complete opening. Such brush models now remain
non-solid to the player. Version-1 saves preserve their entity/event indices;
the new hinge is appended and saved collider indices are reconstructed.

The sanitizer-backed real-map test walks through the passage from both the train
side and the opposite side, without jumping or noclip, asserts that the rotor
turns and stays anchored, and checks the player does not enter solid geometry.

The rebuilt app was also tested in Azahar by loading a separate copy of the old
checkpoint at the gate and holding forward through it. The player crossed into
the room beyond the rotor. The original user checkpoint was left unchanged.

See [the customs queue regression](CUSTOMS_QUEUE_FIX.md) for the queue-to-checkpoint route test and remaining limits.
