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

Most of the YM2164 behavioural fidelity work for XCent lives **outside**
this fork — in XCent's own register pre/post-scaling tables and firmware
emulation layer. The chip-level divergences carried inside this fork are
deliberately narrow.

**File sizes** (against upstream Nuked-OPM, master branch):

| File   | Upstream lines | Fork lines | Δ  |
|--------|---------------:|-----------:|---:|
| `.c`   | 2,241          | 2,263      | +22 |
| `.h`   | 289            | 304        | +15 |

The +37-line difference is accounted for entirely by the additions listed
below. (The remaining ~95% of the diff is mechanical symbol renaming.)

### 1. Mechanical symbol rename (zero behaviour change)

To allow the unmodified upstream OPM and this fork to be linked into the
same translation unit (XCent's test build does this for cross-validation),
all public identifiers were renamed:

  - Header guard:  `_OPM_H_`     → `_OPP_H_`
  - Type:          `opm_t`        → `opp_t`
  - Functions:     `OPM_Reset`    → `OPP_Reset`,
                   `OPM_Write`    → `OPP_Write`,
                   `OPM_Clock`    → `OPP_Clock`,
                   `OPM_Read`     → `OPP_Read`,
                   `OPM_SetIC`    → `OPP_SetIC`,
                   `OPM_ReadIRQ`  → `OPP_ReadIRQ`,
                   `OPM_ReadCT1`  → `OPP_ReadCT1`,
                   `OPM_ReadCT2`  → `OPP_ReadCT2`
  - Enum:          `opm_flags_*`  → `opp_flags_*`  (values unchanged:
                   `opp_flags_none = 0`, `opp_flags_ym2164 = 1`)
  - Internal      `opm_*`        → `opp_*`        (struct field prefixes
    naming:                                       and helper-function names)

Every OPP-named field already present in upstream (e.g. `eg_tl_opp`,
`opp_tl_cnt`, `opp_tl[32]`, `opp_flags_ym2164`) is **upstream Nuked-OPM
behaviour** — Nuke.YKT models the YM2164 OPP variant natively. None of
those fields are XCent additions.

### 2. License-header amendment

Source-file headers in `opp.c` and `opp.h` carry an additional copyright
line acknowledging the Knives On Strings modifications, alongside the
preserved Nuke.YKT original. The licence terms (LGPL-2.1-or-later) are
unchanged.

### 3. New mechanism: per-slot attack-rate skip (XCent issue DSP23)

This is the only behavioural addition. It exists to allow XCent to hit
attack times that fall between the granularity of the OPM AR table
(specifically: hardware DX100 measurements at AR=18–30 produced
~105–115 ms attacks, but the closest upstream-reachable AR=13 produces
~95 ms; no upstream AR exists between 95 ms and 140 ms).

**Mechanism**: per slot, optionally skip every Nth `eg_inc=1` tick while
the envelope is in the attack state. `skip=N` makes the attack
`(N+1)/N` times longer (e.g. `skip=10` → 1.10×). `skip=0` disables
the mechanism (default).

**Where it lives**:

| What                                       | File   | Lines (approx) |
|--------------------------------------------|--------|----------------|
| `opp_atk_skip[32]`, `opp_atk_skip_cnt[32]` | `opp.h` | 226–232      |
| `OPP_SetAttackSkip` declaration            | `opp.h` | 298          |
| Attack-skip logic in EG inc handler        | `opp.c` | ~666–676     |
| `OPP_SetAttackSkip` definition             | `opp.c` | 2257–2263    |

That's the entire chip-internal divergence. ~10 lines of behavioural code
plus a 7-line public API.

If you are diffing against upstream Nuked-OPM and find a divergence that
is not on this list, that is a documentation bug — please open an issue.

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
