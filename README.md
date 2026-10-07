# Smart Thermostat Savings Calculator

**[Live demo →](https://riko-miyagi.github.io/smart-thermostat-savings-calculator/)**

An interactive calculator estimating how much a smart thermostat could save a Canadian household on heating and cooling, based on where they live, their home, their heating system, and their daily routine — written in plain language, since most people don't know (or care) what a "programmable thermostat" or "AFUE" is.

## Why this exists

Smart thermostats are usually sold on a vague promise — "save energy" — without much grounding in what that actually means for a specific home. The real answer depends on where you live (a Winnipeg winter and a Vancouver winter are not the same heating problem), how big and well-insulated your home is, what you heat with, and how you already manage the temperature day to day. This tool tries to make that estimate concrete and personal rather than a flat marketing percentage, while staying usable by someone with zero HVAC background.

## What it does

- Searchable city input covering roughly 40 cities and towns across every Canadian province and territory, instead of a short fixed list.
- Plain-language questions: home size, insulation level, heating system, how you currently manage the temperature, whether your home sits empty during the day, and how big a temperature swing you'd be comfortable with.
- Estimates baseline annual heating and cooling energy use with the degree-day method, adjusted for home size and insulation.
- Calculates smart-thermostat savings using the U.S. Department of Energy's published temperature-setback rule of thumb, scaled by how many hours a day it would apply, plus a small "automation" bonus that's bigger if you're not currently doing any scheduling yourself.
- Converts energy saved into dollars (local provincial utility rates, or your own real rates if you enter them) and emissions avoided (provincial grid carbon intensity).
- Estimates **payback period** — how long it would take a smart thermostat to pay for itself, based on what you tell it one costs.
- Lets you override the default rates with your own real electricity, gas, or oil price for a more personal estimate.
- Translates total emissions avoided into human-scale terms: equivalent trees-worth of carbon absorption, and kilometres not driven.

## How the model works

**Baseline demand.** A reference case — a 1,500 sq ft Toronto home with average insulation, heated by a gas furnace — is assumed to use about 2,200 m³ of natural gas a year for heating and 1,800 kWh of electricity for cooling. Every other city, home size, and insulation level scales from that reference in proportion to local heating and cooling degree days (HDD/CDD, base 18°C), floor area, and an insulation multiplier (a drafty older home uses about 25% more energy than average; a well-insulated newer one, about 20% less).

**City search.** Roughly 40 cities spanning every province and territory each have their own approximate HDD/CDD. Electricity, gas, and grid-carbon figures are set at the *provincial* level, since utility rates and grid mix are provincial/utility matters, not city-specific ones — see [`data/province_rates.csv`](data/province_rates.csv) and [`data/city_reference_data.csv`](data/city_reference_data.csv). If an exact town isn't listed, the nearest city on the list is a reasonable stand-in, the same approximation real energy-modeling tools make when mapping a location to its nearest reference weather station. Natural gas distribution isn't actually available everywhere — it's far less common in Atlantic Canada and the territories — so those provinces carry a placeholder gas rate, flagged in the data file, in case someone selects "gas furnace" there anyway.

**Heating system conversion.** The reference gas usage is first converted into a "useful heat" energy demand (accounting for a gas furnace's ~92% efficiency), then divided by whichever system's efficiency applies: gas furnace (~92% AFUE), oil furnace (~85% AFUE), electric heaters (~100% efficient but a much higher primary-energy cost), or a cold-climate heat pump (seasonal COP of ~2.8).

**Savings from temperature setbacks.** The core, user-controllable driver of the estimate, based on the U.S. Department of Energy's rule of thumb: roughly 1% off a heating or cooling bill per degree Fahrenheit set back for 8 hours, about 1.8% per degree Celsius. Applied to the setback amount chosen, scaled by how many hours a day it would realistically apply — 8 hours (overnight only) if the home isn't empty during the day, or 17 hours (overnight plus a typical workday) if it is.

**The "smart" bonus.** On top of setback savings, a small additional bonus (5%, 3%, or 1%) reflects that automation catches things people forget — biggest for someone doing no scheduling at all today, smallest for someone who already has a full schedule.

**Payback period.** Thermostat cost (whatever the user enters, defaulting to $220) divided by total estimated annual savings. It's a simple payback calculation — it doesn't account for financing, rebates, or installation complexity.

**Costs and emissions.** By default, dollar savings use approximate, illustrative electricity and natural-gas/oil rates by province (heating oil is flat-priced at ~$1.25/L everywhere, since it's typically sold by local distributors rather than a regulated utility). Anyone who knows their actual rates can enter them directly for a more personal number. Emissions use approximate provincial grid carbon intensities — Quebec, Manitoba, and BC run mostly on hydro and are extremely low-carbon; Alberta and Saskatchewan rely much more heavily on fossil generation — plus standard combustion factors for natural gas (1.88 kg CO2/m³) and heating oil (2.75 kg CO2/L).

| Province/territory | Electricity | Natural gas | Grid intensity |
|---|---|---|---|
| Ontario | $0.13/kWh | $0.38/m³ | 40 gCO2/kWh |
| Quebec | $0.07/kWh | $0.45/m³ | 2 gCO2/kWh |
| British Columbia | $0.11/kWh | $0.40/m³ | 12 gCO2/kWh |
| Alberta | $0.17/kWh | $0.28/m³ | 410 gCO2/kWh |
| Manitoba | $0.09/kWh | $0.30/m³ | 10 gCO2/kWh |
| Saskatchewan | $0.15/kWh | $0.30/m³ | 580 gCO2/kWh |
| Nova Scotia | $0.17/kWh | $0.55/m³ (placeholder) | 390 gCO2/kWh |
| New Brunswick | $0.12/kWh | $0.55/m³ (placeholder) | 300 gCO2/kWh |
| Prince Edward Island | $0.16/kWh | $0.55/m³ (placeholder) | 30 gCO2/kWh |
| Newfoundland and Labrador | $0.12/kWh | $0.55/m³ (placeholder) | 25 gCO2/kWh |
| Yukon | $0.14/kWh | $0.55/m³ (placeholder) | 50 gCO2/kWh |
| Northwest Territories | $0.30/kWh | $0.55/m³ (placeholder) | 180 gCO2/kWh |
| Nunavut | $0.35/kWh | $0.55/m³ (placeholder) | 700 gCO2/kWh |

## A note on precision

This is an educational estimator, not a utility bill. Degree-day values, utility rates, and grid emission factors are approximate figures representative of each city/province, not pulled live from a utility or grid operator. Real savings depend heavily on an individual home's actual insulation, air sealing, and household habits beyond what a handful of inputs can capture. Treat the output as directionally useful for comparing scenarios, not as an exact prediction for one specific house.

## Tech stack

Plain HTML, CSS, and JavaScript — no build step, no framework, no charting library. City search uses a native HTML `<datalist>` rather than a custom dropdown widget, which keeps it fully keyboard- and screen-reader-accessible for free. A standalone project with its own visual identity, separate from [riko-miyagi.github.io](https://riko-miyagi.github.io), which links to it.

## Running it locally

No build process — clone the repo and open `index.html` directly in a browser, or serve the folder with any static file server.

```
git clone https://github.com/riko-miyagi/smart-thermostat-savings-calculator.git
cd smart-thermostat-savings-calculator
open index.html
```

## License

MIT — see [LICENSE](LICENSE).
