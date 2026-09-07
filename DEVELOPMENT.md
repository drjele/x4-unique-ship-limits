# Development

## Checks and formatting

Use Python 3.10 or newer and Bash. Install the pinned tools in a virtual environment:

```bash
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements-dev.txt
export PATH="$PWD/.venv/bin:$PATH"
python3 scripts/check.py
```

Run `python3 scripts/check.py --fix` to format Python and shell and normalize text whitespace. The same checks run on pushes and pull requests. XML is checked for well-formedness; game schemas, XPath matches and gameplay require separate X4 validation. Blender scripts are parsed and linted without importing Blender.

Use UTF-8, LF, a final newline, spaces and no trailing whitespace. Indent code with four spaces and workflow YAML with two. Use descriptive names, uppercase shell variables, constant-first equality comparisons and explicit boolean checks. Ruff's E712 rule is disabled to retain explicit boolean comparisons. Keep shell free of prose comments. Keep only short, non-obvious constraints in code; put explanations here. XML continuation attributes may align with their opening attribute. Preserve XPath selectors, savegame identifiers and embedded game expressions when applying formatting.

## Installation and publishing helpers

`install.sh` and `publish.sh` both source `lib/find_x4.sh`. The library searches usual Steam roots and additional library folders. `X4_PATH`, `X_TOOLS_PATH`
and `PROTON_PATH` override discovery. Proton Experimental is preferred when found; otherwise the helper uses the last matching Proton directory it encounters.

Installation replaces the extension directory with a copy of `extension/`. Refresh it after edits; X4 enumerates real extension directories, so a symlink does not substitute for installation. Restart the game after installing or removing.

Publishing stages a separate copy inside the game's extensions directory. The repository keeps its readable extension id; `steam/workshop-id` holds the numeric Workshop id. The helper changes only the staged manifest, runs the interactive WorkshopTool and restores the manual installation after success. On Linux it runs WorkshopTool through Proton and maps paths through drive Z. A failed upload can leave the staged copy behind; rerun `./install.sh` to restore it.

## Release metadata

`content.xml` uses an integer version multiplied by 100 and an ISO release date. The date matches the corresponding released entry in `CHANGELOG.md`. Development changes belong under `Unreleased`; they do not advance the manifest's release version or date. An unreleased scaffold may retain its initial creation date until its first release. Keep existing extension ids stable.

## Implementation constraints

### extension/md/drjele_unique_ship_limits.xml

Configuration. Re-read on every savegame load, so editing a value below and reloading is enough - no new game required. Each one can also be overridden at runtime without touching this file, by setting the matching global variable:
&lt;set_value name="global.$DrJeleUniqueShipAstridUnlimited" exact="1"/&gt; Those globals are also what the in-game options menu writes - see drjele_unique_ship_limits_options.xml.

Nothing here patches a vanilla file. Every `limited` ship ware is capped by one profile-level counter, `limited_blueprint_<ware id>`, which the ship and station configuration menus read with GetUserDataSigned and compare against the ships the player currently owns. Writing that counter is the entire mechanism, so the mod is save-compatible in both directions and cannot collide with another mod's XPath.

signed="true" on every write is mandatory, not stylistic. The menus read the key back with GetUserDataSigned; a key written unsigned reads as empty, and the lua's `tonumber(...) or 0` turns that into a limit of zero, which locks the ship out entirely instead of freeing it.

Counters whose DLC is not installed are written anyway. The name is only a string, so an absent ware costs nothing. The Erlking path is the one place that has to resolve real wares, and both lookups there carry @ so the script stays quiet without Tides of Avarice.

The apply cue is delayed by five seconds for two separate reasons. Configuration hangs off the same md.Setup.Start signal and the order between cues on one signal is not defined, so an immediate apply would read an unpublished table roughly half the time. And the DLC setup scripts reset these counters to 1 from &lt;patch sinceversion="N"&gt; blocks, which run once when a savegame written against an older script version is loaded - re-applying after them on every load is what keeps a future game update from quietly restoring the cap.

Every setting is looked up the same way: the global the options menu writes wins, otherwise the table Configuration publishes, otherwise the vanilla literal. All of them are global variables, so a configuration that fails to load degrades to the vanilla limit of 1 rather than to 0.

The four ship settings are switches, not numbers, because the counter has no "no limit" value - a lifted cap is just a number nothing will reach. $UnlimitedLimit holds that number, in the configuration cue rather than in the menu, so someone who wants an exact cap instead of none has one place to change and the menu stays a set of yes/no questions.

The Erlking cue names the macro directly, so it hangs off a parent that only completes when @ware.ship_pir_xl_battleship_01_a resolves. Without Tides of Avarice the child never starts listening and the macro reference is never evaluated. The parent is deliberately not instantiated: it completes once and its child keeps listening from then on, where instantiating would add a duplicate listener on every savegame load.

The Erlking is a different mechanism and gets its own path. Its ware carries no counter; instead Story_Research_Erlking.Player_Erlking_Build_Added removes research_erlking_core the instant a build is queued, and the ware needs that research as a production precursor. Restoring it is exactly what vanilla's own Player_Erlking_Build_Cancelled does, so this mod runs the same two actions - on the same event, with a delay, because both cues trigger off event_player_build_added and the order between two scripts is not defined.

The blueprint check is the story gate and must stay. player.blueprints.{$ErlkingWare}.any.exists is only true after the story has handed the blueprint over, so the switch can never let a player build an Erlking they were not already entitled to.

The guards are nested rather than combined with `and` because MD does not promise to short-circuit; with $ErlkingWare null, a single combined condition would still evaluate player.blueprints.{null}.

The delay is one second, not five. Vanilla removes the research inside the event handler, so any delay at all gives the ordering; the length only sets how wide a window a save can land in and store the research as still missing. A five second delay was measured being beaten by a save taken 2.65 seconds after the build was queued. Nothing is lost when that happens - the load-time Apply calls the same library - but the shorter window is worth having.

If the delayed cue turns out to miss a route to queueing an Erlking, the fallback is a poll on the same three-way guard, as a cue with instantiate="true" checkinterval="1min" - never as a cue that resets itself, because checkinterval does not throttle those: after a reset the first condition check happens immediately.

### extension/md/drjele_unique_ship_limits_options.xml

In-game options, through SirNukes Mod Support APIs. Everything that touches that API lives in this file, so if the API is not installed nothing here ever runs and the mod keeps working off the constants in drjele_unique_ship_limits.xml.

All five callbacks only ever write the global override variables the rest of the mod already reads, so the apply step knows nothing about menus. They normalize the widget's value to 1 or 0 rather than passing it through, so the apply step compares against integers whatever the API hands back.

The API stores an option's value in the savegame under its $id, and hands it straight back to the widget as its start value. A stored value that no longer fits the widget fails validation and takes the whole Extension Options menu down with it, so the type and range of an option must never change under a $id that has already shipped - give it a new one instead. The ids here say what they are rather than what they were during development, and the four ship ones were renamed when they stopped being sliders.

Every callback signals Reapply, because the counters have to be rewritten for the change to reach the menus; the Erlking switch does too, so that turning it on restores the research immediately rather than at the next load.
