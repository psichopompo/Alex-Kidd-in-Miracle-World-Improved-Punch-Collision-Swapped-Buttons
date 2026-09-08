<div align="center">
  <img width="512" height="174" alt="7206ef1be0a3d44c57fa8214fc74421e" src="https://github.com/user-attachments/assets/e5acbd4b-ecc4-4fd6-aa03-042f8e0dc6e4" />
</div>

## Basic Information

**Hack title:** Alex Kidd in Miracle World — Improved Punch Collision & Swapped Buttons

**Platform:** Sega Master System (SMS)

**Game:** Alex Kidd in Miracle World

**Region:** USA / Europe

**Genre:** Action / Platformer

**Category:** Improvement

**Author:** *Psicopompo*

**Version:** 1.0

**Release date:** 2026-09-08

**Patch format:** IPS

---

## Description

This hack makes two quality-of-life improvements to Alex Kidd in Miracle World.

### 1. More forgiving punch collision against breakable blocks

**The problem.** In the original game, a punch is only checked for block
collision a *single time* — the instant the punch button is pressed — and
against a *single point* (one 8×8-pixel tile). If Alex is mid-jump or the punch
hasn't fully extended yet, that one-shot check can miss, so the block does not
break even though the fist ends up visibly overlapping it. Landing a punch
"just in the corner" or while descending/ascending from a jump frequently fails
to break the block for this reason.

<img width="256" height="192" alt="Alex Kidd in Miracle World (USA, Europe) (Rev 1)-260908-073735" src="https://github.com/user-attachments/assets/274eadf8-eaaf-4d85-8fb5-22d82e26263b" /> <img width="256" height="192" alt="Alex Kidd in Miracle World (punch fix)-260908-080324" src="https://github.com/user-attachments/assets/89408e6b-4874-4e85-9748-0bd28196722a" />



**The fix.** Block collision is now checked on *every frame* while the punch is
extended (the four calls to the punch-tick routine are redirected to a small
helper). The check uses the fist's actual hitbox — the same 8×5-pixel box the
game already uses to hit enemies — with **1 pixel of extra margin** per side
(10×7 px total). Six sample points (3 columns × 2 rows) are tested so the wider
box leaves no gaps. If any point touches a breakable block, the block breaks.

As a result:

- Any **1-pixel contact** now breaks the block — including corner hits and
  punches thrown mid-jump.
- Blocks still **do not** break without contact (no "phantom reach" / no
  breaking blocks the fist never touches).
- A guard prevents the same block from being broken twice by one punch.

### 2. Swapped the two action buttons

The game's default mapping is **Button 1 = Jump, Button 2 = Punch**. This hack
swaps them to the more conventional **Button 1 = Punch, Button 2 = Jump**.

The swap is done at the single point where the controller is read
(`saveInput`), so it applies consistently across the whole game.

---

## ROM / ISO Information

Apply the IPS patch to the following **unmodified** ROM:

- **File name:** `Alex Kidd in Miracle World (USA, Europe) (Rev 1).sms`
- **Size:** 131,072 bytes (128 KB)
- **CRC32:** `AED9AAC4`
- **MD5:** `f43e74ffec58ddf62f0b8667d31f22c0`
- **SHA-1:** `6d052e0cca3f2712434efd856f733c03011be41c`

> Note: the patch targets the **Rev 1** build of the USA/Europe release. It
> will not apply correctly to Rev 0 or to the Japanese version, which have
> different code layout.

---

## Patch File Checksums

- **File name:** `AlexKidd_punch_collision.ips`
- **Size:** 159 bytes
- **CRC32:** `E486EEBD`
- **MD5:** `f74569acf7d5651e8d8daff387c2bbfd`
- **SHA-1:** `453d47cc4d9a4d96bb9f27dc728bceb0dbf8e4a5`

---

## Patched ROM (for reference / verification)

After applying the patch, the resulting ROM should match:

- **Size:** 131,072 bytes (128 KB)
- **CRC32:** `DD4E9200`
- **MD5:** `a76f432b484aa5653a3f4256dc7537a0`
- **SHA-1:** `025163f4e4ff8a251e2c72429f2c5543268e6de4`

---

## Technical Summary (what was changed — 121 bytes total)

| Address (SMS CPU) | Change |
|---|---|
| `$2AE9, $2CE6, $338E, $34FF` | `call $462E` → `call $7F7C` (redirect the four punch-tick calls to the helper, for per-frame block checking) |
| `$7F7C–$7FE9` | Injected helper: continuous block-collision check + hitbox tables + button-swap routine (110 bytes) |
| `$03C4–$03C6` | `ld hl,$C006` → `jp $7FD5` (route controller reading through the button-swap routine) |

The new code lives in the unused (`$FF`) padding at the end of the fixed bank
`$4000–$7FFF` (just before the `$7FF0` header). The game's mapper only pages
`$8000–$BFFF`, so this region is always accessible. No existing code, header,
or checksum was touched. The helper restores all registers it must preserve
(notably `HL = $C006` when returning to `saveInput`).

---

## Verification

- Unit tests of the injected Z80 routine (a small interpreter running the
  actual patched bytes):
  - Block contact breaks (center, edge, +1 px margin, corner): 8/8 ✓
  - No-contact / "air" does not break: ✓
  - No double-break per punch: ✓
  - Button swap: 7 bit-level cases + 4 end-to-end `saveInput` cases ✓
- IPS patch round-trip (apply → compare byte-for-byte): ✓
