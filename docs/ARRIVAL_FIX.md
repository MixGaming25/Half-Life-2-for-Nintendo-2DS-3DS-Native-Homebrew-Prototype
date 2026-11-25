# Intro and train arrival fixes — 14 November 2026

The user's existing arrival checkpoint reproduced an early stop at 3.15 simulated
seconds. It restored unconsumed one-shot outputs from `intro_train_path_0`, which
sent `SetSpeed 0.0` to all three coaches after the rear coach reached that node.
Loading now keeps initialization outputs consumed. Existing version-1 saves
remain readable; the original save file is not deleted or overwritten by the fix.

Door conversion now preserves Source's toggle flag. Those doors remain open
until a close/toggle input rather than automatically closing after their wait.
Save loading restores mutable lock flags while retaining newly converted,
immutable behavior flags. Old queued automatic cinematic startup is discarded;
loading a station checkpoint no longer starts the introduction again.

Scene loads display a progress screen and synchronize GPU work before replacing
scene resources. The introductory clock catches up when rendering falls below
20 FPS, and the bottom screen shows elapsed intro seconds. The G-Man model still
uses a static pose: original skeletal/facial animation is not implemented.

Validation:

- The old checkpoint was loaded in a native harness, without modifying the user's
  file. Before: all coaches stopped near the tunnel at 3.15 seconds. After: they
  reach the platform, and both sliding door panels remain fully open at 60 seconds.
- The real-map harness now asserts each coach finishes within 32 Source units of
  its authored station position, at zero speed, with the exit door held open.
- Native tests cover an explicit close after a toggle door has remained open for
  ten seconds, and save compatibility with consumed initialization events and
  new immutable flags.
- In a private Azahar display, the actual menu paths New Game → B skip and pause
  → Load Game both reach the train. Loading the user's old checkpoint was checked
  using a separate copy in the QA profile; the open doors were observed. The
  introduction also completed automatically and loaded the train without B.

The 27-test suite passes with the native collision/runtime harnesses using
AddressSanitizer and UndefinedBehaviorSanitizer. This fixes these arrival bugs;
it does not certify an end-to-end Point Insertion playthrough or hardware speed.
