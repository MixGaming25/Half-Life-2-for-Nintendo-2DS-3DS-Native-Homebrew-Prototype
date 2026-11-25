# Customs queue and checkpoint regression

The queue was blocked by several independent omissions:

- Static fences used their enclosing model bounds. Those boxes filled the narrow
  walking lanes. Conversion now uses the supplied VPHY convex collision pieces
  for supported static models and records any bounds fallback in `level.json`.
- NPC movement used the player's width and clip mask. Citizens now use a 26-unit
  hull and NPC clip contents; the player keeps the 32-unit hull and player-only
  barriers. Collision packages retain the Source contents bits. Old collision
  packages remain readable.
- The act-busy queue only assigned initial hints. It now handles activation,
  ordered slots, forward movement, pending releases, player-blocked slots and
  `OnNPCLeftQueue` with the departing citizen as activator.
- The converter omitted `OutValue` because it only retained keys starting with
  `On`. Extended output blocks now preserve those connections, and `logic_case`
  `InValue` selects the original schedules for the departing actor.
- Scripted schedules now resolve class selectors and their `goalent`. Walking
  links between original map nodes are prepared offline, using static collision,
  the NPC hull and step solver. Runtime routing and bounded local yielding help
  walkers get around fences and other NPCs. This is limited navigation, not the
  complete Source AI system.
- The arrest scene had speech timing but no walking. VCD MOVE_TO cues now resolve
  scene actor/target bindings and dispatch walking goals. Scene completion waits
  for those walks. NPC-only filtered triggers can fire, so the arrested citizen
  activates the checkpoint gate's original disable event.
- Rotating model doors now rotate their collision as well as their mesh. Authored
  start-open positions, travel angles and angular speeds are retained.

## Saves

Legacy entity, event and string indices are retained. New entity definitions and
extended output blocks are appended. Loading reconstructs collision indices and
new navigation routes, transfers output fire counts and retains current immutable
asset flags. A saved, already-started old queue is reattached to its hints and
previous advance triggers. The consumed arrest scene can be replayed if its
player gate remains closed. The user's original save is not deleted.

## Verification

The 34-test suite passes. Native fixtures exercise queue blocking/release,
counter-to-case-to-schedule activation, scene movement delaying completion,
NPC-only filtered triggers, rotating door collision, the different clip masks
and hull widths, and VPHY coordinate/plane validation.

The sanitizer-backed map test starts at the middle queue lane with the original
spawn/setup relays and two earlier advance requests as preconditions. It walks
through the third map trigger, around the switchback, past all three citizens,
and through the checkpoint to Source position approximately (-4216, -584).
The queue counter reaches three. All six walking segments complete without
jumping, noclip or disabling the player barrier manually; the last segment
required the arrest scene's NPC trigger to release that barrier.

An older arrival checkpoint was also read with the new assets and completed the
60-second train test, retaining the open exit door. A synthetic stalled save made
with the prior entity package also recovered: all three citizens left the queue
and the gate disabled. In Azahar, loading and walking forward from the queue
was checked visually; an ARM-generated save confirmed queue counter 3, all three
citizens released, and the checkpoint barrier disabled.

This does not certify a full
chapter playthrough or original hardware performance.

Format references:

- [Valve collision contents and masks](https://github.com/ValveSoftware/source-sdk-2013/blob/master/src/public/bspflags.h)
- [SourceIO VPHY/IVPS layout reader](https://github.com/REDxEYE/SourceIO/blob/master/library/models/phy/phy.py)
