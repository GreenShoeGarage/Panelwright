# PANELWRIGHT

**Field Instrument 042**

A single-file, browser-based UI designer for the Arduino Giga Display Shield.
Drag LVGL widgets onto a true 1:1 canvas, lay them out across one or more
screens, pick a theme, and export a complete Arduino sketch ready to flash.

No build step. No CDN. No accounts. Open the HTML file in any modern browser
and start drafting.

---

## What it is

PANELWRIGHT is a workbench for laying out touch UIs on the Giga Display Shield
without writing LVGL boilerplate by hand. It models the display at its native
resolution (800 x 480 or 480 x 800), supports multi-screen projects with
proper `lv_scr_load` navigation, ships with a Tradigital theme preset, and
emits an Arduino C++ sketch that compiles against the standard Arduino Giga
library stack.

It is a drafting tool. It does not run your code, simulate touch events, or
flash the board. It produces source you paste into the Arduino IDE.

## What it is not

Not a full IDE. Not a runtime emulator. Not a substitute for reading the LVGL
documentation when you start writing logic. The generated sketch is a scaffold
with stub event handlers; the work of giving those handlers meaning is yours.

---

## Widgets supported

| Widget    | LVGL primitive   | Notes                                            |
|-----------|------------------|--------------------------------------------------|
| Label     | lv_label         | Text, color, font size, alignment                |
| Button    | lv_btn           | Label child, color, radius, navigate-to-screen, custom event code |
| Slider    | lv_slider        | Min, max, value, color, custom event code        |
| Switch    | lv_switch        | On/off, color, custom event code                 |
| Checkbox  | lv_checkbox      | Label, checked state, color, custom event code   |
| Bar       | lv_bar           | Min, max, value, color                           |
| Arc       | lv_arc           | Min, max, value, color                           |
| Meter     | lv_meter         | Dial gauge with arc indicator and needle         |
| Dropdown  | lv_dropdown      | Options list, default selection, custom event code |
| TextArea  | lv_textarea      | Initial text, placeholder, one-line mode         |
| Roller    | lv_roller        | Options list, default, normal/infinite, custom event code |
| Tabview   | lv_tabview       | Named tabs, top/bottom/left/right placement      |
| List      | lv_list          | Mixed headers (`# Group`) and button rows        |
| Chart     | lv_chart         | Line, bar, or scatter; up to three sample series |
| LED       | lv_led           | Color, brightness 0-255, on/off                  |
| Icon      | lv_label + LV_SYMBOL_* | Pick from 49 built-in LVGL symbols (wifi, settings, play, etc) |
| Keyboard  | lv_keyboard      | Mode (text lower/upper/special/number), TextArea binding |
| Calendar  | lv_calendar      | Year/month/day; today date highlighted           |

Fonts snap to the nearest `lv_font_montserrat_NN` size that ships with the
default LVGL Arduino build. If the size you pick is not enabled in your
`lv_conf.h`, either enable that font macro or step down to size 16, which is
always on.

For Tabview, the generated sketch creates the tabs and leaves their content
empty with TODO markers showing where to add child widgets. For Chart, the
generated sketch seeds each series with sample sine data at boot so the chart
displays something the moment it runs; replace that loop with real data from
your sensors.

---

## Quick start

1. Open `panelwright.html` in a browser.
2. The strip below the toolbar lists screens. The project starts with one
   screen called `main`. Click `+` to add another.
3. Drag a widget tile from the left palette onto the dark canvas in the center.
4. Click the widget to select it; the right panel shows its properties.
5. Drag the body to move. Drag the corner handles to resize. Use arrow keys
   to nudge by one pixel; hold Shift for ten.
6. To wire a button to switch screens, select it, set Action to "Go to screen",
   and pick the Target screen from the dropdown.
7. When the layout looks right, click **Generate Code** in the top toolbar.
8. In the code modal, set the **Sketch name** field if you want something
   other than the default `panelwright_ui`. Then either:
   - **Copy** the source and paste it into a new Arduino sketch, or
   - **Download .ino** to grab the raw source file, or
   - **Download .zip** to grab a ready-to-open Arduino sketch folder
     containing `sketchname/sketchname.ino`. Unzip it anywhere and double
     click the `.ino` to open in the Arduino IDE.

A handful of placeholder widgets appear on first launch so the canvas is not
blank. Click **New** in the toolbar to clear them.

---

## Multi-screen workflow

A PANELWRIGHT project can contain any number of screens, each with its own
set of widgets. The screens strip below the toolbar shows every screen as a
tab. Click a tab to switch the canvas to that screen.

Each screen has three controls in its tab:

- **Star** marks the default screen. The default screen is the one loaded by
  `lv_scr_load` in `setup()` when the board boots. Exactly one screen is the
  default at any time.
- **Pencil** opens a rename prompt. Names are sanitized to letters, digits,
  and underscores, since they are used as C identifiers in the generated
  sketch (`scr_main`, `scr_settings`, and so on).
- **X** deletes the screen. The last remaining screen cannot be deleted. Any
  button navigation actions pointing at a deleted screen are cleared.

In the generated sketch, each screen becomes its own `build_screen_NAME()`
function. `setup()` calls each one in turn, then calls `lv_scr_load` on the
default screen. To navigate at runtime, the generator wires button event
handlers to call `lv_scr_load(scr_OTHERNAME)` when their Action property is
set to "Go to screen".

```cpp
static lv_obj_t * scr_main;
static lv_obj_t * scr_settings;

static void go_settings_btn_event(lv_event_t * e) {
  lv_scr_load(scr_settings);
}

static void build_screen_main(void) {
  scr_main = lv_obj_create(NULL);
  // widgets...
}

void setup() {
  // ...
  build_screen_main();
  build_screen_settings();
  lv_scr_load(scr_main);
}
```

---

## Themes

The Theme selector in the top toolbar switches between two presets:

- **LVGL Default** is the stock blue-on-dark theme that ships with LVGL.
- **Tradigital** is amber primary (`#D4A574`), teal secondary (`#6FB5A8`),
  dark mode on. The canvas preview tints toward warm dark, default button
  colors lean amber, and the generated sketch calls
  `lv_theme_default_init(NULL, lv_color_hex(0xD4A574), lv_color_hex(0x6FB5A8),
  true, &lv_font_montserrat_16)` before any screen is built.

The choice is saved with the project file and applies to every screen at
runtime. Per-widget color overrides still win wherever you set them
explicitly in the properties panel.

Next to the Theme selector there is an **Apply** button. It walks every widget
on every screen and recolors them to match the current theme palette: buttons
go to the theme primary, slider and bar fills go to the theme primary, chart
series rotate through the theme's primary, secondary, and contrast colors,
meter arcs and needles get their warm pair. Labels, text areas, icons, and
LEDs are left alone because their colors are usually deliberate. The action
goes through the normal undo stack, so an accidental click is one Ctrl+Z away.

---

## Alignment guides

Drag a widget around and PANELWRIGHT shows thin amber guide lines whenever
one of its edges or centers comes within five pixels of another widget's
edge or center, or the canvas center, or a canvas edge. The dragged widget
snaps to the matching position, so building a row of evenly aligned buttons
is a matter of dragging them roughly into place and letting the guides do
the precision work. Guides clear the moment you release the pointer.

The 10 pixel grid snap is independent of alignment guides; both fire during
the same drag, with alignment taking precedence when both want to move the
widget to the same final position.

---

## Per-widget event code

Interactive widgets (button, slider, switch, checkbox, dropdown, roller) have
an **Event Code** field in their property panel. Whatever you type is inserted
directly into the generated event handler body. The field accepts raw C++.

When the field is empty, the generator falls back to a default body that
either prints to Serial or, for navigation buttons, calls `lv_scr_load`. When
the field is non-empty:

- For non-navigation buttons, your code replaces the Serial print stub.
- For navigation buttons, the `lv_scr_load` call still happens, and your code
  runs immediately after.
- For sliders, switches, checkboxes, dropdowns, and rollers, the generator
  still emits the line that reads the widget's current value into a local
  variable (`int32_t v`, `bool on`, or `uint16_t sel`). Your code can use that
  variable as written.

Use this for the small bits of logic that belong with a widget: forwarding a
slider value to a PWM pin, toggling an LED from a switch, sending a quick
serial command on a button press. For anything larger, leave the field empty
and write the logic in the body of your sketch instead.

---

## Hardware target

Arduino Giga R1 WiFi with the Giga Display Shield attached. The shield provides
an 800 x 480 capacitive touch panel; PANELWRIGHT lets you toggle between
landscape and portrait orientation, which simply swaps which dimension is
passed to `Arduino_H7_Video`.

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

The generated sketch targets LVGL v8 API, which is what the current Arduino
LVGL library ships. If you have upgraded to v9 locally, `lv_meter` has been
renamed to `lv_scale` and the meter widget's generated code will need a small
edit. The other widgets remain source-compatible.

---

## Anatomy of the generated sketch

The output is organized into clearly labeled sections so it is easy to extend.

```
Header banner with display size, screen count, theme
Display and touch detector instances
Screen handles (one lv_obj_t * per screen)
Widget handles (one lv_obj_t * per widget, plus button labels)
Forward declarations for build_screen_NAME() functions
apply_theme() function           (only when theme is Tradigital)
Event handlers (one per interactive widget; navigation buttons jump screens)
build_screen_NAME() functions    (one per screen)
setup() builds every screen, then loads the default
loop()  runs the LVGL timer
```

Each widget gets a unique global `lv_obj_t *` pointer named after the widget
name from the Identity field in the properties panel. References to that
handle elsewhere in your sketch are guaranteed safe; you can read a slider,
change a label, or toggle a switch from anywhere using the same name you gave
it in the designer.

Interactive widgets (button, slider, switch, checkbox, dropdown, roller)
receive pre-wired event callbacks of the form `widgetname_event` registered
on the appropriate `LV_EVENT_*` code. By default the stubs print to Serial
so the sketch is testable the moment it boots, and buttons with their Action
set to "Go to screen" call `lv_scr_load` instead of printing. If you fill in
the **Event Code** property on a widget, your code is dropped into the handler
body in place of the default stub. See the "Per-widget event code" section
above for the exact rules.

---

## Project files

PANELWRIGHT can save and load its own layouts as JSON. Use **Save .json** in
the toolbar to download a snapshot, and **Load** to restore one. The format
is human-readable and lives in a single file with no external assets.

```json
{
  "version": 2,
  "tool": "PANELWRIGHT",
  "orientation": "landscape",
  "theme": "tradigital",
  "screens": [
    {
      "id": "scr_1",
      "name": "main",
      "isDefault": true,
      "widgets": [
        { "id": "w_1", "type": "label", "name": "title_lbl",
          "x": 40, "y": 30, "w": 300, "h": 40,
          "props": { "text": "Control Panel", "color": "#d4a574",
                     "fontSize": 28, "align": "left" } }
      ]
    }
  ],
  "nextId": 2,
  "nextScreenId": 2,
  "nameCounts": { "label": 1 }
}
```

Old v1 project files (single `widgets` array, no screens) are migrated on
load. The widgets are wrapped into a single screen called `main` marked as
the default, and saving the file again writes it back in v2 format.

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

The canvas renders an approximation of the active theme, not a pixel-exact
mirror. The intent is to give you a credible feel for layout, spacing, and
density at the device's true resolution. Colors are surfaced as hex values
and converted to `lv_color_hex(0xRRGGBB)` calls in the generated sketch.

Geometry is stored as integers. The properties panel exposes X, Y, width, and
height as numeric inputs so layouts can be tuned to exact coordinates when a
mockup or print fixture requires it.

Widget z-order maps directly to the order of `lv_obj_t` creation in the
generated sketch, which controls draw order on the device. Raise, Lower,
To Front, and To Back in the properties panel reorder the widget array
accordingly.

For the meter widget, the angle controls describe the sweep on the canvas in
the conventional clockwise-from-east system (0 = east, 90 = south). The
generator translates these into the (sweep_angle, rotation) pair that
`lv_meter_set_scale_range` expects.

---

## PANELWRIGHT View

`panelwright-view.html` is a separate, standalone HTML file that loads a
saved project JSON and renders the screens as a clickable mockup. Buttons
with a goto action navigate between screens just like they will on the
device. Useful for walking through a UI design before flashing, or for
showing a project to someone who does not need the editor.

Open the file in a browser, click **Open .json** (or drag a JSON file onto
the page), and the project renders inside a virtual device shell at one to
one. The Reset button returns to the project's default screen.

The viewer ships with the same eighteen widget renderers as the editor and
honors the project's theme. It does not allow editing, generating code, or
saving; it is read-only.

### Editor and viewer linkage

When the editor and viewer are deployed together on the same origin, the
editor's **Open Viewer** button hands the current in-progress project to
the viewer via `sessionStorage` and opens it in a new tab. The viewer reads
the stash on startup, renders the project automatically, and clears the
stash so a manual refresh starts clean. The viewer's **\u2190 Editor**
link returns to the editor's parent path.

The default deployment layout assumes the editor sits at `/panelwright/`
(served as `index.html` from that folder) and the viewer at
`/panelwright/viewer/` (also `index.html`). The editor opens `viewer/`
relative to its own URL and the viewer links back to `../`. If you place
the files at different paths, edit the two link strings accordingly.

Each file remains fully usable on its own. If `sessionStorage` is
disabled or the viewer is opened directly, the empty state and **Open
.json** flow still work as expected.

---

## File structure

```
panelwright.html        The editor. Drop it anywhere.
panelwright-view.html   The companion viewer. Load a saved .json to walk it.
README.md               This file.
```

That is the whole project. There is no asset folder, no node_modules, no
package.json, no service worker. Everything PANELWRIGHT needs to run is
inside its two HTML files, and each one stands on its own.

---

## Known limitations

- Tabview content is not laid out inside the designer. The generator creates
  the tabs and leaves them as empty containers with TODO markers; widgets that
  belong inside a tab are added by hand in your copy of the sketch.
- No image widget. The Giga Display Shield supports `lv_img` with image
  descriptors compiled in, but image asset handling is outside the scope of
  a single-file tool.
- Chart data is sample sine data inserted at boot. Wire `lv_chart_set_next_value`
  to your real source in your sketch's loop or a timer callback.
- The Event Code property covers small per-widget logic. For anything more
  substantial, leave the field empty and write that logic in your sketch.

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

---

Field Instrument 042 // v0.2 // mbparks.com / greenshoegarage.com
