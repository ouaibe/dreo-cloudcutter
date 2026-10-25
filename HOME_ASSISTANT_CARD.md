# Home Assistant dashboard cards for the fan

There are two ways to put the fan on a dashboard. Both show every control in one place (on/off, speed, mode, oscillation, auto-off timer, temperature), and both stay in sync with the fan's physical buttons.

| | [Native](#option-1-native-zero-dependencies) | [Modern](#option-2-modern) |
| --- | --- | --- |
| Needs | nothing: paste one card | 2 template helpers + 1 HACS card (optional) |
| Mode | segmented buttons | segmented buttons |
| Oscillation | dropdown (`30 Degrees`…) | segmented buttons (`Off · 30° · 60° · 90° · 120°`) |
| Auto-off timer | minutes slider (0-720) | one-tap presets (`Off · 15 min · 30 min · 1 h · 2 h · 4 h`) |
| Look | a stack of tiles, each in its own frame | one card, one frame |

## Requirements (both options)

- The current [`Dreo_DR-HTF004S.yaml`](./code/Dreo_DR-HTF004S.yaml) flashed on the fan. It provides the `Fan` entity and the `Auto-Off Timer` and needs ESPHome 2026.10 or newer.
- Home Assistant **2026.10** or newer, for the tile card's `select-options` button style.

## Entity IDs

The examples use the entity IDs Home Assistant creates for the repo's YAML as-is (`friendly_name: Dreo DR-HTF004S`), so every ID starts with `dreo_dr_htf004s`. If you renamed your device, replace that prefix everywhere. You can check your IDs under **Settings → Devices & services → ESPHome → your fan**.

| Used for | Entity |
| --- | --- |
| Fan (on/off, speed) | `fan.dreo_dr_htf004s_fan` |
| Mode | `select.dreo_dr_htf004s_fan_mode` |
| Oscillation angle | `select.dreo_dr_htf004s_oscillation_angle` |
| Auto-off timer (minutes) | `number.dreo_dr_htf004s_auto_off_timer` |
| Temperature | `sensor.dreo_dr_htf004s_air_temperature` |

> **Tip:** the temperature arrives with many decimals. Open the sensor's settings in HA and set **Display precision** to one decimal. It applies to both options.

To add a card, go to **Edit dashboard → Add card → Manual** and paste the YAML. For several fans, add one card per fan with its own prefix and `heading`.

---

## Option 1: Native (zero dependencies)

This option uses only built-in cards and the entities the firmware creates. Nothing else to install or configure.

```
  Dreo Fan                                🌡 22.2 °C  ⏲ 45 min
┌─────────────────────────────────────────────────────────────┐
│ 🌀 Fan                                     On · 58 %         │
│ ━━━━━━━━━━━━━━━━━━━●━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━   │
└─────────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────────┐
│ 🌀 Mode                                                      │
│ [▓ Normal ▓│ Auto │ Natural │ Sleep ]                        │
└─────────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────────┐
│ ↔ Oscillation                          [ 30 Degrees      ▾ ] │
└─────────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────────┐
│ ⏲ Auto-off timer                                  45 min     │
│ ━━━━━━●━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━   │
└─────────────────────────────────────────────────────────────┘
```

```yaml
type: vertical-stack
cards:
  - type: heading
    heading: Dreo Fan
    icon: mdi:fan
    heading_style: title
    badges:
      - type: entity
        entity: sensor.dreo_dr_htf004s_air_temperature
        icon: mdi:thermometer
        show_icon: true
        show_state: true
      - type: entity
        entity: number.dreo_dr_htf004s_auto_off_timer
        icon: mdi:timer-outline
        show_icon: true
        show_state: true
        color: orange
        visibility:
          - condition: numeric_state
            entity: number.dreo_dr_htf004s_auto_off_timer
            above: 0
  - type: tile
    entity: fan.dreo_dr_htf004s_fan
    name: Fan
    color: amber
    state_content:
      - state
      - preset_mode
      - percentage
    features_position: bottom
    features:
      - type: fan-speed
  - type: tile
    entity: select.dreo_dr_htf004s_fan_mode
    name: Mode
    icon: mdi:fan-auto
    color: teal
    hide_state: true
    features_position: bottom
    features:
      - type: select-options
        style: buttons
        options:
          - Normal
          - Auto
          - Natural
          - Sleep
  - type: tile
    entity: select.dreo_dr_htf004s_oscillation_angle
    name: Oscillation
    icon: mdi:arrow-oscillating
    color: blue
    hide_state: true
    features_position: inline
    features:
      - type: select-options
        style: dropdown
  - type: tile
    entity: number.dreo_dr_htf004s_auto_off_timer
    name: Auto-off timer
    icon: mdi:timer-outline
    color: blue-grey
    features_position: bottom
    features:
      - type: numeric-input
        style: slider
```

---

## Option 2: Modern

This option builds one card with one frame. Oscillation and the timer get short, one-tap labels, and the current setting stays highlighted.

```
┌ Dreo Fan ─────────────────────────── 🌡 22.2 °C  ⏲ 1 h ┐
│ 🌀 Fan                                    On · Auto      │
│ ━━━━━━━━━━━━━━━━●━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━   │
│ 🌀 Mode                                                  │
│ [ Normal │▓ Auto ▓│ Natural │ Sleep ]                    │
│ ↔ Oscillation                                            │
│ [ Off │▓ 30° ▓│ 60° │ 90° │ 120° ]                       │
│ ⏲ Auto-off                                       1 h     │
│ [ Off │ 15 min │ 30 min │▓ 1 h ▓│ 2 h │ 4 h ]            │
└──────────────────────────────────────────────────────────┘
```

It needs:
- **Two template helpers** that give oscillation and the timer short labels (step 1).
- **[`vertical-stack-in-card`](https://github.com/ofekashery/vertical-stack-in-card)** from HACS, for the single frame. It's optional: without it, replace the first line of the card with `type: vertical-stack`. Everything works the same, but the heading and rows are drawn as separate pieces.

### Step 1: two helpers with short labels

The fan's own selects use long labels (`30 Degrees`) and the timer is a number of minutes, which makes for crowded buttons. These two template selects give them short labels (`30°`, `1 h`) and translate in both directions. They don't add any logic: picking `1 h` simply sets the timer to 60.

Add this to `configuration.yaml`, then restart Home Assistant or reload **Template entities** from **Developer tools → YAML**:

```yaml
template:
  - select:
      - name: "Dreo DR-HTF004S Oscillation"
        unique_id: dreo_dr_htf004s_oscillation
        icon: mdi:arrow-oscillating
        state: >
          {% set m = {'OFF':'Off','30 Degrees':'30°','60 Degrees':'60°','90 Degrees':'90°','120 Degrees':'120°'} %}
          {{ m.get(states('select.dreo_dr_htf004s_oscillation_angle'), 'Off') }}
        options: "{{ ['Off','30°','60°','90°','120°'] }}"
        select_option:
          - action: select.select_option
            target:
              entity_id: select.dreo_dr_htf004s_oscillation_angle
            data:
              option: >
                {{ {'Off':'OFF','30°':'30 Degrees','60°':'60 Degrees','90°':'90 Degrees','120°':'120 Degrees'}[option] }}

      - name: "Dreo DR-HTF004S Auto-off"
        unique_id: dreo_dr_htf004s_auto_off
        icon: mdi:timer-outline
        # A preset value shows its label; anything else (e.g. a running countdown) shows "N min".
        state: >
          {% set n = states('number.dreo_dr_htf004s_auto_off_timer') | float(0) | int %}
          {% set m = {0:'Off',15:'15 min',30:'30 min',60:'1 h',120:'2 h',240:'4 h'} %}
          {{ m[n] if n in m else n ~ ' min' }}
        options: "{{ ['Off','15 min','30 min','1 h','2 h','4 h'] }}"
        select_option:
          - action: number.set_value
            target:
              entity_id: number.dreo_dr_htf004s_auto_off_timer
            data:
              value: >
                {{ {'Off':0,'15 min':15,'30 min':30,'1 h':60,'2 h':120,'4 h':240}[option] }}
```

This creates `select.dreo_dr_htf004s_oscillation` and `select.dreo_dr_htf004s_auto_off`.

**Prefer the UI?** Go to **Settings → Devices & services → Helpers → Create helper → Template → Select**. Paste the `state` and `options` templates, then add the action from `select_option`. Optionally link the helper to the fan's device so it shows up on the device page. After creating it, check its entity ID: linking it to a device can add a prefix such as the area name. Rename it so it matches the card below.

Want different timer presets? Edit the same values in all three places: `m`, `options` and the `value` map.

### Step 2: the card

```yaml
type: custom:vertical-stack-in-card
cards:
  - type: heading
    heading: Dreo Fan
    icon: mdi:fan
    heading_style: title
    badges:
      - type: entity
        entity: sensor.dreo_dr_htf004s_air_temperature
        icon: mdi:thermometer
        show_icon: true
        show_state: true
      - type: entity
        entity: select.dreo_dr_htf004s_auto_off
        icon: mdi:timer-outline
        show_icon: true
        show_state: true
        color: orange
        visibility:
          - condition: state
            entity: select.dreo_dr_htf004s_auto_off
            state_not: "Off"
  - type: tile
    entity: fan.dreo_dr_htf004s_fan
    name: Fan
    color: amber
    state_content:
      - state
      - preset_mode
      - percentage
    features_position: bottom
    features:
      - type: fan-speed
  - type: tile
    entity: select.dreo_dr_htf004s_fan_mode
    name: Mode
    icon: mdi:fan-auto
    color: teal
    hide_state: true
    features_position: bottom
    features:
      - type: select-options
        style: buttons
        options:
          - Normal
          - Auto
          - Natural
          - Sleep
  - type: tile
    entity: select.dreo_dr_htf004s_oscillation
    name: Oscillation
    icon: mdi:arrow-oscillating
    color: blue
    hide_state: true
    features_position: bottom
    features:
      - type: select-options
        style: buttons
  - type: tile
    entity: select.dreo_dr_htf004s_auto_off
    name: Auto-off
    icon: mdi:timer-outline
    color: blue-grey
    features_position: bottom
    features:
      - type: select-options
        style: buttons
```

> **About HACS cards:** any custom card runs inside your Home Assistant session with your user's permissions. `vertical-stack-in-card` is small (around 330 lines, no network access) and easy to read; it's worth glancing at the changes before you accept an update.

---

## How the controls behave

- **Mode:** `Normal` means a manual speed. `Auto`, `Natural` and `Sleep` are the fan's presets. Moving the speed slider switches back to `Normal`. When the fan is off, no mode is highlighted, and picking one turns the fan on in that mode.
- **Oscillation:** `Off` / `OFF` stops it; any angle starts it at that angle. The Fan entity's own oscillation toggle (used by HomeKit, Google, etc.) restores the last angle.
- **Auto-off timer:** the fan's built-in "turn off after N minutes" timer. `0` / `Off` means no timer. It is cleared when the fan turns off and is not kept across reboots. In the modern card, a value that isn't one of the presets shows as `N min` with no button highlighted.
- Everything stays in sync with the fan's physical buttons, because the fan reports each change over UART.
