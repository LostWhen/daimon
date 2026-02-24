# Daimon

**Army List → UnitCrunch Converter for Warhammer 40,000 (10th Edition)**

**[→ Use it now](https://lostwhen.github.io/daimon/)**

Daimon takes a `.json` army list exported from [ListForge](https://listforge.club), [New Recruit](https://newrecruit.eu), or any app that uses the BattleScribe roster schema and converts it into an import file for [UnitCrunch](https://www.unitcrunch.com), the 40K probability and damage simulator.

No server, no account, no data leaves your browser. Drop a file in, get a file out.

---

## Quick Start

### Option A: One-Click from ListForge
1. Build your army list in [ListForge](https://listforge.club)
2. Click the **Daimon** export button (ListForge handles the rest)
3. Daimon opens, converts your list, and auto-downloads a `.txt` file
4. In UnitCrunch, click **Import** and select the `.txt` file
5. All your units appear as profiles, ready to sim

### Option B: File Upload
1. Export your army list as `.json` from ListForge, New Recruit, or another BSData-compatible app (see [Compatibility](#compatibility))
2. Open [Daimon](https://lostwhen.github.io/daimon/)
3. Drop the `.json` file onto the page (or click to browse)
4. A `.txt` file downloads automatically
5. In UnitCrunch, click **Import** and select the `.txt` file
6. All your units appear as profiles, ready to sim

---

## Compatibility

Daimon reads any JSON file that follows the [BattleScribe roster schema](http://www.battlescribe.net/schema/rosterSchema).

| App | Export Format | Works with Daimon |
|---|---|---|
| ListForge | JSON (+ native deep-link) | ✓ Tested extensively — one-click integration available |
| New Recruit | JSON | ✓ Tested — see [Validation](#validation) |
| BattleScribe | XML (.ros / .rosz) | ✗ Not supported — BattleScribe exports XML, not JSON |

If you test Daimon with an app not listed here, please open an issue with your results.

---

## What Gets Converted

Everything that UnitCrunch can simulate is extracted and mapped automatically.

**Defensive Stats** — Toughness, Save, Wounds, and Invulnerable Save from unit stat profiles. Mixed stat-line units (e.g., Exarch + Aspect Warriors, Nob + Boyz) are split into separate model types with correct wound values and model counts.

**Weapons** — All ranged and melee weapon profiles with Attacks, BS/WS, Strength, AP, and Damage. Dice expressions (D6, 2D6, D6+3) are preserved as strings. Units with different loadout options that produce identical stat lines are deduplicated.

**Weapon Abilities** — Each keyword on a weapon profile maps to the corresponding UnitCrunch modifier: Sustained Hits (any value), Lethal Hits, Devastating Wounds, Twin-linked, Torrent, Heavy, Rapid Fire (any value), Anti-\* (2+–6+), Blast, Indirect Fire, Lance, Melta (2–4), Ignores Cover, Pistol, Assault, Hazardous, Precision, Psychic.

**Unit Abilities** — Parsed from ability description text on the datasheet:

| Ability | How It's Mapped |
|---|---|
| Invulnerable saves | Set on model type; extracted from Invuln profiles and description text; distinguishes bearer-only vs unit-wide |
| Feel No Pain | Standard FNP and conditional (psychic, mortal wounds) |
| Rerolls (hit/wound) | Reroll 1s and reroll all failed; leader aura patterns detected |
| Stat modifiers | +1/−1 to hit, wound, save, damage, strength, toughness |
| Critical hit/wound effects | Threshold changes (5+), extra hits on crits, lethal/devastating on crits |
| Damage reduction | −1 damage, halve damage |
| Benefit of Cover | Detected from description text |
| Conditional abilities | "Once per battle" / "until end of phase" → start inactive; user toggles on in UC |
| Leader auras | "While this model is leading a unit" → start active |
| Pre-bake annotations | Equipment and conditional ability bonuses baked into stats by BSData → flagged as informational notes |

---

## What You'll Need to Set Up Manually

Some abilities are game-state-dependent and can't be automatically converted. After importing, you may want to configure these in UnitCrunch by hand:

- **Detachment rules** — Daimon extracts unit data, not detachment-wide rules (e.g., War Horde's Sustained Hits on charge, Seer Council's Strands of Fate). Add these as global modifiers in UC.
- **Stratagems** — Not parsed. Apply as needed per matchup in UC.
- **Mortal wound output** — Abilities that inflict mortal wounds (on charge, on death, etc.) aren't mapped since UC models attack sequences, not event triggers.
- **Movement and deployment** — Deep Strike, Advance bonuses, transport capacity — not relevant to damage simulation.
- **Board-state auras** — "While within 6″ of this model" effects require positioning context UC doesn't have.
- **Enhancements** — Enhancement points are correctly included in profile totals. Stat modifications from enhancements are detected but suppressed because BSData bakes enhancement bonuses directly into weapon and unit stat profiles already. Enhancement ability descriptions are preserved for reference.

---

## Validation

### Systematic Verification (Phase 4)

Automated field-by-field verification against Wahapedia CSV data exports using a custom verification harness. Every converted profile is compared against Wahapedia's ground truth for unit stats (M/T/Sv/W/LD/OC/Invuln), weapon profiles (A/BS/WS/S/AP/D), weapon keywords, and parsed abilities. Cross-builder validation confirmed both ListForge and New Recruit produce identical converter output for the same units.

| Faction | Profiles Tested | Matched | Discrepancies | Converter Bugs |
|---|---|---|---|---|
| Custodes | 31 | 31/31 (100%) | 2 | 0 |
| Aeldari | 61 | 61/61 (100%) | 18 | 0 |
| Necrons | 51 | 51/51 (100%) | 6 | 0 |
| Drukhari | 37 | 34/37 (92%) | 29 | 0 |
| Chaos Daemons | 74 | 74/74 (100%) | 17 | 0 |
| Space Marines | 91 | 91/91 (100%) | 20 | 0 |
| Chaos Space Marines | 55 | 55/55 (100%) | 17 | 0 |
| **Total** | **400** | **397/400 (99%)** | **109** | **0** |

**0 converter bugs found across 400 profiles, 7 factions, 2 list builders.**

All 109 discrepancies were classified into three categories — none are converter errors:
- **BSData↔Wahapedia source disagreements** — BSData and Wahapedia occasionally differ on specific stat values. BSData is typically more current, reflecting recent dataslate and FAQ updates before Wahapedia catches up.
- **BSData pre-baked conditional bonuses** — BSData bakes certain loadout-dependent stat modifiers (e.g., mark-specific buffs, conditional attack bonuses) directly into weapon profiles. These appear as "wrong" values against Wahapedia's base stats but are correct for that specific loadout.
- **Harness matching limitations** — Cross-faction units, dual-profile weapon sub-profile selection, and multi-character mixed stat lines cause false positives in the automated comparison.

The 3 unmatched profiles in Drukhari were units that exist in the BSData catalogue but are absent from Wahapedia's CSV data — the harness correctly flagged these as unverifiable rather than erroneously matching them to wrong units.

### UnitCrunch Round-Trip Verification

Full round-trip (source JSON → Daimon → UnitCrunch import → UnitCrunch export → structural diff, zero differences):

| Faction | Detachment | Profiles | Weapons | Abilities |
|---|---|---|---|---|
| Tyranids | Subterranean Assault | 10 | 21 | 22 |
| Tyranids | Invasion Fleet | 9 | 16 | 24 |
| Tyranids | Invasion Fleet | 11 | 17 | 27 |
| Chaos Daemons | Scintillating Legion | 8 | 19 | 10 |
| Custodes | (full faction) | 31 | — | — |
| Necrons | (full faction) | 51 | — | — |

### Stat Audit Verification

Imported successfully into UnitCrunch (all files accepted by UC's format validator; source JSON stat audit confirmed zero discrepancies in stats, weapons, and invulns):

| Faction | Detachment | Profiles | Weapons | Abilities |
|---|---|---|---|---|
| Tyranids | Subterranean Assault | 10 | 19 | 21 |
| Emperor's Children | Coterie of the Conceited | 11 | 41 | 14 |
| Blood Angels | Rage-Cursed Onslaught | 11 | 43 | 28 |
| Orks | War Horde | 18 | 71 | 28 |
| Aeldari | Seer Council | 16 | 62 | 19 |
| Space Wolves | Stormlance Task Force | 13 | 32 | 33 |

Combined: 117 profiles, 341 weapons, 226 abilities across 7 factions. Zero data errors found.

### New Recruit Compatibility Testing

v0.73.22 — 7 NR exports covering 7 factions, all invulnerable saves cross-referenced against Wahapedia:

| Faction | Source | Profiles | Weapons | Abilities | Invulns |
|---|---|---|---|---|---|
| Chaos Daemons | NR | 11 | 20 | 6 | 11/11 |
| Blood Angels | NR | 11 | 27 | 28 | 4/4 |
| Death Guard | NR | 12 | 26 | 23 | 8/8 |
| Grey Knights | NR | 8 | 19 | 12 | 4/4 |
| Imperial Knights | NR | 6 | 27 | 14 | 6/6 |
| Tyranids | NR | 13 | 25 | 24 | 4/4 |
| Orks | NR | 15 | 44 | 26 | 15/15 |

Combined NR: 76 profiles, 188 weapons, 133 abilities, 52 invulnerable saves. Zero errors. Enhancement points (9 units) correctly summed from upgrade children. Enhanced unit variants correctly deduplicated as separate profiles.

---

## Technical Details

- Single HTML file (~102KB), zero external dependencies
- Runs entirely client-side in the browser
- Output uses msgpack binary encoding matching UnitCrunch's native import/export format
- `appVersion` field set to `0.73.17` (UC validates this against known version strings; arbitrary values cause silent rejection)
- `exported` timestamp encoded as msgpack uint64 (`0xCF`) — UC's parser is type-strict and rejects float64 (`0xCB`) encoding for this field
- ListForge deep-link uses URL hash fragment (`#/listforge-json/`) — encoded data never leaves the browser
- Deep-link decoding uses native browser `DecompressionStream('gzip')` API — no external decompression libraries

---

## Name

**Daimon** — from Walter Jon Williams' novel *Aristoi* (1992). The Aristoi partition their consciousness into *daimones*: specialized sub-processes that each handle a single task autonomously. This tool is a daimon — one job, one transformation, done.

---

## Version History

### v0.74.1
**Pre-bake annotation system (Phase 1).**

BSData sometimes pre-bakes equipment or conditional ability bonuses directly into unit stat lines. Users running simulations in UnitCrunch had no way to know which stats included these bonuses vs. base datasheet values. Daimon now detects pre-baked stats and adds informational annotations to each affected profile.

What gets detected:
- **Equipment stat modifications** — Invulnerable saves from shields/wargear, Leadership from Daemonic Icons, Wounds from Relic Shields, Attacks from god-specific marks (Tzeentch +3A)
- **Equipment weapon keyword grants** — Icon of Flame granting Ignores Cover, Gitfinder Gogglez granting Ignores Cover
- **Once-per-battle conditional abilities** — Abilities that temporarily modify stats or grant weapon keywords for one phase only (Distraction Grot, Ammo Runt, Finest Hour, Thrilling Spectacle)

Annotations appear as unchecked abilities in UnitCrunch's abilities panel, named `⚠ Pre-baked: {source}` with a description explaining what's modified and whether the bonus is permanent (equipment) or temporary (once per battle). Equipment annotations say "Stats shown include this bonus." Temporal annotations say "Stats shown may include this temporary bonus."

Tested across 5 faction files (145 units): 22 annotations detected, 0 false positives.

### v0.74.0
**ListForge deep-link integration.**

ListForge can now send army lists directly to Daimon via a URL deep link, eliminating the file export/upload step. Users click a button in ListForge, Daimon opens with the list pre-loaded, converts it, and auto-downloads the UnitCrunch import file.

How it works:
- ListForge encodes the BSData JSON as gzip + base64 and appends it to a URL hash fragment
- Daimon reads the hash, decodes with the browser-native `DecompressionStream` API, and feeds the JSON to the existing converter
- The `#` hash fragment means no data ever leaves the browser — no server involved
- File upload (drag-and-drop) continues to work exactly as before

Also added: Landing page footer with version compatibility info and links to ListForge, UnitCrunch, New Recruit, and GitHub.

Validated: Deep-link decode → convert → UnitCrunch import round-trip confirmed working. File upload regression tested with zero changes to existing behavior.

### v0.73.23
**Fix: Torrent weapons incorrectly assigned BS 4+ instead of null.**

When BSData encodes a weapon's BS as N/A (the standard representation for Torrent weapons, which auto-hit and have no ballistic skill), the converter's `parseStatValue()` returned the string `"N/A"`. The weapon builder then checked `typeof chars.bs === 'number'`, found it false, and fell back to `bs: 4`. The same logic applied to WS.

The fix distinguishes between "field is present but non-numeric" (set to `null`) and "field is completely absent" (keep the default of 4). This is a one-line change per field in `parseWeaponProfile()`.

Weapons affected in Custodes testing:
- Galatus warblade ranged profile (Contemptor-Galatus Dreadnought) — Torrent, Twin-linked, Ignores Cover
- Twin plasma projector (Telemon Heavy Dreadnought) — Torrent, Twin-linked
- Witchseeker flamer (Witchseekers) — Torrent, Ignores Cover

Simulation impact: Low — UnitCrunch applies Torrent as auto-hit regardless of BS value, so the incorrect BS 4+ was effectively ignored. However, the profile data was wrong and would display incorrectly in UC's stat summary.

Validated: Regression tested across all 14 test files (176 profiles, 13 factions). Zero errors, identical profile and deduplication counts.

### v0.73.22
**New Recruit compatibility + enhancement points fix.**

Two bugs fixed, both additive — zero existing lines removed or modified.

1. **Invulnerable save extraction (NR-specific):** New Recruit uses five different formats for invulnerable save ability profiles, none of which matched Daimon's existing regex patterns. `extractInvulnSave` now handles all NR formats: bare value (`$text: "4+"`), parenthesized name (`"Invulnerable Save (4+)"`), sentence descriptions (`"This model has a 4+ invulnerable save."`), unit-wide sentences (`"Models in this unit have a 5+ invulnerable save."`), and named variants (`"Invulnerable Save: Fluxmaster"`). This also fixed two wrong-value bugs where text fallbacks were grabbing invulnerable save values from unrelated aura or faction rule text (Zoanthropes: was 6+, should be 4+; Trukk: was 5+, should be 6+).

2. **Enhancement points (NR + ListForge):** BSData stores enhancement costs as separate `costs` entries on upgrade child selections, not summed into the parent unit cost. `processSelectionUnit` now sums costs from direct `type=upgrade` children. This also fixes deduplication of enhanced unit variants — previously three Lords of Change at 270/300/280pts were collapsed into one profile at 270pts because their fingerprints were identical without the enhancement cost.

Affected units: 25 dropped invulns and 2 wrong values across 7 NR files. 14 units with incorrect points across both NR (9) and ListForge (5) exports. All invulnerable save values verified against Wahapedia datasheets.

Validated: 76 NR profiles + 6 ListForge files regression tested. Zero errors, zero regressions.

### v0.73.21
**Fix: Dynamic fallback for Rapid Fire and Sustained Hits values not in the preset map.**

Previously, the converter had hardcoded mappings for Sustained Hits 1, 2, D3 and Rapid Fire 1, 2, D3. Any weapon with a value outside that list (e.g., Rapid Fire 3, Rapid Fire 5, Sustained Hits 3) had the ability silently dropped — the lookup found nothing and moved on. This affected both the weapon keyword parser and the bracket keyword parser (for abilities like `[SUSTAINED HITS 3]` in description text).

The fix adds dynamic fallback branches that construct the correct UnitCrunch ability structure for any arbitrary value, using the same `makeSustainedEffect()` and `increaseWeaponAttacks` patterns already used by the preset mappings.

Weapons affected in testing:
- Vertus Hurricane Bolter (Custodes) — Rapid Fire 3
- Gauss Flux Arc / Monolith (Necrons) — Rapid Fire 3
- Gauss Flayer Array / Ghost Ark, Doomsday Ark (Necrons) — Rapid Fire 5
- Pavane of Slaanesh focused witchfire (Chaos Daemons) — Sustained Hits 3
- Shrieker Cannon selectable ability (Drukhari/Aeldari) — `[SUSTAINED HITS 3]` via bracket path

Validated: Full round-trip (JSON → Daimon → UC import → UC export → structural diff) confirmed zero differences for Custodes (31 profiles) and Necrons (51 profiles). All preset values (RF 1/2/D3, SH 1/2/D3) regression-tested with zero changes.

### v0.73.20
**Initial public release.** 117 profiles, 341 weapons, 226 abilities across 7 factions validated with zero data errors. See [Validation](#validation) for full breakdown.

---

## Contributing

Found a bug, tested with a new faction, or want to improve ability parsing? Open an issue or submit a pull request.

---

## License

MIT

---

## Author

[LostWhen](https://github.com/LostWhen)
