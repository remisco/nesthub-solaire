# Nest Hub Solaire — Home Assistant solar dashboard for Google Nest Hub

**English** · [Français](README.fr.md)

[![Open in HACS](https://my.home-assistant.io/badges/hacs_repository.svg)](https://my.home-assistant.io/redirect/hacs_repository/?owner=remisco&repository=nesthub-solaire&category=plugin)
[![HACS Custom](https://img.shields.io/badge/HACS-Custom-41BDF5.svg)](https://hacs.xyz/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

A full-screen, animated **Home Assistant Lovelace card** designed for the **Google Nest Hub** (1024 × 600). It shows **solar production**, **home power consumption**, **hot water** left in the tank, **showers remaining**, **EDF Tempo** colours and the **live weather**, drawn over an illustrated background animated in HTML/SVG.

Install it in one click with **HACS**, then cast it permanently to your Nest Hub with a ready-made automation.

![Home Assistant solar dashboard on a Google Nest Hub: solar production, consumption, hot water, EDF Tempo and weather](docs/apercu.png)

*Preview with sample values. The card's labels are in French.*

## Features

| Area | Values | Animation |
|---|---|---|
| Solar production | Live power, energy produced today | Light sweep and sparkles on the panels (speed follows production), dimmed at night, inverter LED, energy flow to the house |
| Consumption | Power, "100 % solar · surplus X W" or "Grid: X W" | Rotating light arcs (speed follows consumption), green when self-consuming, orange on grid power |
| Hot water | Litres of 38 °C water available, tank temperature | Rippling water level coloured by temperature, bubbles, thermometer glow |
| Showers remaining | Number of showers | One falling drop per remaining shower |
| EDF Tempo today / tomorrow | Colour and tariff | Text and calendar icon in the day's colour, with a glow |
| Daily production peak | Peak value and time (read from history) | Radiating sun, flickering lightning bolt, wave on the curve's summit |
| Weather | Current conditions and temperature | Sun, moon and stars, drifting clouds, rain, downpour, snow, thunderstorm, fog, wind |
| EDF Tempo days left | Blue / White / Red | — |

Everything is computed inside the card: **no template sensors, no other custom cards and no images to copy**. It works with any solar inverter integrated in Home Assistant (Enphase Envoy, SolarEdge, Fronius, Huawei, Shelly EM…).

## Installation with HACS (recommended)

1. Click the **Open in HACS** button above, or in HACS go to **⋮ → Custom repositories** and add `https://github.com/remisco/nesthub-solaire` with the **Dashboard** category.
2. Click **Download**, then reload your browser.

HACS adds the `/hacsfiles/nesthub-solaire/nesthub-solaire-card.js` resource automatically.

### Manual installation

Copy [`dist/nesthub-solaire-card.js`](dist/nesthub-solaire-card.js) to `config/www/`, then add the resource `/local/nesthub-solaire-card.js` (type *JavaScript module*) under **Settings → Dashboards → ⋮ → Resources**.

## Configuration

Create a view of type **Panel** (a single full-screen card) with the path `solaire`, and add the card:

```yaml
views:
  - title: Solaire
    path: solaire
    type: panel
    cards:
      - type: custom:nesthub-solaire-card
        production: sensor.solar_power
        consommation: sensor.house_power
        energie_jour: sensor.solar_energy_today
        import_reseau: sensor.grid_import_power
        temperature_ballon: sensor.water_heater_temperature
        meteo: weather.forecast_home
```

Replace the entities with your own. Option names are in French.

### Options

| Option | Required | Default | Description |
|---|---|---|---|
| `production` | yes | — | Power produced by the solar panels (W or kW) |
| `consommation` | yes | — | Power consumed by the house (W or kW) |
| `energie_jour` | no | — | Energy produced today |
| `import_reseau` | no | consumption − production | Power drawn from the grid |
| `seuil_import` | no | `50` | Below this imported power (W), the house is shown as "100 % solar" |
| `temperature_ballon` | no | — | Hot water tank temperature (°C) |
| `eau_chaude` | no | computed | Your own hot water litres sensor, if you already have one |
| `volume_ballon` | no | `300` | Tank capacity (L) |
| `temperature_douche` | no | `38` | Shower temperature (°C) |
| `temperature_eau_froide` | no | `12` | Mains cold water temperature (°C) |
| `litres_par_douche` | no | `60` | Litres of 38 °C water per shower (about 8 minutes with a standard shower head) |
| `meteo` | no | `weather.forecast_home` | Weather entity |
| `soleil` | no | `sun.sun` | Used to show the moon at night |
| `tempo_aujourdhui` | no | `sensor.rte_tempo_couleur_actuelle` | Today's Tempo colour |
| `tempo_demain` | no | `sensor.rte_tempo_prochaine_couleur` | Tomorrow's Tempo colour |
| `jours_bleu`, `jours_blanc`, `jours_rouge` | no | `sensor.rte_tempo_cycle_jours_restants_*` | Tempo days left |

The default Tempo entities come from the [RTE Tempo](https://github.com/hekmon/rtetempo) integration (HACS). Tempo is a French electricity tariff; outside France, just leave these entities out.

**Hot water calculation**: litres at 38 °C = `volume × (T_tank − 38) / (38 − T_cold_water)`, then showers = litres ÷ 60, rounded down. This is mixed water (tank water blended with cold water): at 70 °C, a 300 L tank delivers about 370 L at 38 °C, i.e. 6 showers.

## Showing it on the Nest Hub

### Requirements

- The Nest Hub must be added to Home Assistant through the **Google Cast** integration.
- Home Assistant must be reachable over **HTTPS with a valid certificate**: the Nest Hub refuses plain HTTP pages. Either:
  - Home Assistant Cloud (Nabu Casa);
  - the **Duck DNS** add-on with Let's Encrypt (free), then set the `https://…` URL in *Settings → System → Network* (both internal **and** external URL).

### Always-on display

Create an automation, switch to YAML mode and paste [`automations/nesthub_affichage_permanent.yaml`](automations/nesthub_affichage_permanent.yaml), replacing `media_player.nest_hub` with your display.

Every minute, it casts the view again if the screen is idle (photo frame, clock or off). It never interrupts a video or another app.

To try it right away: *Developer tools → Actions → `cast.show_lovelace_view`* with your display, `dashboard_path: lovelace` and `view_path: solaire`.

## Good to know

- The Nest Hub does not pass touch input to the Home Assistant cast: the card is made to be looked at.
- The Nest Hub stops the cast after about ten minutes; the automation restarts it within a minute.
- The card keeps the Nest Hub's 1024 × 600 layout and scales to the available width on other screens.
- To reach Home Assistant from outside without opening a port, the **Tailscale** add-on works very well alongside it.

## YAML version without a custom card

The [`installation-manuelle/`](installation-manuelle/) folder contains the earlier all-YAML version (a picture-elements view with button-card and card-mod, template sensors, theme and background image), for those who prefer to edit everything by hand.

## License

[MIT](LICENSE)
