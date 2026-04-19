# AGENTS.md — Totem ZMK Config

Context file for AI coding agents. Read this before touching any keymap or config file.

---

## Physical layout

The Totem is a 38-key column-staggered split keyboard.

```
Left hand                                      Right hand

        ┌─────┬─────┬─────┬─────┬─────┐   ┌─────┬─────┬─────┬─────┬─────┐
        │ LT0 │ LT1 │ LT2 │ LT3 │ LT4 │   │ RT0 │ RT1 │ RT2 │ RT3 │ RT4 │
        ├─────┼─────┼─────┼─────┼─────┤   ├─────┼─────┼─────┼─────┼─────┤
        │ LM0 │ LM1 │ LM2 │ LM3 │ LM4 │   │ RM0 │ RM1 │ RM2 │ RM3 │ RM4 │
┌─────┐ ├─────┼─────┼─────┼─────┼─────┤   ├─────┼─────┼─────┼─────┼─────┤ ┌─────┐
│ LB0 │ │ LB1 │ LB2 │ LB3 │ LB4 │ LB5 │   │ RB0 │ RB1 │ RB2 │ RB3 │ RB4 │ │ RB5 │
└─────┘ └─────┴─────┴─────┴─────┴─────┘   └─────┴─────┴─────┴─────┴─────┘ └─────┘
                    ┌─────┬─────┬─────┐   ┌─────┬─────┬─────┐
                    │ LH0 │ LH1 │ LH2 │   │ RH0 │ RH1 │ RH2 │
                    └─────┴─────┴─────┘   └─────┴─────┴─────┘
```

### CRITICAL — outer bottom-row keys (LB0 and RB5)

**LB0 (left) and RB5 (right) are standalone outer-column keys.**
They do NOT align vertically with any key in the top or home rows.
They sit to the *outside* of the 5-column inner grid.

Do not assume LB0 is "below LT0/LM0" in a column sense — it is not.
Do not assume RB5 is "below RT4/RM4" — it is not.

In `config/totem.keymap`, the `ZMK_BASE_LAYER` macro indentation makes this
visible: LB0 and RB5 are placed *before* and *after* the five inner bottom-row
keys respectively:

```
ZMK_BASE_LAYER(Base,
    /* top row   */  &key  &key  &key  &key  &key  ,  &key  &key  &key  &key  &key  ,
    /* home row  */  &key  &key  &key  &key  &key  ,  &key  &key  &key  &key  &key  ,
    /* [LB0] outer */ &kp ESC
    /* LB1–LB5  */         &key  &key  &key  &key  &key  ,
    /* RB0–RB4  */                                     &key  &key  &key  &key  &key
    /* [RB5] outer */                                                             &key ,
    /* thumbs   */         &key  &key  &key  ,  &key  &key  &key
)
```

### Default Base layer bindings by position

| Label | Column aligns with | Default key        | Notes                    |
|-------|--------------------|--------------------|--------------------------|
| LB0   | outer (no column)  | `ESC`              | standalone outer key     |
| LB1   | Q / A              | `Z` (hml LALT)     | HRM: ALT                 |
| LB2   | W / S              | `X` (hml LCTRL)    | HRM: CTRL                |
| LB3   | E / D              | `C` (hml LGUI)     | HRM: GUI                 |
| LB4   | R / F              | `V` (hml LSHIFT)   | HRM: SHIFT               |
| LB5   | T / G              | `B`                | plain                    |
| RB0   | Y / H              | `N`                | plain                    |
| RB1   | U / J              | `M` (hmr RSHFT)    | HRM: SHIFT               |
| RB2   | I / K              | `,` (hmr RGUI)     | HRM: GUI                 |
| RB3   | O / L              | `.` (hmr RCTRL)    | HRM: CTRL                |
| RB4   | P / Ç              | `-` (hmr LALT)     | HRM: ALT (note: LALT)    |
| RB5   | outer (no column)  | `BSPC/DEL`         | standalone outer key     |

`KEYS_L = LT0–LT4 LM0–LM4 LB0–LB5` (16 keys)
`KEYS_R = RT0–RT4 RM0–RM4 RB0–RB5` (16 keys)
`THUMBS = LH0–LH2 RH0–RH2` (6 keys)

---

## Layers

| Index | Name       | Purpose                                      |
|-------|------------|----------------------------------------------|
| 0     | BASE       | QWERTY, Portuguese layout                    |
| 1     | COLEMAK    | Colemak-DH, Portuguese layout                |
| 2     | NUM        | Numpad + arrows + sticky mods                |
| 3     | GAMING     | Gaming QWERTY, no HRMs, no combos            |
| 4     | GAME_PLUS  | Gaming sub-layer (numbers, extra binds)      |
| 5     | FUN        | F-keys, BT profiles, output toggle           |

Combos are active only on BASE and COLEMAK (index 0 and 1).

---

## Home-row mods (HRMs)

HRMs live on the **bottom row**, not the home row (moved in PR #10).

- Left order (pinky → index): LALT · LCTRL · LGUI · LSHIFT
- Right order (index → pinky): RSHFT · RGUI · RCTRL · LALT

Config (`MAKE_HRM` macro):
- `flavor = "balanced"`
- `tapping-term-ms = <280>` (or 240 after quick-wins PR)
- `quick-tap-ms = <175>`
- `require-prior-idle-ms = <150>`
- `hold-trigger-on-release`
- `hold-trigger-key-positions` restricted to opposite hand + all thumbs

`hml` (left HRM) triggers on `KEYS_R THUMBS`.
`hmr` (right HRM) triggers on `KEYS_L THUMBS`.

---

## Combos

Symbol-heavy approach: most symbols (parens, brackets, braces, slash, pipe,
`@`, `#`, `=`, `%`, `$`, accent keys, etc.) are combos — there is no
dedicated SYM layer. Active only on BASE + COLEMAK.

Timing defines:
```
COMBO_TERM_FAST   // chord window for fast combos (adjacent keys)
COMBO_TERM_SLOW   // chord window for combos involving HRM positions
COMBO_IDLE_FAST   // require-prior-idle for fast combos
COMBO_IDLE_SLOW   // require-prior-idle for slow combos
```

**Bottom-row combo positions are adjacent to, not on, the letter keys:**
- `copy`  = LB2+LB3 (X+C positions) → emits `Ctrl+C`
- `paste` = LB1+LB2 (Z+X positions) → emits `Ctrl+V`

`ZMK_COMBO_8` wraps combos that land on HRM positions so they fire as
tap-only (workaround for ZMK issue #544).

---

## Portuguese layout gotchas

- `PT_GRAVE = LS(PT_ACUTE)` — Shift+acute produces grave on PT layout
- `PT_GRAVE` is a dead key; a standalone bare grave requires a trailing space
- `magic_shift` uses `qsk` with `ignore-modifiers`, which means sticky shift
  persists through combo chord events. The `acute_macro` and `grave_macro`
  macros explicitly release LSHFT/RSHFT before emitting the accent key to
  prevent sticky shift from converting acute → grave unintentionally.
- Do not remove the shift-release preamble from those macros without
  rethinking the `qsk` / `magic_shift` interaction.

---

## Build / CI

Targets in `build.yaml`:
- `xiao_ble/nrf52840/zmk` + `totem_left`
- `xiao_ble/nrf52840/zmk` + `totem_right`
- `nice_nano//zmk` + `totem_dongle dongle_display`
- `nice_nano//zmk` + `settings_reset`
- `xiao_ble/nrf52840/zmk` + `settings_reset`

`keymap-drawer` auto-regenerates `keymap-drawer/totem.yaml` and
`keymap-drawer/totem.svg` via the GitHub Actions CI job. Do not manually edit
those generated files.

West modules (`config/west.yml`):
| Module              | Remote     | Pin    |
|---------------------|------------|--------|
| zmk                 | zmkfirmware| main   |
| zmk-helpers         | urob       | main   |
| zmk-auto-layer      | urob       | main   |
| zmk-dongle-display  | englmaxi   | main   |

---

## Repo conventions

- Only `main` is a long-lived branch.
- Feature work on short-lived `feat/*` or `claude/*` branches.
- Merge via squash: `gh pr merge <n> --squash --delete-branch`
- Conflicts resolved locally: `git merge --squash <remote-branch>`, fix
  conflicts, commit, push, then `gh pr close <n>` with a comment.
- Single-maintainer repo — avoid unsolicited refactors outside task scope.

---

## Gotchas for agents

1. **LB0 ≠ below LT0/LM0.** LB0 is the standalone outer left bottom key.
   RB5 is the standalone outer right bottom key. Neither aligns with any
   top/home-row column. (This is the most common mistake.)
2. Combos on the bottom row use positions *adjacent to* the target letters,
   not on the letters themselves.
3. `COMBO_IDLE_FAST = 150` is load-bearing — do not add fast combos without
   a `require-prior-idle-ms` guard.
4. `totem_dongle.conf` USB 1.1 compat: do not remove `CONFIG_ZMK_USB_BOOT=y`,
   `CONFIG_USB_CDC_ACM=n`, or `CONFIG_USB_HID_POLL_INTERVAL_MS=10`.
5. `PT_GRAVE` is a dead key — standalone use needs a trailing space.
6. `magic_shift` + `qsk` with `ignore-modifiers` leaks sticky shift through
   combos. The accent macros compensate; keep that pattern intact.
7. The `keymap-drawer/` files are auto-generated. Do not hand-edit them.
