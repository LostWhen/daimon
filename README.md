# Daimon

**Army List → UnitCrunch Converter for Warhammer 40,000 (10th Edition)**

[**→ Use it now**](https://lostwhen.github.io/daimon/)

Daimon takes a `.json` army list exported from [ListForge](https://listforge.club), [New Recruit](https://www.newrecruit.eu/), or any app that uses the [BattleScribe roster schema](http://www.battlescribe.net/schema/rosterSchema) and converts it into an import file for [UnitCrunch](https://www.unitcrunch.com/), the 40K probability and damage simulator.

No server, no account, no data leaves your browser. Drop a file in, get a file out.

---

## Quick Start

1. Export your army list as `.json` from **ListForge**, **New Recruit**, or another BSData-compatible app (see [Compatibility](#compatibility))
2. Open **[Daimon](https://lostwhen.github.io/daimon/)**
3. Drop the `.json` file onto the page (or click to browse)
4. A `.txt` file downloads automatically
5. In **[UnitCrunch](https://www.unitcrunch.com/)**, click **Import** and select the `.txt` file
6. All your units appear as profiles, ready to sim

## Compatibility

Daimon reads any JSON file that follows the BattleScribe roster schema (`http://www.battlescribe.net/schema/rosterSchema`).

| App | Export Format | Works with Daimon |
|-----|-------------|-------------------|
| [ListForge](https://listforge.club) | JSON | ✓ Tested extensively |
| [New Recruit](https://www.newrecruit.eu/) | JSON | ✓ Tested — see [Validation](#validation) |
| BattleScribe | XML (`.ros` / `.rosz`) | ✗ Not supported — BattleScribe exports XML, not JSON |

If you test Daimon with an app not listed here, please open an issue with your results.

## What Gets Converted

Everything that UnitCrunch can simulate is extracted and mapped automatically.

**Defensive Stats** — Toughness, Save, Wounds, and Invulnerable Save from unit stat profiles. Mixed stat-line units (e.g., Exarch + Aspect Warriors, Nob + Boyz) are split into separate model types with correct wound values and model counts.

**Weapons** — All ranged and melee weapon profiles with Attacks, BS/WS, Strength, AP, and Damage. Dice expressions (`D6`, `2D6`, `D6+3`) are preserved as strings. Units with different loadout options that produce identical stat lines are deduplicated.

**Weapon Abilities** — Each keyword on a weapon profile maps to the corresponding UnitCrunch modifier: Sustained Hits (any value), Lethal Hits, Devastating Wounds, Twin-linked, Torrent, Heavy, Rapid Fire (any value), Anti-\* (2+–6+), Blast, Indirect Fire, Lance, Melta (2–4), Ignores Cover, Pistol, Assault, Hazardous, Precision, Psychic.

**Unit Abilities** — Parsed from ability description text on the datasheet:

| Ability | How It's Mapped |
|---------|----------------|
| Invulnerable saves | Set on model type; extracted from Invuln profiles and description text; distinguishes bearer-only vs unit-wide |
| Feel No Pain | Standard FNP and conditional (psychic, mortal wounds) |
| Rerolls (hit/wound) | Reroll 1s and reroll all failed; leader aura patterns detected |
| Stat modifiers | +1/−1 to hit, wound, save, damage, strength, toughness |
| Critical hit/wound effects | Threshold changes (5+), extra hits on crits, lethal/devastating on crits |
| Damage reduction | −1 damage, halve damage |
| Benefit of Cover | Detected from description text |
| Conditional abilities | "Once per battle" / "until end of phase" → start inactive; user toggles on in UC |
| Leader auras | "While this model is leading a unit" → start active |

## What You'll Need to Set Up Manually

Some abilities are game-state-dependent and can't be automatically converted. After importing, you may want to configure these in UnitCrunch by hand:

- **Detachment rules** — Daimon extracts unit data, not detachment-wide rules (e.g., War Horde's Sustained Hits on charge, Seer Council's Strands of Fate). Add these as global modifiers in UC.
- **Stratagems** — Not parsed. Apply as needed per matchup in UC.
- **Mortal wound output** — Abilities that inflict mortal wounds (on charge, on death, etc.) aren't mapped since UC models attack sequences, not event triggers.
- **Movement and deployment** — Deep Strike, Advance bonuses, transport capacity — not relevant to damage simulation.
- **Board-state auras** — "While within 6″ of this model" effects require positioning context UC doesn't have.
- **Enhancements** — Enhancement points are correctly included in profile totals. Stat modifications from enhancements are detected but suppressed because BSData bakes enhancement bonuses directly into weapon and unit stat profiles already. Enhancement ability descriptions are preserved for reference.

## Validation

All testing was done with competitive 2000-point lists using specific detachments, not raw datasheet dumps.

**Full round-trip verification** (source JSON → Daimon → UnitCrunch import → UnitCrunch export → structural diff, zero differences):

| Faction | Detachment | Profiles | Weapons | Abilities |
|---------|------------|----------|---------|-----------|
| Tyranids | Subterranean Assault | 10 | 21 | 22 |
| Tyranids | Invasion Fleet | 9 | 16 | 24 |
| Tyranids | Invasion Fleet | 11 | 17 | 27 |
| Chaos Daemons | Scintillating Legion | 8 | 19 | 10 |
| Custodes | (full faction) | 31 | — | — |
| Necrons | (full faction) | 51 | — | — |

**Imported successfully into UnitCrunch** (all files accepted by UC's format validator; source JSON stat audit confirmed zero discrepancies in stats, weapons, and invulns):

| Faction | Detachment | Profiles | Weapons | Abilities |
|---------|------------|----------|---------|-----------|
| Tyranids | Subterranean Assault | 10 | 19 | 21 |
| Emperor's Children | Coterie of the Conceited | 11 | 41 | 14 |
| Blood Angels | Rage-Cursed Onslaught | 11 | 43 | 28 |
| Orks | War Horde | 18 | 71 | 28 |
| Aeldari | Seer Council | 16 | 62 | 19 |
| Space Wolves | Stormlance Task Force | 13 | 32 | 33 |

**Combined:** 117 profiles, 341 weapons, 226 abilities across 7 factions. Zero data errors found.

Additional testing from earlier development validated 254 profiles across 7 army files including Necrons, Custodes, and Drukhari — see the [session handoff document](https://github.com/LostWhen/daimon/wiki) for full history.

**New Recruit compatibility testing** (v0.73.22 — 7 NR exports covering 7 factions, all invulnerable saves cross-referenced against [Wahapedia](https://wahapedia.ru/)):

| Faction | Source | Profiles | Weapons | Abilities | Invulns |
|---------|--------|----------|---------|-----------|---------|
| Chaos Daemons | NR | 11 | 20 | 6 | 11/11 |
| Blood Angels | NR | 11 | 27 | 28 | 4/4 |
| Death Guard | NR | 12 | 26 | 23 | 8/8 |
| Grey Knights | NR | 8 | 19 | 12 | 4/4 |
| Imperial Knights | NR | 6 | 27 | 14 | 6/6 |
| Tyranids | NR | 13 | 25 | 24 | 4/4 |
| Orks | NR | 15 | 44 | 26 | 15/15 |

**Combined NR:** 76 profiles, 188 weapons, 133 abilities, 52 invulnerable saves. Zero errors. Enhancement points (9 units) correctly summed from upgrade children. Enhanced unit variants correctly deduplicated as separate profiles.

**Automated Wahapedia ground-truth verification** (v0.73.23 — converter output compared field-by-field against [Wahapedia](https://wahapedia.ru/) CSV data exports using an automated verification harness):

| Faction | Profiles | Stats Checked | Weapons Checked | Result |
|---------|----------|---------------|-----------------|--------|
| Custodes | 31/31 | T, Sv, W, Invuln | A, BS/WS, S, AP, D + keywords | 29/31 clean, 2 BSData↔Wahapedia source disagreements |

The two flagged profiles are source data differences, not converter bugs: Telemon Heavy Dreadnought (BSData pre-bakes a conditional +2A bonus into the weapon stat) and Venatari Custodians (BSData and Wahapedia disagree on wounds — pending manual verification against the official Forge World datasheet).

## Technical Details

- Single HTML file (~93KB), zero external dependencies
- Runs entirely client-side in the browser
- Output uses [msgpack](https://msgpack.org/) binary encoding matching UnitCrunch's native import/export format
- `appVersion` field set to `0.73.17` (UC validates this against known version strings; arbitrary values cause silent rejection)
- `exported` timestamp encoded as msgpack uint64 (`0xCF`) — UC's parser is type-strict and rejects float64 (`0xCB`) encoding for this field

## Name

*Daimon* — from Walter Jon Williams' novel [*Aristoi*](https://en.wikipedia.org/wiki/Aristoi_(novel)) (1992). The Aristoi partition their consciousness into daimones: specialized sub-processes that each handle a single task autonomously. This tool is a daimon — one job, one transformation, done.

## Version History

### v0.73.23

**Fix: Torrent weapons incorrectly assigned BS 4+ instead of null.**

When BSData encodes a weapon's BS as `N/A` (the standard representation for Torrent weapons, which auto-hit and have no ballistic skill), the converter's `parseStatValue()` returned the string `"N/A"`. The weapon builder then checked `typeof chars.bs === 'number'`, found it false, and fell back to `bs: 4`. The same logic applied to WS.

The fix distinguishes between "field is present but non-numeric" (set to `null`) and "field is completely absent" (keep the default of 4). This is a one-line change per field in `parseWeaponProfile()`.

**Weapons affected in Custodes testing:**
- Galatus warblade ranged profile (Contemptor-Galatus Dreadnought) — Torrent, Twin-linked, Ignores Cover
- Twin plasma projector (Telemon Heavy Dreadnought) — Torrent, Twin-linked
- Witchseeker flamer (Witchseekers) — Torrent, Ignores Cover

**Simulation impact:** Low — UnitCrunch applies Torrent as auto-hit regardless of BS value, so the incorrect BS 4+ was effectively ignored. However, the profile data was wrong and would display incorrectly in UC's stat summary.

**Validated:** Regression tested across all 14 test files (176 profiles, 13 factions). Zero errors, identical profile and deduplication counts.

### v0.73.22

**New Recruit compatibility + enhancement points fix.**

Two bugs fixed, both additive — zero existing lines removed or modified.

**1. Invulnerable save extraction (NR-specific):** New Recruit uses five different formats for invulnerable save ability profiles, none of which matched Daimon's existing regex patterns. `extractInvulnSave` now handles all NR formats: bare value (`$text: "4+"`), parenthesized name (`"Invulnerable Save (4+)"`), sentence descriptions (`"This model has a 4+ invulnerable save."`), unit-wide sentences (`"Models in this unit have a 5+ invulnerable save."`), and named variants (`"Invulnerable Save: Fluxmaster"`). This also fixed two wrong-value bugs where text fallbacks were grabbing invulnerable save values from unrelated aura or faction rule text (Zoanthropes: was 6+, should be 4+; Trukk: was 5+, should be 6+).

**2. Enhancement points (NR + ListForge):** BSData stores enhancement costs as separate `costs` entries on upgrade child selections, not summed into the parent unit cost. `processSelectionUnit` now sums costs from direct `type=upgrade` children. This also fixes deduplication of enhanced unit variants — previously three Lords of Change at 270/300/280pts were collapsed into one profile at 270pts because their fingerprints were identical without the enhancement cost.

**Affected units:** 25 dropped invulns and 2 wrong values across 7 NR files. 14 units with incorrect points across both NR (9) and ListForge (5) exports. All invulnerable save values verified against Wahapedia datasheets.

**Validated:** 76 NR profiles + 6 ListForge files regression tested. Zero errors, zero regressions.

### v0.73.21

**Fix: Dynamic fallback for Rapid Fire and Sustained Hits values not in the preset map.**

Previously, the converter had hardcoded mappings for Sustained Hits 1, 2, D3 and Rapid Fire 1, 2, D3. Any weapon with a value outside that list (e.g., Rapid Fire 3, Rapid Fire 5, Sustained Hits 3) had the ability silently dropped — the lookup found nothing and moved on. This affected both the weapon keyword parser and the bracket keyword parser (for abilities like `[SUSTAINED HITS 3]` in description text).

The fix adds dynamic fallback branches that construct the correct UnitCrunch ability structure for any arbitrary value, using the same `makeSustainedEffect()` and `increaseWeaponAttacks` patterns already used by the preset mappings.

**Weapons affected in testing:**
- Vertus Hurricane Bolter (Custodes) — Rapid Fire 3
- Gauss Flux Arc / Monolith (Necrons) — Rapid Fire 3
- Gauss Flayer Array / Ghost Ark, Doomsday Ark (Necrons) — Rapid Fire 5
- Pavane of Slaanesh focused witchfire (Chaos Daemons) — Sustained Hits 3
- Shrieker Cannon selectable ability (Drukhari/Aeldari) — [SUSTAINED HITS 3] via bracket path

**Validated:** Full round-trip (JSON → Daimon → UC import → UC export → structural diff) confirmed zero differences for Custodes (31 profiles) and Necrons (51 profiles). All preset values (RF 1/2/D3, SH 1/2/D3) regression-tested with zero changes.

### v0.73.20

Initial public release. 117 profiles, 341 weapons, 226 abilities across 7 factions validated with zero data errors. See [Validation](#validation) for full breakdown.

## Contributing

Found a bug, tested with a new faction, or want to improve ability parsing? [Open an issue](https://github.com/LostWhen/daimon/issues) or submit a pull request.

## License

[MIT](LICENSE)

## Author

[LostWhen](https://github.com/LostWhen)
