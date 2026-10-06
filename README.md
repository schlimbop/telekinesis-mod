# Telekinesis Combat — 0.1.0 development build

An SKSE C++ implementation of combat telekinesis for Skyrim SE/AE. One target at a time: suspend a humanoid or loose item, move it with your aim, pull/push its hold distance, throw it, slam it, or execute a weakened humanoid. Fast impacts apply native health damage with player attribution, independently of Havok's incidental damage.

**Status:** compiled x64 development build available. Release compilation/linking, the C++ combat math test, and all six Python tool/record checks passed on GitHub's Windows runner. The downloaded DLL exports `SKSEPlugin_Load`, `SKSEPlugin_Query`, and `SKSEPlugin_Version`; it imports only standard Windows system DLLs. The companion and package hashes were checked locally. **Gameplay and SSEEdit/Creation Kit validation have not been performed.** This is a compiled experimental build, with in-game acceptance checks still required.

[Successful build and artifact](https://github.com/schlimbop/telekinesis-mod/actions/runs/37462262493), compiled commit `e88d51a2b88eff628de993ec8546a5afb3c44f2a`, 6 October 2026. Install `TelekinesisCombat-0.1.0.zip` directly; a C++ compiler is needed only if you choose to rebuild the source.

## Contents

- `src/main.cpp`: engine integration, physics controller, keyboard input, impact damage, finisher, config validation, lifecycle cleanup.
- `src/combat_math.h`, `tests/math.cpp`: portable combat math and runnable C++ checks.
- `Data/TelekinesisCombat.esp`: six new forms, Skyrim.esm master, ESL flag, no vanilla overrides.
- `Data/SKSE/Plugins/TelekinesisCombat.ini`: controls, costs, forces, resistance and damage settings.
- `tools/companion.py`: regenerate the ESP from an owned Skyrim.esm without writing to the master.
- `tools/package.py`: source or mod-manager ZIP, hashes, DLL architecture check.
- `tools/inspect_binary.py`: read and check SKSE DLL exports/imports without executing it.
- `.github/workflows/build.yml`: Windows build, tests, package artifact.
- `Papyrus/`: optional cosmetic sound/shader listener; no Papyrus loop or native gameplay dependency.
- `docs/CreationKit.md`: exact records, manual recreation, optional sound integration.
- `docs/TESTING.md`: compilation and in-game acceptance gates.

## Runtime targets and requirements

Target list: SE **1.5.97**, AE **1.6.640 / 1.6.659 / 1.6.1170 / 1.6.1179**, and current Steam **1.7.104**. These are implementation targets, not a tested compatibility matrix. Unknown versions and VR are rejected. CommonLib's version-aware accessors and Address Library relocations are used; there are no hand-coded runtime instruction patches.

Install SKSE and Address Library for your **exact executable version**. The [official SKSE page](https://skse.silverlock.org/) lists Steam 1.7.104 with SKSE 2.3.1, GOG 1.6.1179 with 2.2.6, and SE 1.5.97 with 2.0.20 as of 6 October 2026. Older AE installations need their matching archived SKSE. Address Library must provide every relocation used by this build. Game Pass, Epic, consoles and VR are outside this project's scope.

[CommonLibSSE NG v10.1.0](https://github.com/alandtse/CommonLibSSE-NG/releases/tag/v10.1.0) is pinned in CMake. You need Visual Studio 2022 with current Desktop Development with C++, Windows SDK, CMake 3.24+, Git, vcpkg, and Python 3.10+. spdlog, rapidcsv and DirectXTK are CommonLib build dependencies. The plugin itself adds no configuration-parser dependency.

## Build

Run from this project's directory in a Visual Studio developer PowerShell. Replace the vcpkg path with your installation:

```powershell
cmake -S . -B build -A x64 -DCMAKE_TOOLCHAIN_FILE=C:/vcpkg/scripts/buildsystems/vcpkg.cmake -DVCPKG_TARGET_TRIPLET=x64-windows-static -DCMAKE_MSVC_RUNTIME_LIBRARY=MultiThreaded
cmake --build build --config Release --parallel
ctest --test-dir build -C Release --output-on-failure
python -m unittest discover -s tests -v
python tools/package.py --dll build/Release/TelekinesisCombat.dll
```

The resulting `dist/TelekinesisCombat-0.1.0.zip` contains an ESP, INI, DLL and documentation. Packaging fails if the DLL is missing or is not an x64 Windows DLL. It cannot certify gameplay; complete the acceptance checks before treating a build as playable.

For portable math checks without CommonLib or Skyrim, configure with `-DTKC_MATH_ONLY=ON`, then build and run CTest. To regenerate the companion:

```powershell
python tools/companion.py "C:/Program Files (x86)/Steam/steamapps/common/Skyrim Special Edition/Data/Skyrim.esm"
```

The generator uses the [TES5Edit record definitions](https://github.com/TES5Edit/TES5Edit/blob/dev-4.1.5/Core/wbDefinitionsTES5.pas) for record layouts. It reads vanilla effect templates, copies a small whitelist of record fields, and writes a separate new ESP. It copies no game asset files. Open the ESP in SSEEdit and run **Check for Errors** before an in-game test. The included ESP was structurally checked by the Python validator, not by SSEEdit/Creation Kit.

## Installation and first use

After building and passing checks, install the release ZIP with MO2 or Vortex. Enable `TelekinesisCombat.esp` and launch through SKSE. Manual layout under your game's `Data` directory:

```text
TelekinesisCombat.esp
SKSE/Plugins/TelekinesisCombat.dll
SKSE/Plugins/TelekinesisCombat.ini
```

The companion uses the ESL flag and old-runtime-safe IDs 0x800–0x805. No FormID compaction is needed. Never compact or renumber it after installation; if rebuilding it manually, update the INI to the resulting local IDs.

For this first version, acquire the control spell through the console:

```text
help "Combat Telekinesis" 4
player.addspell <the SPEL FormID shown by help>
```

Equip **Combat Telekinesis** in either hand. The hotkeys are the casting controls; the mouse spell-cast buttons only play the harmless control effect. Vanilla Telekinesis remains available separately. No vendor injection or auto-granted spell is included.

| Key | Action |
| --- | --- |
| G | Aim at a target to grab; press again to safely drop |
| H | Launch in the camera/crosshair direction |
| J, held | Pull the held target nearer |
| K, held | Push the held target farther while retaining the grip |
| L | Release in a high-speed downward slam |
| X, held | Charge crush for 1.5 seconds on a humanoid at ≤25% health |

Aim moves the suspension anchor. H is the forward throw/repulse action. Pull/push are distance controls while gripping, not area blasts. Allow a brief moment for the target's ragdoll to form before throwing. There is no multi-target arsenal, disarm, anatomical destruction or custom killmove animation in this version.

## Configuration and gameplay

The INI uses Windows profile syntax. Close the game before editing it; configuration is loaded at DataLoaded. Controls are DirectInput **keyboard scan codes**, not virtual-key numbers. Defaults are G=34, H=35, J=36, K=37, L=38, X=45. Duplicate controls reset to defaults. Invalid numbers fall back to defaults and are logged; valid values are bounded. Gamepad mappings are not implemented.

Costs default to 15 Magicka on grab, 12/second to hold, 10 to launch/slam and 40 to execute. Actor costs multiply by `clamp(scale³ × (1 + positive level difference × LevelResistance), 1, 8)`. Object holds use the base cost. Insufficient Magicka drops the target or rejects the action. This is manual cost accounting; Alteration cost-reduction perks and XP progression are not yet integrated.

Only humanoids (`ActorTypeNPC`, optionally `ActorTypeUndead`) are eligible. Children, essential actors, dragons, giants, killmove actors, over-scale/over-height actors and already-paralyzed actors are rejected. Protected actors are excluded by default. Exclusions also apply to native damage where relevant. Custom races without these keywords cannot be gripped; extremely unusual skeletons may fail the ragdoll check and be released. Dragons/large creatures can still be struck by a launched object, but cannot be suspended.

Objects must already have dynamic collision bodies and be loose weapons, armor, miscellaneous items, ingredients or potions. Doors, furniture, statics, containers, equipped items and objects in inventories are excluded. Taking an object for telekinesis does not silently transfer ownership. Normal Havok collisions/ownership behavior remains active.

`LaunchSpeed` and `SlamSpeed` are Skyrim world units/second; they are converted to Havok units using engine world-scale values. `SpringGain` controls responsiveness. The hold uses bounded linear velocities and never repeatedly teleports the actor reference. Swords/daggers receive approximate +Z-forward alignment; axes/maces and multi-body/custom meshes tumble normally. Disable `OrientSwords` for incompatible meshes.

Damage is a tunable gameplay formula: `mass × speed² × multiplier / 1,000,000`, capped by `MaxImpactDamage`, with a minimum speed. Actor mass is a configurable proxy; object mass uses inventory weight, clamped to the object cap. Weapons get a 2× multiplier. This is not physically calibrated kinetic energy. Each launch can apply native damage **once**, at its first qualifying impact, to an actor struck and to the thrown actor itself. Sweeps use previous/current physics positions, torso bounds and obstruction rays; rapid directional velocity loss also detects walls/floors. Havok may additionally apply incidental damage.

Executions use health damage, a stock magical visual/sound effect, a notification and a mod event. They do not simulate a skull or modify head meshes. Optional Papyrus can add a more suitable crack sound and shader; see CreationKit.md.

## Stability and compatibility

Input and all game-object access execute on the game thread. A timer thread queues at most one task and touches no Skyrim objects. Havok queries/mutations use the cell's world lock. Bodies are discovered afresh; only reference handles are retained across updates. Flight tracking is bounded. A menu pause, load, major hitch, cell transition, unequip or failed handle ends a hold. Dropping removes this plugin's suspension spell without dispelling another mod's paralysis.

Suspension is a dedicated two-second paralysis spell, refreshed while controlled. After an interruption or save/load, its native effect can persist for up to two seconds and then expires; no permanent paralysis AV edits, serialized native handles, gravity overrides or disabled AI are stored. A throw refreshes paralysis only during its short tracked flight. There is no guarantee that a custom animation/physics stack will recover correctly until tested.

Avoid using Better Telekinesis or another ragdoll controller on the **same target**. Test separately with Precision, PLANCK-like actor physics changes, animation behavior replacements, custom skeletons and mods that alter paralysis recovery. A vanilla-effect overhaul should not replace these private effects, although broad runtime patches may still influence them. The ESP makes no vanilla overrides and needs only Skyrim.esm.

Known prototype limits: torso approximations rather than exact weapon/limb contact; actor movement can create false positives; a center ray can miss grazing environmental contacts; a speed-loss check can confuse third-party forces with collisions. Sweeps reduce tunnelling but do not enable continuous Havok collision for every weapon shape. Small items may still pass through thin meshes visually at high speed. Reduce speeds if needed. Native player damage attribution is provided; full crime reporting, kill XP and perk interactions need in-game verification. No claim of established stability or measured frame-time performance is made.

If initialization fails, inspect `TelekinesisCombat.log` in the SKSE log directory. Missing companion spells disable gameplay and produce a notification. Never delete a required Address Library error to force an unsupported executable to run.

For removal, drop held targets, wait at least ten seconds for tracked effects to expire, and save away from combat. Disable the plugin/ESP together in a test profile. Keep the original save when evaluating this development build.
