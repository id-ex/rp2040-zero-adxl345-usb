# RP2040-Zero + ADXL345 USB Accelerometer for Klipper

[Русская версия](README.ru.md) · [Configuration example](config/adxl.cfg)

## Preview

<p align="center">
  <a href="3drender3.png"><img src="3drender3.png" width="32%" alt="Sensor board assembly"></a>
  <a href="3drender2.png"><img src="3drender2.png" width="32%" alt="Enclosure with cover removed"></a>
  <a href="3drender1.png"><img src="3drender1.png" width="32%" alt="Closed enclosure"></a>
</p>
<p align="center">
  <a href="photo3.jpg"><img src="photo3.jpg" width="32%" alt="Build photo 3"></a>
  <a href="photo2.jpg"><img src="photo2.jpg" width="32%" alt="Build photo 2"></a>
  <a href="photo1.jpg"><img src="photo1.jpg" width="32%" alt="Build photo 1"></a>
</p>
<p align="center"><sub>CAD renders and the finished build · Click an image to open it</sub></p>

## Why this project?
Ready-made USB accelerometers for printer calibration can cost noticeably more than two inexpensive modules — RP2040-Zero and ADXL345 (GY-291). This project combines them into a compact USB sensor with a printed enclosure. Pin headers and short jumpers make the assembly simple, with very few parts. Actual savings depend on local prices.

The accelerometer measures mechanical vibrations. Klipper uses these measurements to select Input Shaping filters that reduce ringing and ghosting on printed surfaces. This helps choose accelerations that maintain acceptable print quality. The sensor itself does not suppress vibrations or fix mechanical problems. Once calibration is complete, it can be removed: the filters work without it.

The repository contains enclosure body and cover models in [STEP](STEP/) for CAD and [STL](STL/) for printing, renders, build photographs and a `config/adxl.cfg` example.

## Parts

- [RP2040-Zero](https://aliexpress.ru/item/1005006354505058.html?sku_id=12000036866363667&spm=a2g2w.productlist.search_results.1.4eba1ee6ZgtkmF)
- [ADXL345 GY-291](https://aliexpress.ru/item/1005001621867550.html?sku_id=12000016846764576&spm=a2g2w.productlist.search_results.0.4a32301cQ4KS5m)
- 8-pin header.
- Good-quality USB data cable.
- 3D-printed enclosure (design a mounting solution for your own printer).

Before ordering, check the selected product variant, pin layout and module dimensions: listings may change.

## Wiring
This pinout matches the assembled device's configuration. It is not the GP0–GP3 layout found in some Pico tutorials. Follow GPIO labels rather than the board's orientation in photographs.

| ADXL345 pin | RP2040-Zero | Function |
|---|---|---|
| VCC | 3V3 | Module power |
| GND | GND | Ground |
| CS | GP29 | Chip select |
| INT1 | GP28 | Connected in this build; unused by Klipper |
| INT2 | GP27 | Connected in this build; unused by Klipper |
| SCL / SCLK | GP14 | SPI clock |
| SDA / MOSI | GP15 | Data from controller |
| SDO / MISO | GP26 | Data from sensor |

Software SPI is used. INT1/INT2 connections are optional for ADXL345 operation in Klipper; do not configure the corresponding GPIOs as outputs. Logic is 3.3 V; RP2040 GPIOs are not 5 V tolerant. Short SPI connections reduce noise susceptibility but do not eliminate interference. The USB cable must support data transfer.

## Firmware
The RP2040 acts as an additional MCU. USB connects to the Klipper host, not the motor-control board.

Build firmware from the Klipper source version installed on your host. For RP2040-Zero, select Raspberry Pi RP2040 architecture, RP2040 processor (if listed separately), and USB communication:

```bash
cd ~/klipper
make menuconfig
make clean
make
```

1. Hold BOOT while connecting the Zero to your computer.
2. Copy `out/klipper.uf2` to the `RPI-RP2` drive. The board reboots after copying.

## Configuration
The portable example is in `config/adxl.cfg`. Copy it next to `printer.cfg` and specify your board's serial path.

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
    117.5, 117.5, 20 # Set coordinates for your printer
```

Replace the serial placeholder with the actual path from `ls /dev/serial/by-id/`. The `/dev/adxl` alias only works if separately configured on the host. The probe point is specific to this printer; check travel limits and clearance on yours. If the sensor orientation changes, set `axes_map` in `[adxl345]` as needed.

## Enabling and calibration
Keep the sensor include disabled by default: `#[include adxl.cfg]`.

1. Rigidly attach the sensor to the component being measured. Soft tape and a loose enclosure distort measurements. The cable must not obstruct movement.
2. Connect USB to the Klipper host. Place `adxl.cfg` next to `printer.cfg`.
3. Enable this line in `printer.cfg`:
   ```ini
   [include adxl.cfg]
   ```
4. Select Save & Restart.
5. Check the sensor in the Mainsail console:
   ```text
   ACCELEROMETER_QUERY
   MEASURE_AXES_NOISE
   ```
6. Follow the calibration instructions at [klipper3d.org](https://www.klipper3d.org/Measuring_Resonances.html).

The acceleration recommended by the test is not applied automatically — review it separately. Do not copy another printer's frequencies: use your own measurements.

## Disconnecting after calibration
1. Keep the saved `[input_shaper]` settings in `printer.cfg` / the SAVE_CONFIG block — they are needed during printing.
2. Comment out only the sensor include:
   ```ini
   #[include adxl.cfg]
   ```
3. Select Save & Restart, then unplug USB and remove the sensor.

Do not move `[input_shaper]` into the removable `adxl.cfg`. Unplugging USB while the include is active makes Klipper lose the additional MCU and stop. For the next calibration, reconnect the module and enable the include again. Repeat measurements after changes to mechanics, belt tension or head/bed mass.

## Publication and license
Original project files (configuration, documentation, enclosure CAD and images) are released under the [MIT License](LICENSE). This does not cover third-party component models. Only the enclosure body and cover are published as 3D files, in [STEP](STEP/) and [STL](STL/) formats.
