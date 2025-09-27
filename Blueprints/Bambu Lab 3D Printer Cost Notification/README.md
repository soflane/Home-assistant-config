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
- **Telegram Bot** photo + caption (MarkdownV2) or text-only fallback
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
