# Shelly Automatic Awning Controller — Weather-Based (Open-Meteo)

A JavaScript automation script for Shelly devices that automatically closes a motorized awning/cover in response to rain, wind, or gusts — including short-term forecasts — and reopens it to its previous position once the weather stabilizes.

## Features

- Polls the [Open-Meteo](https://open-meteo.com/) API every 10 minutes for current conditions and a 30-minute forecast (precipitation, weather code, wind speed, gusts, temperature).
- Closes the cover automatically when rain, strong wind, or gusts are detected now **or** forecast in the next 15–30 minutes.
- Remembers the awning's position before closing and restores it once a configurable number of consecutive "good" readings confirm stable weather.
- Detects manual interventions (movements not triggered by the script) and suspends automation for a configurable window, so the user's manual control isn't immediately overridden.
- Skips automatic actions below a configurable temperature threshold to protect the motor.
- Persists state (manual override timestamp, saved position) via Shelly's KVS, so it survives reboots.
- Emits a `tenda_alert` event and disables/re-enables scheduled jobs accordingly, so downstream Scenes/Automations can react.
- Optional Telegram notification support (disabled by default, easy to enable).

## Configuration

All tunable parameters are centralized at the top of the script: location coordinates, polling interval, wind/gust/temperature/rain thresholds, and manual-override duration.

## Requirements

- A Shelly 2PM (or Plus/Gen3/Gen4 2PM) configured in **Cover** mode, controlling a bidirectional awning motor.
- Firmware with scripting support and the `Cover`, `Schedule`, `KVS`, and `HTTP` component APIs.
- An internet-connected Wi-Fi network (the device calls the Open-Meteo API directly).

## Wiring the Shelly 2PM

The Shelly 2PM should be wired in **Cover (Roller) mode**, which supports three operating modes: detached, single input, or dual input. In detached mode, the device can only be controlled through its WebUI/App; even if buttons or switches are connected, they cannot control the motor directly. This is the recommended mode for this script, since automation is fully API-driven and detached mode avoids interference from physical buttons that the script would otherwise need to distinguish from its own commands via the `source` field.

**Detached mode wiring:**

| Terminal | Connect to |
|---|---|
| L1, L2 | Live (Phase) |
| N | Neutral |
| Motor common wire | Neutral |
| O1 | Motor direction wire 1 (e.g. open) |
| O2 | Motor direction wire 2 (e.g. close) |

If you'd rather keep a physical control button available (useful for clean manual overrides, since the script already detects a `source` other than `script`/`SHOS`/`timer`), you can use single-input or dual-input mode instead, wiring the switch(es) to S1 (and S2).

⚠️ **Before wiring anything:** switch off the circuit breaker and verify there is no voltage present using a multimeter/phase tester.

## RC Snubbers

An RC snubber limits voltage transients, consisting of a resistor (R) and a capacitor (C) in series. It's intended for Shelly Plus 1/1PM and Shelly 2PM devices, which can restart when switching inductive loads such as fans, motors, or roller shutters; the snubber mitigates this by providing a short-term alternative current path so the inductive load discharges safely. It should be wired between the Output and Neutral terminals, in parallel with the inductive load being switched.

For a bidirectional motor like an awning, Shelly recommends **two RC snubbers**, one per output:

- **RC #1**: in parallel between **O1** and **N**
- **RC #2**: in parallel between **O2** and **N**

In practice, community feedback is mixed: some users always install them as per the manual, others report no issues without them on small/modern motors — it depends on the motor type (older/classic inductive motors benefit more). If your Shelly resets or briefly blacks out when the motor starts/stops, that's a sign you need them. Since this script tracks state (manual override, saved position) via KVS, an unexpected device reset mid-closure could leave the awning partway or cause state to be lost — so installing the snubbers preventively is recommended.

## Setting up the schedules this script suspends/re-enables

This script does not create schedules itself — it acts on whichever schedules already exist in the Shelly app, enabling/disabling them via `Schedule.List` + `Schedule.Update`. To set this up:

1. In the Shelly app (or the device's WebUI), go to **Settings → Schedules** (the clock icon on the device page).
2. Create the desired time-based schedules for the awning, e.g.:
   - Auto-open at 8:00 AM
   - Auto-close at 8:00 PM
   - (any others, e.g. midday closing in summer)
3. **No special naming is required**: the script iterates over *all* existing schedules (`result.jobs`) and disables them (`enable:false`) when it closes the awning for bad weather, then re-enables them (`enable:true`) once the weather is confirmed stable again — so your daily routines won't conflict with the emergency closure in the meantime.
4. If you later want to exclude specific schedules from this logic (e.g. a safety schedule that should always stay active), the script would need a filter by schedule `id` — this isn't implemented yet.

## License

_Add your preferred license here (e.g. MIT)._
