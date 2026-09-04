# Cover-Automation-Shelly
A JavaScript automation script for Shelly devices that automatically closes a motorized awning/cover in response to rain, wind, or gusts — including short-term forecasts — and reopens it to its previous position once the weather stabilizes.

Features

Polls the Open-Meteo API every 10 minutes for current conditions and a 30-minute forecast (precipitation, weather code, wind speed, gusts, temperature).
Closes the cover automatically when rain, strong wind, or gusts are detected now or forecast in the next 15–30 minutes.
Remembers the awning's position before closing and restores it once a configurable number of consecutive "good" readings confirm stable weather.
Detects manual interventions (movements not triggered by the script) and suspends automation for a configurable window, so the user's manual control isn't immediately overridden.
Skips automatic actions below a configurable temperature threshold to protect the motor.
Persists state (manual override timestamp, saved position) via Shelly's KVS, so it survives reboots.
Emits a tenda_alert event and disables/re-enables scheduled jobs accordingly, so downstream Scenes/Automations can react.
Optional Telegram notification support (disabled by default, easy to enable).

Configuration is centralized at the top of the script: location coordinates, polling interval, wind/gust/temperature/rain thresholds, and manual-override duration.
