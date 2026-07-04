# Irrigation Controller (Home Assistant)

Two-part system for managing irrigation across multiple zones:

1. **`irrigation_zone_controller.yaml`** (blueprint) — one instance per zone. Reactively enforces soil moisture limits, rain delay, and rain forecast, and supports a manual override timer for deep-watering. Does **not** handle scheduling — it reacts to whatever opens the valve.
2. **`irrigation_sequencer.yaml`** (automation, edit directly for your zones) — runs twice daily at set times, walks your zones in order, checks moisture and rain/forecast for each zone right before it would start, waters or skips accordingly, and always pauses a fixed gap before the next zone regardless of whether the previous one watered. This is what actually opens each zone's valve on schedule.

This split exists because a single shared calendar-per-zone approach means a skipped zone just waits for its own next scheduled time instead of letting the next zone go — the sequencer fixes that by owning the order and moving on immediately.

## Features

- **Ordered daily sequencing** — zones run one after another in the order you list them, twice daily (5am/3pm by default); a skipped zone doesn't block the rest.
- **Soil moisture aware** — the sequencer only starts a zone if moisture is at/below min; the per-zone blueprint stops it early if moisture reaches max mid-run.
- **Rain-aware, checked per zone** — right before each zone would start, the sequencer re-checks the shared rain delay switch and any forecast sensors, so a change in rain conditions partway through a run affects zones not yet started. The per-zone blueprint also reacts if rain/forecast changes mid-run on a zone that's already watering.
- **Max duration safety cap** — each zone has its own timeout (20 minutes by default) so a stuck sensor or valve can't run forever.
- **Fixed gap between zones, always** — a configurable pause (1 minute by default) runs after every zone, whether it watered or was skipped, so equipment like a well pump hub never sees continuous flow across zones, and only one zone is ever active at a time.
- **Manual override with a real escape hatch** — start a zone's timer directly to force watering for a set duration (e.g. deep-watering new plants), bypassing both the moisture thresholds and rain delay/forecast. Watering runs until the timer finishes, is paused, or is canceled, then everything reverts automatically to normal threshold-based behavior — no settings to remember to reset.
- **Notifications** at every state change (started, stopped, skipped, canceled) with current moisture readings and skip reason.
- **One blueprint, many zones** — all per-zone entities (soil sensor, thresholds, valve, timer, rain delay, forecast sensors, notify target) are exposed as blueprint inputs, so each zone's reactive guard is just a new automation instance created from a form in the UI — no copy-pasting or editing YAML per zone.

## Requirements

- A soil moisture sensor per zone
- Two `input_number` helpers per zone (min/max moisture threshold)
- A `valve` entity (or adaptable to a switch) controlling each zone's water
- A `timer` helper per zone for manual overrides
- A shared `input_boolean` for rain delay
- (Optional) One or more numeric sensors for rain forecast (chance of rain %, expected precip amount, etc.) — must report a plain number, not a condition string. Weather integrations that only expose a `weather.xxx` entity typically need a small template sensor to extract a numeric value before they'll work here.
- A notify service (e.g. a mobile app notify target)

## Installation

1. Copy `irrigation_zone_controller.yaml` into `config/blueprints/automation/<your_folder>/` in your Home Assistant config, or import it via **Settings → Automations & Scenes → Blueprints → Import Blueprint** using the URL to the raw file itself (e.g. `https://raw.githubusercontent.com/<user>/<repo>/main/irrigation_zone_controller.yaml`, or the GitHub blob URL) — not just the repo's main page.
2. Reload blueprints (**Blueprints → ⋮ → Reload Blueprints**) or restart Home Assistant.
3. Create a new automation from the blueprint for each zone and fill in the inputs.
4. Edit `irrigation_sequencer.yaml` directly with your actual entity IDs, start time, and zone order (this one isn't a blueprint — it's specific to your setup), then import it as a regular automation (paste into a new automation's YAML editor, or add the file to `config/automations.yaml`-style storage and reload automations).

## License

MIT

