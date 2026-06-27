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

## What landed recently

Eight waves of editor work since the initial 18-widget release:

1. Project metadata, dirty indicator, keyboard nudging, snap-to-margin guides,
   copy/paste/duplicate, and a right-click context menu.
2. Multi-select with rubber-band box-select, group drag, group nudge, group
   align/distribute, and a right-click group menu.
3. Layer outline panel with glyph + name + type rows, drag-to-reorder z-order,
   and click-and-drag numeric scrubbing on every geometry and property number
   input (Shift for 10x, Alt for 0.1x).
4. Pre-generate validation pass that catches duplicate names, C++ reserved
   words, stale navigation targets, and orphan keyboard bindings; a
   board-profile abstraction (Giga + Display Shield, Custom); and a
   `arduino_secrets.h` split for WiFi credentials in zip exports.
5. Tab sub-canvases: tabview tabs are now editable nested workspaces.
   Double-click a tabview to enter a tab, lay widgets inside the tab content
   area, and the generator emits children with the correct LVGL parent.
6. Codegen refactored into section emitters; new `ui.h` / `ui.cpp` split for
   larger projects. The .ino keeps the hardware (display, touch,
   peripherals, setup/loop); ui.cpp holds the theme, event handlers, and
   per-screen builders; ui.h is the contract between them.
7. Docked live preview in the editor; Fill Data mode in the viewer. The
   editor gets a Preview button that opens a side panel mirroring the active
   screen in real time, with goto buttons clickable for nav testing. The
   viewer gets a Fill Data button that lists every value-bearing widget on
   the current screen with type-appropriate controls (range, text,
   checkbox, select) so you can walk through what the design will look
   like with realistic data.
8. SVG export. An Export SVG button in the toolbar opens a small modal
   for choosing scope (current screen vs. all screens packed into a zip)
   and whether to wrap each screen in the device-shell frame. The output
   uses HTML in `foreignObject` so every browser renders it pixel-perfect
   with the editor canvas. PNG rasterization is deferred to a future round
   that ships native SVG primitives.

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
| Image     | lv_image (v9) / lv_img (v8) | Upload PNG/JPEG/GIF/WebP; auto-resized + converted to RGB565 bytes embedded in flash, fit modes (cover, contain, stretch), opacity |

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

## Starter templates

The **Templates** button in the toolbar opens a gallery of pre-built starter projects. Each card shows a live SVG thumbnail rendered from the template's actual widget tree, so the preview is exactly what loads. Clicking a card replaces the current canvas with the template (with a confirmation prompt if you have unsaved work).

Templates ship as complete v3 project blobs, so loading one sets up not just the widgets but also the theme, peripherals, and project metadata. Peripherals required by a template are pre-enabled and called out as amber tags on the card.

**Current templates:**

| Template | Use case | Pre-enabled peripherals |
|----------|----------|-------------------------|
| Blank | Empty 800 x 480 canvas, starting fresh | none |
| Bench Multimeter | Big value readout with mode + range selectors and HOLD / REL / MAX-MIN buttons | none |
| Thermostat | Setpoint arc, current-temperature label, and HEAT / COOL / OFF / AUTO mode buttons | none |
| Weather Station | Four readouts (temperature, humidity, pressure, wind) in a 2 x 2 grid plus a 24-hour trend chart | WiFi |
| Sensor Recorder | Live data chart with sample rate, threshold sliders, record toggle, and export-to-SD button | SD card |
| Audio Spectrum | Eight frequency-band bars, peak meter, and gain slider | Microphone (PDM) |

After loading, every widget is editable, every peripheral toggle is yours to change, and **Generate Code** produces a working sketch. Templates are intended as starting points, not finished products; expect to customize them.

Adding your own: PANELWRIGHT defines templates as inline data in `panelwright.html`. Search for `const TEMPLATES = [` to find the list. Each entry has `id`, `name`, `description`, optional `tags`, and a `build()` function that returns a project blob. Use the `tplWidget()` helper to construct widgets with sensible default props. To distribute a template, export your project to `.json` from the toolbar and share that file; another user can load it through **Load**.

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

## Editing tab content

The Tabview widget is more than a static container: each of its tabs is its
own editable workspace.

Drop a Tabview from the palette and you see the tab bar drawn on the canvas
with placeholder content that says "(double-click to edit tab content)". To
enter a tab, either:

- double-click the tabview on the canvas, which opens the tab marked as the
  preview tab in the properties panel, or
- select the tabview and click one of the **Enter tab** buttons in the
  properties panel (one per tab), or
- use the tab switcher buttons in the scope breadcrumb once you are already
  inside one of the tabs.

While editing a tab:

- a breadcrumb appears above the canvas showing
  `[screen] > [tabview name] > [tab name]`, with click-to-pop crumbs and an
  **Exit tab** button on the right;
- the parent screen's widgets dim out and the host tabview gets an amber
  outline so you keep your context;
- a dashed amber frame marks the tab content area, and widget coordinates
  inside the tab are relative to that frame's top-left (LVGL parent
  semantics, so the generator can hand them straight to LVGL);
- drag, drop, nudge, multi-select, align, distribute, copy, paste, and
  numeric scrubbing all work exactly as they do at the screen root, just
  clamped to the tab content bounds.

Tab names are edited as one-per-line in the properties panel. Renaming
preserves the existing per-tab widget arrays; reducing the line count drops
the trailing tabs.

The generator emits one `lv_tabview_add_tab(...)` call per tab. Tabs with
children inside them recurse: the generator outputs
`lv_btn_create(<tv>_tab_0)` (or whichever widget type) with the tab handle as
the parent. Tabs without children stay as a `(void)<tv>_tab_N;` line that
suppresses the unused-variable warning.

Validation walks every tab on every screen. Widget names across screens and
tabs share a single namespace; a duplicate inside a tab will show up in the
validation modal just like one on a screen, and clicking the entry jumps
into the tab.

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

### Editor light mode

The toolbar has a small moon (☾) / sun (☀) button between the
Grid/Snap toggles and the Undo/Redo group. Click it to toggle the editor
itself (toolbar, palette, properties panel, modals) between dark mode
(default) and a warm cream/sepia light mode tuned for the tradigital paper
aesthetic. The choice persists in `localStorage` under
`panelwright_editor_theme`.

The editor theme is independent of the LVGL canvas theme. Two specific
implications worth knowing:

- You can preview a Tradigital (dark) screen on a light editor or an LVGL
  Default (light) screen on a dark editor; the canvas honors its own theme.
- The device shell (the black frame around the canvas) always stays dark
  because it represents the physical Giga board; only the surrounding
  chrome shifts.

Things that stay consistent across editor modes by design: the serial
monitor log (terminal feel), the device shell, and the LVGL canvas itself.

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

## Serial Monitor (Web Serial)

The **Serial** button in the toolbar opens a drawer at the bottom of the editor that connects to your Giga over USB using the browser's Web Serial API. It reads live UART output from the running sketch and lets you send commands back, closing the edit/flash/debug loop without leaving PANELWRIGHT.

**Connecting.** Flash the Giga with a sketch that includes `Serial.begin(115200)` (every generated PANELWRIGHT sketch does this automatically). Click **Serial** in the toolbar, click **Connect**, pick the Giga's port from the browser dialog. The status dot turns teal and starts pulsing once the port is open.

**Baud rates.** Default is 115200, matching the generated sketch. Other common rates (9600, 19200, 38400, 57600, 230400, 460800, 921600) are in the picker. Changing baud while connected is disabled; disconnect first.

**Log highlighting.** Incoming lines are color-coded by content:
- `[boot]` prefix lines render in cyan, so the staged peripheral init markers from the generated `setup()` stand out
- Lines containing `error`, `fail`, `panic`, `hardfault`, `fault`, or `assert` render in red
- Lines containing `warn`, `warning`, or `deprecated` render in amber
- `[debug]` prefix lines render in muted gray
- Sent commands appear as `> text` in amber-bright

This pairs directly with the staged boot diagnostics already generated by PANELWRIGHT: when a peripheral init faults, the last `[boot] init_X...` you see (without a matching `done`) is the culprit. The log shows it in cyan; the red error line that often follows on the next print is the root cause.

**Sending commands.** Type into the input at the bottom of the drawer and press **Enter** (or click **Send**). The line ending picker controls the byte sent after your text: LF (`\n`) for typical Arduino `Serial.readStringUntil('\n')` parsers, CRLF for some terminal programs, or None for raw byte streams. Sent text appears in the log with a `>` prefix so you can see exactly what went out.

**Auto-scroll.** On by default. Toggle off when you scroll up to inspect history; new lines stop yanking the view away. Toggle back on when you're done.

**Clear.** Empties the log without disconnecting.

**Browser support.** Web Serial is implemented in Chromium-family browsers (Chrome, Edge, Opera, Brave, Arc). Firefox and Safari do not expose it; the drawer still opens but shows a small note explaining why Connect is disabled. The API requires a user gesture for the initial `requestPort()`, so the connect flow always opens through the picker dialog. There's no automatic reconnect on page reload.

**Unplugging the device.** If you disconnect the USB cable while connected, the read loop notices the broken stream, closes the port, and updates the UI to "Disconnected" with a `--- Device disconnected ---` line. Reconnect by clicking Connect again and re-picking the port.

**Privacy.** The Web Serial picker only shows ports the user explicitly grants access to. The browser remembers approved ports per origin, but PANELWRIGHT doesn't itself persist any port reference, so each session starts with the picker dialog.

---

## Onboard hardware

The **Hardware** button in the toolbar opens a modal listing the onboard
peripherals available on the Giga R1 mainboard and the Giga Display Shield.
Each peripheral has a toggle and a small set of configuration controls.
Enabled peripherals appear as amber indicator dots in the status bar so a
glance at the editor shows what hardware your sketch will touch.

These are project-level, not per-screen. None of them render anything on the
canvas. They show up only in the generated sketch as a set of includes,
globals, init functions, and a call from `setup()`. The body of each init
function is intentionally readable: print on failure, print on success, ready
to extend.

| Peripheral  | Library / API           | Notes                                          |
|-------------|-------------------------|------------------------------------------------|
| Microphone  | `<PDM.h>`               | Sample rate, channels, ring buffer size        |
| IMU         | `Arduino_BMI270_BMM150` | Bosch BMI270 6-axis. Optional accel/gyro helper emit toggles |
| Camera      | `camera.h` + sensor lib | HM01B0, GC2145, OV7670, OV7675; resolution, format, frame rate |
| SD Card     | `SDMMCBlockDevice` + `FATFileSystem` | Mounts the Display Shield microSD at /sd |
| WiFi        | `<WiFi.h>`              | SSID and password stored as literals (move to arduino_secrets.h before publishing) |
| BLE         | `<ArduinoBLE.h>`        | Sets local name, calls advertise; you add services |
| RGB LED     | (pinMode / analogWrite) | Onboard active-low LED; boot color configurable |
| RTC         | `<mbed_rtc_time.h>`     | STM32H7 internal RTC, backed by VBAT coin cell |

Helpers are wired up for the ones that need them: `read` callback for the
microphone, `read_acceleration()` and `read_gyroscope()` for the IMU,
`capture_camera_frame()` for the camera, `set_rgb_led(r,g,b)` for the LED,
`rtc_now_string()` for the RTC. Each peripheral block in the generated sketch
begins with a comment describing how to consume it, including inline examples
for the patterns most people reach for first.

To call a peripheral from a widget, drop the relevant line into the widget's
**Event Code** field. For example, on a switch that toggles the onboard LED:

```cpp
set_rgb_led(on ? 0 : 32, on ? 0 : 32, on ? 96 : 0);
```

Or to drive an arc from the IMU pitch, in the loop body of your sketch:

```cpp
float ax, ay, az;
if (read_acceleration(ax, ay, az)) {
  int pitch_deg = (int)(atan2(ay, az) * 180.0f / PI);
  lv_arc_set_value(tilt_arc, pitch_deg + 90);
}
```

The Arduino_BMI270_BMM150 library is the standard IMU library for boards
that carry the Bosch BMI270. The Display Shield uses the BMI270 alone (no
magnetometer), so the BMM150 portion of the library is inert; accelerometer
and gyroscope work exactly as documented.

---

## Cookbook: wiring widgets to hardware

PANELWRIGHT generates the UI scaffolding. Making widgets actually do something usually comes down to a few lines of C++ wired in at the right place. There are three places to put code:

**Event Code field** (per widget). Open the widget's properties and use the `Event Code` textarea. The snippet is inserted into that widget's LVGL event handler and runs when the widget is interacted with (slider value changes, button clicked, switch toggled). Local variable `e` is the `lv_event_t *`. The widget itself is available by its name (e.g. `sld_1`).

**Setup code field** (project level). Open Project Details and use the `Setup code` textarea. Runs once at the end of `setup()` after the UI is built and any peripherals are initialised. Right place for `pinMode`, sensor library begins, serial protocols, etc.

**Loop code field** (project level). Same Project Details modal. Inserted at the end of `loop()` after `lv_timer_handler()`. Right place for continuous polling: read a sensor, update a widget value, watch a flag.

All three persist in the project JSON so they survive regeneration. You never have to hand-edit the generated .ino unless you want to.

**Boot ordering.** The generated `setup()` initialises the display first, then **builds and loads the UI**, then forces a paint with one `lv_timer_handler()` call, and only then runs the peripheral inits. This means the screen comes up as soon as the board boots: if a peripheral init crashes or hangs (camera with no sensor attached, WiFi with placeholder credentials, SD with a missing card), you'll see the UI already on the display rather than a dark screen and the red HardFault LED. The trade-off is that your `Setup code` (which runs after peripherals) cannot assume the UI hasn't been touched yet, but for everything in this cookbook that's fine.

Every recipe below references widgets by their default auto-generated name (`btn_1`, `sld_1`, `mtr_1`, etc.). Rename your widget in the properties panel and use that name instead. Recipes assume LVGL v9 (the default). If you're on v8, button creation is `lv_btn_create` and the meter uses `lv_meter_*` instead of the v9 scale+arc composite, but the rest is identical.

### Input widgets driving hardware

**Button toggles a GPIO pin.** Drop a Button widget. Open its properties and put this in `Event Code`:

```c
static bool relay_on = false;
relay_on = !relay_on;
digitalWrite(2, relay_on ? HIGH : LOW);
```

In Project Details `Setup code`, declare the pin: `pinMode(2, OUTPUT);`. Tapping the button now toggles GPIO 2. If your button has visual feedback you want, also update its label: `lv_label_set_text(btn_1_label, relay_on ? "ON" : "OFF");`.

**Switch enables a feature.** Drop a Switch widget named `sw_pump`. In its `Event Code`:

```c
bool on = lv_obj_has_state(sw_pump, LV_STATE_CHECKED);
digitalWrite(3, on ? HIGH : LOW);
Serial.print("pump: "); Serial.println(on);
```

Setup code: `pinMode(3, OUTPUT);`. The switch's checked state drives the pin directly.

**Slider sets PWM duty cycle.** Drop a Slider widget named `sld_speed`, range 0 to 255. In its `Event Code`:

```c
int val = lv_slider_get_value(sld_speed);
analogWrite(5, val);
```

Dragging the slider sends PWM out on pin 5 (motor driver, LED dimmer, fan controller).

**Dropdown selects a mode.** Drop a Dropdown with options `Auto\nManual\nOff`, named `dd_mode`. In its `Event Code`:

```c
uint16_t sel = lv_dropdown_get_selected(dd_mode);
switch (sel) {
  case 0: Serial.println("AUTO");   break;
  case 1: Serial.println("MANUAL"); break;
  case 2: Serial.println("OFF");    digitalWrite(2, LOW); break;
}
```

**Roller picks a numeric value.** Drop a Roller with options `1\n2\n5\n10\n20\n50`, named `rl_gain`. In its `Event Code`:

```c
uint16_t idx = lv_roller_get_selected(rl_gain);
char opt[16];
lv_roller_get_selected_str(rl_gain, opt, sizeof(opt));
int gain = atoi(opt);
// use gain for your DSP, amplifier, etc.
```

**Keyboard captures into a textarea.** Set a Keyboard widget's TextArea property to point to your textarea. Touch input flows automatically. To grab what was typed:

```c
const char * text = lv_textarea_get_text(ta_1);
```

Read this from your save button's Event Code or anywhere appropriate.

### Sensors driving widgets

These go in `Loop code`. Update widgets every iteration. Throttle if needed.

**A0 → Meter (gauge).** Drop a Meter named `mtr_temp`, range 0 to 100. `Loop code`:

```c
static uint32_t last = 0;
if (millis() - last > 50) {  // 20 Hz update
  last = millis();
  int raw = analogRead(A0);
  int pct = map(raw, 0, 1023, 0, 100);
  lv_arc_set_value(mtr_temp_indic, pct);
}
```

The `mtr_temp_indic` handle is exposed at file scope as the inner arc. The throttling matters because LVGL repaints on every value change.

**A0 → Bar (linear progress).** Drop a Bar named `bar_lvl`, range 0 to 1023. `Loop code`:

```c
static uint32_t last = 0;
if (millis() - last > 50) {
  last = millis();
  lv_bar_set_value(bar_lvl, analogRead(A0), LV_ANIM_OFF);
}
```

**A0 → Label (text readout).** Drop a Label named `lbl_temp`. `Loop code`:

```c
static uint32_t last = 0;
if (millis() - last > 250) {  // 4 Hz, slow enough to read
  last = millis();
  float volts = analogRead(A0) * (3.3f / 1023.0f);
  char buf[16];
  snprintf(buf, sizeof(buf), "%.2f V", volts);
  lv_label_set_text(lbl_temp, buf);
}
```

**A0 → Chart (time-series).** Drop a Chart named `cht_log` with type Line, 60 points. The series handle `cht_log_ser1` is exposed at file scope. `Loop code`:

```c
static uint32_t last = 0;
if (millis() - last > 200) {  // 5 Hz
  last = millis();
  lv_chart_set_next_value(cht_log, cht_log_ser1, analogRead(A0));
}
```

Five samples per second on a 60-point chart means a 12-second window. For multi-series charts the additional handles are `cht_log_ser2` and `cht_log_ser3`.

**Digital pin → LED widget.** Drop an LED named `led_door`. `Loop code`:

```c
if (digitalRead(7) == HIGH) lv_led_on(led_door);
else                        lv_led_off(led_door);
```

Setup: `pinMode(7, INPUT_PULLUP);` for a door-switch input.

### Hardware devices

Eight onboard peripherals can be enabled from the Onboard Hardware modal. Each one generates an init function and a set of helper globals you can use from `Setup code`, `Loop code`, or any widget's `Event Code`. The exact symbols are listed at the top of each subsection so you know what's available without reading the generated .ino.

#### Microphone (PDM digital mic)

**Available after enabling:**
- `int16_t mic_buffer[N]`, circular PCM buffer, N is the sample count you picked
- `volatile int mic_samples_ready`, number of fresh samples since you last cleared this flag; the PDM callback `on_mic_data()` sets it automatically
- `MIC_CHANNELS`, `MIC_SAMPLE_RATE`, constants matching your settings

**Peak meter on a Bar.** Bar `bar_mic` range 0 to 32767. `Loop code`:

```c
if (mic_samples_ready) {
  int16_t peak = 0;
  for (int i = 0; i < mic_samples_ready; i++) {
    int16_t v = abs(mic_buffer[i]);
    if (v > peak) peak = v;
  }
  mic_samples_ready = 0;
  lv_bar_set_value(bar_mic, peak, LV_ANIM_OFF);
}
```

The `mic_samples_ready` flag is the gate, so this only runs when fresh data arrives, no `millis()` throttle needed.

**RMS level on an Arc (smoother VU meter).** Arc `arc_vu` range 0 to 100. `Loop code`:

```c
if (mic_samples_ready) {
  uint64_t sumSq = 0;
  for (int i = 0; i < mic_samples_ready; i++) {
    sumSq += (int32_t)mic_buffer[i] * mic_buffer[i];
  }
  float rms = sqrtf((float)sumSq / mic_samples_ready);
  mic_samples_ready = 0;
  int pct = (int)(rms / 32767.0f * 100.0f);
  lv_arc_set_value(arc_vu, constrain(pct, 0, 100));
}
```

**Trigger something on a loud sound.** Button-style threshold detection in `Loop code`:

```c
if (mic_samples_ready) {
  int16_t peak = 0;
  for (int i = 0; i < mic_samples_ready; i++) {
    if (abs(mic_buffer[i]) > peak) peak = abs(mic_buffer[i]);
  }
  mic_samples_ready = 0;
  static uint32_t lastTrigger = 0;
  if (peak > 8000 && millis() - lastTrigger > 500) {
    lastTrigger = millis();
    lv_led_on(led_alert);
    // turn it off again after 200 ms via your own timer
  }
}
```

#### IMU (BMI270 6-axis)

**Available after enabling:**
- `IMU`, the library global (provides `accelerationAvailable()`, `gyroscopeAvailable()`, etc.)
- `read_acceleration(ax, ay, az)`, helper, returns `bool`, fills g-units (-4 to +4 by default)
- `read_gyroscope(gx, gy, gz)`, helper, returns `bool`, fills deg/sec

**Bubble level: tilt on X axis drives an arc.** Arc `arc_tilt`, range 0 to 180. `Loop code`:

```c
float ax, ay, az;
if (read_acceleration(ax, ay, az)) {
  // ax in g's (-1 to +1 in normal use). Map to 0..180 for arc position.
  int v = (int)((ax + 1.0f) * 90.0f);
  lv_arc_set_value(arc_tilt, constrain(v, 0, 180));
}
```

**Display pitch / roll on labels.** Labels `lbl_pitch` and `lbl_roll`. `Loop code`:

```c
static uint32_t last = 0;
float ax, ay, az;
if (millis() - last > 100 && read_acceleration(ax, ay, az)) {
  last = millis();
  int pitch = (int)(atan2f(ay, az) * 180.0f / PI);
  int roll  = (int)(atan2f(ax, az) * 180.0f / PI);
  char buf[16];
  snprintf(buf, sizeof(buf), "%+3d deg", pitch); lv_label_set_text(lbl_pitch, buf);
  snprintf(buf, sizeof(buf), "%+3d deg", roll);  lv_label_set_text(lbl_roll, buf);
}
```

**Shake to advance a roller.** Roller `rl_mode`. `Loop code`:

```c
float ax, ay, az;
if (read_acceleration(ax, ay, az)) {
  float mag = sqrtf(ax*ax + ay*ay + az*az);
  // A normal still reading is ~1.0g. A shake exceeds 1.8g briefly.
  static uint32_t lastShake = 0;
  if (mag > 1.8f && millis() - lastShake > 600) {
    lastShake = millis();
    uint16_t cur = lv_roller_get_selected(rl_mode);
    uint16_t total = lv_roller_get_option_cnt(rl_mode);
    lv_roller_set_selected(rl_mode, (cur + 1) % total, LV_ANIM_ON);
  }
}
```

**Rotation rate on a chart.** Chart `cht_gyro` with one series, range -250 to 250. `Loop code`:

```c
static uint32_t last = 0;
float gx, gy, gz;
if (millis() - last > 50 && read_gyroscope(gx, gy, gz)) {
  last = millis();
  lv_chart_set_next_value(cht_gyro, cht_gyro_ser1, (lv_coord_t)gz);
}
```

#### Camera

**Available after enabling:**
- `Camera cam`, the camera object
- `FrameBuffer cam_fb`, destination buffer
- `capture_camera_frame(timeout_ms)`, helper, returns 0 on success

**Capture on button press, show status.** Button's `Event Code`:

```c
if (capture_camera_frame(3000) == 0) {
  lv_label_set_text(lbl_status, "Frame captured");
  // cam_fb.getBuffer() is the raw pixel data, length getBufferSize()
} else {
  lv_label_set_text(lbl_status, "Capture failed");
}
```

**Capture and save to SD as raw bytes.** Enable both Camera and SD card. Button `Event Code`:

```c
if (capture_camera_frame(3000) != 0) {
  lv_label_set_text(lbl_status, "Capture failed");
  return;
}
char path[32];
snprintf(path, sizeof(path), "/sd/cap_%lu.raw", millis());
FILE * f = fopen(path, "wb");
if (f) {
  fwrite(cam_fb.getBuffer(), 1, cam_fb.getBufferSize(), f);
  fclose(f);
  lv_label_set_text(lbl_status, "Saved");
}
```

For PNG or JPEG, you would need an encoder library on top of the raw buffer; that's outside PANELWRIGHT's scope.

#### SD card

**Available after enabling:**
- `sd_block_device`, the SDMMC block device
- `sd_fs`, `mbed::FATFileSystem` mounted at `/sd`
- All access is through C stdio (`fopen`, `fprintf`, `fread`, `fclose`) with paths under `/sd/...`

**Append to a CSV log on a button press.** Button `Event Code`:

```c
FILE * f = fopen("/sd/log.csv", "a");
if (f) {
  fprintf(f, "%lu,%d,%d\n",
          millis(),
          analogRead(A0),
          lv_obj_has_state(sw_pump, LV_STATE_CHECKED) ? 1 : 0);
  fclose(f);
  lv_label_set_text(lbl_status, "Logged");
}
```

**Read a single config value at boot.** Project `Setup code`:

```c
FILE * f = fopen("/sd/config.txt", "r");
if (f) {
  int threshold = 0;
  if (fscanf(f, "threshold=%d", &threshold) == 1) {
    lv_slider_set_value(sld_thresh, threshold, LV_ANIM_OFF);
  }
  fclose(f);
}
```

**Continuous logging while a switch is on.** `Loop code`:

```c
static uint32_t last = 0;
if (lv_obj_has_state(sw_log, LV_STATE_CHECKED) && millis() - last > 1000) {
  last = millis();
  FILE * f = fopen("/sd/stream.csv", "a");
  if (f) {
    fprintf(f, "%lu,%d\n", millis(), analogRead(A0));
    fclose(f);
  }
}
```

`fclose` on every write is slow but safe. For high-rate logging, batch writes with `fwrite` into a buffer and flush periodically.

#### WiFi

**Available after enabling:**
- `WIFI_SSID`, `WIFI_PASSWORD`, credential constants (from arduino_secrets.h or inlined)
- `init_wifi()` runs in setup() and blocks for up to 20 seconds attempting to connect
- Full Arduino WiFi API: `WiFi.RSSI()`, `WiFi.localIP()`, `WiFi.status()`

**Show IP address at boot.** Label `lbl_ip`. Project `Setup code` (runs after init_wifi):

```c
char buf[24];
IPAddress ip = WiFi.localIP();
snprintf(buf, sizeof(buf), "%d.%d.%d.%d", ip[0], ip[1], ip[2], ip[3]);
lv_label_set_text(lbl_ip, buf);
```

**Live RSSI readout.** Label `lbl_rssi`. `Loop code`:

```c
static uint32_t last = 0;
if (millis() - last > 2000) {
  last = millis();
  char buf[32];
  if (WiFi.status() == WL_CONNECTED) {
    snprintf(buf, sizeof(buf), "WiFi %d dBm", WiFi.RSSI());
  } else {
    snprintf(buf, sizeof(buf), "WiFi: offline");
  }
  lv_label_set_text(lbl_rssi, buf);
}
```

**HTTP GET on button press.** Button `Event Code`:

```c
#include <WiFiClient.h>  // put this at the top of the .ino in setup code? No,
                          // includes must be hand-edited at the top of the file
WiFiClient client;
if (client.connect("example.com", 80)) {
  client.println("GET / HTTP/1.0");
  client.println("Host: example.com");
  client.println();
  // Read response, stuff first line into a label, etc.
  lv_label_set_text(lbl_status, "Sent");
  client.stop();
} else {
  lv_label_set_text(lbl_status, "Conn failed");
}
```

For includes you genuinely have to edit the top of the generated .ino. The `#include` directives sit above the PANELWRIGHT-managed region; they don't get clobbered when you regenerate.

#### BLE

**Available after enabling:**
- `BLE_LOCAL_NAME`, the advertised name constant
- `init_ble()` runs in setup() and starts advertising
- Full ArduinoBLE API: define your `BLEService` and `BLECharacteristic` objects manually

The peripheral mode generated by PANELWRIGHT advertises a name but doesn't define any services. You wire those up yourself.

**Switch state synced over BLE.** Add at the top of your .ino (above the PANELWRIGHT block) and the `Setup code` field, paired:

Top of .ino (hand-edited; persists across regeneration since it's outside the managed region):
```c
BLEService    sw_svc("19B10000-E8F2-537E-4F6C-D104768A1214");
BLEByteCharacteristic sw_char("19B10001-E8F2-537E-4F6C-D104768A1214", BLERead | BLEWrite | BLENotify);
```

Project `Setup code`:
```c
sw_svc.addCharacteristic(sw_char);
BLE.addService(sw_svc);
sw_char.writeValue((uint8_t)0);
sw_char.setEventHandler(BLEWritten, [](BLEDevice c, BLECharacteristic ch) {
  uint8_t v;
  ch.readValue(v);
  if (v) lv_obj_add_state(sw_remote, LV_STATE_CHECKED);
  else   lv_obj_remove_state(sw_remote, LV_STATE_CHECKED);
});
```

The switch widget's own `Event Code` should echo state back over BLE so phone apps see toggles from the touch screen:

```c
bool on = lv_obj_has_state(sw_remote, LV_STATE_CHECKED);
sw_char.writeValue((uint8_t)(on ? 1 : 0));
```

**Notify a sensor reading.** Notifying-only characteristic. After defining `temp_char` similarly, in `Loop code`:

```c
static uint32_t last = 0;
if (BLE.connected() && millis() - last > 500) {
  last = millis();
  int16_t reading = analogRead(A0);
  temp_char.writeValue(reading);  // BLENotify pushes to subscribers
}
BLE.poll();  // process incoming BLE events
```

The `BLE.poll()` call should run every loop iteration when BLE is enabled, otherwise incoming writes from the phone won't be handled.

#### RGB LED (onboard, active-low)

**Available after enabling:**
- `set_rgb_led(r, g, b)`, drives all three channels at once. **Each channel is binary** (on for any value >= 1, off for 0). This is intentional, see note below.
- `RGB_DEFAULT_R`, `RGB_DEFAULT_G`, `RGB_DEFAULT_B`, constants for the boot color you picked
- `LEDR`, `LEDG`, `LEDB`, pin macros (active-low)

**Why binary and not PWM:** The Giga's RGB LED pins share PWM timer hardware with the Arduino_H7_Video peripheral. Calling `analogWrite()` on `LEDR`/`LEDG`/`LEDB` after `Display.begin()` corrupts the timer state and triggers a HardFault (red LED flashing, board unresponsive). `digitalWrite()` doesn't touch any timer, so it's safe alongside the display. The trade-off is 8 colors (000, 001, 010, 011, 100, 101, 110, 111) instead of 16.7 million. For most status-indicator uses this is fine.

**Status indicator: green when WiFi connected, red otherwise.** `Loop code`:

```c
static uint32_t last = 0;
static bool wasOnline = false;
if (millis() - last > 500) {
  last = millis();
  bool online = (WiFi.status() == WL_CONNECTED);
  if (online != wasOnline) {
    wasOnline = online;
    set_rgb_led(online ? 0 : 1, online ? 1 : 0, 0);
  }
}
```

**Switch toggles white indicator.** Switch widget `Event Code`:

```c
bool on = lv_obj_has_state(sw_busy, LV_STATE_CHECKED);
set_rgb_led(on ? 1 : 0, on ? 1 : 0, on ? 1 : 0);
```

**Color picker from three switches.** Switches `sw_r`, `sw_g`, `sw_b`. Put this in each switch's `Event Code`:

```c
set_rgb_led(lv_obj_has_state(sw_r, LV_STATE_CHECKED) ? 1 : 0,
            lv_obj_has_state(sw_g, LV_STATE_CHECKED) ? 1 : 0,
            lv_obj_has_state(sw_b, LV_STATE_CHECKED) ? 1 : 0);
```

Eight colors total. Use a dropdown widget with named options if you want a friendlier UX than three switches.

**Pulse on button press.** Button `Event Code`:

```c
set_rgb_led(0, 0, 1);  // blue on
lv_timer_t * t = lv_timer_create([](lv_timer_t * t) {
  set_rgb_led(0, 0, 0);
  lv_timer_del(t);
}, 150, NULL);
```

**If you really need brightness/PWM** on the RGB LED, you need to either (a) drive an external RGB LED on a different PWM-capable pin that doesn't conflict with the H7 video timers, or (b) write a manual software-PWM loop using a fast `lv_timer` that toggles the digital pins at a duty cycle. The official Arduino guidance for the Giga + Display Shield is to leave the onboard RGB LED in digital mode while the display is active.

#### RTC (STM32H7 internal)

**Available after enabling:**
- `rtc_now_string()`, returns a `String` formatted `"YYYY-MM-DD HH:MM:SS"`
- Raw access via C `time()`, `localtime()`, `strftime()`, `set_time()`
- The clock is battery-backed only if your board has the VBAT coin cell populated

**Live clock display.** Label `lbl_clock`. `Loop code`:

```c
static uint32_t last = 0;
if (millis() - last > 1000) {
  last = millis();
  lv_label_set_text(lbl_clock, rtc_now_string().c_str());
}
```

**Set the clock from a button.** Button "SET TIME NOW" `Event Code`:

```c
// Replace with the current epoch when flashing. Or use WiFi/NTP to fetch.
set_time(1750000000);  // 2025-06-15 something
lv_label_set_text(lbl_status, "Clock set");
```

**Timestamp every log entry.** Combine SD card + RTC. Button `Event Code`:

```c
FILE * f = fopen("/sd/log.csv", "a");
if (f) {
  fprintf(f, "%s,%d\n", rtc_now_string().c_str(), analogRead(A0));
  fclose(f);
}
```

**Schedule something at a specific time of day.** `Loop code`:

```c
static int lastFiredHour = -1;
time_t now = time(NULL);
struct tm * t = localtime(&now);
if (t->tm_hour == 6 && t->tm_min == 0 && lastFiredHour != t->tm_hour) {
  lastFiredHour = t->tm_hour;
  // Once per day at 06:00:
  set_rgb_led(255, 200, 100);  // sunrise color
  digitalWrite(2, HIGH);        // turn on the coffee maker
}
if (t->tm_hour != 6) lastFiredHour = -1;  // arm again for tomorrow
```

### Image widget

The Image widget embeds a PNG, JPEG, GIF, or WebP into your sketch as a static RGB565 byte array in flash. There is no SD card or filesystem involvement at runtime; the pixel data is part of the compiled binary and lives at a known address.

**Adding an image.** Drop the Image widget on the canvas, then open the properties panel and click the file picker. PANELWRIGHT decodes the file in the browser, downsamples it to fit within the widget box (capped at 240 x 180 pixels to keep flash usage reasonable), converts to RGB565 little-endian, and caches the bytes inside the project JSON. Both the original preview and the byte buffer travel with the project.

**What the codegen produces.** For each image widget with a cached upload, the sketch includes a file-scope byte array and matching descriptor:

```c
// img_logo: 64 x 48 (6144 bytes)
static const uint8_t img_logo_data[] = {
  0x00, 0xF8, 0x00, 0xF8, ...
};
static const lv_image_dsc_t img_logo_dsc = {
  .header.magic = LV_IMAGE_HEADER_MAGIC,
  .header.cf = LV_COLOR_FORMAT_RGB565,
  .header.w = 64,
  .header.h = 48,
  .header.stride = 128,
  .data_size = sizeof(img_logo_data),
  .data = img_logo_data,
};
```

The widget builder then attaches it:

```c
img_logo = lv_image_create(scr_main);
lv_obj_set_pos(img_logo, 100, 50);
lv_obj_set_size(img_logo, 64, 48);
lv_image_set_src(img_logo, &img_logo_dsc);
```

On LVGL v8 the type is `lv_img_dsc_t`, the create call is `lv_img_create`, and the source-setter is `lv_img_set_src`. Color format is `LV_IMG_CF_TRUE_COLOR`. PANELWRIGHT switches between v8 and v9 based on your `Meta` panel selection. Either way `LV_USE_IMAGE` (v9) or `LV_USE_IMG` (v8) is enabled in the generated `lv_conf.h` automatically.

**Fit modes.** `cover` fills the widget box, cropping if aspect ratios differ. `contain` fits inside the box and letterboxes the rest. `stretch` distorts the image to fill exactly. The chosen mode applies to the editor preview; on hardware the image is drawn at its uploaded pixel dimensions and aligned by LVGL's default placement (top-left of the widget box). Re-upload at the desired final pixel size if you need precise on-device geometry.

**Opacity.** A value of 0 to 100. When less than 100, the generated code emits `lv_obj_set_style_img_opa(img_X, lvOp, 0)` with the mapped 0 to 255 LVGL alpha. Useful for watermarks or background panels.

**Flash cost.** Each image consumes `width * height * 2` bytes of flash. A 240 x 180 image is 86,400 bytes; a 64 x 64 icon is 8,192 bytes. The status bar shows current image flash usage; the properties panel shows per-image size. The Giga has 2 MB of flash, of which roughly 600 KB is available after Arduino core, LVGL, and your peripheral libraries link in.

**Updating an image at runtime.** The descriptor and its bytes are `const`, so they live in flash and cannot be modified. To swap images at runtime, define multiple Image widgets (each with its own descriptor) and toggle visibility with `lv_obj_add_flag(handle, LV_OBJ_FLAG_HIDDEN)` / `lv_obj_clear_flag`. Or, for true dynamic imagery, hand-author a second `lv_image_dsc_t` in SRAM, populate the pixel buffer in your loop, and call `lv_image_set_src` to switch.

### Cross-widget patterns

**Slider drives a meter live.** Slider named `sld_x` (0 to 100), Meter named `mtr_x` (0 to 100). In the slider's `Event Code`:

```c
lv_arc_set_value(mtr_x_indic, lv_slider_get_value(sld_x));
```

That's it. The meter follows the slider without any loop code.

**Button advances a roller.** Useful when the roller is a step-through wizard rather than a free-pick. Button's `Event Code`:

```c
uint16_t cur = lv_roller_get_selected(rl_wizard);
uint16_t total = lv_roller_get_option_cnt(rl_wizard);
lv_roller_set_selected(rl_wizard, (cur + 1) % total, LV_ANIM_ON);
```

**Threshold logic: slider sets a level, LED reflects whether A0 exceeds it.** Slider `sld_thresh` (0 to 1023), LED `led_alarm`. `Loop code`:

```c
static uint32_t last = 0;
if (millis() - last > 50) {
  last = millis();
  int reading = analogRead(A0);
  int threshold = lv_slider_get_value(sld_thresh);
  if (reading > threshold) lv_led_on(led_alarm);
  else                     lv_led_off(led_alarm);
}
```

**Switch enables continuous logging to SD.** Switch `sw_log`, button `btn_savepoint`. `Loop code`:

```c
static uint32_t last = 0;
if (lv_obj_has_state(sw_log, LV_STATE_CHECKED) && millis() - last > 1000) {
  last = millis();
  File f = SD.open("/stream.csv", FILE_WRITE);
  if (f) {
    f.print(millis()); f.print(",");
    f.println(analogRead(A0));
    f.close();
  }
}
```

### Notes and gotchas

LVGL widget value setters do not trigger event callbacks. Setting `lv_slider_set_value(sld, 50)` from your code will not fire the slider's `Event Code`, only user touch will. Use Event Code for *reactions to user input*, and loop/setup code (or direct calls in event handlers of other widgets) for *programmatic updates*.

The widget handle name in C++ is exactly the `Name` field in the properties panel. The editor sanitises invalid characters; if you rename a widget after writing custom code, update your code to match.

LVGL event handlers run on the LVGL thread. Heavy work (SD writes, network calls) will pause the UI. For anything taking more than a few milliseconds, set a flag in the event handler and do the work from `loop()` instead.

For sensor reads in `loop()`, always throttle with a `millis()` check. Without throttling you'll be calling `lv_arc_set_value` thousands of times per second, which churns LVGL's invalidate-and-redraw logic and can make the UI feel sluggish on the Giga.

The meter widget exposes two extra file-scope handles in v9 mode: `<name>_scale` and `<name>_indic`. The indic is what you update with `lv_arc_set_value` to change the displayed value. The scale is the tick-marks layer underneath. You almost always want the indic.

### Inner handles cheat sheet

Most widgets expose just one file-scope C++ handle: the widget's `Name` from the properties panel. A few composite widgets expose additional handles for their inner parts so you can drive them from custom code.

| Widget | Main handle | Inner handles | Notes |
|--------|-------------|---------------|-------|
| label | `lbl_1` | none | `lv_label_set_text(lbl_1, "x")` |
| button | `btn_1` | `btn_1_label` | The button's text is in the child label. Use `lv_label_set_text(btn_1_label, "ok")` to change it. |
| slider | `sld_1` | none | Value via `lv_slider_get_value` / `lv_slider_set_value`. |
| switch | `sw_1` | none | State via `lv_obj_has_state(sw_1, LV_STATE_CHECKED)`. |
| checkbox | `cb_1` | none | Same state pattern as switch. |
| bar | `bar_1` | none | `lv_bar_set_value(bar_1, v, LV_ANIM_OFF)`. |
| arc | `arc_1` | none | `lv_arc_set_value(arc_1, v)`. |
| meter (v9) | `mtr_1` | `mtr_1_scale`, `mtr_1_indic` | Container + tick scale + value arc. Update the value with `lv_arc_set_value(mtr_1_indic, v)`. The scale is for restyling tick marks. |
| meter (v8) | `mtr_1` | none exposed | v8 uses `lv_meter_scale_t *` and `lv_meter_indicator_t *` which are different types. Tracked inside the builder; reach them via `lv_meter_add_*` if you regenerate scales at runtime. |
| dropdown | `dd_1` | none | Selection via `lv_dropdown_get_selected`. |
| textarea | `ta_1` | none | Text via `lv_textarea_get_text` / `lv_textarea_set_text`. |
| roller | `rl_1` | none | Selection via `lv_roller_get_selected` / `lv_roller_get_selected_str`. |
| tabview | `tv_1` | `tv_1_tab_0`, `tv_1_tab_1`, ... | One handle per tab. Add widgets to a tab at runtime with `lv_obj_t * new_lbl = lv_label_create(tv_1_tab_2);`. Switch tabs programmatically with `lv_tabview_set_active(tv_1, idx, LV_ANIM_ON)`. |
| list | `lst_1` | none | Items don't get individual handles. Access them at runtime with `lv_obj_get_child(lst_1, idx)`, or add new ones with `lv_list_add_btn(lst_1, LV_SYMBOL_FILE, "New")`. |
| chart | `cht_1` | `cht_1_ser1`, `cht_1_ser2`, `cht_1_ser3` | One handle per series (up to 3). Push new values with `lv_chart_set_next_value(cht_1, cht_1_ser1, v)`. These are `lv_chart_series_t *`, not `lv_obj_t *`. |
| led | `led_1` | none | `lv_led_on(led_1)` / `lv_led_off(led_1)` / `lv_led_set_color(led_1, lv_color_hex(0xff0000))`. |
| icon | `icn_1` | none | Symbol is a label under the hood. Change with `lv_label_set_text(icn_1, LV_SYMBOL_WIFI)`. |
| keyboard | `kbd_1` | none | Bound to a textarea via `lv_keyboard_set_textarea(kbd_1, ta_1)` (done automatically if you set the target in properties). |
| calendar | `cal_1` | none | Get the highlighted date with `lv_calendar_get_pressed_date(cal_1, &date)`. |
| image | `img_1` | `img_1_dsc` (file-scope), `img_1_data` (byte array) | The descriptor is the canonical reference (`lv_image_set_src(img_1, &img_1_dsc)`). To swap images at runtime, define multiple Image widgets and toggle `LV_OBJ_FLAG_HIDDEN`. |

When in doubt, look at the generated `// ---- Widget handles ----` section near the top of the .ino. Everything declared there is fair game from anywhere in the file.

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

## Installing the project's lv_conf.h

Every Montserrat font size that LVGL renders is gated by a compile-time
define in `lv_conf.h`. The default `lv_conf.h` enables a small handful of
sizes; the rest get stripped from the binary so they don't waste flash.
That means if your PANELWRIGHT project uses font size 32 but your installed
`lv_conf.h` has `LV_FONT_MONTSERRAT_32` set to 0, the sketch fails to
compile with `'lv_font_montserrat_32' was not declared in this scope`.

PANELWRIGHT solves this by emitting a tuned `lv_conf.h` in the project
bundle that enables exactly the sizes your screens use and leaves the rest
disabled. Each enabled size costs roughly 3 KB of flash, so the file is
sized to the project. The status bar's **Memory** readout shows the
running total.

### Steps

1. **Generate the bundle.** Click **Generate Code** in the toolbar, then in
   the modal click **Download .zip**. The bundle contains
   `your_sketch/your_sketch.ino` and a tuned `lv_conf.h`.

2. **Locate your sketchbook.** Arduino IDE: Settings → "Sketchbook
   location". The default is `~/Documents/Arduino` on macOS and Linux and
   `Documents\Arduino` on Windows.

3. **Open the LVGL library folder in Finder.** On macOS the fastest path is
   `Cmd + Shift + G` to bring up "Go to Folder", then paste:

   ```
   ~/Documents/Arduino/libraries/lvgl/
   ```

   Press Return. Substitute the path that matches your sketchbook location
   if you have moved it (a CloudDocs path is common if you keep your
   sketchbook in iCloud Drive).

   If you do not see an `lvgl` folder in `libraries/`, install LVGL first
   through the Library Manager and run step 3 again.

4. **Back up the stock `lv_conf.h`.** Rename the existing file to
   `lv_conf.h.original`. This is your fallback if anything goes wrong and
   the file you will restore when working on other sketches that do not
   share this project's font set.

5. **Drop in the bundled `lv_conf.h`.** Copy the `lv_conf.h` from your
   PANELWRIGHT project bundle into `libraries/lvgl/`, replacing the stock
   file.

6. **(Optional) Verify the enabled sizes.** Open the new `lv_conf.h` in a
   text editor and search for `LV_FONT_MONTSERRAT_`. The sizes your
   project uses should each show `1`. For a project using sizes 14, 18,
   and 32:

   ```c
   #define LV_FONT_MONTSERRAT_14 1
   #define LV_FONT_MONTSERRAT_18 1
   #define LV_FONT_MONTSERRAT_32 1
   ```

   Sizes you do not use should stay at `0` to keep flash usage down.

7. **Compile.** Open `your_sketch/your_sketch.ino` in Arduino IDE, select
   Tools → Board → Arduino Mbed OS Giga Boards → Arduino Giga R1, then
   click Verify. A clean compile means every referenced font is enabled.

8. **Upload.** Connect the Giga over USB and click Upload. Your UI should
   appear with every size rendering correctly.

### One-time setup vs per-project

Most Field Instruments will reuse a similar set of font sizes (14, 18,
and 24 are the workhorses). Once you have installed the bundled
`lv_conf.h` for the first project, subsequent projects that use the same
sizes or a subset of them need no further changes. Only when a project
introduces a new size does the bundled `lv_conf.h` need to be reinstalled.

If you want one `lv_conf.h` that covers everything, you can enable every
Montserrat size you commonly reach for (or all 23 stock sizes, which adds
about 70 KB of flash). The Giga has plenty of flash to absorb this; the
trade-off is one larger binary versus the swap-and-rebuild dance on every
project change.

### Troubleshooting

`'lv_font_montserrat_NN' was not declared in this scope` means that size
is not enabled in your current `lv_conf.h`. Open the file, find
`LV_FONT_MONTSERRAT_NN` (substitute your size), change `0` to `1`, save,
and recompile. Or re-export the project bundle, which will set every
needed size automatically.

A Library Manager update can silently overwrite your custom `lv_conf.h`.
If your sketch suddenly stops finding font sizes, that is the most likely
cause. Restore from your `lv_conf.h.original` backup, or re-export the
PANELWRIGHT bundle and copy `lv_conf.h` over again. Pinning the LVGL
library version in Library Manager prevents the silent updates.

If you change which font sizes your project uses (renaming a header,
adding a screen with a large display value, etc.), regenerate the bundle
and replace `lv_conf.h` again. The font advisory in the code preview
modal lists every size your current project needs so you can cross-check
without leaving PANELWRIGHT.

---

## LVGL version compatibility

The **LVGL version** dropdown in Project Details switches the codegen
between two API targets:

- **LVGL 9.x** (default, current Arduino Library Manager version)
- **LVGL 8.x** (legacy, ships in older Arduino board packages)

The differences live entirely in the generator, not the editor. Picking
v9 emits `lv_scale_create` + `lv_arc_create` for the Meter widget (the
v9 way; `lv_meter` was removed in v9) and the single-argument
`lv_tabview_create(parent)` with separate `lv_tabview_set_tab_bar_*`
calls. Picking v8 emits the old `lv_meter_create` + scale + indicator
chain and the three-argument `lv_tabview_create(parent, dir, size)`.

If you compile and see `lv_meter_create was not declared in this
scope` (along with errors mentioning `lv_meter_scale_t`,
`lv_meter_indicator_t`, `lv_meter_add_arc`, etc.), the project is set
to v8 but your installed LVGL is v9. Switch the project to v9 and
regenerate.

Font sizes (`lv_font_montserrat_X`) are emitted with `#if
LV_FONT_MONTSERRAT_X` compile-time guards. If the board package's
`lv_conf.h` does not enable the requested size, the code falls back to
`LV_FONT_DEFAULT` (always defined). This avoids the "undeclared"
compile error when a generated sketch asks for a font size that your
board's configuration didn't compile in.

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

## Single .ino vs split layout

In **Project Details** the **Code layout** option chooses between two outputs.
The choice only affects what the zip export writes; the on-screen Generate
Code preview always shows the single-file form, because it is the most
readable representation of the whole project.

**Single .ino** (the default) puts everything in one file. Header comment,
includes, display and touch declarations, peripheral globals and init
functions, screen and widget handle declarations, theme, event handlers,
screen builders, `setup()`, and `loop()`. Three hundred lines for a small
project, two thousand for a large one. Zero indirection.

**Split into ui.h / ui.cpp** is the option for larger projects. The zip
contains three files:

- `<sketch>.ino` is the Arduino entry point. It owns the hardware: display
  and touch declarations, every peripheral's includes and init functions,
  `setup()` (which calls peripheral inits then `ui_init()`), and `loop()`.
- `ui.h` is the contract between the .ino and the LVGL logic. It declares
  the screen and widget handles as `extern`, exposes a single
  `void ui_init(void)` entry point, and includes `lvgl.h`.
- `ui.cpp` owns the UI logic. It defines every handle, holds the theme
  function, every event handler, every `build_screen_*` function, and
  implements `ui_init()` (theme apply + every screen build + load default).

The split exists because past a few hundred widgets the single-file output
becomes harder to navigate than separate compilation units. The .ino stays
short. The ui.cpp groups all the LVGL code together where it is easy to
read and to hand-edit after generation.

One caveat for users with custom Event Code: peripheral state like
`mic_buffer` lives in the .ino. If your custom code in an event handler
references a peripheral global, you will need to add a matching `extern`
declaration to the top of ui.cpp by hand. The default event handlers
(navigation, `Serial.println` stubs, value reads via `lv_*_get_value`) need
no externs and compile cleanly.

The validation pass, the `arduino_secrets.h` companion, the .gitignore, and
the tab sub-canvas codegen all work in both layouts.

---

## Auto-save

The editor writes a snapshot of the live project to `localStorage` on
every mutation (drag, drop, nudge, scrub, property edit, screen add,
peripheral toggle), debounced 500ms. On startup, if a saved snapshot is
present, it is restored automatically, and a small dismissable banner
near the top of the workspace confirms the restore with the original
time stamp. The banner has a **Start fresh** button that clears the
saved snapshot and reloads the default seed project.

Auto-save is independent of **Save .json**: the manual save downloads a
file for archival and version control, while auto-save is the safety
net for in-progress sessions. Browsers can and do discard background
tabs to reclaim memory; when that tab is revisited, the editor reloads
its HTML and would lose the JavaScript state that was holding the
project. Auto-save means the project comes back instead.

The storage key is `panelwright_autosave`, and the payload is a single
JSON blob with the same shape used internally for undo. If `localStorage`
is unavailable (private browsing, quota exceeded, file:// origin in
some browser configurations) the editor falls back to running without
the safety net; no error is shown.

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

## Docked live preview

The **Preview** button in the toolbar opens a side panel between the canvas
and the property panel. It renders the active screen at every edit, scaled
to fit the dock, with no selection chrome, no resize handles, and no drag
listeners. The preview shows what the device will actually display.

Goto buttons in the preview are clickable: clicking one switches the
editor's active screen, so you can walk a navigation flow without leaving
the editor. Tab content renders at its real position inside the parent
tabview (using the tabview's `activeTab` prop to pick which tab to show),
so previewing a screen with a tabview shows the tab that the device will
boot to.

The preview updates on every editor action: drag, drop, nudge, property
edit, scrub, theme change, screen switch, undo, redo. Toggle it off when
you do not need it; the dock takes about 380 pixels of horizontal screen
real estate.

---

## Image export

The **Export** button in the toolbar opens a small modal with four
options: **Scope** (current screen, or all screens packed into a zip),
**Format**, **PNG scale** (PNG only), and **Device shell**.

Three formats are available:

**PNG (raster, 2x by default)** rasterizes the screen at the chosen
scale: 1x produces a native 800x480 pixel image, 2x produces 1600x960
(the default, retina-ready), 3x and 4x for print or oversized
documentation. The rasterizer takes the native primitive SVG, loads it
into an off-screen `<canvas>`, and exports the canvas as a PNG blob.

**SVG, native primitives (universal)** emits `<rect>`, `<text>`,
`<circle>`, `<path>`, `<line>`, `<polygon>`, and `<polyline>` elements
that any SVG-aware tool can read. Inkscape, Illustrator, Affinity
Designer, GIMP, Preview, ImageMagick, and any browser all render the
result identically. File sizes are typically 1 to 5 KB per screen since
there is no embedded CSS payload.

**SVG, HTML in foreignObject (browser-only)** wraps the editor's HTML
widget renderers in an SVG `<foreignObject>` element with the relevant
page CSS inlined. This produces a pixel-perfect match with the editor
canvas (every CSS gradient, drop shadow, and font carries through) but
only browser engines render it. Most photo editors and vector tools
show a blank canvas when opening a foreignObject SVG. Use this mode
only when the SVG is going to be viewed in a browser context.

PNG and Native SVG both go through the same primitive renderers, so a
PNG export at 1x and a Native SVG export at 1x produce visually
identical output (the SVG can be scaled losslessly afterward, the PNG
cannot). For documentation that will be inserted into Markdown or a
slide deck, PNG at 2x is usually right. For documentation that needs
to scale arbitrarily, Native SVG.

There is one visual cost to the native renderers (which both PNG and
Native SVG use): certain CSS effects (linear gradients on slider knobs,
subtle box shadows on the device shell, the exact LVGL UI font) are
approximated rather than reproduced. The shapes, positions, sizes, and
colors all match the editor; a generic sans-serif font stands in for
the editor's UI font, and a few subtle gradient ramps become solid
fills. For matching the editor exactly to the pixel, the foreignObject
SVG mode remains available (and renders to PNG via any browser's
"save as image" feature).

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

### Fill Data mode

The viewer's **Fill Data** button (visible once a project is loaded) opens
a slide-out panel from the right side of the stage. Every value-bearing
widget on the active screen gets a row with a type-appropriate control:

- Sliders, bars, arcs, and meters get a range slider plus a number input,
  bounded by the widget's min and max.
- Labels and textareas get a text input.
- Switches, checkboxes, and the LED widget get a checkbox.
- Dropdowns and rollers get a select with the widget's options.

Each row has a small amber dot that lights up when an override is active.
The **Clear overrides** button in the panel footer resets everything to
the widget defaults from the saved project.

Overrides do not persist. They are a what-if tool for walking through what
the design will look like with realistic data ("battery at 23 percent",
"WiFi off", "temperature reading 87 degrees") before you flash the board
and feed it real values. Changing screens preserves overrides on the
screen you left; loading a new project clears everything.

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
