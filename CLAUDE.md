# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ESP32-S3 based multimedia display device with four demonstration applications:
- **DEMO_LVGL** - LVGL graphics framework demo
- **DEMO_PIC** - JPEG image viewer (reads from SD card `/pic` folder)
- **DEMO_MJPEG** - MJPEG video player (reads from SD card `/mjpeg` folder)
- **DEMO_MP3** - MP3 audio player (reads from SD card `/music` folder)

Hardware: JC3248W535EN board with 320x480 QSPI LCD (AXS15231B controller), touch panel, SD/TF card, I2S audio output.

## Build System

### PlatformIO (recommended)

Each demo has a `platformio.ini` for building with PlatformIO. Requires the pioarduino platform for Arduino ESP32 3.x support (ESP-IDF 5.1).

```bash
cd 1-Demo/Demo_Arduino/DEMO_LVGL
pio run              # Compile
pio run -t upload    # Upload to board
pio device monitor   # Serial monitor
```

Key `platformio.ini` settings:
```ini
platform = https://github.com/pioarduino/platform-espressif32/releases/download/51.03.07/platform-espressif32.zip
board = esp32-s3-devkitc-1
framework = arduino
board_build.arduino.memory_type = qio_opi
build_flags = -DBOARD_HAS_PSRAM -DLV_CONF_INCLUDE_SIMPLE
lib_deps = lvgl/lvgl@^8.3.11
lib_extra_dirs = ../libraries
```

**Important:** The standard `espressif32` platform uses Arduino ESP32 2.x which is incompatible with this project's ESP-IDF 5.1 APIs. Use the pioarduino platform.

### Arduino IDE (alternative)

1. Copy libraries from `1-Demo/Demo_Arduino/libraries/` to Arduino's libraries folder
2. Select "ESP32-S3 Dev Module" board
3. Enable PSRAM, set PSRAM speed to 120M (required for MJPEG)
4. Open desired `DEMO_*.ino` file and upload

**Serial monitor:** 115200 baud

## Repository Structure

```
1-Demo/Demo_Arduino/
├── libraries/           # Required Arduino libraries (lvgl, ESP32_JPEG, audioI2S)
├── esp32s3/             # ESP32-S3 SDK config (copy to Arduino package for 120M PSRAM)
├── DEMO_LVGL/           # LVGL graphics demo
├── DEMO_PIC/            # JPEG image viewer
├── DEMO_MJPEG/          # MJPEG video player
├── DEMO_MP3/            # MP3 audio player
└── TF file/             # SD card folder structure template
```

## Architecture

Each demo follows the same structure:
- `DEMO_*.ino` - Entry point, initializes SD card, display, creates UI
- `esp_bsp.c/h` - Board Support Package (display init, touch, LVGL task)
- `esp_lcd_axs15231b.c/h` - QSPI LCD driver
- `esp_lcd_touch.c/h` - I2C touch panel driver
- `lv_port.c/h` - LVGL porting layer
- `pincfg.h` - GPIO pin definitions
- `display.h` - Display constants (320x480, RGB565)
- `lv_conf.h` - LVGL configuration

**PlatformIO structure** (created for pio builds):
- `src/` - Source files (main.cpp copied from .ino, plus .c files)
- `include/` - Header files (.h files)
- `platformio.ini` - Build configuration

**Key patterns:**
- Display access protected by mutex: `bsp_display_lock()` / `bsp_display_unlock()`
- Large buffers allocated from SPIRAM: `heap_caps_aligned_alloc(..., MALLOC_CAP_SPIRAM)`
- Demo selection via `#define CURR_DEMO` (1=LVGL, 2=PIC, 3=MJPEG, 4=MP3)

**LVGL demos** (in DEMO_LVGL, edit `src/main.cpp` setup function):
- `lv_demo_widgets()` - UI with Profile/Analytics/Shop tabs
- `lv_demo_benchmark()` - Performance testing
- `lv_demo_music()` - Music player interface
- `lv_demo_stress()` - Stress testing

## Hardware Pin Configuration

| Function | GPIO |
|----------|------|
| LCD CS | 45 |
| LCD PCLK | 47 |
| LCD DATA0-3 | 21, 48, 40, 39 |
| LCD DC | 8 |
| LCD Backlight | 1 |
| Touch SCL/SDA | 8, 4 |
| Touch INT | 3 |
| SD CLK/CMD/D0 | 12, 11, 13 |
| I2S BCK/LRCK/DO | 42, 2, 41 |
| Battery ADC | 5 |

## Dependencies

- **LVGL 8.3.9+** (not v9) - Graphics library
- **ESP32_JPEG** - Hardware JPEG decoder
- **JPEGDEC** - Software JPEG fallback
- **ESP32-audioI2S 3.0.12** - MP3 decoding and I2S output

## LVGL 8.x Coding Rules

**IMPORTANT:** This project uses LVGL 8.x. Do NOT use LVGL 7.x patterns.

**Events (obligatoire) :**
```c
// Correct - LVGL 8.x event callback
lv_obj_add_event_cb(btn, event_handler, LV_EVENT_CLICKED, NULL);

static void event_handler(lv_event_t *e) {
    lv_event_code_t code = lv_event_get_code(e);
    lv_obj_t *obj = lv_event_get_target(e);
    if (code == LV_EVENT_CLICKED) {
        // Handle click
    }
}
```

```c
// WRONG - LVGL 7.x (won't compile)
lv_btn_set_action(btn, LV_BTN_ACTION_CLICK, callback);  // Does not exist
lv_btn_get_state(btn);  // Does not exist
```

**Other LVGL 8.x changes:**
- Use `lv_obj_set_style_*()` instead of `lv_style_set_*()`
- Use `lv_obj_add_style()` instead of `lv_obj_set_style()`
- Screens: `lv_scr_act()` returns active screen, `lv_scr_load()` to switch

## Reference Documentation

- User manual: `6-User_Manual/Getting started JC3248W535 .pdf`
- Pin diagrams: `5-IO pin distribution/`
- IC datasheets: `4-Driver_IC_Data_Sheet/`
