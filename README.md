# **BESS-Trading-Algorithm**

This script simulates a Battery Energy Storage System (BESS) trading algorithm that performs arbitrage based on Day-Ahead electricity prices in Greece. It fetches the prices from the ENTSO-E Transparency Platform, determines optimal charging and discharging times based on price thresholds, and visualizes the results in an interactive dashboard.


-----
-----


## Key Features & Functionality
* **Live Price Fetching:** Fetches today's 24-hour Day-Ahead electricity prices for Greece directly from the ENTSO-E Transparency Platform.
* **Arbitrage Strategy:** Charges the battery during the cheapest hours of the day and discharges during the most expensive, maximizing the daily profit.
* **Efficiency Modeling:** Accounts for real-world charging and discharging losses using the one-way efficiency factor (√0.90 ≈ 94.87%).
* **Interactive Dashboard:** Displays the full 24-hour simulation in a three-panel Plotly chart, rendered directly in the browser.


-----


## Dashboard
- 📈 **Market Price per Hour** — Full Day-Ahead price curve with green upward triangles (buy) and red downward triangles (sell) marking each decision
- 🔋 **State of Charge per Hour** — Filled area chart showing the battery energy level (MWh) rising during charging and falling during discharging
- 💰 **Profit / Loss per Hour** — Bar chart showing the net cash flow (€) for each hour: green for revenue, red for cost


-----


## Battery Parameters
| Parameter | Value |
|---|---|
| Max Capacity | 10 MWh |
| Max Power | 2 MW |
| Round-Trip Efficiency | 90% |
| One-Way Efficiency | ≈ 94.87% |
| Initial State of Charge | 0 MWh |


-----


## Mathematical Logic
### 1) Buy / Sell Thresholds
All 24 hourly prices are sorted in ascending order. The battery charges when the price falls within the cheapest 30% and discharges when it falls within the most expensive 30%.

$$Price_{\text{buy}} = \text{sorted}(Price)_{[7]} \quad \text{(8th lowest price)}$$

$$Price_{\text{sell}} = \text{sorted}(Price)_{[16]} \quad \text{(17th lowest price)}$$

### 2) Charging Logic
To fill the battery to its maximum capacity without exceeding it, the energy drawn from the grid accounts for charging losses:

$$Energy_{\text{grid}} = \min\left(Power_{\text{max}},\ \frac{Capacity_{\text{max}} - Capacity_{\text{current}}}{\eta}\right)$$

$$Capacity_{\text{new}} = Capacity_{\text{current}} + Energy_{\text{grid}} \cdot \eta$$

$$\text{Cost} = - Energy_{\text{grid}} \cdot Price_{\text{hour}}$$

### 3) Discharging Logic
The energy delivered to the grid is limited by both the power rating and the available stored energy after losses:

$$Energy_{\text{grid}} = \min\left(Power_{\text{max}},\ Capacity_{\text{current}} \cdot \eta\right)$$

$$Capacity_{\text{new}} = Capacity_{\text{current}} - \frac{Energy_{\text{grid}}}{\eta}$$

$$\text{Revenue} = + Energy_{\text{grid}} \cdot Price_{\text{hour}}$$


-----


## Data Source
* **ENTSO-E Transparency Platform:** The sole data source of the application. Day-Ahead Market wholesale prices are retrieved for the Greek bidding zone using Greece's official country code (`GR`). Data is scoped to the current calendar day (00:00–24:00 Europe/Athens). To use this API, a free account and personal API token are required. Registration is available at [transparency.entsoe.eu](https://transparency.entsoe.eu).


-----


## Setup
### Requirements
- Python 3.8+
- Active ENTSO-E API token

### Installation
```bash
pip install pandas plotly entsoe-py python-dotenv
```

### API Key Configuration
Create a `.env` file in the project folder:
```
ENTSOE_API_KEY=your-api-key-here
```
### Run
```bash
python BESS_trading.py
```


-----


## Libraries & Frameworks

| Package | Purpose |
|---|---|
| `pandas` | Data processing and resampling |
| `plotly` | Interactive dashboard and charts |
| `entsoe-py` | ENTSO-E API client |
| `python-dotenv` | Secure API key management |

-----

## Technical Limitations

* **Simplified Strategy:** The rule-based threshold strategy does not guarantee maximum profit. An optimal solution would require Linear Programming (LP) or dynamic optimization techniques.
* **Single-Day Scope:** The simulation covers one calendar day only. Multi-day backtesting is not currently supported.

-----

## License
MIT License: Free to use, modify, and distribute with attribution.

-----
-----

## **Developer**
**Dimitrios Poulos** <br>
*Electrical & Computer Engineering Student*
* **Release Date:** June 2026
