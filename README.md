# PANELWRIGHT

**Field Instrument 042**

A single-file, browser-based UI designer for the Arduino Giga Display Shield.
Drag LVGL widgets onto a true 1:1 canvas, edit their properties, and export
a complete Arduino sketch ready to flash.

No build step. No CDN. No accounts. Open the HTML file in any modern browser
and start drafting.

---

## What it is

PANELWRIGHT is a workbench for laying out touch UIs on the Giga Display Shield
without writing LVGL boilerplate by hand. It models the display at its native
resolution (800 x 480 or 480 x 800), renders an approximation of the LVGL
default theme on screen, and emits an Arduino C++ sketch that compiles against
the standard Arduino Giga library stack.

It is a drafting tool. It does not run your code, simulate touch events, or
flash the board. It produces source you paste into the Arduino IDE.

## What it is not

Not a full IDE. Not a runtime emulator. Not a substitute for reading the LVGL
documentation when you start writing logic. The generated sketch is a scaffold
with stub event handlers; the work of giving those handlers meaning is yours.

---

## Widgets supported

| Widget    | LVGL primitive   | Notes                                    |
|-----------|------------------|------------------------------------------|
| Label     | lv_label         | Text, color, font size, alignment        |
| Button    | lv_btn           | Label child, color, radius               |
| Slider    | lv_slider        | Min, max, value, color                   |
| Switch    | lv_switch        | On/off, color                            |
| Checkbox  | lv_checkbox      | Label, checked state, color              |
| Bar       | lv_bar           | Min, max, value, color                   |
| Arc       | lv_arc           | Min, max, value, color                   |
| Dropdown  | lv_dropdown      | Options list, default selection          |
| TextArea  | lv_textarea      | Initial text, placeholder, one-line mode |
| Roller    | lv_roller        | Options list, default, normal/infinite   |

Fonts snap to the nearest `lv_font_montserrat_NN` size that ships with the
default LVGL Arduino build. If the size you pick is not enabled in your
`lv_conf.h`, either enable that font macro or step down to size 16, which is
always on.

---

## Quick start

1. Open `panelwright.html` in a browser.
2. Drag a widget tile from the left palette onto the dark canvas in the center.
3. Click the widget to select it; the right panel shows its properties.
4. Drag the body to move. Drag the corner handles to resize. Use arrow keys
   to nudge by one pixel; hold Shift for ten.
5. When the layout looks right, click **Generate Code** in the top toolbar.
6. Copy or download the resulting `.ino`, drop it into a new Arduino sketch
   folder, and flash.

A handful of placeholder widgets appear on first launch so the canvas is not
blank. Click **New** in the toolbar to clear them.

---

## Hardware target

Arduino Giga R1 WiFi with the Giga Display Shield attached. The shield provides
an 800 x 480 capacitive touch panel; PANELWRIGHT lets you toggle between
landscape and portrait orientation, which simply swaps which dimension is
passed to `Arduino_H7_Video`.

The starting URL for the shield, including pinout and library installation
instructions, is the official Arduino tutorial linked from the documentation
hub.

---

## Required Arduino libraries

Install these through the Arduino Library Manager:

```
Arduino_H7_Video
Arduino_GigaDisplayTouch
lvgl
ArduinoGraphics
```

`ArduinoGraphics` is pulled in transitively by some library versions, so if
the Library Manager flags it as already present, that is expected.

The generated sketch includes the three primary headers:

```cpp
#include "Arduino_H7_Video.h"
#include "Arduino_GigaDisplayTouch.h"
#include "lvgl.h"
```

---

## Anatomy of the generated sketch

The output is organized into clearly labeled sections so it is easy to extend.

```
Header banner
Display and touch detector instances
Widget handle declarations (one lv_obj_t * per widget, plus button labels)
Event handler stubs (one per interactive widget)
build_ui() function
setup()
loop()
```

Each widget gets a unique global `lv_obj_t *` pointer named after the widget
name from the Identity field in the properties panel. References to that
handle elsewhere in your sketch are guaranteed safe; you can read a slider,
change a label, or toggle a switch from anywhere using the same name you
gave it in the designer.

Interactive widgets (button, slider, switch, checkbox, dropdown, roller)
receive pre-wired event callbacks of the form `widgetname_event` registered
on the appropriate `LV_EVENT_*` code. The stubs print to Serial so the sketch
is testable the moment it boots; replace the print with your logic when ready.

A representative example for a button named `start_btn`:

```cpp
static lv_obj_t * start_btn;
static lv_obj_t * start_btn_label;

static void start_btn_event(lv_event_t * e) {
  Serial.println("start_btn pressed");
}

// inside build_ui()
start_btn = lv_btn_create(scr);
lv_obj_set_pos(start_btn, 40, 140);
lv_obj_set_size(start_btn, 140, 50);
lv_obj_set_style_bg_color(start_btn, lv_color_hex(0x2E6CDA), 0);
lv_obj_set_style_radius(start_btn, 5, 0);
start_btn_label = lv_label_create(start_btn);
lv_label_set_text(start_btn_label, "START");
lv_obj_set_style_text_color(start_btn_label, lv_color_hex(0xFFFFFF), 0);
lv_obj_center(start_btn_label);
lv_obj_add_event_cb(start_btn, start_btn_event, LV_EVENT_CLICKED, NULL);
```

---

## Project files

PANELWRIGHT can save and load its own layouts as JSON. Use **Save .json** in
the toolbar to download a snapshot, and **Load** to restore one. The format
is human-readable and lives in a single file with no external assets.

```json
{
  "version": 1,
  "tool": "PANELWRIGHT",
  "orientation": "landscape",
  "widgets": [
    { "id": "w_1", "type": "label", "name": "lbl_1",
      "x": 40, "y": 40, "w": 280, "h": 40,
      "props": { "text": "GIGA DISPLAY", "color": "#d4a574",
                 "fontSize": 28, "align": "left" } }
  ],
  "nextId": 2,
  "nameCounts": { "label": 1 }
}
```

Project files are forward-compatible within the v1 format.

---

## Keyboard reference

| Key                    | Action                          |
|------------------------|---------------------------------|
| Click                  | Select widget                   |
| Drag                   | Move widget                     |
| Corner / edge handles  | Resize                          |
| Delete or Backspace    | Delete selected widget          |
| Arrows                 | Nudge by 1 px                   |
| Shift + Arrows         | Nudge by 10 px                  |
| Ctrl + D               | Duplicate selected widget       |
| Ctrl + Z               | Undo                            |
| Ctrl + Shift + Z       | Redo                            |
| Ctrl + Y               | Redo                            |
| Escape                 | Deselect                        |

Snap-to-grid is on by default at 10 pixels. Toggle Grid and Snap independently
from the top toolbar.

---

## Design notes

The canvas renders an approximation of the LVGL default dark theme, not a
pixel-exact mirror. The intent is to give you a credible feel for layout,
spacing, and density at the device's true resolution. Colors are surfaced as
hex values and converted to `lv_color_hex(0xRRGGBB)` calls in the generated
sketch.

Geometry is stored as integers. The properties panel exposes X, Y, width, and
height as numeric inputs so layouts can be tuned to exact coordinates when a
mockup or print fixture requires it.

Widget z-order maps directly to the order of `lv_obj_t` creation in the
generated sketch, which controls draw order on the device. Raise, Lower,
To Front, and To Back in the properties panel reorder the widget array
accordingly.

---

## File structure

```
panelwright.html    The entire application. Drop it anywhere.
README.md           This file.
```

That is the whole project. There is no asset folder, no node_modules, no
package.json, no service worker. Everything PANELWRIGHT needs to run is
inside the HTML file.

---

## Known limitations

- No multi-screen workflow yet. All widgets are created on the active screen
  via `lv_scr_act()`. If you need multiple screens, generate the layout for
  each in turn and combine the resulting `build_ui` functions manually.
- No image widget. The Giga Display Shield supports `lv_img` with image
  descriptors compiled in, but image asset handling is outside the scope of
  a single-file tool.
- Theme is fixed to LVGL's default. Custom themes are applied at compile time
  in the sketch.
- Event handler bodies are stubs. PANELWRIGHT is a layout tool, not a logic
  editor.

---

## Why this exists

The LVGL API is fine. Calculating widget coordinates by squinting at a ruler
and recompiling to see whether a button landed where you wanted is not. This
instrument exists so the layout phase can happen in the medium where layout
actually belongs, which is in front of you, with your hands on it.

The generated sketch is meant to be readable. The point is not to hide LVGL
behind a wrapper; the point is to give you a head start on writing LVGL
yourself.

---

## License

GPL-3.0
