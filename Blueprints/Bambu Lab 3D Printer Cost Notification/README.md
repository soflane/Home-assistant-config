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

   [![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2Fsoflane%2FHome-assistant-config%2Frefs%2Fheads%2Fmain%2FBlueprints%2FBambu+Lab+3D+Printer+Cost+Notification%2FBambu-Labs-3DPrinter-Print-Cost-Notification.yaml)


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
| **Electricity Cost per kWh**                                 | ✅        | e.g. `0.40` for €0.40/kWh.                                   |
| **Printer Purchase Cost**                                    | ✅        | e.g. `1000`.                                                 |
| **Printer Lifetime (hours)**                                 | ✅        | e.g. `6000`.                                                 |
| **Telegram Chat ID(s)**                                      | ✅        | One or more chat IDs (comma-separated) . See below to find your chat ID(s).  e.g. `-1001234567890, 12345678` |
| **Sleep Mode Toggle**                                        |          | `input_boolean` to suppress pause alerts (DND).              |
| **Cumulative Stats helpers** *(Filament Weight, Filament Cost, Electrical Cost, Total Cost, Last Print kWh)* |          | Create `input_number` helpers if you want totals updated after each print: `cumulative_filament_weight` (g), `cumulative_filament_cost`, `cumulative_electrical_cost`, `cumulative_total_cost`, and (optionally) `last_print_kwh_used`. |
| **Disable Image Sending**                                    |          | If `true`, always send text (no image).                      |
| **Currency Symbol**                                          | ✅        | Defaults to `$`. Use `€`, `£`, etc. Only symbol (no spaces). Used for display only |

> **Why length is optional?** Cost uses **weight × price/kg**. Length (m/ft) is nice for reporting but not essential.
