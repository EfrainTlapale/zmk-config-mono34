# zmk-config-mono34

ZMK config for **mono34** — a 34-key unibody (monoblock) keyboard with a
Sweep/Ferris-style column stagger, built for a **SparkFun Pro Micro RP2040**
(`sparkfun_pro_micro_rp2040//zmk`) or any other Pro Micro–compatible controller.

```
╭─────┬─────┬─────┬─────┬─────╮ ╭─────┬─────┬─────┬─────┬─────╮
│  Q  │  W  │  E  │  R  │  T  │ │  Y  │  U  │  I  │  O  │  P  │
├─────┼─────┼─────┼─────┼─────┤ ├─────┼─────┼─────┼─────┼─────┤
│  A  │  S  │  D  │  F  │  G  │ │  H  │  J  │  K  │  L  │  ;  │
├─────┼─────┼─────┼─────┼─────┤ ├─────┼─────┼─────┼─────┼─────┤
│  Z  │  X  │  C  │  V  │  B  │ │  N  │  M  │  ,  │  .  │  /  │
╰─────┴─────┴─────┼─────┼─────┤ ├─────┼─────┼─────┴─────┴─────╯
                  │ NAV │ SFT │ │ CTL │ SYM │
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

4 rows × 10 columns, `col2row` (diode cathodes face the row lines — the stripe
on the diode points at the row). Pin numbers in the overlay are **Pro Micro pad
numbers**, not RP2040 GPIO numbers, which is what keeps the shield portable
across controllers.

| Matrix | Pro Micro pad | RP2040 GPIO | Direction |
| ------ | ------------- | ----------- | --------- |
| col 0  | D0            | GPIO1       | output    |
| col 1  | D1            | GPIO0       | output    |
| col 2  | D2            | GPIO2       | output    |
| col 3  | D3            | GPIO3       | output    |
| col 4  | D4            | GPIO4       | output    |
| col 5  | D5            | GPIO5       | output    |
| col 6  | D6            | GPIO6       | output    |
| col 7  | D7            | GPIO7       | output    |
| col 8  | D8            | GPIO8       | output    |
| col 9  | D9            | GPIO9       | output    |
| row 0  | D18 / A0      | GPIO26      | input     |
| row 1  | D19 / A1      | GPIO27      | input     |
| row 2  | D20 / A2      | GPIO28      | input     |
| row 3  | D21 / A3      | GPIO29      | input     |

Note that D0 and D1 are **crossed** on this board (D0 → GPIO1, D1 → GPIO0).
That is handled by the board's `pro_micro` connector node, so you only ever
think in D-numbers.

Left unused and available for an encoder, LEDs, or a trackball: **D10**
(GPIO21), **D14** (GPIO20), **D15** (GPIO22), **D16** (GPIO23).

These pins are a starting point — change them in
`config/boards/shields/mono34/mono34.overlay` to match however your PCB is
actually routed. If your diodes point the other way, set
`diode-direction = "row2col"` and swap the GPIO flags between `col-gpios` and
`row-gpios` (outputs get `GPIO_ACTIVE_HIGH`, inputs get
`GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN`).

## Layers

| # | Name  | Reached by                       |
| - | ----- | -------------------------------- |
| 0 | Base  | default                          |
| 1 | Sym   | hold right inner thumb           |
| 2 | Nav   | hold left inner thumb (Space)    |
| 3 | Num   | Sym + Nav together (tri-layer)   |
| 4 | Media | hold right pinky on Num          |
| 5 | Mouse | hold left index home row on Nav  |

`&bootloader` and `&sys_reset` live on the Media layer's bottom row.

## Building

Push to GitHub and the Actions workflow produces a `firmware` artifact
containing `mono34.uf2`. To flash:
hold **BOOT** while plugging in (or double-tap **RESET**), then drop the `.uf2`
onto the `RPI-RP2` drive.

`config/west.yml` tracks ZMK `main` on purpose. Do **not** pin it to the current
`v0.3.0` release: at that tag this board still lives at the old
`app/boards/arm/` path with the plain id `sparkfun_pro_micro_rp2040`, and the
`//zmk` board variant used in `build.yaml` does not exist yet. Tracking `main`
means an upstream breaking change can break a build that worked yesterday — if
that happens, pin `revision:` to a known-good commit SHA rather than to v0.3.0.

To fully wipe the flash — the RP2040 equivalent of a settings reset — flash
[`flash_nuke.uf2`](https://datasheets.raspberrypi.com/soft/flash_nuke.uf2)
first, then reflash the firmware.

## ZMK Studio

The build has ZMK Studio enabled (`studio-rpc-usb-uart` snippet +
`CONFIG_ZMK_STUDIO=y`), so the keymap can be edited live at
[zmk.studio](https://zmk.studio) over USB. Press `&studio_unlock` — the top-left
key on the Media layer — to unlock it for editing.

## Notes on the controller

The RP2040 has no radio, so this is a USB-only keyboard: there is no `&bt`
behavior, no battery reporting, and no deep-sleep power saving to configure.
If you port a keymap from a wireless 34-key board, strip out the `&bt` bindings
or the build will fail.
