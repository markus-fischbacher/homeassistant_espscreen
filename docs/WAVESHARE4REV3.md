# Waveshare ESP32-S3-Touch-LCD-4, Hardware Revision 3.0, draft/experimental

Not yet added to Tessera Screen Manager (its `boards.json` has not been regenerated for this board - see
"Not done yet" below). This is the plain **ESP32-S3-Touch-LCD-4** (no "B", no "V4.0" suffix): a 4-inch
480 x 480 IPS panel with an ST7701S driver over RGB, GT911 capacitive touch, and a **TCA9554** I/O expander.

It is **not** the same board as either of these two, which look similar but use other pins:
- [WAVESHARE4B.md](WAVESHARE4B.md) - the "4B" board.
- [WAVESHARE4V4.md](WAVESHARE4V4.md) - Hardware Revision 4.0 of this same board name, which has Waveshare's
  CH32V003 expander instead of a TCA9554.

Physical acceptance has not been performed. The RGB data bus pins, the init-SPI pins and the display
timings are **not confirmed on real hardware** - see "Hardware references" below for exactly which parts
are solid and which are guesses.

## Install

Tessera Screen Manager cannot build this board yet (its board picker only lists what is in `boards.json`,
which this board has not been added to). Until it is, build and flash it directly with ESPHome instead of
through the app, then let Home Assistant pick it up through the regular ESPHome integration.

### Option A - ESPHome add-on inside Home Assistant (no separate computer needed)

1. Install the **ESPHome** add-on from the Home Assistant add-on store, if you don't have it yet.
2. In the ESPHome dashboard, create a new device and skip the guided setup to get a blank YAML editor.
3. Replace its contents with:

   ```yaml
   substitutions:
     DEVICE_NAME: "waveshare-rev3"
     DEVICE_FRIENDLY_NAME: "Waveshare Rev3 Test"

   packages:
     display:
       url: https://github.com/markus-fischbacher/homeassistant_espscreen.git
       ref: add-waveshare-4-rev3-board
       files: [packages/waveshare4rev3.yaml]
       refresh: 0s

   wifi:
     ssid: !secret wifi_ssid
     password: !secret wifi_password
     power_save_mode: none
     ap:
       ssid: "Tessera ${DEVICE_FRIENDLY_NAME}"
       password: !secret ap_password

   captive_portal:

   api:
     encryption:
       key: !secret api_encryption_key

   ota:
     - platform: esphome
       password: !secret ota_password
   ```

4. Add `wifi_ssid`, `wifi_password`, `ap_password`, `api_encryption_key` (32 random bytes, base64) and
   `ota_password` to the add-on's `secrets.yaml`.
5. Connect the board over USB, choose **Install -> Plug into this computer**, and flash it.
6. Home Assistant's ESPHome integration discovers the device on your Wi-Fi by itself afterwards.

### Option B - ESPHome CLI from a checkout

```bash
git clone --branch add-waveshare-4-rev3-board https://github.com/markus-fischbacher/homeassistant_espscreen.git
cd homeassistant_espscreen
cp checkout/secrets.yaml.example checkout/secrets.yaml   # fill in Wi-Fi and the keys
esphome run checkout/waveshare4rev3.yaml
```

Requires Python and `pip install esphome==2026.9.0` (the version this repository is tested against,
`requirements.txt`) on whichever computer runs the command.

Keep a USB cable and `esptool`/the ESPHome web flasher ready for a recovery reflash: with low-confidence
pins, a dark screen or dead touch on the first boot is expected, not a sign of bricked hardware.

## What is and isn't confirmed

| Area | Confidence | Source |
|---|---|---|
| I2C bus (SDA GPIO15, SCL GPIO7) | High | Waveshare's own V3.0 example repo + vincentmakes/ComfoSense-Touch |
| TCA9554 address (0x20) and its pin roles (EXIO0 touch reset, EXIO1 backlight enable, EXIO2 LCD reset) | High | Same two sources, independently |
| GT911 touch interrupt (GPIO16, direct) | High | Same two sources |
| ST7701S init sequence (`model: ST7701S` in ESPHome's `mipi_rgb`) | Medium | Cross-checked against Waveshare's own bundled driver library (`esp_lcd_st7701_rgb.c`); generic, not vendor-tuned for this exact glass |
| RGB data bus pins (DE 40, VSYNC 39, HSYNC 38, PCLK 41, R/G/B as in the YAML) | High (source), untested on hardware | Waveshare's V3.0 branch, `examples/Arduino-v3.1.3/examples/01_HelloWorld.ino` (`Arduino_ESP32RGBPanel`); also matches [ay129-35MR](https://github.com/ay129-35MR/Waveshare-ESP32-S3-Touch-LCD-4-Home-Assistant-Display) |
| Init-SPI pins (CLK GPIO2, MOSI GPIO1, CS GPIO42) | High (source), untested on hardware | Same file (`Arduino_SWSPI`) |
| HSYNC timings (pulse 8, back porch 50, front porch 10) and VSYNC timings (pulse 8, back porch 20, front porch 10) | High (source), untested on hardware | Same file; previously a guess copied from the "4B" board |
| Pixel clock (16 MHz), `pclk_inverted`, `color_order`, `invert_colors` | Guess | Not given by the Arduino example; kept from the "4B" board |
| Flash size, UART | Guess | Copied from the "4B" board, not verified |

If the picture is wrong or missing, check in this order: pixel clock and polarity/colour settings, then the
timings, then the pins. The Waveshare wiki and schematic were not read (bot check), so the pins are confirmed
against Waveshare's example code only, not against the schematic.

## Hardware references

- [waveshareteam/ESP32-S3-Touch-LCD-4, branch V3.0](https://github.com/waveshareteam/ESP32-S3-Touch-LCD-4/tree/V3.0) -
  Waveshare's own example repository for this exact board and revision.
- [vincentmakes/ComfoSense-Touch](https://github.com/vincentmakes/ComfoSense-Touch) - a community firmware that
  auto-detects Revision 3 (TCA9554) vs Revision 4 (CH32V003) hardware; its `src/board_config.h` is the second
  independent confirmation of the I2C bus, expander address and GT911 pin.
- [ay129-35MR/Waveshare-ESP32-S3-Touch-LCD-4-Home-Assistant-Display](https://github.com/ay129-35MR/Waveshare-ESP32-S3-Touch-LCD-4-Home-Assistant-Display) -
  the only source found for the RGB bus and init-SPI pins; its own `esphome.yaml` is incomplete and untested.
- [ESPHome MIPI RGB](https://esphome.io/components/display/mipi_rgb/), `model: ST7701S`.
- Waveshare's `www.waveshare.com/wiki/ESP32-S3-Touch-LCD-4` wiki page could not be read while drafting this board
  (it returns an automated bot-check page to scripted fetches); re-checking it against the table above would
  raise several of these from Low to High confidence.

## Not done yet

- `screen_manager/app/boards.json` has not been regenerated (`tools/generate_board_shapes.py`), so Tessera
  Screen Manager's board picker does not list this board.
- `packages/waveshare4rev3.yaml` and `checkout/waveshare4rev3.yaml` were written by hand, mirroring the "4B"
  board's generated files, since Python (`tools/generate_entries.py`) was not available while drafting this.
  Re-run that generator to confirm they match.
- No physical acceptance: nobody has confirmed any of the Low-confidence pins or the timings on real hardware.

## What to report while testing

1. Board revision (confirm "Rev 3.0" is printed on it, and that the I/O expander is the TCA9554, not a
   CH32V003) and whether it boots and reaches the display's `init_sequence` without a crash.
2. Does the backlight come on? Confirms the EXIO1 expander pin role.
3. Does the picture appear at all, and if so, is it the right colours, mirrored, shifted, or torn? Points at
   the RGB bus pins and the timings respectively.
4. Does touch register anywhere on the glass, and does it land where you actually tapped? Confirms the GT911
   interrupt pin and its orientation.
5. Any reset, brownout, or I2C error in the log, especially right after power-on.

Do not publish Wi-Fi passwords or API/OTA keys with logs.
