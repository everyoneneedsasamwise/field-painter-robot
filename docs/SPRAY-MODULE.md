# Spray Module — "Kill Zone" Blanket Sprayer

## Overview

Blanket spray mode for killing all vegetation in designated zones: fence lines, gravel driveways, rock beds, sidewalk cracks, and any area where nothing should grow.

**No AI needed.** No precision targeting. Draw the zone → robot sprays everything.

## How It Works

```
App: Draw spray zones on satellite view
  - Fence lines (path + width)
  - Gravel areas (polygon fill)
  - Rock beds (polygon fill)
        │
        ▼
Path Planner generates coverage pattern
  - Lines: follow path with spray width offset
  - Areas: parallel stripe fill pattern (like mowing)
        │
        ▼
Robot drives pattern, pump + solenoid ON
  - Nozzle faces DOWN (not rear like paint mode)
  - Wide fan pattern for maximum coverage
  - Speed calibrated to spray rate
        │
        ▼
Everything in the zone gets sprayed
```

## Hardware

**Shares 100% of the paint module hardware.** No new purchases needed.

| Part | Shared With | Notes |
|------|-------------|-------|
| 12V Diaphragm Pump | Paint module | Same pump, same wiring |
| 12V Solenoid Valve | Paint module | Same valve |
| 1 Gallon Tank | Paint module | Rinse between uses! |
| Silicone Tubing | Paint module | Same lines |
| Flat Fan Spray Tip | Paint module | Use wider fan tip for coverage |

### Optional Upgrade
- **Second wider nozzle** ($4) — 110° fan for maximum blanket coverage
- **Nozzle quick-connect** ($6) — swap between paint (narrow) and spray (wide) tips without tools

## Spray Solutions

### For Fence Lines & Gravel (Kill Everything)

| Solution | Recipe | Cost | Effectiveness | Notes |
|----------|--------|------|---------------|-------|
| **Horticultural Vinegar** | 30% acetic acid, straight | $15/gal | ★★★★ | Burns foliage on contact, may need repeat |
| **Salt + Vinegar + Soap** | 1 gal vinegar + 1 cup salt + 1 tbsp dish soap | $5/gal | ★★★★★ | Nuclear option — salt sterilizes soil |
| **Boiling Vinegar Mix** | Same as above, pre-heated | $5/gal | ★★★★★ | Even faster kill |
| **Commercial Organic** | Avenger, BurnOut (citric acid) | $25/gal | ★★★★ | Ready-to-use, certified organic |
| **Glyphosate** | Roundup concentrate, diluted | $20/gal | ★★★★★ | Most effective, systemic kill, controversial |

### Recommendations by Zone

| Zone | Best Solution | Why |
|------|---------------|-----|
| Fence lines (dirt) | Salt + vinegar | Cheap, don't care about soil |
| Gravel driveway | Salt + vinegar | Want long-term prevention |
| Rock beds | Horticultural vinegar | Less soil damage if near garden |
| Sidewalk cracks | Salt + vinegar | Sterilize the cracks |
| Near garden edges | Vinegar only (NO salt) | Salt can leach into garden soil |

⚠️ **Never use salt solution near gardens, trees, or lawn edges.** Salt travels through soil with water and will kill anything nearby.

## App — Spray Zone Designer

### Drawing Tools

**Fence Line Mode:**
1. Tap start of fence on satellite view
2. Tap along fence to trace the path
3. Set spray width (1-4 feet from fence, both sides or one side)
4. Robot follows the path, spraying the set width

**Area Fill Mode:**
1. Tap corners to draw polygon around gravel/rock area
2. Robot calculates parallel stripe pattern to cover entire area
3. Set overlap percentage (default 20% to avoid gaps)
4. Stripe spacing = nozzle spray width × (1 - overlap)

**Exclusion Zones:**
- Mark trees, posts, or anything to skip
- Robot turns off spray within exclusion radius

### Schedule
- Set recurring spray runs (monthly, bi-weekly)
- App reminds you to fill tank before scheduled run
- Track spray history per zone

## Coverage Math

| Setting | Value |
|---------|-------|
| Nozzle spray width (110° fan at 12" height) | ~24 inches |
| Robot speed while spraying | 0.3 m/s (~1 ft/s) |
| Pump flow rate | 0.8 GPM |
| Coverage per gallon | ~400 sq ft |
| 1 gallon tank capacity | ~400 sq ft per fill |

**Typical usage:**
- 100 ft fence line, 2 ft wide spray = 200 sq ft → half a tank
- 10x20 ft gravel pad = 200 sq ft → half a tank
- Full perimeter fence (200 ft) + driveway → 1-2 tank fills

## Operating Procedure

### Before First Use
1. Flush paint system thoroughly with water (if previously used for paint)
2. Run clean water through pump for 30 seconds
3. Fill tank with chosen solution
4. Test spray on a small area to verify coverage and flow rate

### Spray Session
1. Fill tank with solution
2. Open app → Spray Mode
3. Select pre-drawn zones or draw new ones
4. Hit START
5. Robot follows path/fills area with spray
6. Notification when complete or tank empty

### After Use
1. Rinse tank with clean water
2. Run pump for 15 seconds to flush lines
3. This prevents corrosion (vinegar) or chemical residue

## Safety

- Wear gloves when handling horticultural vinegar (30% is caustic)
- Keep pets/kids off sprayed areas for 24 hours
- Don't spray on windy days (drift onto lawn/garden)
- Robot should check wind sensor (future add-on) or warn in app
- Vinegar fumes can be strong — robot handles the exposure, not you!

## Integration with Other Modules

The spray module is the simplest of the three — it's just the paint module with different liquid and a wider nozzle pointed down. The real value is in the app's zone management:

- **Saved zones persist** — define once, spray monthly with one tap
- **Zone types** help the app suggest the right solution
- **History tracking** shows which zones are due for re-treatment
- **Overlap with laser module** — laser handles the lawn, spray handles the dead zones
