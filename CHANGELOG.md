# Changelog

All notable changes to this project will be documented in this file.

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

[v1.0.0]: https://github.com/drjele/x4-unique-ship-limits/releases/tag/v1.0.0
