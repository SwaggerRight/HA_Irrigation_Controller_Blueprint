# Irrigation Controller (Home Assistant)

Two-part system for managing irrigation across multiple zones:

1. **`irrigation_zone_controller.yaml`** (blueprint) — one instance per zone. Purely reactive: enforces a soil moisture max threshold, rain delay, and rain forecast, and supports a manual override timer for deep-watering. Does **not** handle scheduling or a min threshold — it only reacts to whatever opens the valve and decides when to stop it.
2. **`irrigation_sequencer.yaml`** (automation, edit directly for your zones) — runs twice daily at set times, walks your zones in order, checks moisture and rain/forecast for each zone right before it would start, waters or skips accordingly, confirms the valve actually opened and closed before moving on, and always pauses a fixed gap before the next zone regardless of outcome. This is what actually opens each zone's valve on schedule.

This split exists because a single shared calendar-per-zone approach means a skipped zone just waits for its own next scheduled time instead of letting the next zone go — the sequencer fixes that by owning the order and moving on immediately.

## Features

- **Ordered daily sequencing** — zones run one after another in the order you list them, twice daily (5am/3pm by default); a skipped zone doesn't block the rest.
- **Soil moisture aware** — the sequencer only starts a zone if moisture is at/below its min threshold; the per-zone blueprint stops it early if moisture reaches max mid-run.
- **Sensor-unavailable safety check** — if a zone's soil sensor is `unknown`/`unavailable` when checked, that zone is skipped (not watered) with a distinct warning notification, instead of silently treating a missing reading as 0% moisture and watering anyway.
- **Rain-aware, checked per zone** — right before each zone would start, the sequencer re-checks the shared rain delay switch and any forecast sensors, so a change in rain conditions partway through a run affects zones not yet started. The per-zone blueprint also reacts if rain/forecast changes mid-run on a zone that's already watering.
- **Efficient triggers** — the blueprint's moisture-max stop uses a `numeric_state` trigger tied directly to the threshold entity, so it only fires when moisture actually crosses that line — not on every routine sensor update.
- **Valve open/close confirmation** — the sequencer waits for the valve to confirm it actually reached "open" before starting the watering-duration clock (handles slow/cloud-based valves), and waits for confirmed "closed" — whether that closure came from the moisture/rain guard or from the sequencer's own timeout — before moving to the next zone. If closure can't be confirmed, you get a distinct "VALVE NOT CONFIRMED CLOSED" alert rather than silent assumption.
- **Max duration safety cap** — each zone has its own timeout (20 minutes by default) so a stuck sensor or valve can't run forever.
- **Fixed gap between zones, always** — a configurable pause (1 minute by default) runs after every zone, whether it watered or was skipped, so equipment like a well pump hub never sees continuous flow across zones, and only one zone is ever active at a time.
- **Manual override with a real escape hatch** — start a zone's timer directly to force watering for a set duration (e.g. deep-watering new plants), bypassing both the moisture threshold and rain delay/forecast. Watering runs until the timer finishes, is paused, or is canceled, then everything reverts automatically to normal threshold-based behavior — no settings to remember to reset.
- **Notifications** at every state change (started, stopped, skipped, canceled, sensor unavailable, valve not confirmed) with current moisture readings and reason.
- **One blueprint, many zones** — all per-zone entities the blueprint actually needs (soil sensor, max threshold, valve, timer, rain delay, forecast sensors, notify target) are exposed as blueprint inputs, so each zone's reactive guard is just a new automation instance created from a form in the UI — no copy-pasting or editing YAML per zone.

## Requirements

- A soil moisture sensor per zone
- Two `input_number` helpers per zone (min and max moisture threshold) — min is used by the sequencer, max is used by the blueprint
- A `valve` entity (or adaptable to a switch) controlling each zone's water
- A `timer` helper per zone for manual overrides
- A shared `input_boolean` for rain delay
- (Optional) One or more numeric sensors for rain forecast (chance of rain %, expected precip amount, etc.) — must report a plain number, not a condition string. Weather integrations that only expose a `weather.xxx` entity typically need a small template sensor to extract a numeric value before they'll work here.
- A notify service (e.g. a mobile app notify target)

## Installation

1. Copy `irrigation_zone_controller.yaml` into `config/blueprints/automation/<your_folder>/` in your Home Assistant config, or import it via **Settings → Automations & Scenes → Blueprints → Import Blueprint** using the URL to the raw file itself (e.g. `https://raw.githubusercontent.com/<user>/<repo>/main/irrigation_zone_controller.yaml`, or the GitHub blob URL) — not just the repo's main page.
2. Go to **Settings → Developer Tools → YAML** and click **Reload Automations** to pick up blueprint changes (there's no separate "Reload Blueprints" button in current Home Assistant — reloading automations covers blueprint-based ones too).
3. Create a new automation from the blueprint for each zone and fill in the inputs. When updating an existing zone's inputs after a blueprint change, open that automation once and re-save it so any removed/renamed fields clear out.
4. Edit `irrigation_sequencer.yaml` directly with your actual entity IDs, start times, and zone order (this one isn't a blueprint — it's specific to your setup), then paste it into a new automation's YAML editor (or add the file to your automations and reload).

## License

MIT


