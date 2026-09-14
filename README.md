# RP2040-Zero + ADXL345 USB Accelerometer

A low-cost, DIY USB accelerometer for Klipper Input Shaper calibration, with a compact custom enclosure.

By [id-ex](https://github.com/id-ex) · [MIT License](LICENSE)

[Русская версия](README.ru.md) · [Klipper configuration](config/adxl.cfg)

## Preview

<p align="center">
  <a href="3drender1.png"><img src="3drender1.png" width="32%" alt="Closed enclosure"></a>
  <a href="3drender2.png"><img src="3drender2.png" width="32%" alt="Enclosure with cover removed"></a>
  <a href="3drender3.png"><img src="3drender3.png" width="32%" alt="Sensor board assembly"></a>
</p>
<p align="center">
  <a href="photo1.jpg"><img src="photo1.jpg" width="32%" alt="Build photo 1"></a>
  <a href="photo2.jpg"><img src="photo2.jpg" width="32%" alt="Build photo 2"></a>
  <a href="photo3.jpg"><img src="photo3.jpg" width="32%" alt="Build photo 3"></a>
</p>
<p align="center"><sub>CAD renders and the finished build · Click an image for full resolution</sub></p>

## Why build it?

Ready-made USB accelerometers can cost noticeably more than an RP2040-Zero and an ADXL345 breakout. This project combines these inexpensive modules using short connections and a printed enclosure. The pin arrangement makes a compact soldered assembly practical. Actual savings depend on local prices and shipping.

An accelerometer measures printer vibrations so Klipper can select Input Shaper filters to reduce ringing/ghosting. It helps tune acceleration for better print quality, but does not fix loose belts or other mechanical problems. The sensor is only needed during calibration; saved filters work without it.

## Parts

- [RP2040-Zero — AliExpress](https://aliexpress.ru/item/1005006354505058.html?sku_id=12000036866363667&spm=a2g2w.productlist.search_results.1.4eba1ee6ZgtkmF) (Waveshare pin layout)
- [ADXL345 breakout — AliExpress](https://aliexpress.ru/item/1005001621867550.html?sku_id=12000016846764576&spm=a2g2w.productlist.search_results.0.4a32301cQ4KS5m) (GY-291 used here)
- Short wires/pin headers and soldering tools
- USB data cable
- Printed enclosure and a rigid mounting solution

Check the selected product variant, pin layout and module dimensions before ordering; listings may change.

## Wiring — this build

**Use GPIO labels, not physical pin numbers. This is not the GP0–GP3 wiring found in some Pico tutorials.** Software SPI is used.

| ADXL345 | RP2040-Zero | Function |
|---|---|---|
| VCC | 3V3 | Module power |
| GND | GND | Ground |
| CS | GP29 | Chip select |
| SCL / SCLK | GP14 | SPI clock |
| SDA / MOSI | GP15 | Controller → sensor |
| SDO / MISO | GP26 | Sensor → controller |
| INT1 | GP28 | Connected in this assembly; unused by Klipper |
| INT2 | GP27 | Connected in this assembly; unused by Klipper |

INT1/INT2 are optional and require no configuration. Do not configure their connected GPIOs as outputs. RP2040 GPIOs are **3.3 V only, not 5 V tolerant**. Verify the power-input requirements of your particular breakout before soldering. Disconnect power while wiring.

Keep SPI connections short. This reduces noise susceptibility but does not eliminate interference. Secure the USB cable without restricting printer movement.

## Firmware

Build from the Klipper source version used by your printer host. Back up any existing `.config` before changing build settings for the main printer board.

```bash
cd ~/klipper
make menuconfig
```

Select Raspberry Pi RP2040 architecture, RP2040 processor if separately listed, and USB communication. Save, then build:

```bash
make clean
make
```

Hold **BOOT** while connecting the Zero to a computer. Copy `out/klipper.uf2` to the mounted `RPI-RP2` drive. The board reboots automatically. Connect it to the **Klipper host**, not the printer's motor-control board.

## Klipper configuration

Copy [`config/adxl.cfg`](config/adxl.cfg) next to `printer.cfg`. On the host, find the board:

```bash
ls /dev/serial/by-id/
```

Replace the serial placeholder in the example with your board's actual path:

```ini
[mcu adxl]
serial: /dev/serial/by-id/usb-Klipper_rp2040_REPLACE_WITH_YOUR_ID-if00

[adxl345]
cs_pin: adxl:gpio29
spi_software_sclk_pin: adxl:gpio14
spi_software_mosi_pin: adxl:gpio15
spi_software_miso_pin: adxl:gpio26

[resonance_tester]
accel_chip: adxl345
probe_points:
    117.5, 117.5, 20
```

The probe point is an example from an Ender-3 build: check travel limits and clearance on your printer. Set `axes_map` if needed for your sensor orientation. Do not duplicate existing MCU, ADXL345 or resonance tester sections.

Some custom hosts use `/dev/adxl` instead of a by-id path. That alias requires host-side setup and is **not** created by this configuration.

Enable in `printer.cfg`, with the sensor connected:

```ini
[include adxl.cfg]
```

Save and restart Klipper.

## Calibration

Never run calibration during a print. Rigidly mount the sensor: loose mounts and soft foam tape distort measurements. Check that the enclosure and cable cannot collide with moving parts.

Check communication and noise in the Klipper console:

```text
ACCELEROMETER_QUERY
MEASURE_AXES_NOISE
```

For an **Ender-3 / moving-bed printer**, measure axes separately:

1. Mount the sensor on the print head for X:
   ```text
   G28
   SHAPER_CALIBRATE AXIS=X
   SAVE_CONFIG
   ```
2. Move the sensor to the bed for Y, verify its response, then run:
   ```text
   G28
   SHAPER_CALIBRATE AXIS=Y
   SAVE_CONFIG
   ```

`SAVE_CONFIG` saves the filter settings and restarts Klipper. Recommended acceleration limits are not applied automatically; review them separately. If moving the sensor requires unplugging USB, disable its include and restart first; re-enable after reconnecting.

For other kinematics, follow the mounting guidance in the [official resonance measurement documentation](https://www.klipper3d.org/Measuring_Resonances.html).

## Disconnect after calibration

Keep `[input_shaper]` and its saved values in `printer.cfg` / the SAVE_CONFIG block. **Do not put them in the removable sensor configuration.**

1. Comment out the include:
   ```ini
   #[include adxl.cfg]
   ```
2. Save and restart Klipper.
3. Unplug USB and remove the sensor.

Unplugging while the include is enabled causes an MCU connection error and stops Klipper. Reconnect and enable the include for future calibration. Recheck after changes to belts, mechanics or moving mass.

## Project files

- `config/adxl.cfg` — portable configuration template; edit the serial path
- `rp2040z+adxl345-case.step` — enclosure body
- `rp2040z+adxl345-cover.step` — enclosure cover
- `3drender1.png`, `3drender2.png`, `3drender3.png` — CAD renders
- `photo1.jpg`, `photo2.jpg`, `photo3.jpg` — build photos

STEP models can be opened in CAD and exported as STL/3MF for slicing. Check fit and mounting clearance for your own modules. Only the enclosure body and cover STEP files are published. The FreeCAD assembly, external component reference models and local backups are excluded from Git.

## License

Original project files (configuration, documentation, enclosure CAD and images) are released under the [MIT License](LICENSE). Third-party component models are not covered by this grant and remain subject to their respective licenses; they are excluded pending review. See [PUBLISHING.md](PUBLISHING.md).
