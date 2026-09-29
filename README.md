# TEG Loop Simulator

An interactive 3D simulator for recovering data-centre waste heat with thermoelectric generators (TEGs). It models the whole loop:

**Data hall → shell-and-tube heat exchanger → TEG array → DC-DC converter → battery → back to the IT power bus**

Built for the CHE 383 / 482 / 483 Group 11 capstone at the University of Waterloo.

It is a single self-contained `index.html` with no build step. Open it in a browser, or serve it with GitHub Pages.

## Running it

- **Locally:** download `index.html` and open it in Chrome, Edge, Firefox or Safari. It needs an internet connection the first time, to load three.js and the fonts from a CDN.
- **GitHub Pages:** Settings → Pages → *Deploy from a branch* → `main` / `(root)`. The simulator is then served at `https://<user>.github.io/<repo>/`. Pages on a private repository needs a paid GitHub plan. On a free account, make the repo public first.

## What you can change

| Area | Parameters |
|---|---|
| Data hall | Rated IT load, rack density, utilisation, daily and weekend load profile, cooling solution (air/CRAH, rear-door HX, direct-to-chip W32/W45, single-phase immersion), coolant, flow rate, IT supply limit, loop pump head and efficiency, PUE |
| Heat exchanger | Hot fluid on tube or shell side, construction material, 1–4 shells in series with a separate flow arrangement for each shell (counterflow, parallel, TEMA E, crossflow), tube count, passes, OD, wall thickness, length, square or triangular pitch, pitch ratio, baffle spacing, shell wall, cooling-tower or chiller facility water, fouling resistances, optional manual film coefficients, allowable pressure drop |
| TEG | Location (A: rack exhaust air · B: coolant bypass stack · C: shell exterior, channel head, or wrapped return pipe), coverage, heat sink type and resistance, material presets or custom Seebeck coefficient / resistivity / thermal conductivity, output decay per year, couples, leg geometry, contact resistance, substrate, thermal interface material, module size, MPPT or fixed load ratio, series string length, converter efficiency and standby loss |
| Battery | LFP, NMC, LTO, sodium-ion, VRLA, or vanadium redox flow (electrolyte volume sets the refuel capacity), cells in series and parallel, cell Ah, charge rate, discharge power, SOC window, round-trip and bus efficiency, cycle and calendar life, self-discharge, dispatch strategy (pass-through, store-and-trickle, time-of-use, backup reserve) |
| Economics | Flat or Ontario-style time-of-use tariff, grid emissions factor, module / install / sink / power-electronics / battery / balance-of-system costs, maintenance, discount rate, horizon |

Four presets are included: the proposal baseline (Solution C), Solution A, Solution B, and a warm-water W45 case with a chilled-water sink.

## What it reports

- **Design constraints** from the proposal, checked at the worst case (peak load, warmest facility water):
  - IT supply temperature limit
  - Facility water class
  - Coolant temperature change caused by the TEGs ≤ 1 °C
  - Added pressure drop ≤ 5 % of circuit head
  - Pump power increase < 2 %
  - Tube and shell velocity windows
  - Hot-junction material limit
  - Leg packing in the module footprint
  - Service-access coverage
  - Recovery ≥ 100 W per 25 kW rack
  - Net output after fans and pumps
  - Payback ≤ 10 years
  - Battery charge acceptance
- **Energy chain** each hour, from IT load down to the energy delivered to the bus.
- **Temperature-drop breakdown** across the fluid film, wall, interfaces, TEG legs and heat sink.
- **Per-shell table:** tube and shell Reynolds numbers with flow regime, film coefficients, U, NTU, ε, duty and temperatures.
- **7-day hourly trends:** TEG output, battery state of charge, and coolant supply and return temperatures.
- **Lifetime projection:** annual energy delivered, cumulative cash, payback, NPV, levelised cost and battery replacements.

## Model

Each hour is solved as a steady state. The coolant return temperature is found so that the heat exchanger plus the TEGs reject exactly the heat the IT load puts into the liquid loop.

- **Tube side:** Gnielinski correlation (turbulent), Sieder–Tate (laminar), blended through the transition region; Petukhov friction factor.
- **Shell side:** Kern's method. Bundle diameter comes from standard tube-count constants.
- **Shells:** ε-NTU per shell, with temperature-dependent viscosity, so the flow regime can change from shell to shell. Multi-shell units are counter-current between shells.
- **TEG:** 1-D module model with Peltier, Fourier and Joule terms. Electrical contact resistance, substrates, interface material and heat-sink resistance sit in series, and the junction temperatures are solved iteratively. Thermoelectric properties are held at roughly 50 °C values.
- **Battery:** round-trip efficiency is split between charge and discharge. Capacity fades with both equivalent full cycles and calendar age, and the pack is replaced at 70 % state of health.

## Sources and caveats

- Material presets for Type I, Type II and the commercial Bi₂Te₃ baseline follow Nozariasbmarz et al., *iScience* 23, 101340 (2020).
- Liquid-cooling classes (W32, W45) follow ASHRAE TC 9.9.
- Costs, battery data and tariff values are editable estimates, not quotes.
- The default grid emissions factor is the national figure used in the proposal. Ontario's grid is much cleaner, so lower it for an Ontario site.
- This is a screening model for comparing design options. It is not a substitute for a rated heat-exchanger design (for example Aspen EDR or HTRI).
