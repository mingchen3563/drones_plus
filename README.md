# Drones Plus

A [Space Haven](https://store.steampowered.com/app/979110/Space_Haven/) mod that rebalances player bot stations so they pull more from the ship's grid and stop devouring Energy Cells.

## What it changes

Three stations are retuned via surgical XML patches:

| Station | mid | `energyUsePerSec` | Energy Cell consumption |
|---|---|---|---|
| Logistic Bot Station | 2881 | 5 → **8** | 1 every 12s → **removed** |
| Salvage Bot Station | 2893 | 5 → **8** | 1 every 6s → **removed** |
| Combat Bot Station | 4527 | 5 → **8** | 1 every 6s → **removed** |

In exchange for a slightly heavier continuous draw on the power grid, your bots no longer need a steady supply of Energy Cells (element 1926). Keep generators and batteries healthy and your fleet stays charged forever.

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

<Operation Class="NodeRemove">
    <xpath>/data/Element/me[@mid='2881']/data/l/element/features/roboDock/oneChargeNeeds</xpath>
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

Six operations total — three `AttributeSet`, three `NodeRemove`, all `result: OK`.

## License

MIT. Do what you want.

## Credits

- [spacehaven-modloader](https://github.com/Spacehaven-modding-tools/spacehaven-modloader) team for the patch system.
- Bugbyte for Space Haven.
