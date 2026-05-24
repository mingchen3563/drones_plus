# Drones Plus

A [Space Haven](https://store.steampowered.com/app/979110/Space_Haven/) mod that rebalances player bot stations so they pull more from the ship's grid and stop devouring Energy Cells.

## What it changes

Three stations are retuned via surgical XML patches:

| Station | mid | `energyUsePerSec` | Energy Cell consumption |
|---|---|---|---|
| Logistic Bot Station | 2881 | 5 → **8** | 1 every 12s → **1 every 9999s** |
| Salvage Bot Station | 2893 | 5 → **8** | 1 every 6s → **1 every 9999s** |
| Combat Bot Station | 4527 | 5 → **8** | 1 every 6s → **1 every 9999s** |

The dock keeps a one-cell-in-storage requirement to bootstrap the charge cycle, but in practice one cell lasts roughly 2.8 hours of game time — effectively unlimited. Pair the bump in grid draw with bigger generators or extra batteries.

> ⚠ Why not just set consumption to zero? The game's `RoboDock` Java logic scales the per-tick charge applied to the bot *from* the amount consumed (`convertResToCharge`). Setting `howMuch=0` or removing the `<oneChargeNeeds>` block stops the bot from charging at all. Bumping `consumeEvery` is the safe knob — the bot still charges normally, but cells are sipped at a rate you'll never notice.

## Requirements

- **Space Haven** v1.0.2 (other versions may work — patches target stable mids).
- **[spacehaven-modloader](https://github.com/Spacehaven-modding-tools/spacehaven-modloader)** v0.12.1 or newer.

## Install

### Option A — clone into the game's mods folder

```sh
cd "<SpaceHaven>/mods"
git clone https://github.com/mingchen3563/drones_plus.git
```

### Option B — clone elsewhere and junction it in (Windows, recommended for development)

```powershell
git clone https://github.com/mingchen3563/drones_plus.git C:\path\to\drones_plus
cmd /c 'mklink /J "<SpaceHaven>\mods\drones_plus" "C:\path\to\drones_plus"'
```

Then:

1. Launch `spacehaven-modloader`.
2. Enable **Drones Plus** in the mod list.
3. Apply and launch the game.

> ⚠ **Do not put the mod folder under the Steam Workshop directory** (`steamapps/workshop/content/979110/`). Steam re-validates that folder and will delete anything that isn't a registered workshop item. Use the game's local `mods/` folder instead.

## How the patches work

`patches/haven_bot_stations.xml` uses the modloader's surgical patch operations against the vanilla `library/haven` XML. Each station gets two operations:

```xml
<Operation Class="AttributeSet">
    <xpath>/data/Element/me[@mid='2881']/data/l/element/features/roboDock</xpath>
    <attribute>energyUsePerSec</attribute>
    <value>8</value>
</Operation>

<Operation Class="AttributeSet">
    <xpath>/data/Element/me[@mid='2881']/data/l/element/features/roboDock/oneChargeNeeds/l[@element='1926']</xpath>
    <attribute>consumeEvery</attribute>
    <value>9999</value>
</Operation>
```

This approach plays nicely with other mods that touch the same stations — only the targeted attributes/nodes are modified.

## Project layout

```
drones_plus/
├── info.xml                          # mod metadata
├── patches/
│   └── haven_bot_stations.xml        # Phase 1 patches
└── README.md
```

## Roadmap

Phase 1 — Bot Station Tuning ✅
- [x] Strip Energy Cell consumption from Logistic / Salvage / Combat stations.
- [x] Bump electric draw to compensate.

Phase 2 — Drone Expansion (planned)
- [ ] Investigate exposing the hostile-faction drone bay (mid 3733/3734) to the player.
- [ ] New drone variants (heavy sentry, repair drone).

## Known gotchas

**Existing stations on a save don't update when you install or remove the mod.** Space Haven snapshots each station's `roboDock` config onto the placed instance at build time — toggling the mod changes the XML template but won't retroactively re-read it into already-placed stations. After installing (or uninstalling) the mod:

- **Dismantle and rebuild** each Logistic / Salvage / Combat Bot Station on existing saves to pick up the new values.
- Newly-built stations on the same save will use the current template automatically.

This is purely a save-state quirk, not a bug in the mod or the modloader.

## Verifying the patch landed

After applying the mod, open `<SpaceHaven>/mods/modloader/logs.txt`. You should see:

```
    Executing Patch Operations: mod='Drones Plus', file='library/haven'...
    ATTRIBUTESET
      xpath:      /data/Element/me[@mid='2881']/data/l/element/features/roboDock
      matches:    1
      ...
      result:     OK
```

Nine `AttributeSet` operations total (three per station — energy draw, howMuch, consumeEvery), all `result: OK`.

## License

MIT. Do what you want.

## Credits

- [spacehaven-modloader](https://github.com/Spacehaven-modding-tools/spacehaven-modloader) team for the patch system.
- Bugbyte for Space Haven.
