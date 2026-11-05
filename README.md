
# Doodle🖌️🎨 (arduino-touch-paint)
**Turn a bare‑metal Arduino Mega into a live paint canvas!**  
Touch‑responsive 3.5″ TFT lets you draw, pick colors, and watch raw calibration magic in real time. Dive in, tweak the resistive magic, and make the screen come alive.  

---  

## About the Project  

This repository contains a **hardware‑first paint application** built around an **Arduino Mega 2560** and a **3.5" TFT LCD Touch Shield (ILI9486 driver)**.  
- The sketch reads raw resistive‑touch values, maps them to screen pixels, and lets the user draw free‑hand lines and select colors directly on the display.  
- The biggest technical challenge was **calibration**: converting the analog resistance measurements (`TS_LEFT`, `TS_RT`, `TS_TOP`, `TS_BOT`) into an accurate pixel matrix.  
- The project started from a DIYmall tutorial, was stripped down to the essentials, and then expanded with a custom calibration routine and a minimal UI for color selection.  

The result is a **stand‑alone “paint” canvas** that lives entirely on the microcontroller—no PC, no OS, just raw Arduino code and a few well‑chosen libraries.  

---  

## Tech Stack  

| Layer | Technology / Library | Purpose |
|-------|----------------------|---------|
| **Microcontroller** | Arduino Mega 2560 | Provides enough GPIO pins and memory for the TFT shield and touch controller. |
| **Display** | 3.5" TFT LCD Touch Shield (ILI9486) | 480 × 320 pixel colour display with built‑in resistive touch layer. |
| **Graphics** | `MCUFRIEND_kbv` + `Adafruit_GFX` | Low‑level driver for the ILI9486 and high‑level drawing primitives (lines, rectangles, text). |
| **Touch** | `TouchScreen.h` (modified) | Reads raw X/Y resistance values from the touch panel and returns calibrated coordinates. |
| **Development Environment** | Arduino IDE (or PlatformIO) | Compiles and uploads the `.ino` sketches to the Mega. |
| **Media** | `MOV01361.AVI` | Demo video showing the canvas in action (useful for README or GitHub preview). |

---  

## Repository Structure  

```
arduino-touch-paint/
├─ MOV01361.AVI                     # Demo video – shows drawing, color picking and calibration UI.
├─ TouchScreen_Calibr_native/       # Full‑featured calibration sketch and helper files.
│   ├─ TouchScreen_Calibr_native.ino # Main sketch that runs the calibration routine and demo.
│   ├─ TouchScreen_kbv.cpp           # Implementation of the custom TouchScreen class that works
│   │                                 # with the MCUFRIEND_kbv driver (raw ADC reads, scaling, etc.).
│   └─ TouchScreen_kbv.h             # Header declaring the custom TouchScreen class, calibration
│                                     # constants, and helper functions.
└─ Touch_shield_new/                # Minimal “paint” application – the final user‑facing sketch.
    └─ Touch_shield_new.ino          # Core paint program: draws lines, handles colour palette,
                                      # and calls the calibrated TouchScreen class for touch input.
```

### File‑by‑file description  

| File | What it does |
|------|--------------|
| **MOV01361.AVI** | A short video (≈30 s) captured from the Arduino’s TFT screen. Demonstrates the canvas, colour picker, and the raw calibration screen. Great for a quick visual preview. |
| **TouchScreen_Calibr_native/TouchScreen_Calibr_native.ino** | The **calibration demo**. Starts by prompting the user to touch four corner points, records the raw ADC values, computes the calibration constants (`TS_LEFT`, `TS_RT`, `TS_TOP`, `TS_BOT`) and stores them in EEPROM. After calibration it runs a simple drawing demo using those constants, proving that the mapping works. |
| **TouchScreen_Calibr_native/TouchScreen_kbv.cpp** | Source implementation of a **custom TouchScreen class** that directly interfaces with the `MCUFRIEND_kbv` driver. It contains: <br>• Low‑level `readTouch()` that returns raw X/Y/pressure values. <br>• `getPoint()` that applies the calibration constants and returns a `TSPoint` in pixel coordinates. <br>• Helper functions for pressure filtering and optional smoothing. |
| **TouchScreen_Calibr_native/TouchScreen_kbv.h** | Header file exposing the custom class, the `TSPoint` struct, and the calibration constants. Also declares the EEPROM load/save helpers used by the calibration sketch. |
| **Touch_shield_new/Touch_shield_new.ino** | The **final paint application** that end‑users will upload. It: <br>1. Loads previously saved calibration data (or falls back to defaults). <br>2. Initializes the TFT (`MCUFRIEND_kbv`) and the custom `TouchScreen` object. <br>3. Draws a colour palette bar at the top of the screen. <br>4. In the main loop, reads touch points, maps them to screen coordinates, and draws lines in the currently selected colour. <br>5. Handles “long‑press” on the palette to change colour. The code is deliberately lightweight to keep RAM usage under the Mega’s limits. |

---  

## Getting Started  

### Prerequisites  

1. **Hardware**  
   - Arduino Mega 2560 (or compatible board).  
   - 3.5" TFT LCD Touch Shield with ILI9486 driver (the Arduino‑compatible “Touch Shield” from DIYmall works).  
   - USB cable for programming.  
   - Optional: Stylus or conductive tip for more precise drawing.  

2. **Software**  
   - **Arduino IDE** (v1.8.19 or later) **or** PlatformIO.  
   - Libraries (install via *Sketch → Include Library → Manage Libraries*):  
     - `MCUFRIEND_kbv` (latest)  
     - `Adafruit GFX Library` (latest)  
     - `TouchScreen` (the original library – we replace it with our custom version, but the original is needed for the header files).  

3. **Serial Monitor** (optional) – to view calibration status or debug prints.  

### Installation  

```bash
# 1. Clone the repository
git clone https://github.com/yourusername/arduino-touch-paint.git
cd arduino-touch-paint

# 2. Open the Arduino IDE
#    (or open the folder in PlatformIO)
```

#### Library setup (Arduino IDE)

1. Open **Sketch → Include Library → Manage Libraries…**  
2. Search for **MCUFRIEND_kbv** → Install.  
3. Search for **Adafruit GFX** → Install.  
4. Search for **TouchScreen** → Install (the official library).  

#### Adding the custom TouchScreen class  

The custom class lives in `TouchScreen_Calibr_native/`.  
- Copy `TouchScreen_kbv.h` and `TouchScreen_kbv.cpp` into a folder called `libraries/TouchScreen_kbv/` inside your Arduino sketchbook (or place them in the same folder as the sketch).  
- Restart the Arduino IDE so the new library is recognized.  

### How to Run  

#### 1️⃣ First‑time Calibration (only needed once or after hardware changes)

1. Open **`TouchScreen_Calibr_native/TouchScreen_Calibr_native.ino`** in the IDE.  
2. Select **Tools → Board → Arduino Mega 2560** and the correct COM port.  
3. Click **Upload**.  
4. When the sketch starts, the screen will ask you to **touch the four corners** of the display.  
5. Follow the prompts; the sketch will store the calibration constants in the Mega’s EEPROM.  
6. After calibration, a simple drawing demo will run—verify that the stylus follows your finger accurately.  

> **Tip:** If you need to reset the EEPROM (e.g., after a bad calibration), open the Serial Monitor, set the baud rate to **9600**, and send the character **`r`**. The sketch will clear the stored constants and force a new calibration on next start.  

#### 2️⃣ Run the Paint Application  

1. Open **`Touch_shield_new/Touch_shield_new.ino`**.  
2. Verify that the same board/port settings are selected.  
3. Click **Upload**.  
4. When the program starts you’ll see a colour palette bar at the top.  
5. **Touch and drag** anywhere on the canvas to draw with the selected colour.  
6. **Long‑press** (≈1 s) on a colour square to switch to that colour.  

The sketch reads touch points via `TouchScreen.getPoint()`, which already applies the calibration data saved earlier, so you should get pixel‑perfect drawing.  

### Optional: Using the Demo Video  

The file `MOV01361.AVI` can be opened with any standard media player (VLC, Windows Media Player, etc.) to see a quick showcase of the canvas in action. Feel free to embed it in GitHub’s README using a GIF or a short clip if you publish the repo on GitHub Pages.  

---  

## Contributing  

Contributions are welcome! Typical ways to help:  

- **Improve calibration** – add auto‑recalibration or a UI button to trigger it.  
- **Add more UI features** – e.g., brush size selector, eraser mode, save‑to‑SD.  
- **Port to other boards** – adapt the code for Arduino Uno, ESP32, or STM32.  

Please fork the repo, make your changes, and open a Pull Request. Include a short description of what you changed and why.  

---  

## License  

This project is licensed under the **MIT License** – see the `LICENSE` file for details.  

---  

### Happy painting! 🎨  

If you run into any issues, open an **Issue** on GitHub and we’ll try to help. Enjoy turning raw resistive touch into a living digital canvas.  
