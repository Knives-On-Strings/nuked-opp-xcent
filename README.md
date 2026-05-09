# nuked-opp-xcent

A modified fork of [Nuked-OPM](https://github.com/nukeykt/Nuked-OPM) by
Nuke.YKT, with Yamaha YM2164 (OPP) behavioural tweaks. This source is
distributed publicly to satisfy the source-availability obligations of the
LGPL-2.1 licence under which the binary distribution of XCent — a Yamaha
DX100 emulation by Knives On Strings — links it.

If you are an XCent end user looking to relink the XCent product against a
modified copy of this code, this repository is the source you need.
If you are a developer interested in YM2164 emulation in general, the
upstream Nuked-OPM repository is almost certainly what you want first;
the divergences here are XCent-specific and may not be useful out of
context.

## Files

- `opp.c` — main emulation source
- `opp.h` — public API header
- `LICENSE` — verbatim GNU Lesser General Public Licence, version 2.1

## Provenance

This is a fork of [Nuked-OPM](https://github.com/nukeykt/Nuked-OPM) by
Nuke.YKT, which is itself based on YM2151 die analysis by gtr3qq
(siliconpr0n.org) and YM2164 (OPP) decap also by gtr3qq.

The fork was created by Knives On Strings as part of the XCent project to
add YM2164-specific behavioural tweaks needed for accurate Yamaha DX100
emulation. The original work and the substantive emulation logic are
Nuke.YKT's; the modifications here are narrow in scope and documented
below.

## Licence

GNU Lesser General Public Licence, version 2.1 or any later version
(LGPL-2.1-or-later). Full text in `LICENSE`.

```
Copyright (C) 2020, 2026 Nuke.YKT  (original Nuked-OPM)
Copyright (C) 2026 Knives On Strings  (modifications)

This library is free software; you can redistribute it and/or modify it
under the terms of the GNU Lesser General Public License as published by
the Free Software Foundation; either version 2.1 of the License, or (at
your option) any later version.

This library is distributed in the hope that it will be useful, but
WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY
or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU Lesser General Public
License for more details.

You should have received a copy of the GNU Lesser General Public License
along with this library; if not, see <https://www.gnu.org/licenses/>.
```

## Modifications relative to upstream Nuked-OPM

The substantive behavioural fidelity work for the YM2164 lives outside
this file in the XCent product itself (mostly as register pre/post-scaling
tables in XCent's `RomTables.h` and `FirmwareLogic.cpp`). The chip-level
divergences carried in this fork are deliberately small:

1. **Symbol renames** — `OPM_*` → `OPP_*`, `opm_t` → `opp_t`. This lets
   OPP and the unmodified OPM coexist in the same translation unit (the
   XCent test build links both for cross-validation; see XCent issue
   tracker `CODE14`).

2. **OPP-mode IC reset flag** — `opp_flags_ym2164` is used at chip reset
   to select YM2164 behaviour where it differs from YM2151.

3. **OPP-only TL ramp / attack-tick mechanism** — present but currently
   inactive (see comments around `kArRemap` in XCent's `RomTables.h`); the
   AR remap is applied register-side rather than chip-internal because the
   chip-side ramp introduced unwanted envelope ripple during testing.

If you are diffing against upstream Nuked-OPM and find a divergence that
is not in this list, that is a documentation bug — please open an issue.

## Tags and XCent release correspondence

Per LGPL §6, the source you receive must correspond to the binary
distribution you are linking against. Each XCent release tags this
repository with a matching version (e.g. `v0.14.0-rc1`). To get the
source corresponding to a specific XCent build:

```bash
# Replace v0.14.0-rc1 with the version shown in XCent's About modal
git clone https://github.com/Knives-On-Strings/nuked-opp-xcent
cd nuked-opp-xcent
git checkout v0.14.0-rc1
```

If a tag does not yet exist for the XCent version you are running,
contact `knivesonstrings@gmail.com` and we will provide the
corresponding source. Knives On Strings will honour such requests for at
least three (3) years from the date of distribution of the corresponding
release, in accordance with LGPL §6.

## Building this code in isolation

This repository ships only the chip emulator. It is not a standalone
synthesiser or plugin. To use it, you would need to drive its register
interface yourself:

```c
#include "opp.h"

opp_t chip;
OPP_Reset(&chip, opp_flags_ym2164);
OPP_Write(&chip, 0, 0x20);  // address
OPP_Write(&chip, 1, 0xC7);  // data
// ... clock the chip ...
int32_t output[2];
uint8_t sh1, sh2, so;
OPP_Clock(&chip, output, &sh1, &sh2, &so);
```

The full register map and timing model are documented in upstream
Nuked-OPM and in XCent's `Docs/specs/02-YM2164-FM-ENGINE.md` (not
included in this repository). For a working integration example, see
XCent's `src/fm_engine/NukedEngine.h`.

There is no build system for this repository. Compile `opp.c` with any
C99-compliant C compiler.

## Reporting issues

- Bugs in the **upstream emulation** (anything that affects YM2151 / OPM
  behaviour and is not specific to OPP) — report to upstream
  Nuke.YKT at https://github.com/nukeykt/Nuked-OPM. Do not report
  upstream issues here.
- Bugs in the **XCent-specific OPP modifications** listed above —
  report at https://github.com/Knives-On-Strings/xcent/issues or via
  email at `knivesonstrings@gmail.com`.

## Acknowledgements

- **Nuke.YKT** for Nuked-OPM and the broader Nuked-* chip-emulation
  family. None of this work would exist without theirs.
- **gtr3qq** (siliconpr0n.org) for the YM2164 and YM2151 decap and
  die-shot work.
- **John McMaster** (siliconpr0n.org) for additional FM-chip die analysis.
