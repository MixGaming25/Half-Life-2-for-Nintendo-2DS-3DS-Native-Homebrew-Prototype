# Entity runtime milestone 04

This is the historical milestone-04 report. See [current chapter status](POINT_INSERTION_STATUS.md) for milestone-05 additions and limitations.

The converter retains repeated BSP entity keys. `entities.bin` stores 751 entity
definitions, 667 output connections, shared MDL/brush templates and compressed
procedural pose deltas for the current train-station region. `entities.json`
reports model failures and unsupported map classes. No model/material extraction
errors remain in this conversion.

## Supported subset

| Area | Behavior |
| --- | --- |
| Entity output connections | Preserve duplicates, relative delays and fire counts; named/wildcard targets; `!self`, `!caller`, entity `!activator` |
| Common inputs | `Kill`, `Enable`, `Disable`, `Toggle`, `FireUser1` through `FireUser4` |
| Startup | `logic_auto` `OnMapSpawn` and `OnNewGame` |
| Triggers | Approximate brush AABB intersection with the player; start/end touch, once firing, multiple refire delay; simple player-name/class filters |
| Relays/timers | `Trigger`, `FireTimer`, `RefireTime`, `ResetTimer`; enable/disable; fire-once relay flag |
| Counters | `Add`, `Subtract`, `SetValue`, `SetValueNoFire`, `GetValue`; clamping, `OutValue`, hit-limit events; explicit connection parameters take precedence over output values |
| Cases | `PickRandom`, `PickRandomShuffle`; shuffle avoids reuse within a cycle |
| NPC steering | Direct named target/path-corner movement with collision and wall sliding; simple follow/wait interaction for citizens; player-facing idle behavior |
| NPC inputs | `SetTarget`, `SetGoal`, `StartPatrolling`, `StopPatrolling`, `StopMoving`; scripting markers |
| Scripted sequences | `BeginSequence`, `CancelSequence`, bounded movement/teleport modes; sequence outputs around a fixed-duration placeholder action |
| Buttons | Reach/line-of-sight use and `Press`; `OnPressed` |
| Sliding doors | Use, `Open`, `Close`, `ToggleDoor`, `Lock`, `Unlock`; opening vector/speed/wait; completion events; reverse closing if blocked by the player |
| Physics props | Fixed orientation, swept bounds, gravity, bounce, friction, sleep/wake, use/push, hold/drop/throw, approximate impulse transfer |
| Physics inputs | `Wake`, `Sleep`, `EnableMotion`, `DisableMotion` |

Unsupported inputs/targets are counted rather than treated as successfully
executed. In particular, VCD `Start` does **not** automatically emit a fabricated
`OnCompletion`. The HUD displays dispatched event counts, queue length and
misses. Source relationship/combat inputs, speech, sound and most visual effects
are unsupported.

The runtime uses 60 Hz fixed simulation steps, a 2,048-entry delayed event queue
and a maximum of 256 dispatches per tick. Full queues and recursive chains are
bounded and increment an overflow diagnostic. Frame time is capped to avoid an
unbounded catch-up loop. Render scratch allocation is bounded to 90,000 vertices;
models nearest the camera are considered first.

## NPC and animation limits

Humanoid MDL/VVD/VTX meshes are loaded with skins and reduced LODs. Their bind-pose
weights drive an offline procedural arm-rest pose and four walk phases. The
original animation files, animation blending/IK, expressions, voice scenes,
weapons, damage, factions, act-busy queues and combat schedules are absent.
Scanners/cameras remain at their authored positions rather than using humanoid
gravity. NPC movement has no full navigation graph and can stop at obstacles.
Citizens following on use is a prototype interaction, not recreated HL2 dialogue.

`scripted_sequence` actions last 1.5 seconds after reaching their marker and do
not decode the named clip. `aiscripted_schedule` has only a rudimentary start
handler. There is no complete choreography or campaign progression.

## Collision and physics limits

Dynamic solids use oriented model bounds converted to a fixed world AABB. Those
bounds can block visible gaps. Trigger checks are AABB approximations, so convex
or rotated trigger volumes are not exact. Physics props are initially asleep to
preserve their authored placement and keep idle simulation inexpensive. They
wake on use, explicit map input or a supported prop impulse.

The current solver has no angular integration, mass/inertia model, constraints,
ragdolls, breakage, damage, buoyancy, or true contact manifolds. Sliding doors
have approximate blocking behavior; rotating and parented doors/trains are not
implemented. An unsupported rotating-door open request is reported and does not
pretend to open the door. Player-use inputs targeting the special `!player`
entity and unsupported filter combinations are not emulated.

## Verification

Run `python3 -m unittest discover -s tests -v` for the 15 tests. Two native C
harnesses run under AddressSanitizer and UndefinedBehaviorSanitizer. They cover
movement, duplicate event export, delayed/limited outputs, parameter propagation,
startup/timers/triggers, counter updates, blocked sliding doors, scripted actor
movement, physics gravity/sleep, grab/throw/impulse transfer, malformed packages,
and event recursion budgets.

The runtime harness can also load the real prepared data:

```sh
cc -std=c11 -Wall -Wextra -Werror -fsanitize=address,undefined -g -Isource \
  tests/runtime_test.c source/runtime.c source/runtime_assets.c source/collision.c \
  -lm -o build/runtime-test
./build/runtime-test romfs/entities.bin romfs/collision.bin
```

The initial map smoke test simulated three seconds with finite entity states;
19 unsupported startup input/target requests were reported. This validates the
implemented subset, not the original station timeline. NPC rendering was
visually checked in Azahar 2126.2. Original Nintendo hardware remains untested.
