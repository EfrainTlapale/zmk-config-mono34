# zmk-config-mono34

ZMK config for **mono34** — a 34-key unibody (monoblock) keyboard with a
Sweep/Ferris-style column stagger. Wireless, on a single **nice!nano v2**
(`nice_nano//zmk`) soldered flush to the back of the ergogen PCB
`mono34_unibody_inserts` (`ergogen/config.yaml`), in the printed case in
`cad/src/unibody_inserts/`.

```
╭─────┬─────┬─────┬─────┬─────╮ ╭─────┬─────┬─────┬─────┬─────╮
│  Q  │  W  │  E  │  R  │  T  │ │  Y  │  U  │  I  │  O  │  P  │
├─────┼─────┼─────┼─────┼─────┤ ├─────┼─────┼─────┼─────┼─────┤
│  A  │  S  │  D  │  F  │  G  │ │  H  │  J  │  K  │  L  │  :  │
├─────┼─────┼─────┼─────┼─────┤ ├─────┼─────┼─────┼─────┼─────┤
│  Z  │  X  │  C  │  V  │  B  │ │  N  │  M  │  ,  │  .  │ GUI │
│     │     │     │     │     │ │     │     │     │     │  /  │
╰─────┴─────┴─────┼─────┼─────┤ ├─────┼─────┼─────┴─────┴─────╯
                  │ EXT │ SFT │ │ CTL │ SYM │
                  │ SPC │     │ │ ENT │     │
                  ╰─────┴─────╯ ╰─────┴─────╯
```

## Layout

Unlike the Sweep, the two halves are butted together into a single 10-column
grid — the column stagger is kept, the gap is not. The four thumb keys sit
centred under columns 4–7. Key order is identical to Sweep/Cradio (three rows
of ten left-to-right, then the four thumbs left-to-right), so bindings can be
copied between this keymap and any other 34-key ZMK config unchanged.

## Wiring

4 rows × 10 columns, `col2row` (diode cathodes face the row lines). The pins are
the PCB's nets, which ergogen names after the Pro Micro pads: `P16` on the PCB
is `&pro_micro 16` (D16) in the overlay.

| Matrix | PCB net / `&pro_micro` | nRF52840 | Direction |
| ------ | ---------------------- | -------- | --------- |
| col 0 — left pinky  | D16 | P0.10 | output |
| col 1 — left ring   | D14 | P1.11 | output |
| col 2 — left middle | D15 | P1.13 | output |
| col 3 — left index  | D18 | P1.15 | output |
| col 4 — left inner  | D19 | P0.02 | output |
| col 5 — right inner | D5  | P0.24 | output |
| col 6 — right index | D6  | P1.00 | output |
| col 7 — right middle| D7  | P0.11 | output |
| col 8 — right ring  | D8  | P1.04 | output |
| col 9 — right pinky | D9  | P1.06 | output |
| row 0 — top         | D21 | P0.31 | input  |
| row 1 — home        | D20 | P0.29 | input  |
| row 2 — bottom      | D0  | P0.08 | input  |
| row 3 — thumbs      | D4  | P0.22 | input  |

The thumbs share the index and inner columns of their half, so they land on
row 3 under columns 3–6. D16 and D10 are the nRF52840's NFC pins; the overlay
sets `&uicr { nfct-pins-as-gpios; }` so D16 works as a column.

Free for later: D1, D2, D3 (enough for a nice!view), D10, and the nice!nano's
extra pins P1.01, P1.02, P1.07, which the PCB breaks out.

## Layers

| # | Name  | Reached by                       |
| - | ----- | -------------------------------- |
| 0 | Base  | default                          |
| 1 | Sym   | hold right inner thumb           |
| 2 | Ext   | hold left inner thumb (Space)    |
| 3 | Num   | Sym + Ext together (tri-layer)   |
| 4 | Misc  | hold right pinky on Num          |
| 5 | Mouse | hold E on Ext                    |

`&sys_reset` and `&bootloader` live on the Misc layer's left column. The
Misc layer's left home row holds Bluetooth: profiles 0, 1, 2, then
`BT_CLR` (forget the current profile's host).

Extras carried over from the endgame config: **J + K** chorded is `ESC`
(50 ms window, 150 ms prior-idle guard), the left shift thumb is a
quick-release sticky shift, and right `/` is a `LGUI` hold-tap. The `Ext`
layer has held mods on the home row and sticky mods on the row below,
plus `HYPER` (`LS(LC(LA(LGUI)))`) on the left index inner column.

## Building

Push to GitHub and the Actions workflow produces a `firmware` artifact with
`mono34.uf2` and `settings_reset.uf2`. To flash: double-tap the nice!nano's
reset (the PTS636 under the hole in the base, or `&bootloader` on the Misc
layer), then drop the `.uf2` onto the `NICENANO` drive.

If pairing gets into a bad state, flash `settings_reset.uf2` (it wipes the
stored Bluetooth bonds), then flash `mono34.uf2` again and re-pair.

`config/west.yml` tracks ZMK `main` on purpose: `nice_nano//zmk` is the
hardware-model-v2 name from the Zephyr 4.1 update, and the `&uicr` NFC
property needs it too. Tracking `main` means an upstream breaking change can
break a build that worked yesterday; if that happens, pin `revision:` to a
known-good commit SHA.

## Power

The 401230 cell (130 mAh) is soldered to the PCB's battery pads and goes
battery + → power switch (Alps SSSS811101, rear edge) → RAW. The nice!nano
charges it over USB at about 100 mA, **only with the power switch ON**: OFF
disconnects the battery. `CONFIG_ZMK_SLEEP` puts the board into deep sleep
after 15 minutes idle; any key wakes it.
