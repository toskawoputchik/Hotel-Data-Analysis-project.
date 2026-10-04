# Hotel-Data-Analysis-project.
Python Analysis Script (hotel_analysis.py) to generate sample data, perform the required metrics, and save visualization charts.  GitHub README.md template structured precisely as requested.
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import seaborn as.sns as sns  # Optional, standard matplotlib works fine too

# Set random seed for reproducibility
np.random.seed(42)

# 1. Generate Sample Dataset
dates = pd.date_range(start="2025-01-01", end="2025-12-31", freq="D")
n_days = len(dates)

rooms_available = np.full(n_days, 150)  # Assuming a 150-room hotel
rooms_sold = np.random.randint(80, 145, size=n_days)
adr = np.random.uniform(100, 250, size=n_days).round(2)
revenue = (rooms_sold * adr).round(2)

df = pd.DataFrame(
    {
        "Date": dates,
        "Rooms Available": rooms_available,
        "Rooms Sold": rooms_sold,
        "ADR": adr,
        "Revenue": revenue,
    }
)

# 2. Perform Analysis & Feature Engineering
df["Occupancy Rate"] = (df["Rooms Sold"] / df["Rooms Available"]) * 100
df["Month"] = df["Date"].dt.to_period("M")
df["Day of Week"] = df["Date"].dt.day_name()

# Aggregate Monthly Performance
monthly_perf = (
    df.groupby("Month")
    .agg(
        {
            "Revenue": "sum",
            "Rooms Sold": "sum",
            "Rooms Available": "sum",
            "ADR": "mean",
        }
    )
    .reset_index()
)
monthly_perf["Monthly Occupancy (%)"] = (
    monthly_perf["Rooms Sold"] / monthly_perf["Rooms Available"]
) * 100
monthly_perf["Month"] = monthly_perf["Month"].astype(str)

# Aggregate Weekly Trends (Day of Week)
dow_order = [
    "Monday",
    "Tuesday",
    "Wednesday",
    "Thursday",
    "Friday",
    "Saturday",
    "Sunday",
]
weekly_trends = (
    df.groupby("Day of Week")
    .agg({"Revenue": "mean", "Occupancy Rate": "mean"})
    .reindex(dow_order)
    .reset_index()
)

print("--- Data Sample ---")
print(df.head())
print("\n--- Monthly Performance Summary ---")
print(monthly_perf)

# 3. Generate and Save Charts
plt.style.use("seaborn-v0_8-whitegrid" if "seaborn-v0_8-whitegrid" in plt.style.available else "default")

# Chart 1: Monthly Revenue Trend
plt.figure(figsize=(10, 5))
plt.plot(
    monthly_perf["Month"],
    monthly_perf["Revenue"],
    marker="o",
    color="b",
    linewidth=2,
)
plt.title("Monthly Revenue Performance (2025)", fontsize=14, fontweight="bold")
plt.xlabel("Month", fontsize=12)
plt.ylabel("Revenue ($)", fontsize=12)
plt.xticks(rotation=45)
plt.tight_layout()
plt.savefig("monthly_revenue.png")
plt.close()

# Chart 2: Day of the Week Occupancy Trend
plt.figure(figsize=(10, 5))
plt.bar(
    weekly_trends["Day of Week"],
    weekly_trends["Occupancy Rate"],
    color="teal",
    alpha=0.8,
)
plt.title(
    "Average Occupancy Rate by Day of the Week", fontsize=14, fontweight="bold"
)
plt.xlabel("Day of Week", fontsize=12)
plt.ylabel("Occupancy Rate (%)", fontsize=12)
plt.ylim(0, 100)
plt.tight_layout()
plt.savefig("weekly_occupancy_trends.png")
plt.close()

print(
    "\nAnalysis complete! Charts 'monthly_revenue.png' and 'weekly_occupancy_trends.png' saved successfully."
)
