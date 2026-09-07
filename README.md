# Unique Ship Limits for X4: Foundations

<p align="center">
  <img src="extension/preview.jpg" alt="Unique Ship Limits" width="512">
</p>

A handful of ships in X4 are capped at one. The Astrid is the blunt case — the game tells you so in as many words: *"There is no way to manufacture another Astrid-class ship once the ones already in existence are destroyed."* Lose it and the wharf entry stays greyed out with a padlock, blueprint or no blueprint.

The cap turns out to be one number. Every `limited` ship ware has a counter in your profile called `limited_blueprint_<ware>`, the ship and station configuration menus read it back with `GetUserDataSigned` and compare it against the ships you currently own, and nothing else enforces it anywhere. This mod puts a switch on that number.

The Erlking works differently and gets its own switch — see below.

**Requires X4: Foundations 9.00.** No DLC required, and no hard dependency on other mods. Works on an existing savegame. Every default is the vanilla value, so installing it changes nothing until you turn a switch on.

## Install

```bash
./install.sh
```

The helper copies `extension/` into the game's `extensions/<extension-id>`
directory, using the id in `extension/content.xml`. It searches the usual Steam layouts and additional library folders. To choose an installation:

```bash
X4_PATH="/path/to/X4 Foundations" ./install.sh
```

Restart X4 after installing or updating. To remove the manual installation:

```bash
./install.sh --uninstall
```

## The settings

Five switches, all off by default, all off meaning vanilla.

| Switch                             | Ships it covers                                                                                                      |
|------------------------------------|----------------------------------------------------------------------------------------------------------------------|
| **Astrid unlimited**               | `ship_gen_m_yacht_01_a` — the Northriver yacht from Tides of Avarice                                                 |
| **Experimental Shuttle unlimited** | `ship_ter_s_xperimental_01_a` — from the Timelines epilogue                                                          |
| **Boron story corvette unlimited** | `ship_bor_m_corvette_02_a` — the Kingdom End story reward                                                            |
| **Timelines racers unlimited**     | all four at once: `ship_arg_s_racer_01_a`, `ship_gen_s_racer_01_a`, `ship_par_s_racer_01_a`, `ship_tel_s_racer_01_a` |
| **Erlking stays buildable**        | `ship_pir_xl_battleship_01_a` — a different mechanism entirely, see below                                            |

There is no "no limit" value for the counter, so a lifted cap is simply a number nothing will reach — 1000. If you would rather cap a ship at some exact number instead of removing the cap, change `$UnlimitedLimit` in the configuration cue of `extension/md/drjele_unique_ship_limits.xml`; the switches then set that number instead of 1000.

The cap counts ships you *currently own*, not ships you have ever built, so under vanilla rules losing one already frees the slot — the switch is for the case where you want two at the same time, or where the ship is gone for good.

**None of this hands you a blueprint.** Every one of these ships still needs the blueprint its story gives you, and the Erlking still needs its own. If you have not done the story, this mod does nothing for that ship.

### The Erlking

The Erlking has no counter. Its ware lists `research_erlking_core` as a production research precursor, and the vanilla story script takes that research away the moment you queue a build:

```xml

<remove_encyclopedia_entry type="researchables" item="'research_erlking_core'"/>
<remove_research ware="ware.research_erlking_core"/>
```

So the first one is free and the second is impossible until you go and recover the power core again. With the switch on, this mod puts the research back — the same two actions vanilla itself runs when you *cancel* an Erlking build — a few seconds after the build is queued, and again on every savegame load. You keep as many as you can pay for.

It only ever does that once you already hold the Erlking blueprint, so nothing about the story up to that point changes.

## How it works

No vanilla file is patched. The whole mod is two MD scripts and five `set_userdata` calls, which is why it is save-compatible in both directions and cannot collide with another mod's XPath.

The applied value is written on every `md.Setup.Start` — a new game and every load — after a short delay. The delay is doing two jobs. The configuration cue hangs off the same signal and the order between cues on one signal is not defined, so without it the apply step would read an unpublished table about half the time. And the DLC setup scripts reset these counters to 1 from `<patch sinceversion="N">` blocks, which run once when a savegame written by an older script version is loaded; re-applying afterwards on every load means a future game update quietly putting the cap back does not survive the next reload.

`signed="true"` on the write is not optional. The menus read the key with `GetUserDataSigned`, and a value written unsigned reads back as nothing, which `tonumber(...) or 0` turns into a limit of **zero** — the ship would be locked out completely.

Every setting is read defensively — the global the options menu writes, else the table the configuration cue publishes, else the vanilla literal — so a configuration that fails to load degrades to stock behaviour rather than to zero.

Ships whose DLC you do not own are still written. The key is only a string, so writing `limited_blueprint_ship_bor_m_corvette_02_a` without Kingdom End installed costs nothing and means nothing. The Erlking path is the one place that has to look up real wares, and both lookups there are guarded with `@`.

## Where the value actually lives

**Not in your savegame.** `set_userdata` writes to `userdata.xml`, next to your saves — the same profile-level store that holds your custom gamestart unlocks:

```
$HOME/.config/EgoSoft/X4/<userid>/userdata.xml
```

Two consequences worth knowing before you turn a switch on:

- a raised limit applies to **every save on this profile**, not just the one you were playing;
- it **survives uninstalling the mod**, because an uninstalled mod gets no chance to put it back.

To go back to stock: turn every switch off, load a save so the apply step runs, save, and only then remove the extension. Or edit the seven `limited_blueprint_*` lines in `userdata.xml` by hand with the game closed.

## Configuring without the menu

The in-game switches need [SirNukes Mod Support APIs](https://steamcommunity.com/sharedfiles/filedetails/?id=2042901274); without it the mod runs off the constants at the top of `extension/md/drjele_unique_ship_limits.xml`, which are re-read on every savegame load. Editing one and reloading is enough — no new game.

Each can also be overridden at runtime without touching the file:

```xml

<set_value name="global.$DrJeleUniqueShipAstridUnlimited" exact="1"/>
<set_value name="global.$DrJeleUniqueShipExperimentalShuttleUnlimited" exact="1"/>
<set_value name="global.$DrJeleUniqueShipBoronCorvetteUnlimited" exact="1"/>
<set_value name="global.$DrJeleUniqueShipTimelinesRacerUnlimited" exact="1"/>
<set_value name="global.$DrJeleUniqueShipErlkingRebuildable" exact="1"/>
```

Then signal `md.DrJele_UniqueShipLimits.Reapply`, or just reload.

Set `$DebugChance` to 100 in the configuration cue to have every applied change written to the debug log.

## Debugging

Add this to the game's launch options — Steam, right click X4, **Properties → General → Launch Options**:

```
-debug all -logfile debuglog.txt
```

The log lands next to your savegames: `$HOME/.config/EgoSoft/X4/<userid>/debuglog.txt` on Linux, `Documents\Egosoft\X4\<userid>\debuglog.txt` on Windows. If Steam is installed as a snap it runs the game with a redirected home, which puts both under `~/snap/steam/common/`.

The mod is silent by default. Set `$DebugChance` to `100` in the configuration cue of `extension/md/drjele_unique_ship_limits.xml`, re-run `./install.sh` and restart, and every applied change is written out:

```
DrJele Unique Ship Limits: astrid 1000, experimental shuttle 1, boron corvette 1, timelines racers 1
DrJele Unique Ship Limits: erlking research restored
```

The faster check needs no log at all. With the game closed:

```bash
grep limited_blueprint "$HOME/.config/EgoSoft/X4/"*/userdata.xml
```

Those seven lines are the entire effect of this mod: `1` is vanilla, `1000` is a lifted cap.

## Status

**Verified in game on 9.00**, on a savegame several hundred hours old, with all five switches on. The mechanism itself is read straight out of the shipped files rather than inferred:

| Claim                                                   | Where it comes from                                                                                                                   |
|---------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------|
| the cap is `limited_blueprint_<ware>` and nothing else  | `ui/addons/ego_detailmonitorhelper/helper.lua`, and its four callers in the two config menus                                          |
| vanilla writes it with `set_userdata ... signed="true"` | `ego_dlc_pirate/md/setup_dlc_pirate.xml`, `ego_dlc_boron/md/setup_dlc_boron.xml`, `ego_dlc_timelines/md/story_timelines_epilogue.xml` |
| it counts ships owned, not ships built                  | `menu_ship_configuration.lua` fills `usedLimitedMacros` from `C.GetUsedLimitedShips()`                                                |
| the Erlking is gated by research, not by a counter      | `ego_dlc_pirate/md/story_research_erlking.xml`, cues `Player_Erlking_Build_Added` and `Player_Erlking_Build_Cancelled`                |

What the run confirms:

| Check             | Result                                                                                                                                                                |
|-------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Applied on load   | all seven counters went from `1` to `1000` in `userdata.xml` on the first load after installing                                                                       |
| Astrid buildable  | a second Astrid, `THG-786`, was ordered at a player wharf and finished, alongside the one already owned — the case the game's own flavour text calls impossible       |
| Erlking buildable | a second Erlking, `HAV-485`, was ordered and finished, and `research_erlking_core` is back in the completed research list afterwards, so a third can still be ordered |
| Script errors     | none from this mod                                                                                                                                                    |
| Save independence | the mod does not appear in the savegame's `<patches>` list, because `content.xml` declares `save="0"` — the save does not depend on it                                |

Two things the run does **not** separate. `RestoreErlkingResearch` is called both from the build event and from the load-time apply, and a savegame cannot say which of the two fired — what is proven is that the library and its guards work, not that the one second delay on `ErlkingBuildQueued` is enough on its own. Isolating that needs a run with `$DebugChance` at 100.

An earlier attempt at this check was inconclusive for a related reason: the save was taken 2.65 seconds after the build was queued, inside what was then a five second delay, so it recorded the research as still stripped. The delay is now one second. A save inside that window is not a failure — the next load restores it — but it makes the check meaningless.

Also still unknown is whether every route to queueing an Erlking raises `event_player_build_added`. If it does not, the fallback is a poll on the same guard.

One log line worth noting, 1.5 seconds before the Astrid build was queued:

```
[=ERROR=] GetMissingLoadoutBlueprints(): Failed to retrieve defensible with ID '0' and macro with name ''.
```

It appears once, in the ship configuration menu's own code. This mod patches no lua and cannot call that function, so at most it unlocked a menu path that was never reachable before; the purchase itself went through and the ship is building. Not reproduced since.

## Publishing to the Steam Workshop

Install **X Tools** (Steam app 282160) and keep Steam running and logged in with an account that owns X4. On Linux, install Proton as well; on Windows, run the helper from Git Bash, MSYS or Cygwin.

```bash
./publish.sh publish
./publish.sh update "what changed"
```

Use `publish` once, then `update` with a change note. `X4_PATH`,
`X_TOOLS_PATH` and `PROTON_PATH` override automatic discovery. The staging location must contain an `extensions` directory.

The first upload records the numeric id in `steam/workshop-id`; retain that file for future updates. The readable id in the repository's `content.xml`
stays unchanged. After publishing, open the printed Workshop URL, complete any required Steam agreement and choose the item's visibility. Avoid keeping both the manual installation and a subscription to the same mod enabled.

Update the manifest version and release date together with `CHANGELOG.md`
when releasing. See [Development](DEVELOPMENT.md) for staging, platform and release conventions.

## Development

See [DEVELOPMENT.md](DEVELOPMENT.md) for setup, code style, validation and release conventions.

## Legal

MIT, see [`LICENSE`](LICENSE). Non-commercial fan project; X4: Foundations belongs to Egosoft GmbH and this project is not affiliated with or endorsed by Egosoft.
