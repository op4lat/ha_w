
# ha_w - Home Assistant Weather Utilities

A collection of lightweight, template-based weather utilities for **Home Assistant**, focusing on converting raw wind speed data into the descriptive **Beaufort Wind Scale**.

## 🌬️ Beaufort Wind Scale Sensor

This project provides a single, efficient template sensor that converts a numeric Beaufort scale (0–12) into human-readable descriptions for **Land**, **Sea**, and a **Short Summary**.

Instead of creating multiple entities, this implementation uses **attributes** to keep your entity registry clean while providing rich data for dashboards and automations.

### Features

*   **Single Entity**: One sensor (`sensor.beaufort_scale`) instead of three.
*   **Rich Attributes**: Includes `land`, `sea`, `number`, and a descriptive `state`.
*   **Modern Syntax**: Uses the modern `template:` integration with `state` and `attributes`.
*   **Dashboard Ready**: Optimized for use with **Mushroom Cards** and **Bubble Cards**.
*   **Auto-Loading**: Designed for `!include_dir_merge_list` setups.

### Installation

1.  **Place the File**:
    Copy `beaufortscale.yaml` into your Home Assistant templates directory:
    ```bash
    /root/config/templates/beaufortscale.yaml
    ```

2.  **Verify Configuration**:
    Ensure your `configuration.yaml` includes the directory merge command:
    ```yaml
    template: !include_dir_merge_list templates
    ```
    *This automatically loads all `.yaml` files in the `/root/config/templates/` folder. No need to list individual files.*

3.  **Prerequisites**:
    You need an existing sensor that provides the **numeric Beaufort scale** (0–12). By default, this template expects:
    *   `sensor.st_beaufort_scale`

    *If your sensor has a different ID, update the `states('sensor.st_beaufort_scale')` calls in the template.*

4.  **Restart**:
    Go to **Developer Tools > YAML > Check Configuration**, then **Restart** Home Assistant.

### Configuration (`beaufortscale.yaml`)

```yaml
- sensor:
    - name: "beaufort_scale"
      unique_id: beaufort_scale
      state: >
        {% set bft = states('sensor.st_beaufort_scale') | float(0) | int %}
        {% set desc = {
          0: 'Calm', 1: 'Light air', 2: 'Light breeze', 3: 'Gentle breeze', 
          4: 'Moderate breeze', 5: 'Fresh breeze', 6: 'Strong breeze', 
          7: 'High wind', 8: 'Gale', 9: 'Strong gale', 10: 'Storm', 
          11: 'Violent storm', 12: 'Hurricane force'
        } %}
        {{ desc.get(bft, 'Unknown') }}
      attributes:
        number: >
          {{ states('sensor.st_beaufort_scale') | float(0) | int }}
        land: >
          {% set bft = states('sensor.st_beaufort_scale') | float(0) | int %}
          {% set land_map = {
            0: 'Calm. Smoke rises vertically.',
            1: 'Direction shown by smoke drift.',
            2: 'Wind felt on face; leaves rustle.',
            3: 'Leaves and small twigs in constant motion.',
            4: 'Raises dust and loose paper; small branches moved.',
            5: 'Small trees in leaf begin to sway.',
            6: 'Large branches in motion; whistling in telegraph wires.',
            7: 'Whole trees in motion; inconvenience felt walking.',
            8: 'Twigs break off trees; generally impedes progress.',
            9: 'Slight structural damage (chimney pots and slates removed).',
            10: 'Seldom experienced inland; trees uprooted.',
            11: 'Very rarely experienced; accompanied by widespread damage.',
            12: 'Hurricane force. Devastation.'
          } %}
          {{ land_map.get(bft, 'Unknown') }}
        sea: >
          {% set bft = states('sensor.st_beaufort_scale') | float(0) | int %}
          {% set sea_map = {
            0: 'Sea like a mirror.',
            1: 'Ripples with appearance of scales.',
            2: 'Small wavelets; crests have glassy appearance.',
            3: 'Large wavelets; crests begin to break.',
            4: 'Small waves becoming longer; fairly frequent white horses.',
            5: 'Moderate waves; many white horses; chance of spray.',
            6: 'Large waves begin to form; white foam crests everywhere.',
            7: 'Sea heaps up; white foam blown in streaks.',
            8: 'Moderately high waves; edges of crests break into spindrift.',
            9: 'High waves; dense streaks of foam; sea begins to roll.',
            10: 'Very high waves; overhanging crests; visibility affected.',
            11: 'Exceptionally high waves; sea covered with white patches.',
            12: 'Air filled with foam and spray; visibility seriously affected.'
          } %}
          {{ sea_map.get(bft, 'Unknown') }}
```

### Dashboard Usage

#### Mushroom Cards (Recommended)
Use a **Template Card** to show all data in one compact view:

```yaml
type: custom:mushroom-template-card
entity: sensor.beaufort_scale
icon: mdi:weather-windy
icon_color: blue
primary: "{{ states('sensor.beaufort_scale') }}"
secondary: >
  Scale: {{ state_attr('sensor.beaufort_scale', 'number') }}
  Land: {{ state_attr('sensor.beaufort_scale', 'land') }}
  Sea: {{ state_attr('sensor.beaufort_scale', 'sea') }}
multiline_secondary: true
```

#### Bubble Card
Use **sub-buttons** to display attributes interactively:

```yaml
type: custom:bubble-card
card_type: button
button_type: state
entity: sensor.beaufort_scale
name: Beaufort Scale
icon: mdi:weather-windy
show_state: true
sub_button:
  - entity: sensor.beaufort_scale
    name: Scale
    attribute: number
    show_attribute: true
  - entity: sensor.beaufort_scale
    name: Land
    attribute: land
    show_attribute: true
  - entity: sensor.beaufort_scale
    name: Sea
    attribute: sea
    show_attribute: true
```

### File Structure

```text
/root/config/
├── configuration.yaml
│   # Contains: template: !include_dir_merge_list templates
└── templates/
    ├── beaufortscale.yaml   # The active single sensor
    ├── beaufort.yaml        # (Optional/Legacy) Old multi-sensor version
    └── beaufort.disable     # (Reference) Original chart data
```

### Troubleshooting

*   **Entity shows "Unknown"**: Ensure `sensor.st_beaufort_scale` exists and returns a number (0–12).
*   **Visual Editor Warning**: When editing dashboard YAML, you may see *"Visual editor not supported"*. This is normal for attribute-based configurations; simply continue editing in YAML mode.
*   **Old Sensors Persist**: If old `sensor.beaufort_land` or `sensor.beaufort_sea` entities still appear, they are orphaned. You may need to delete them via **Settings > Devices & Services > Entities** or use the `recorder.purge_entities` service.
*   **Typo in State**: If you see "clam" instead of "Calm", check your template for typos in the `state` dictionary.

### License
GNU General Public License v3.0


