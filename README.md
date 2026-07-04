# Irrigation Zone Controller (Home Assistant Blueprint)

A Home Assistant automation blueprint that manages watering for a single irrigation zone using calendar schedules, soil moisture thresholds, manual overrides, and a shared rain delay switch.

## Features

- **Calendar-driven scheduling** — starts and stops watering based on a Home Assistant calendar entity, so you set your schedule the same way you'd set any other event.
- **Soil moisture aware** — skips or stops scheduled watering automatically if the soil is already above your min/max thresholds, and resumes watering mid-schedule if moisture drops.
- **Rain delay support** — a shared `input_boolean` can pause all zones at once when it's raining.
- **Manual override with a real escape hatch** — start a timer to force watering for a set duration (e.g. deep-watering new plants), bypassing both the moisture thresholds and rain delay. Watering runs until the timer finishes, is paused, or is canceled, then everything reverts automatically to normal threshold-based behavior — no settings to remember to reset.
- **Notifications** at every state change (started, stopped, skipped, canceled) with current moisture readings.
- **One blueprint, many zones** — all entities (soil sensor, thresholds, valve, calendar, timer, rain delay, notify target) are exposed as blueprint inputs, so each zone is just a new automation instance created from a form in the UI — no copy-pasting or editing YAML per zone.

## Requirements

- A soil moisture sensor per zone
- Two `input_number` helpers per zone (min/max moisture threshold)
- A `valve` entity (or adaptable to a switch) controlling the zone's water
- A `calendar` entity defining the watering schedule
- A `timer` helper per zone for manual overrides
- A shared `input_boolean` for rain delay
- A notify service (e.g. a mobile app notify target)

## Installation

1. Copy `irrigation_zone_controller.yaml` into `config/blueprints/automation/<your_folder>/` in your Home Assistant config, or import it via **Settings → Automations & Scenes → Blueprints → Import Blueprint** using a raw GitHub URL to this file.
2. Reload blueprints (**Blueprints → ⋮ → Reload Blueprints**) or restart Home Assistant.
3. Create a new automation from the blueprint and fill in the inputs for your zone.
4. Repeat step 3 for each additional zone.

## License

MIT
