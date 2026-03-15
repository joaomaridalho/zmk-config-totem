# Gaming & Function Layers Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a GAMING base layer (with GAME_PLUS sublayer for numbers 1–5) and a FUN overlay layer (F1–F12 + BT/USB controls) to `config/totem.keymap`.

**Architecture:** All changes live in a single file (`config/totem.keymap`). We add includes, defines, a new hold-tap behaviour, update two thumb keys in existing layers, then append three new layers. Each task is a self-contained edit followed by a commit. There is no automated test framework for ZMK — "testing" means verifying the keymap compiles via the GitHub Actions CI build.

**Tech Stack:** ZMK firmware, ZMK helpers (`zmk-helpers/helper.h`), devicetree/keymap syntax.

---

## Chunk 1: Scaffolding — includes, defines, hold-tap

### Task 1: Add BT/output includes and new layer defines

**Files:**
- Modify: `config/totem.keymap` (top of file, lines 1–26)

The keymap currently includes `<behaviors.dtsi>`, `<dt-bindings/zmk/keys.h>`, and others. BT and output control require two additional headers. The layer defines block currently ends at `#define SYM 3`.

- [ ] **Step 1: Add the two new includes**

Open `config/totem.keymap`. After the existing `#include <dt-bindings/zmk/keys.h>` line, add:

```c
#include <dt-bindings/zmk/bt.h>
#include <dt-bindings/zmk/outputs.h>
```

- [ ] **Step 2: Add the three new layer defines**

After `#define SYM 3`, add (use single spaces to match existing style):

```c
#define GAMING 4
#define FUN 5
#define GAME_PLUS 6
```

- [ ] **Step 3: Verify the top of the file looks correct**

The include block should now be (note `num_word.dtsi` and the `KEYMAP_DRAWER` guard remain unchanged):

```c
#include <behaviors.dtsi>
#include <dt-bindings/zmk/keys.h>
#include <dt-bindings/zmk/bt.h>
#include <dt-bindings/zmk/outputs.h>
#include <behaviors/num_word.dtsi>
#ifndef KEYMAP_DRAWER
#include "keys_pt.h"
#endif
#include "zmk-helpers/key-labels/totem.h"
#include "zmk-helpers/helper.h"
```

And the defines block:

```c
#define BASE 0
#define COLEMAK 1
#define NUM 2
#define SYM 3
#define GAMING 4
#define FUN 5
#define GAME_PLUS 6
```

- [ ] **Step 4: Commit**

```bash
git add config/totem.keymap
git commit -m "feat: add BT/output includes and GAMING/FUN/GAME_PLUS layer defines"
```

---

### Task 2: Define `fun_tab` hold-tap and update LH2 in Base & Colemak

**Files:**
- Modify: `config/totem.keymap` (behaviours section and Base/Colemak layer definitions)

`fun_tab` is a hold-tap where tap sends `Tab` and hold momentarily activates the FUN layer. It uses the same `balanced` flavor and timing as the existing `&lt` behaviour. `LH2` is the outermost left thumb key — currently `&kp TAB` in Base and `&trans` in Colemak.

- [ ] **Step 1: Add the `fun_tab` hold-tap definition**

Insert after the `// Macros` comment that follows `ZMK_TAP_DANCE(num_dance, ...)`. Add `fun_tab` as a new block immediately after `num_dance`'s closing ` )` (note: the existing blocks use a leading space — `num_dance` opens with ` ZMK_TAP_DANCE` and closes with ` )`). The `fun_tab` block does not need a leading space.

Add this block:

```c
ZMK_HOLD_TAP(fun_tab,
    bindings = <&mo>, <&kp>;
    flavor = "balanced";
    tapping-term-ms = <240>;
    quick-tap-ms = <150>;
)
```

- [ ] **Step 2: Update LH2 in the Base layer**

In the Base layer definition, the left thumb row currently reads (full line including right side — note the trailing space after `&key_repeat`):

```
                                                       &kp TAB          &kp RET          &magic_shift     , SMART_NUM        &kp SPACE        &key_repeat
```

Change `&kp TAB` to `&fun_tab FUN 0` (preserve the trailing space):

```
                                                       &fun_tab FUN 0   &kp RET          &magic_shift     , SMART_NUM        &kp SPACE        &key_repeat
```

- [ ] **Step 3: Update LH2 in the Colemak layer**

In the Colemak layer definition, the left thumb row currently reads (full line including right side):

```c
                                                       &trans           &trans           &trans           , &trans           &trans           &trans
```

Change the first `&trans` (LH2) to `&fun_tab FUN 0`:

```c
                                                       &fun_tab FUN 0   &trans           &trans           , &trans           &trans           &trans
```

- [ ] **Step 4: Verify both thumb rows read correctly**

Base LH: `&fun_tab FUN 0   &kp RET   &magic_shift`
Colemak LH: `&fun_tab FUN 0   &trans   &trans`

- [ ] **Step 5: Commit**

```bash
git add config/totem.keymap
git commit -m "feat: add fun_tab hold-tap and wire LH2 to FUN layer in Base/Colemak"
```

---

## Chunk 2: New layers

### Task 3: Add GAMING base layer

**Files:**
- Modify: `config/totem.keymap` (append after the Num layer, around line 192)

The GAMING layer is a `ZMK_BASE_LAYER` (same macro as existing layers). Key design:
- Whole alphabet shifted diagonally so WASD movement lands on physical D·X·C·V keys
- LM0=Shift (sprint), LT0=Tab, LB0=Ctrl — no HRMs anywhere
- Left thumb: Space · Enter · tog GAME_PLUS
- Right hand: plain `&kp` passthrough (no hold-tap delays)
- Right thumb: `&mo FUN` · Space · key_repeat

Totem physical position reminder (left, outermost→innermost):
- LT row: LT0 LT1 LT2 LT3 LT4
- LM row: LM0 LM1 LM2 LM3 LM4
- LB row: LB5(extra) LB0 LB1 LB2 LB3 LB4
- LH thumbs: LH2(outer) LH1(mid) LH0(inner)

Right side mirrors: RT/RM/RB/RH in same column order, RB5 is outer extra.

- [ ] **Step 1: Append the GAMING layer after the Num layer**

```c
ZMK_BASE_LAYER(Gaming,
                     &kp TAB          &kp PT_X         &kp PT_C         &kp PT_V         &kp PT_B         , &kp PT_Y         &kp PT_U         &kp PT_I         &kp PT_O        &kp PT_P ,
                     &kp LSHIFT       &kp PT_Q         &kp PT_W         &kp PT_E         &kp PT_R         , &kp PT_H         &kp PT_J         &kp PT_K         &kp PT_L        &kp PT_C_CEDILLA ,
    &kp ESCAPE       &kp LCTRL        &kp PT_A         &kp PT_S         &kp PT_D         &kp PT_F         , &kp PT_N         &kp PT_M         &kp PT_COMMA     &kp PT_DOT      &kp PT_MINUS    &bs_del ,
                                                       &kp SPACE        &kp RET          &tog GAME_PLUS   , &mo FUN          &kp SPACE        &key_repeat
)
```

- [ ] **Step 2: Verify movement cluster positions**

Cross-check against spec:
- `W` (forward) is at LM2 — the physical D key position ✓
- `A` (left strafe) is at LB1 — the physical X key position ✓
- `S` (back) is at LB2 — the physical C key position ✓
- `D` (right strafe) is at LB3 — the physical V key position ✓
- `Shift` (sprint) is at LM0 — the physical A key position ✓

- [ ] **Step 3: Commit**

```bash
git add config/totem.keymap
git commit -m "feat: add GAMING base layer with WASD at D·X·C·V positions"
```

---

### Task 4: Add GAME_PLUS layer

**Files:**
- Modify: `config/totem.keymap` (append after GAMING layer)

GAME_PLUS is a thin overlay: only the top row (LT) has keys 1–5; everything else is transparent. Toggled from LH0 in the GAMING layer — pressing LH0 again exits back to GAMING.

- [ ] **Step 1: Append the GAME_PLUS layer**

```c
ZMK_LAYER(GamePlus,
                     &kp PT_N1        &kp PT_N2        &kp PT_N3        &kp PT_N4        &kp PT_N5        , &trans           &trans           &trans           &trans          &trans ,
                     &trans           &trans           &trans           &trans           &trans           , &trans           &trans           &trans           &trans          &trans ,
    &trans           &trans           &trans           &trans           &trans           &trans           , &trans           &trans           &trans           &trans          &trans          &trans ,
                                                       &trans           &trans           &tog GAME_PLUS   , &trans           &trans           &trans
)
```

Note: `&tog GAME_PLUS` on LH0 toggles the layer off (returns to GAMING). All other keys fall through to GAMING layer underneath.

- [ ] **Step 2: Commit**

```bash
git add config/totem.keymap
git commit -m "feat: add GAME_PLUS sublayer with number keys 1-5 on top row"
```

---

### Task 5: Add FUN layer

**Files:**
- Modify: `config/totem.keymap` (append after GAME_PLUS layer)

FUN is an overlay layer accessible from Base/Colemak (hold LH2) and from GAMING (hold RH0). Left hand has F1–F12 in NUM-mirrored positions. Right hand has BT profile selectors, BT clear, output toggle, layout toggles, and sticky mods.

BT/output key syntax:
- `&bt BT_SEL 0` — select BT profile 0 (same for 1, 2)
- `&bt BT_CLR` — clear current BT profile pairing
- `&out OUT_TOG` — toggle between BT and USB output
- `&tog GAMING` — toggle GAMING base layer on/off
- `&tog COLEMAK` — toggle COLEMAK base layer on/off (existing pattern from NUM layer)

- [ ] **Step 1: Append the FUN layer**

```c
ZMK_LAYER(Fun,
                     &kp F10          &kp F7           &kp F8           &kp F9           &kp F11          , &tog GAMING      &bt BT_SEL 0     &bt BT_SEL 1     &bt BT_SEL 2    &tog COLEMAK ,
                     &kp F12          &kp F4           &kp F5           &kp F6           &none            , &out OUT_TOG     &bt BT_CLR       &none            &none           &none ,
    &kp K_CANCEL     &none            &kp F1           &kp F2           &kp F3           &none            , &none            &sk LSHIFT       &sk RCTRL        &sk RALT        &sk RGUI        &none ,
                                                       &kp F10          &kp RET          &kp F12          , &kp K_CANCEL     &trans           &trans
)
```

- [ ] **Step 2: Verify right-side key positions against spec**

- RT0=`tog GAMING`, RT1=`BT_SEL 0`, RT2=`BT_SEL 1`, RT3=`BT_SEL 2`, RT4=`tog COLEMAK` ✓
- RM0=`OUT_TOG`, RM1=`BT_CLR`, RM2–RM4=`&none` ✓
- RB5=`&none`, RB (inner 4): `sk LSHIFT` `sk RCTRL` `sk RALT` `sk RGUI`, RB4=`&none` ✓
- RH0=`K_CANCEL`, RH1=`&trans`, RH2=`&trans` ✓
- LH2=`F10`, LH1=`Enter`, LH0=`F12` ✓

- [ ] **Step 3: Commit**

```bash
git add config/totem.keymap
git commit -m "feat: add FUN layer with F1-F12, BT profiles, output toggle, layout toggles"
```

---

## Chunk 3: Verification

### Task 6: Trigger CI build and verify

**Files:** None (read-only verification step)

ZMK uses GitHub Actions to build firmware. A successful build is the primary correctness check for keymap syntax — the ZMK compiler will reject malformed devicetree/binding syntax.

- [ ] **Step 1: Push commits to remote**

```bash
git push
```

- [ ] **Step 2: Open GitHub Actions**

Navigate to the repository on GitHub → Actions tab → watch the most recent workflow run.

- [ ] **Step 3: Verify build passes**

Expected: all build jobs complete with green ✓. If any job fails, click into it to read the compiler error — ZMK errors are usually descriptive (e.g., "undefined reference", "wrong number of bindings").

- [ ] **Step 4: Common errors and fixes**

| Error | Likely cause | Fix |
|-------|-------------|-----|
| `undefined symbol GAMING` | Missing `#define GAMING 4` | Check Task 1 Step 2 |
| `macro 'ZMK_HOLD_TAP' requires 2 bindings` | `fun_tab` binding count wrong | Verify `<&mo>, <&kp>` syntax |
| `undefined reference bt BT_SEL` | Missing `#include <dt-bindings/zmk/bt.h>` | Check Task 1 Step 1 |
| `wrong number of keys in layer` | Key count mismatch in a layer | Count keys against Totem's 38-key layout |

- [ ] **Step 5: Flash and manually test**

Once CI passes, download the firmware artifact and flash to your Totem. Manual test checklist:

**FUN layer (hold LH2):**
- [ ] F1–F9 fire correctly in the numpad positions
- [ ] F10/F11/F12 reachable from LT row and thumbs
- [ ] `tog COLEMAK` switches layout (RT4)
- [ ] `tog GAMING` enters gaming layer (RT0)
- [ ] BT_SEL 0/1/2 switch profiles (RT1–RT3)
- [ ] OUT_TOG switches between BT and USB (RM0)
- [ ] BT_CLR clears pairing (RM1)

**GAMING layer (enter via FUN RT0):**
- [ ] WASD fires from D·X·C·V physical positions
- [ ] Shift at A position (LM0)
- [ ] Space on LH2 (left outer thumb)
- [ ] Enter on LH1
- [ ] `tog GAME_PLUS` on LH0 enters number sublayer
- [ ] `&mo FUN` on RH0 (right inner thumb) opens FUN layer while held
- [ ] Right side letters fire with no hold-tap delay

**GAME_PLUS layer (tog from GAMING LH0):**
- [ ] Keys 1–5 fire from top row
- [ ] All other keys pass through to GAMING
- [ ] LH0 again exits back to GAMING
