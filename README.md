# ===============================================================
# IRevolution Project
# A Data-Driven Exploration of Apple’s iPhone Impact in India
# Complete End-to-End Unified Script (Single File)
# ===============================================================

# -----------------------------
# 1. IMPORT LIBRARIES
# -----------------------------
import pandas as pd
import numpy as np
from datetime import datetime
from sklearn.linear_model import LinearRegression

np.random.seed(42)

# -----------------------------
# 2. SIMULATE DATASET (If no CSV provided)
# -----------------------------
years = list(range(2015, 2025))
states = ["Maharashtra","Delhi","Karnataka","Tamil Nadu","Gujarat",
          "Uttar Pradesh","Telangana","West Bengal"]
models = ["iPhone 8","iPhone X","iPhone XR","iPhone 11",
          "iPhone 12","iPhone 13","iPhone 14","iPhone 15"]
channels = ["Online","Offline"]
city_tiers = ["Tier 1","Tier 2","Tier 3"]

data = []

for year in years:
    for state in states:
        for model in models:
            units = np.random.randint(5000, 50000)
            price = np.random.randint(40000, 120000)
            total_market = np.random.randint(200000, 500000)
            production = np.random.randint(100000, 400000)
            exports = np.random.randint(50000, production)
            employment = np.random.randint(2000, 10000)

            data.append([
                year,
                state,
                np.random.choice(city_tiers),
                model,
                units,
                price,
                np.random.choice(channels),
                production,
                exports,
                employment,
                total_market
            ])

columns = [
    "Year","State","City_Tier","Model","Units_Sold","Price",
    "Sales_Channel","Production_Units","Export_Units",
    "Employment","Total_Market_Units"
]

df = pd.DataFrame(data, columns=columns)

# -----------------------------
# 3. DATA CLEANING
# -----------------------------
df.dropna(inplace=True)
df["Revenue"] = df["Units_Sold"] * df["Price"]

# -----------------------------
# 4. KPI CALCULATIONS
# -----------------------------
total_units = df["Units_Sold"].sum()
total_revenue = df["Revenue"].sum()

yearly = df.groupby("Year").agg({
    "Units_Sold":"sum",
    "Revenue":"sum",
    "Production_Units":"sum",
    "Export_Units":"sum",
    "Employment":"sum",
    "Total_Market_Units":"sum"
}).reset_index()

yearly["YoY_Growth_%"] = yearly["Units_Sold"].pct_change()*100
yearly["Market_Share_%"] = (
    yearly["Units_Sold"] / yearly["Total_Market_Units"]
) * 100
yearly["Export_Ratio_%"] = (
    yearly["Export_Units"] / yearly["Production_Units"]
) * 100

# -----------------------------
# 5. MODEL PERFORMANCE
# -----------------------------
model_perf = df.groupby("Model").agg({
    "Units_Sold":"sum",
    "Revenue":"sum"
}).sort_values("Revenue", ascending=False).reset_index()

# -----------------------------
# 6. STATE ANALYSIS (MAP READY)
# -----------------------------
state_perf = df.groupby("State").agg({
    "Units_Sold":"sum",
    "Revenue":"sum"
}).reset_index()

# -----------------------------
# 7. CHANNEL ANALYSIS
# -----------------------------
channel_perf = df.groupby("Sales_Channel").agg({
    "Units_Sold":"sum",
    "Revenue":"sum"
}).reset_index()

# -----------------------------
# 8. PRICE SEGMENTATION
# -----------------------------
def segment(price):
    if price < 50000:
        return "Mid Range"
    elif price < 90000:
        return "Premium"
    else:
        return "Ultra Premium"

df["Price_Segment"] = df["Price"].apply(segment)

segment_perf = df.groupby("Price_Segment").agg({
    "Units_Sold":"sum",
    "Revenue":"sum"
}).reset_index()

# -----------------------------
# 9. CITY TIER ANALYSIS
# -----------------------------
tier_perf = df.groupby("City_Tier").agg({
    "Units_Sold":"sum",
    "Revenue":"sum"
}).reset_index()

# -----------------------------
# 10. FORECASTING (Linear Regression)
# -----------------------------
X = yearly[["Year"]]
y = yearly["Units_Sold"]

model = LinearRegression()
model.fit(X, y)

future_years = pd.DataFrame({"Year":[2025,2026,2027]})
future_years["Forecast_Units"] = model.predict(future_years)

# -----------------------------
# 11. EXPORT ALL TABLEAU FILES
# -----------------------------
df.to_csv("IRevolution_Master_Data.csv", index=False)
yearly.to_csv("IRevolution_Yearly_KPI.csv", index=False)
model_perf.to_csv("IRevolution_Model_Performance.csv", index=False)
state_perf.to_csv("IRevolution_State_Performance.csv", index=False)
channel_perf.to_csv("IRevolution_Channel_Performance.csv", index=False)
segment_perf.to_csv("IRevolution_Segment_Performance.csv", index=False)
tier_perf.to_csv("IRevolution_Tier_Performance.csv", index=False)
future_years.to_csv("IRevolution_Forecast.csv", index=False)

# -----------------------------
# 12. SQL TABLE STRUCTURE
# -----------------------------
sql_schema = """
CREATE TABLE iphone_india_sales (
    Year INT,
    State VARCHAR(50),
    City_Tier VARCHAR(20),
    Model VARCHAR(50),
    Units_Sold INT,
    Price FLOAT,
    Sales_Channel VARCHAR(20),
    Production_Units INT,
    Export_Units INT,
    Employment INT,
    Total_Market_Units INT
);
"""

# -----------------------------
# 13. TABLEAU CALCULATED FIELDS
# -----------------------------
tableau_calculations = """
YoY Growth % =
(SUM([Units_Sold]) - LOOKUP(SUM([Units_Sold]), -1))
/ LOOKUP(SUM([Units_Sold]), -1)

Market Share % =
SUM([Units_Sold]) / SUM([Total_Market_Units])

Revenue =
SUM([Units_Sold] * [Price])

Export Ratio % =
SUM([Export_Units]) / SUM([Production_Units])
"""

# -----------------------------
# 14. DASHBOARD STRUCTURE GUIDE
# -----------------------------
dashboard_guide = """
Dashboard 1: Market Growth
- Line: Year vs Units
