# Chocofi Vial Project

**Last Updated:** 2026-01-14

## Overview

Tools and workflows for managing Chocofi split keyboard with Vial/QMK firmware on Sea-Picro (RP2040) controllers.

## Hardware

| Component | Details |
|-----------|---------|
| Keyboard | Chocofi 36-key split (5 columns per side) |
| Controller | Sea-Picro (RP2040-based, wired) |
| Firmware | Corne/crkbd Vial (42-key layout, outer pinky columns unused) |

**Note:** Also have a separate nice!nano Corne running ZMK (wireless) - different keyboard entirely.

## Firmware

### Current Firmware
- **File:** `crkbd_rev1_vial_promicro_rp2040.uf2` from [Beekeeb docs](https://docs.beekeeb.com/chocofi-keyboard)
- **Location:** `~/Downloads/crkbd_rev1_vial_promicro_rp2040.uf2`
- Uses 42-key Corne layout; Chocofi's missing outer pinky columns show as empty in Vial

### Flashing (RP2040)
```bash
# 1. Double-tap reset button on Sea-Picro to enter bootloader
# 2. RPI-RP2 drive appears (may mount as RPI-RP21 if stale mount exists)
udisksctl mount -b /dev/sdb1
cp ~/Downloads/crkbd_rev1_vial_promicro_rp2040.uf2 /media/timur/RPI-RP2*/
# Board auto-reboots after flash
```

**Important:** Flash BOTH halves with same firmware. Split halves communicate via TRRS cable.

## Vial

- **Binary:** `~/bin/Vial.AppImage` (wrapper script at `~/bin/vial`)
- **Udev rules:** `/etc/udev/rules.d/99-vial.rules` (matches `vial:f64c2b3c` serial)

### Troubleshooting
- **"Failed to communicate"**: Check Raw HID interface exists (`usage_page=0xFF60`)
- **Unlock prompt issues**: Usually means firmware doesn't have proper Vial support
- **No device detected**: Check `lsusb | grep foostan` - should show `4653:0001`

## ZMK to Vial Conversion

Converts ZMK keymap from `zmk-config-chocofi` repo to Vial `.vil` format.

### Scripts
| File | Purpose |
|------|---------|
| `zmk_to_vial.py` | Converts ZMK keymap to Vial layout |
| `template.vil` | Template with tap dances, macros, settings |

### Usage
```bash
# Fetch from GitHub and convert
python3 zmk_to_vial.py --template template.vil --output chocofi.vil

# Or use local keymap file
python3 zmk_to_vial.py --local /path/to/corne.keymap --template template.vil
```

### Layout Mapping
The ZMK keymap uses 42-key Corne positions. Script maps to Vial's 8-row structure:
- Rows 0-3: Left half (3 rows + thumbs)
- Rows 4-7: Right half (reversed, 3 rows + thumbs)

## GitHub Integration

**Repo:** [timur-hassan/zmk-config-chocofi](https://github.com/timur-hassan/zmk-config-chocofi)

### Workflows
| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `build.yml` | Push to config/ | Builds ZMK firmware |
| `build-vial.yml` | After ZMK build succeeds | Converts keymap to Vial format |

### Workflow Files Location
- GitHub: `.github/workflows/`
- Local drafts: `github-workflow/`

## USB Device IDs

| VID:PID | Manufacturer | Firmware |
|---------|--------------|----------|
| `4653:0001` | foostan | Vial/QMK (Chocofi) |
| `1d50:615e` | OpenMoko/ZMK Project | ZMK (nice!nano Corne) |

## Tap-Hold Behavior

**Problem:** ZMK's `hold-preferred` flavor has no direct equivalent in Vial/QMK.

**Solution for thumb keys (Enter/Shift, Space/Ctrl):**
1. Use mod-tap (`LSFT_T(KC_ENT)`) instead of Tap Dance or Space Cadet keys (`KC_SFTENT`)
2. Enable **Permissive Hold** in Vial's QMK Settings

**Why:** Tap Dance and Space Cadet keys use their own timing logic and ignore global settings like Permissive Hold. Only mod-tap (`*_T()`) keycodes respect these settings.

| Key Type | Respects Permissive Hold |
|----------|-------------------------|
| `LSFT_T(KC_ENT)` | ✅ Yes |
| `TD(n)` (Tap Dance) | ❌ No |
| `KC_SFTENT` (Space Cadet) | ❌ No |

## Gotchas

1. **Stale mount points**: `RPI-RP2` may mount as `RPI-RP21` if previous mount wasn't cleaned up
2. **Two Cornes**: Don't confuse Sea-Picro Chocofi (Vial) with nice!nano Corne (ZMK)
3. **Raw HID check**: Vial needs `usage_page=0xFF60` - verify with `cat /sys/class/hidraw/hidrawN/device/report_descriptor | xxd`
4. **Split keyboard**: Only master half shows on USB; slave communicates via TRRS
