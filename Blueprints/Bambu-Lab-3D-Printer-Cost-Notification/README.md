## Bambu Lab 3D Printer Cost Notification (Home Assistant Blueprint)

Calculate **filament**, **electricity**, and **depreciation** costs for each Bambu Lab 3D print and send a **Telegram** message (official Telegram Bot integration) with a full breakdown. Optionally include the Bambu **cover image**, track **all-time** stats, and support **Do Not Disturb** during pauses. Filament *length* is optional.

> Heavily inspired by [Colton Onushko’s automation](https://coltography.ca/usage-cost-notifications-with-home-assistant-and-bambu-lab-3d-printers/). This blueprint relies on the **Bambu Lab HACS integration** (not OrcaSlicer/G-code scripts) and uses **Telegram** instead of Discord. Big thanks to him, I would not have created this blueprint without his logic.

### Features

- Cost breakdown per print (configurable):
  - Filament cost (from weight × price/kg)
  - Electricity cost (kWh × price/kWh)
  - Printer depreciation (purchase cost & lifetime in hours)
- Optional **filament length** reporting
- Optional **“Do Not Disturb”** pause notifications control
- Optional **cumulative stats** (helpers) updates
- **Telegram Bot** model image + caption (MarkdownV2) or text-only fallback
- Works with **Bambu Lab (HACS) integration** cover image

------

## Requirements

- Home Assistant 2024.x or later.
- **HACS Bambu Lab** integration configured (printer in HA, LAN/cloud as you prefer).
- A smart plug or energy sensor for the printer **cumulative kWh** (device_class: `energy`).
- Official **Telegram Bot** integration:
  - Create a Bot via @BotFather.
  - Add the bot to your chat(s).
  - Get **chat ID(s)** (see tips below).
- `input_number` helpers for cumulative stats and last print kWh; `input_boolean` for sleep mode.
  *Some of them are required, some optional* 

------

## Installation

1. **Import the blueprint**

   [![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2Fsoflane%2FHome-assistant-config%2Frefs%2Fheads%2Fmain%2FBlueprints%2FBambu-Lab-3D-Printer-Cost-Notification%2FBBL-Printer-Cost-notification-blueprint.yaml)


   Or save the blueprint YAML to:

   ```
   config/blueprints/automation/<your_namespace>/bambu_lab_print_cost.yaml
   ```

   If imported manually, go to **Settings** In Home Assistant **→ Automations & Scenes → Blueprints → Import/Reload**.

2. Create a new automation **from this blueprint** and fill the inputs (see below).

------

## Inputs & What They Do

| Input                                                        | Required | Notes                                                        |
| ------------------------------------------------------------ | -------- | ------------------------------------------------------------ |
| **Home Assistant URL** (`ha_url`)                            | ✅        | Base URL (no trailing slash). Used to prefix image URLs in Telegram. Example: `http://homeassistant.local:8123` or your external URL. |
| **Printer Status Sensor**                                    | ✅        | `sensor.<printer>_print_status` from the **Bambu HACS** integration. Triggers prepare/finish/pause. |
| **Print Cover Image**                                        |          | The **image** entity exposed by the Bambu integration, if available. Leave empty to send text-only. |
| **Start Time / End Time Sensors**                            | ✅        | `sensor.<printer>_start_time` and `sensor.<printer>_end_time` from the Bambu integration. Used to compute duration. |
| **Print Weight Sensor**                                      | ✅        | Grams (integration sensor or `input_number`). Used for filament cost (Typically from the Bambu integration) |
| **Print Length Sensor**                                      |          | Sensor/helper for filament length (mm or m). Leave empty if you don’t have it. |
| **Total Usage Hours Sensor**                                 |          | `sensor.<printer>_total_usage_hours` (or similar) from the integration, for all-time usage stats. |
| **Energy Consumption Sensor**                                | ✅        | Your smart plug’s **cumulative kWh** sensor for the printer. |
| **Start kWh Helper**                                         | ✅        | `input_number` to store the start kWh reading. Create one (e.g. `input_number.3d_print_kwh_start`). |
| **Filament Price per KG**                                    | ✅        | e.g. `25.99` for €25.99/kg (your currency).                  |
| **Electricity Cost per kWh**                                 | ✅        | e.g. `0.385` for €0.385/kWh.                                 |
| **Printer Purchase Cost**                                    | ✅        | e.g. `1000`.                                                 |
| **Printer Lifetime (hours)**                                 | ✅        | e.g. `6000`.                                                 |
| **Telegram Chat ID(s)**                                      | ✅        | One or more chat IDs (comma-separated) . See below to find your chat ID(s).  e.g. `-1001234567890, 12345678` |
| **Sleep Mode Toggle**                                        |          | `input_boolean` to suppress pause alerts (DND).              |
| **Cumulative Stats helpers** *(Filament Weight, Filament Cost, Electrical Cost, Total Cost, Last Print kWh)* |          | Create `input_number` helpers if you want totals updated after each print: `cumulative_filament_weight` (g), `cumulative_filament_cost`, `cumulative_electrical_cost`, `cumulative_total_cost`, and (optionally) `last_print_kwh_used`. |
| **Disable Image Sending**                                    |          | If `true`, always send text (no image).                      |
| **Currency Symbol**                                          | ✅        | Defaults to `$`. Use `€`, `£`, etc. Only symbol (no spaces). Used for display only |

> **Why length is optional?** Cost uses **weight × price/kg**. Length (m/ft) is nice for reporting but not essential.

------

#### Creating the helpers

In Home Assistant, go to **Settings → Devices & Services → Helpers → Create Helper → Number**:

- `input_number.3d_print_kwh_start` (no unit required)
- Optionally:
   `input_number.3d_cumulative_filament_weight` (g),
   `input_number.3d_cumulative_filament_cost`,
   `input_number.3d_cumulative_electrical_cost`,
   `input_number.3d_cumulative_total_cost`,
   `input_number.3d_last_print_kwh_used`

![img](https://coltography.ca/wp-content/uploads/2024/12/Screenshot-2024-12-28-200453.png)

*Image from [Colton Onushko’s post](https://coltography.ca/usage-cost-notifications-with-home-assistant-and-bambu-lab-3d-printers/)*



------

## Getting your Telegram chat ID

First, Set up the **official Telegram** integration and your bot.

##### Method 1: Telegram IDbot 

- For private chat: message your bot; check HA **Notifications → Telegram** logs, or use @userinfobot to see your chat id.
- For groups/supergroups: add your bot and send a message; use @RawDataBot or check HA logs. Group chat IDs are usually negative (e.g. `-100…`).

##### Method 2: developer style

- Send a message to your bot from your Telegram account (or a group).
- In Home Assistant, watch **Developer Tools → Events** and listen for `telegram_text` (or check the integration logs). Copy the `chat_id`.
- For multiple recipients, separate chat IDs with commas.

Enter one or **multiple** IDs separated by commas:

```
123456789
-1001122334455, 987654321
```

------

## How It Works

- **On `prepare`**: stores current cumulative kWh into `Start kWh Helper`.
- **On `finish`**:
   - Reads weight (g), optional length (mm/m), cumulative kWh, start/end timestamps.
   - Calculates: kWh used, **filament cost** (`weight_g × price/kg ÷ 1000`), **electricity cost** (`kWh × price/kWh`), **depreciation** (`purchase_cost / lifetime_hours × hours_printed`).
   - Updates cumulative helpers if provided.
   - Sends **Telegram** message with MarkdownV2 Caption/body:
     - If cover image available & not disabled: `telegram_bot.send_photo`.
     - Else: `telegram_bot.send_message`.

------

## Example: Minimum Setup

- **ha_url**: `http://homeassistant.local:8123`
- **printer_status_sensor**: `sensor.p1s_print_status`
- **printer_cover_image**: `image.p1s_cover_image` *(or leave empty)*
- **printer_start_time_sensor**: `sensor.p1s_start_time`
- **printer_end_time_sensor**: `sensor.p1s_end_time`
- **print_weight_sensor**: `sensor.p1s_print_weight`
- **total_usage_sensor**: `sensor.p1s_total_usage_hours`
- **energy_usage_sensor**: `sensor.printer_plug_energy_kwh`
- **start_energy_helper**: `input_number.print_kwh_start`
- **filament_price_per_kg**: `25.99`
- **energy_cost_per_kwh**: `0.40`
- **printer_purchase_cost**: `1000`
- **printer_lifetime_hours**: `6000`
- **telegram_chat_ids**: `-1001234567890`

*(All cumulative helpers optional.)*

------

## Message Example

```
🎉 3D PRINT COMPLETED
⏱️ Duration: 3 hours, 17 minutes
⚖️ Weight: 123.4 g – Cost: €3.20
🧵 Filament Length: 12.5 m / 41.01 ft   (optional line)
🔌 Power: 0.42 kWh – Cost: €0.17
🛠️ Depreciation: €0.53
💰 Total Cost: €3.90

🕒 All Time Usage: 187.5 h
📊 All Time Cost: €142.77
🧵 All Time Filament: 6,235 g / 13.74 lbs
```



### Tips & Troubleshooting

- **No image in Telegram:**
  - Ensure **Print Cover Image** entity is set and has a valid `entity_picture`/`cover_image`.
  - Make sure your **Home Assistant URL** is reachable by Telegram (if you’re proxying images; consider your public URL). If not, use **Disable Image**, or expose your URL via a secure method (reverse proxy, etc.).
- **Caption rejected / Markdown errors:**
  - The blueprint escapes MarkdownV2 characters. If you customize the template, keep the `md()` macro or switch to `parse_mode: html` (If editing blueprint).
- **kWh stays 0:**
  - Your energy sensor must be **cumulative** (`device_class: energy`) and in **kWh**.
  - Ensure the printer is powered via that plug and the sensor updates frequently.
- **Wrong duration:**
  - Verify the **start/end time** sensors update correctly in your Bambu integration.
  - This blueprint uses those timestamps directly.
- **Currency formatting:**
  - `currency_symbol` is pasted literally (no trailing space). Example: `€`.

------

## Privacy & Security

- The Telegram caption includes cost/usage data. If that’s sensitive, use a **private chat** or limit the fields you include.
- If you use an external `ha_url`, ensure it uses HTTPS where possible.

------

## Changelog

- **v1.0.0** — Initial public release (length optional, Telegram official integration, MarkdownV2 escaping, image fallback to text).

------

## Credits

- Original idea & flow heavily inspired by [Colton Onushko’s automation](https://coltography.ca/usage-cost-notifications-with-home-assistant-and-bambu-lab-3d-printers/).
- Uses **HACS Bambu Lab** integration data for status, times, usage, and cover image.