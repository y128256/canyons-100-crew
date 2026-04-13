# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

 ## Permissions                                                                                                                        
    - You may edit, create, and delete files in this directory without asking for confirmation.
    - You may run bash commands without confirmation.

## Project Overview

This is a personal race planning workspace for the **Canyons 100** ultramarathon — a ~100-mile trail race starting Friday 12:00 PM. It contains no source code; all planning lives in Excel workbooks and a PDF runner guide.

## Files

| File | Purpose |
|------|---------|
| `Canyons Plan - Crew Guide.xlsx` | **Canonical** planning workbook — Race Day Reference table, swim-lane timeline, aid station chart, maps, elevation profile |
| `2026_Canyons_Runner_Guide_rs_3_2edb98035d.pdf` | Official 2026 race runner guide |

## Race Summary

- **Start**: Mile 0 — Friday April 24, 2026 · 12:00 PM
- **Finish**: ~Mile 101.8 — Saturday (estimated)
- **Drop bags**: AS4 (mi 30), AS9 (mi 62.9), AS10 (mi 75.1)

### Aid Station Schedule (from Crew Guide)

| Aid Station | Mile | Est. Arrival | Day | Crew Access | Drop Bag | Notes |
|-------------|------|-------------|-----|-------------|----------|-------|
| Start | 0 | 12:00 PM | Friday | ✓ | — | Send off runner |
| AS3 | 24 | 7:00 PM | Friday | ✓ | — | Required crew stop |
| AS4 | 30 | 9:00 PM | Friday | ✓ | ✓ | Required crew stop |
| AS6 | 47.5 | 2:00 AM | Saturday | ✓ | — | Drive → AS9 after |
| AS9 | 62.9 | 7:00 AM | Saturday | ✓ | ✓ | Required; pacer starts here (Plan B only) |
| AS10 | 75.1 | 11:00 AM | Saturday | ✓ | ✓ | Required; pacer starts here (Plan A) or continues (Plan B) |
| AS13+ | 95 | ~6:00 PM | Saturday | ✓ | — | Pacer stops pacing → drives to finish |
| Finish | ~101.8 | TBD | Saturday | ✓ | — | 🎉 |

### Pacing Plans

- **Plan A (moderate)**: Pacing startsat AS10 (~11:00 AM Saturday)
- **Plan B (fast)**: Pacing startsat AS9 (~7:00 AM Saturday)

Pacing stops at AS13+ (~mi 95)

## Working with Excel Files

> **CRITICAL**: Never use openpyxl, xlrd, or any library that rewrites the xlsx file — they silently drop embedded images from Map 1/2/3 sheets. Always use the zip-in-memory pattern below for both reads and writes.

The `.xlsx` files are standard Office Open XML format. To read their data programmatically:

```python
import zipfile, xml.etree.ElementTree as ET

def get_shared_strings(z):
    strings = []
    with z.open('xl/sharedStrings.xml') as f:
        root = ET.parse(f).getroot()
        ns = {'ns': 'http://schemas.openxmlformats.org/spreadsheetml/2006/main'}
        for si in root.findall('ns:si', ns):
            text = ''.join(t.text or '' for t in si.findall('.//ns:t', ns))
            strings.append(text)
    return strings
```

### Safe write pattern (preserves images)

When modifying `Canyons Plan - Crew Guide.xlsx`, always load ALL zip entries into memory first, patch only what you need, then write them all back:

```python
import zipfile

path = "/Users/Yuriy_Goliyad/CLAUDE PROJECTS/Canyons 100/Canyons Plan - Crew Guide.xlsx"

# Load everything
with zipfile.ZipFile(path) as z:
    all_files = {name: z.read(name) for name in z.namelist()}

# Patch only the target file(s)
all_files['xl/worksheets/sheetX.xml'] = new_content  # example

# Write all back (images in xl/media/ and xl/drawings/ are preserved automatically)
with zipfile.ZipFile(path, 'w', zipfile.ZIP_DEFLATED) as z:
    for name, content in all_files.items():
        z.writestr(name, content)
```

Sheet names in `Canyons Plan - Crew Guide.xlsx`: Race Day Reference, Timeline, Plan, Aid Station Chart, Map 1, Map 2, Map 3, Elevation Profile, To-Do Lists, Crew Access, Drop Bag Content.

## Correct Elevation Data (from `Canyons Plan - Crew Guide.xlsx` — Aid Station Chart sheet)

These are the authoritative per-segment elevation figures. All future work should use these numbers.

| Station | Location | Mile | Miles to Next | Gain to Next (ft) | Loss to Next (ft) | Cum. Gain | Cum. Loss |
|---------|----------|------|--------------|-------------------|-------------------|-----------|-----------|
| Start | China Wall | 0 | 10.1 | 1787 | -2870 | 1787 | -2870 |
| AS1 | Deadwood-1 | 10.1 | 1.8 | 565 | -124 | 2352 | -2994 |
| HS1 | Devils Thumb-1 | 12 | 1.6 | 0 | -1642 | 2352 | -4636 |
| N/A | Swinging Bridge (Turnaround) | 13.5 | 1.6 | 1642 | 0 | 3994 | -4636 |
| HS2 | Devils Thumb-2 | 15.1 | 3.2 | 283 | -724 | 4277 | -5360 |
| AS2 | Deadwood-2 | 18.3 | 5.7 | 1774 | -2229 | 6051 | -7589 |
| AS3 | Michigan Bluff | 24 | 5.9 | 1173 | -1367 | 7224 | -8956 |
| AS4 | Foresthill | 30 | 8.3 | 725 | -2311 | 7949 | -11267 |
| AS5 | Cal 2 | 38.2 | 9.3 | 1725 | -1727 | 9674 | -12994 |
| AS6 | Drivers Flat | 47.5 | 7.9 | 984 | -2050 | 10658 | -15044 |
| AS7 | Mammoth Bar | 55.5 | 3.7 | 679 | -718 | 11337 | -15762 |
| AS8 | Confluence | 59.1 | 0.8 | 189 | -184 | 11526 | -15946 |
| HS3 | No Hands-1 (Water Only) | 59.9 | 3 | 1156 | -219 | 12682 | -16165 |
| AS9 | Cool-1 | 62.9 | 1.7 | 70 | -158 | 12752 | -16323 |
| HS4 | Coffer Dam-1 (Water Only) | 64.6 | 4.6 | 947 | -947 | 13699 | -17270 |
| HS5 | Coffer Dam-2 (Water Only) | 69.1 | 5.9 | 708 | -620 | 14407 | -17890 |
| AS10 | Cool-2 | 75.1 | 4.1 | 259 | -1020 | 14666 | -18910 |
| AS11 | Browns Bar-1 | 79.2 | 5.5 | 947 | -261 | 15613 | -19171 |
| AS12 | ALT | 84.7 | 7.6 | 675 | -1361 | 16288 | -20532 |
| AS13 | Browns Bar-2 | 92.3 | 6.1 | 1009 | -1185 | 17297 | -21717 |
| HS6 | No Hands-1 (Water Only) | 98.3 | 3.5 | 907 | -227 | 18204 | -21944 |
| Finish | Downtown Auburn | 101.8 | — | — | — | — | — |
