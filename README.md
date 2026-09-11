# zmk-config-mono34

ZMK config for **mono34** — a 34-key unibody (monoblock) keyboard with a
Sweep/Ferris-style column stagger, built for a **Waveshare RP2040-Zero**
(`rp2040_zero`).

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

4 rows × 10 columns, `col2row` (diode cathodes face the row lines — the stripe
on the diode points at the row). Pins go through the board's **`&zero_d`
nexus**, which maps 1:1 onto the RP2040 GPIOs (`&zero_d N` is GPN), so the
numbers here match the silkscreen on the controller.

| Matrix | RP2040 GPIO | Direction |
| ------ | ----------- | --------- |
| col 0  | GP9         | output    |
| col 1  | GP8         | output    |
| col 2  | GP7         | output    |
| col 3  | GP6         | output    |
| col 4  | GP5         | output    |
| col 5  | GP4         | output    |
| col 6  | GP3         | output    |
| col 7  | GP2         | output    |
| col 8  | GP1         | output    |
| col 9  | GP0         | output    |
| row 0  | GP10        | input     |
| row 1  | GP11        | input     |
| row 2  | GP12        | input     |
| row 3  | GP13        | input     |

Note the column order: col 0 (the leftmost column, `Q`/`A`/`Z`) is on **GP9**
and the columns count *down* to GP0 on the right. That mirrors how the ribbon
lands on the board; swap the list around in the overlay if your PCB is routed
the other way.

### Which pins are usable

The RP2040-Zero exposes 29 GPIOs, but not equally:

| Pins | Where |
| ---- | ----- |
| GP0..GP15, GP26..GP29 | castellated pads on the edge |
| GP17..GP25 | solder pads on the **back** of the board |
| GP16 | back pad, and wired to the onboard **WS2812** |

All 14 matrix lines above sit on edge pads. **Avoid GP16 for matrix lines** —
the board devicetree deliberately leaves the RGB LED out (the comment in
`rp2040_zero.dts` notes it stays absent until Zephyr is new enough for PIO), so
it will build fine, but the pin is still physically on the LED's data-in net and
column strobes will make it flicker.

Of the edge pins the matrix leaves free, the trackpad takes **GP14, GP15, GP26
and GP27** (see [Trackpad](#trackpad) below), leaving **GP28** and **GP29** for
an encoder or LEDs. Comment out the `mono34-trackpad.dtsi` include in the
overlay to get the trackpad's four back as well.

These pins are a starting point — change them in
`config/boards/shields/mono34/mono34.overlay` to match however your PCB is
actually routed. If your diodes point the other way, set
`diode-direction = "row2col"` and swap the GPIO flags between `col-gpios` and
`row-gpios` (outputs get `GPIO_ACTIVE_HIGH`, inputs get
`GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN`).

## Trackpad

An **Azoteq TPS43-201A-S** (43 × 43 mm ProxSense standard trackpad module,
IQS572 controller) is wired to the same controller and driven by
[AYM1607/zmk-driver-azoteq-iqs5xx](https://github.com/AYM1607/zmk-driver-azoteq-iqs5xx),
pulled in as a west module in `config/west.yml` (pinned to a commit). TPS43 is
one of the two modules the driver author lists as tested.

All of the devicetree for it lives in
`config/boards/shields/mono34/mono34-trackpad.dtsi`, included from the shield
overlay.

| TPS43 FPC pin | RP2040-Zero pin | Function          |
| ------------- | --------------- | ----------------- |
| VDDHI / VREG  | 3V3             | 3.3 V supply      |
| GND           | GND             | ground            |
| SDA           | GP14            | I2C1 SDA          |
| SCL           | GP15            | I2C1 SCL          |
| RDY           | GP26            | data-ready IRQ    |
| NRST          | GP27            | reset, active low |

All six sit on the same edge of the RP2040-Zero (3V3, GND, GP26–GP29, GP14,
GP15), so the trackpad's FPC breakout only has to reach one side of the board.
Two board defaults get overridden for this:

- The board puts **I2C1 on GP22/GP23**, which are solder pads on the back of the
  RP2040-Zero. The dtsi repins `i2c1_default` onto the edge pins GP14/GP15.
- The board enables the **ADC with a pinctrl state on GP26–GP29**, which
  includes RDY and NRST. Nothing on a USB-only board reads the ADC, so the dtsi
  disables `&adc`.

### Pull-ups and bus speed

The TPS43 module has no I2C pull-up resistors. Fit **2.2 kΩ–4.7 kΩ from SDA and
SCL to 3V3** if you can; the dtsi also enables the RP2040's internal pull-ups
(~55 kΩ) as a fallback and runs the bus at **100 kHz**, which those weak
resistors can actually drive. With proper external pull-ups you can raise
`clock-frequency` back to `400000`.

### Behaviour

The driver reports, out of the box as configured here: cursor movement, tap →
left click, two-finger tap → right click, tap-and-hold → held left click (drag),
and two-finger scroll on both axes with natural scrolling on Y.

Tuning knobs, all in the dtsi:

- **Orientation** — uncomment `switch-xy`, `flip-x` and/or `flip-y` until the
  cursor follows your finger. Which you need depends on which way the FPC tail
  points once the pad is mounted; expect to try one or two combinations.
- **Sensitivity** — add an input processor on the listener, e.g.
  `input-processors = <&zip_xy_scaler 3 2>;` for 1.5× or `<&zip_xy_scaler 1 2>`
  for half speed.
- **Feel** — `bottom-beta` (0 = heaviest filtering/laggiest, 255 = least
  filtered/most responsive) and `stationary-threshold` (pixels of movement
  before the finger counts as moving).
- **Coarse scrolling** — the driver emits one wheel detent per 32 counts.
  Uncomment `CONFIG_ZMK_POINTING_SMOOTH_SCROLLING=y` in `mono34.conf` if your
  host supports HID resolution multipliers.

Optionally, hook the trackpad to the existing Mouse layer so its clicks and
scroll keys land on the thumbs whenever you touch the pad. Add this to
**`mono34.keymap`** rather than the dtsi — that is where the `MOUSE` define is
in scope:

```dts
&trackpad_listener {
    input-processors = <&zip_temp_layer MOUSE 500>;
};
```

(500 ms is how long the layer stays up after you stop moving.)

### If it does not work

Build the firmware with USB logging (uncomment `CONFIG_ZMK_USB_LOGGING=y` in
`mono34.conf`), then watch the CDC-ACM console. The driver logs
`IQS5xx trackpad initialized` on success; `Failed to read system info` or
`Failed to setup device` means the I2C side is not talking — check pull-ups,
that SDA/SCL are not swapped, and that the FPC is seated.

## Layers

| # | Name  | Reached by                       |
| - | ----- | -------------------------------- |
| 0 | Base  | default                          |
| 1 | Sym   | hold right inner thumb           |
| 2 | Ext   | hold left inner thumb (Space)    |
| 3 | Num   | Sym + Ext together (tri-layer)   |
| 4 | Misc  | hold right pinky on Num          |
| 5 | Mouse | hold E on Ext                    |

`&sys_reset` and `&bootloader` live on the Misc layer's left column.

Extras carried over from the endgame config: **J + K** chorded is `ESC`
(50 ms window, 150 ms prior-idle guard), the left shift thumb is a
quick-release sticky shift, and right `/` is a `LGUI` hold-tap. The `Ext`
layer has held mods on the home row and sticky mods on the row below,
plus `HYPER` (`LS(LC(LA(LGUI)))`) on the left index inner column.

## Building

Push to GitHub and the Actions workflow produces a `firmware` artifact
containing `mono34.uf2`. To flash:
hold **BOOT** while plugging in (or hold **BOOT** and tap **RESET**), then drop
the `.uf2` onto the `RPI-RP2` drive.

### Why a ZMK fork?

Upstream ZMK does not ship the Waveshare RP2040-Zero. `config/west.yml` points
at [`nmunnich/zmk`](https://github.com/nmunnich/zmk), which adds
`app/boards/arm/waveshare_rp2040_zero` (board id `rp2040_zero`) together with
the `&zero_d` GPIO nexus this shield's overlay uses.

It is pinned to the SHA `aa2294c` rather than to the `rp2040zero` branch name,
because that is a personal branch and can be rebased or force-pushed. The
Actions workflow is correspondingly pinned to
`build-user-config.yml@v0.3`, to match the Zephyr and toolchain vintage the
fork expects — bumping one without the other is the usual cause of a build that
suddenly stops working.

If the board ever lands upstream, this config can go back to
`zmkfirmware/zmk` and drop the fork.

To fully wipe the flash — the RP2040 equivalent of a settings reset — flash
[`flash_nuke.uf2`](https://datasheets.raspberrypi.com/soft/flash_nuke.uf2)
first, then reflash the firmware.

## Notes on the controller

The RP2040 has no radio, so this is a USB-only keyboard: no bluetooth output,
no battery reporting, and no deep-sleep power saving to configure.

If you port a keymap from a wireless 34-key board, `&bt` bindings do **not**
break the build — the behavior node is declared in ZMK's devicetree regardless
of `CONFIG_ZMK_BLE`, so the keymap still compiles. What gets dropped is the
driver behind it, so those keys are simply **dead at runtime**. That silence is
easy to misread as a wiring fault, so it is worth replacing them with `&none`
(or something useful) rather than leaving them in.
