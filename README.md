# Daimon

**Army List → UnitCrunch Converter for Warhammer 40,000 (10th Edition)**

[**→ Use it now**](https://lostwhen.github.io/daimon/)

Daimon takes a `.json` army list exported from [ListForge](https://listforge.club) or any app that uses the [BattleScribe roster schema](http://www.battlescribe.net/schema/rosterSchema) and converts it into an import file for [UnitCrunch](https://www.unitcrunch.com/), the 40K probability and damage simulator.

No server, no account, no data leaves your browser. Drop a file in, get a file out.

---

## Quick Start

1. Export your army list as `.json` from **ListForge** (or another BSData-compatible app that exports JSON — see [Compatibility](#compatibility))
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
| [New Recruit](https://www.newrecruit.eu/) | JSON | Expected (same schema, not yet tested — [report results](https://github.com/LostWhen/daimon/issues)) |
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
- **Enhancements** — Detected but stat modifications from enhancements are suppressed because BSData bakes enhancement bonuses directly into weapon profiles already.

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

## Technical Details

- Single HTML file (~93KB), zero external dependencies
- Runs entirely client-side in the browser
- Output uses [msgpack](https://msgpack.org/) binary encoding matching UnitCrunch's native import/export format
- `appVersion` field set to `0.73.17` (UC validates this against known version strings; arbitrary values cause silent rejection)
- `exported` timestamp encoded as msgpack uint64 (`0xCF`) — UC's parser is type-strict and rejects float64 (`0xCB`) encoding for this field

## Name

*Daimon* — from Walter Jon Williams' novel [*Aristoi*](https://en.wikipedia.org/wiki/Aristoi_(novel)) (1992). The Aristoi partition their consciousness into daimones: specialized sub-processes that each handle a single task autonomously. This tool is a daimon — one job, one transformation, done.

## Version History

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
