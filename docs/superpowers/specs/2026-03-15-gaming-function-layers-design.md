# Gaming & Function Layers Design

**Date:** 2026-03-15
**Status:** Approved

## Overview

Add two new layers to the Totem ZMK keymap: a **Gaming layer** (with a GAME_PLUS sublayer for number keys) and a **Function layer** (F1–F12 + BT/USB controls). The design follows existing keymap patterns (`ZMK_BASE_LAYER`, `ZMK_HOLD_TAP`) and introduces three new layer indices.

---

## Layer Numbering

| Index | Name | Type |
|-------|------|------|
| 0 | BASE | existing |
| 1 | COLEMAK | existing |
| 2 | NUM | existing |
| 3 | SYM | existing |
| 4 | GAMING | new base layer |
| 5 | FUN | new overlay layer |
| 6 | GAME_PLUS | new overlay layer |

New defines to add:
```c
#define GAMING    4
#define FUN       5
#define GAME_PLUS 6
```

---

## Change to Existing Layers (Base & Colemak)

`LH2` (left outermost thumb, currently plain `&kp TAB`) becomes a hold-tap:

- **Tap:** `Tab`
- **Hold:** `&mo FUN`

Implementation: new `ZMK_HOLD_TAP(fun_tab, ...)` using `balanced` flavor and the same `tapping-term-ms = <240>` / `quick-tap-ms = <150>` as `&lt`. Replace `&kp TAB` with `&fun_tab FUN 0` on `LH2` in both Base and Colemak layers.

`LH0` (`magic_shift`) is **unchanged**.

---

## GAMING Layer (layer 4)

Accessed via `tog GAMING` on the FUN layer. Exited via `tog GAMING` on the FUN layer (accessible from gaming through right thumb `&mo FUN`).

### Left hand

| Row | LB5/LT0 | col1 | col2 | col3 | col4 | LB4/LT4 |
|-----|---------|------|------|------|------|---------|
| LT  | — | `Tab` | `X` | `C` | `V` | `B` |
| LM  | — | `Shift` | `Q` | `W` | `E` | `R` |
| LB  | `ESC`(LB5) | `Ctrl`(LB0) | `A`(LB1) | `S`(LB2) | `D`(LB3) | `F`(LB4) |
| LH  | — | `Space`(LH2) | `Enter`(LH1) | `tog GAME_PLUS`(LH0) | — | — |

Movement cluster: **W=forward, A=left, S=back, D=right** mapped to the physical D·X·C·V key positions (one column right, one row down from standard WASD). Shift at the A physical position for sprint. No HRMs anywhere on this layer.

### Right hand

Plain passthrough — same characters as Base but all plain `&kp` bindings (no HRMs, no hold-tap delays). Right thumb: `&mo FUN` (RH0) · `Space` (RH1) · `key_repeat` (RH2).

---

## GAME_PLUS Layer (layer 6)

Toggled from `LH0` within the Gaming layer. Top row only; everything else transparent.

| Row | Keys |
|-----|------|
| LT  | `1` `2` `3` `4` `5` |
| all others | `&trans` |

Toggling `LH0` again exits GAME_PLUS back to GAMING.

---

## FUN Layer (layer 5)

Accessed by holding `LH2` (tap=Tab, hold=FUN) from Base/Colemak, or by holding `RH0` (`&mo FUN`) from the Gaming layer.

New includes required:
```c
#include <dt-bindings/zmk/bt.h>
#include <dt-bindings/zmk/outputs.h>
```

### Left hand — F keys

F1–F9 mirror the numpad positions in the NUM layer exactly. F10/F11/F12 fill the operator slots and the thumb cluster.

| Row | col0 | col1 | col2 | col3 | col4 |
|-----|------|------|------|------|------|
| LT  | `F10` | `F7` | `F8` | `F9` | `F11` |
| LM  | `F12` | `F4` | `F5` | `F6` | `&none` |
| LB  | `K_CANCEL`(LB5) | `&none`(LB0) | `F1` | `F2` | `F3` | `&none`(LB4) |
| LH  | `F10` | `Enter` | `F12` |

### Right hand — toggles & BT

| Row | col0 | col1 | col2 | col3 | col4 |
|-----|------|------|------|------|------|
| RT  | `tog GAMING` | `BT_SEL 0` | `BT_SEL 1` | `BT_SEL 2` | `tog COLEMAK` |
| RM  | `OUT_TOG` | `BT_CLR` | `&none` | `&none` | `&none` |
| RB  | `&none`(RB5) | `sk LSHIFT` | `sk RCTRL` | `sk RALT` | `sk RGUI` | `&none`(RB4) |
| RH  | `K_CANCEL`(RH0) | `&trans`(RH1) | `&trans`(RH2) |

---

## Implementation Notes

1. Add `#include <dt-bindings/zmk/bt.h>` and `#include <dt-bindings/zmk/outputs.h>` at top of `config/totem.keymap`.
2. Add `#define GAMING 4`, `#define FUN 5`, `#define GAME_PLUS 6`.
3. Define `ZMK_HOLD_TAP(fun_tab, ...)` using `balanced` flavor, `tapping-term-ms = <240>`, `quick-tap-ms = <150>`, bindings `<&mo>`, `<&kp>`.
4. Update `LH2` in Base and Colemak to `&fun_tab FUN 0`.
5. Add `ZMK_BASE_LAYER(Gaming, ...)` — no HRMs, movement cluster at physical D·X·C·V positions.
6. Add `ZMK_LAYER(Fun, ...)` — F keys left, BT/toggles right.
7. Add `ZMK_LAYER(GamePlus, ...)` — numbers 1–5 on top row, rest transparent.
