"""
    Project: BESS Trading Algorithm
    Description: This script simulates a Battery Energy Storage System (BESS) trading algorithm that performs arbitrage based on Day-Ahead electricity prices in Greece. It fetches the prices from the ENTSO-E Transparency Platform, determines optimal charging and discharging times based on price thresholds, and visualizes the results in an interactive dashboard.
    Author: Dimitrios Poulos
"""


import math
import pandas as pd
import plotly.graph_objects as go
from plotly.subplots import make_subplots

from entsoe import EntsoePandasClient
from dotenv import load_dotenv
import os

# ENTSO-E API Connection
# Load the API key from the .env file and connect to the ENTSO-E Transparency Platform
load_dotenv()
API_KEY = os.getenv('ENTSOE_API_KEY')
client = EntsoePandasClient(api_key=API_KEY)



# Battery physical parameters.
MAX_CAPACITY_MWH = 10.0
MAX_POWER_MW = 2.0
ROUND_TRIP_EFFICIENCY = 0.90
INITIAL_SOC_MWH = 0.0
ONE_WAY_EFFICIENCY = math.sqrt(ROUND_TRIP_EFFICIENCY)    # Calculate One-Way Efficiency: Square root of Round-Trip Efficiency.





# Fetch today's 24-hour Day-Ahead prices for Greece from ENTSO-E.
# Prices are published the day before by the market operator.
start = pd.Timestamp('now', tz='Europe/Athens').normalize()
end = start + pd.Timedelta(days=1) 

# Fetch Day-Ahead Prices for Greece
prices_series = client.query_day_ahead_prices(
    country_code='GR',
    start=start,
    end=end
)
prices_series = prices_series.resample('1h').mean()

# Convert to list of 24 hourly prices
prices_eur = prices_series.values.tolist()
hours = list(range(len(prices_eur)))


# Sort prices to find the lowest/highest 30% thresholds
sorted_prices = sorted(prices_eur)
buy_price = sorted_prices[7]    # 8th lowest price.
sell_price = sorted_prices[16]  # 17th highest price.




# Arbitrage Algorithm & Simulation
battery_level_list = []    # Battery List to track the state of charge (SoC) over time.
status_list = []           # Status List to track the battery's operational status (Charging, Discharging, Waiting) over time.
money_list = []            # Money List to track the profit/loss over time.
battery_level = INITIAL_SOC_MWH


# Loop through each hour of the day
for price in prices_eur:
    money = 0.0           # Initialize money for each hour.
    status = 'Waiting'    # Default status is 'Waiting' if no action is taken.

    # Charge Logic: If the price is less than or equal to the buy price and the battery is not full, charge the battery.
    if price <= buy_price and battery_level < MAX_CAPACITY_MWH:
        max_charge_possible = (MAX_CAPACITY_MWH - battery_level) / ONE_WAY_EFFICIENCY    # Calculate the maximum charge possible without exceeding the battery's capacity. We divide by ONE_WAY_EFFICIENCY to account for charging losses in order to fill the remaining capacity perfectly without exceeding it.
        charge_amount = min(MAX_POWER_MW, max_charge_possible)                           # Calculate the actual charge amount based on the maximum power limit and the remaining capacity.
        battery_level += charge_amount * ONE_WAY_EFFICIENCY
        money -= charge_amount * price                                                   # Cost of buying energy
        status = 'Charging'

    # Discharge Logic: If the price is greater than or equal to the sell price and the battery has energy, discharge the battery.
    elif price >= sell_price and battery_level > 0:
        max_discharge_possible = battery_level * ONE_WAY_EFFICIENCY                     # Calculate the maximum discharge possible based on the current battery level.
        discharge_amount = min(MAX_POWER_MW, max_discharge_possible)                    # Calculate the actual discharge amount based on the maximum power limit and the current battery level.
        battery_level -= discharge_amount / ONE_WAY_EFFICIENCY
        money += discharge_amount * price                                               # Revenue from selling energy
        status = 'Discharging'
    
    battery_level_list.append(battery_level)
    status_list.append(status)
    money_list.append(money)




# DataFrame to store the results of the simulation
df = pd.DataFrame({
    'Hour'          : hours,
    'Price'         : prices_eur,
    'Battery_Level' : battery_level_list,
    'Status'        : status_list,
    'Money'         : money_list,
})


# Color mapping for the battery status
COLOR_MAP ={
    'Charging'    : "#00B80F",
    'Discharging' : "#F44336",
    'Waiting'     : "#9E9E9E"
}
status_colors = df['Status'].map(COLOR_MAP).tolist()



# Dashboard Layout
# Row 1: Market prices with buy/sell decision markers.
# Row 2: Battery State of Charge (SoC) over time.
# Row 3: Hourly profit and loss.
fig = make_subplots(
    rows=3, cols=1,
    vertical_spacing=0.25,
    subplot_titles=(
        '📈 Market Price per Hour (€/MWh)',
        '🔋 State of Charge per Hour (MWh)',
        '💰 Profit / Loss per Hour (€)',
    )
)


total_profit = df['Money'].sum()


# Chart 1: Market Price
fig.add_trace(
    go.Scatter(
        x=df['Hour'], y=df['Price'],
        name='Price (€/MWh)',
        line=dict(color='gray')    # Gray line shows the full Day-Ahead price curve.
    ),
    row=1, col=1
)

# Green triangles: Charging (Buying) hours
charge_df = df[df['Status'] == 'Charging']
fig.add_trace(
    go.Scatter(
        x=charge_df['Hour'], y=charge_df['Price'],
        mode='markers',
        marker=dict(color='limegreen', size=10, symbol='triangle-up'),
        name='Charge (Buy)'
    ),
    row=1, col=1
)

# Red triangles: Discharging (Selling) hours
discharge_df = df[df['Status'] == 'Discharging']
fig.add_trace(
    go.Scatter(
        x=discharge_df['Hour'], y=discharge_df['Price'],
        mode='markers',
        marker=dict(color='crimson', size=10, symbol='triangle-down'),
        name='Discharge (Sell)'
    ),
    row=1, col=1
)


# Chart 2: State of Charge 
# Filled area chart showing the battery energy level (MWh) at each hour.
# Rises during charging periods and falls during discharging periods.
fig.add_trace(
    go.Scatter(
        x=df['Hour'], y=df['Battery_Level'],
        name='Battery Level (MWh)',
        fill='tozeroy',
        fillcolor='rgba(65, 105, 225, 0.3)',
        line=dict(color='blue')
    ),
    row=2, col=1
)


# Chart 3: Profit / Loss
# Bar chart showing the hourly profit or loss (€).
profit_colors = ['green' if money > 0 else 'red' if money < 0 else 'gray' for money in df['Money']]
fig.add_trace(
    go.Bar(
        x=df['Hour'], y=df['Money'],
        name='Profit / Loss (€)',
        marker_color=profit_colors,
        showlegend=False
    ),
    row=3, col=1
)
# Manual legend entries for the profit/loss chart.
fig.add_trace(go.Bar(x=[None], y=[None], marker_color='green', name='Revenue (Sell)'), row=3, col=1)
fig.add_trace(go.Bar(x=[None], y=[None], marker_color='red', name='Cost (Buy)'), row=3, col=1)



# Layout and Styling
today = pd.Timestamp('now', tz='Europe/Athens').strftime('%d %B %Y')
fig.update_layout(
    title=dict(
        text=f"Battery Energy Storage System (BESS) Trading Simulation — {today} — Total Profit: €{total_profit:.2f}",
        y=0.98,
        x=0.5,
        xanchor='center',
        yanchor='top'
    ),
    xaxis_title="Hour of the Day",
    yaxis_title="Price (€/MWh)",
    yaxis2_title="Battery Level (MWh)",
    yaxis3_title="Profit / Loss (€)",
    legend=dict(
        orientation="h",
        yanchor="bottom",
        y=1.04,
        xanchor="right",
        x=1
    ),
    height=1000,
    width=1000,
    template='plotly_white'
)

fig.show()