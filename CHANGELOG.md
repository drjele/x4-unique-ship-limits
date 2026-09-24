# Changelog

All notable changes to this project will be documented in this file.

## [Unreleased]

## [v1.1.0] - 2026-09-24 - Debug logging toggle

### Changed

- Detailed Steam Workshop description, covering every setting, where to change it, and the required and optional dependencies.

### Added

- **Debug logging** toggle in Extension Options: switches the mod's debug log output on and off in game, instead of editing `$DebugChance` and reinstalling.
- `publish.sh update` options `--minor`, `--namedesc` and `--readback`, for an update that leaves the version number alone, one that also pushes the name and description to Steam, and one that writes Steam's own text back into `content.xml.steam`.

### Fixed

- `publish.sh` passes `-batchmode`, so an upload no longer waits for a keypress a non-interactive run cannot give it.
- `publish.sh` runs the workshop tool inside the Steam snap's mount namespace. The snap has a private `/tmp`, so the Steam client IPC the tool needs is unreachable from outside it and the upload failed on a Steamworks assertion.
- `publish.sh` shows the tool's output, which Proton otherwise discards, and reads success or failure out of it rather than out of an exit code Proton does not pass on.
- `publish.sh` restores the local installation even when the upload fails, instead of leaving it holding the staged copy with the Workshop id in it.
- `publish.sh update` sends the preview image too, so a refreshed `extension/preview.jpg` reaches the Workshop item instead of leaving the one from the first upload in place.

## [v1.0.0] - 2026-09-07 - Initial release

### Added

- **Astrid unlimited** switch, writing `limited_blueprint_ship_gen_m_yacht_01_a`.
- **Experimental Shuttle unlimited** switch, writing `limited_blueprint_ship_ter_s_xperimental_01_a`.
- **Boron story corvette unlimited** switch, writing `limited_blueprint_ship_bor_m_corvette_02_a`.
- **Timelines racers unlimited** switch, writing all four racer counters at once.
- **Erlking stays buildable** switch, restoring `research_erlking_core` after a build is queued and on every savegame load, so more than one can be ordered.
- In-game options menu through SirNukes Mod Support APIs, with file constants and global overrides as the fallback when it is not installed.
- Existing-save support: the counters are profile-level userdata, applied on every `md.Setup.Start` rather than patched into a cue that has already run.

### Verified

- All seven counters written on the first load after installing, on a savegame several hundred hours old.
- A second Astrid (`THG-786`) and a second Erlking (`HAV-485`) ordered at a player wharf and finished, alongside the ones already owned.
- `research_erlking_core` back in the completed research list after the Erlking build, so a third can still be ordered.

### Notes

- Every switch is off by default and off is the vanilla value, so the mod changes nothing until one is turned on.
- The counter has no "no limit" value, so a lifted cap is set to 1000. `$UnlimitedLimit` in the configuration cue changes what that number is, for anyone who wants an exact cap rather than none.
- No blueprint is granted. Each ship still needs the blueprint its story hands over, and the Erlking switch only acts once the player already holds the Erlking blueprint.
- The counters live in the profile's `userdata.xml`, not in the savegame: a raised limit applies to every save on the profile and survives uninstalling the mod. See the README for how to put it back.

[v1.1.0]: https://github.com/drjele/x4-unique-ship-limits/releases/tag/v1.1.0
[v1.0.0]: https://github.com/drjele/x4-unique-ship-limits/releases/tag/v1.0.0
