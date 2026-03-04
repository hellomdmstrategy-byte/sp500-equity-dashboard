# S&P 500 Mega-Cap Equity Dashboard

**A production-grade, interactive financial analytics dashboard** built with vanilla HTML, CSS, and JavaScript — no frameworks or paid APIs required. Visualizes 16 years of real S&P 500 stock data for the top 8 mega-cap technology equities.

> Built as a marketing analytics portfolio piece demonstrating data visualization, financial literacy, and front-end development skills.

---

## 📊 Live Preview

![Dashboard Preview](preview.png)

> Dark-themed, Power BI / Tableau-inspired layout with animated charts, interactive stock selector, correlation matrix, and a full watchlist table.

---

## 🗂 Dataset

**File:** `sp500_top10_stocks_clean.csv`  
**Source:** Public market data (Yahoo Finance / open financial data APIs)  
**Records:** 35,765 rows  
**Date Range:** January 4, 2010 → February 13, 2026  
**Tickers:** `AAPL`, `AMZN`, `AVGO`, `GOOG`, `META`, `MSFT`, `NVDA`, `TSLA`

### Schema

| Column | Type | Description |
|---|---|---|
| `Date` | string (YYYY-MM-DD) | Trading date |
| `Ticker` | string | Stock symbol |
| `Open` | float | Opening price |
| `High` | float | Daily high |
| `Low` | float | Daily low |
| `Close` | float | Closing price |
| `Adj_Close` | float | Adjusted closing price |
| `Volume` | float | Shares traded |

---

## 📈 Dashboard Features

### 1. Live Ticker Bar
Scrolling marquee showing all 8 tickers with real closing prices and daily % change. Click any ticker to toggle it on/off in the price chart.

### 2. Normalized Price Performance Chart
- Multi-line time series normalized to base 100 at the start of the selected period
- Allows apples-to-apples % return comparison across stocks with very different price levels (e.g., META at $639 vs. NVDA at $182)
- Toggle: **6-month, 1-year, or 2-year** window
- Hover tooltips showing normalized value + raw price per stock
- Gradient area fills per stock for visual depth

### 3. 1-Year Return Rankings
- Horizontal bar chart sorted by 1-year return (best to worst)
- Color-coded: teal for positive, red for negative returns
- 52-week range bands with current price marker for top 5 stocks

### 4. 30-Day Average Volume Chart
- Gradient vertical bar chart showing average daily trading volume
- Reveals liquidity profile — NVDA at 171M shares/day vs. META at 17M

### 5. Pearson Correlation Matrix
- Computed from 24 months of monthly return data
- Heat map from red (negative correlation) → teal (strong positive)
- Hover any cell for exact r value
- Reveals which stocks move together vs. independently

### 6. Watchlist Table
- Full stats table: price, day change, 1Y return, 5Y return, avg volume
- Mini sparkline bar chart per stock (24-month trend)
- Sector classification badges

---

## 🛠 Tech Stack

| Layer | Technology |
|---|---|
| Markup | HTML5 |
| Styling | CSS3 (custom properties, CSS Grid, animations) |
| Charts | Vanilla JavaScript + inline SVG (no Chart.js, no D3) |
| Fonts | Google Fonts — Syne (display) + JetBrains Mono (data) |
| Data | Hardcoded from CSV pre-processing (see below) |

> **No build tools. No npm. No frameworks.** Open `index.html` in any browser.

---

## 🚀 How to Run

### Option 1 — Just open it
```bash
# Clone the repo
git clone https://github.com/YOUR_USERNAME/sp500-equity-dashboard.git
cd sp500-equity-dashboard

# Open in browser (no server needed)
open index.html          # macOS
start index.html         # Windows
xdg-open index.html      # Linux
```

### Option 2 — Local server (recommended for development)
```bash
# Python 3
python3 -m http.server 8080

# Then visit: http://localhost:8080
```

---

## 🔄 Updating the Data

The dashboard uses pre-computed statistics hardcoded in the `STOCKS` object in `index.html`. To refresh with new data from the CSV:

### Step 1 — Install dependencies
```bash
pip install pandas numpy
```

### Step 2 — Run the data prep script
```bash
python3 prepare_data.py
```

This reads `sp500_top10_stocks_clean.csv` and outputs updated values to paste into `index.html`.

### Step 3 — Replace the STOCKS object
In `index.html`, find the `// ─── REAL DATA ───` section and paste the output.

---

## 📁 Repository Structure

```
sp500-equity-dashboard/
│
├── index.html                      # Main dashboard (self-contained)
├── sp500_top10_stocks_clean.csv    # Raw dataset (35,765 rows)
├── prepare_data.py                 # Data prep & stats calculator
├── README.md                       # This file
└── preview.png                     # Dashboard screenshot (optional)
```

---

## 🐍 Data Prep Script

Save as `prepare_data.py` in the same directory as your CSV:

```python
import pandas as pd
import numpy as np
import json

# ── Load ──────────────────────────────────────────────────────────────────────
df = pd.read_csv('sp500_top10_stocks_clean.csv', parse_dates=['Date'])
df = df[df['Ticker'].notna() & (df['Ticker'] != 'Ticker')]  # drop header rows
df['Close'] = pd.to_numeric(df['Close'], errors='coerce')
df['Volume'] = pd.to_numeric(df['Volume'], errors='coerce')
df = df.dropna(subset=['Close'])

# ── Config ────────────────────────────────────────────────────────────────────
TICKERS = ['AAPL', 'AMZN', 'AVGO', 'GOOG', 'META', 'MSFT', 'NVDA', 'TSLA']
COLORS  = {
    'AAPL': '#6aaeff', 'AMZN': '#f5c842', 'AVGO': '#00d4a8',
    'GOOG': '#f06090', 'META': '#8b7cf8', 'MSFT': '#4ecdc4',
    'NVDA': '#ff9f40', 'TSLA': '#e8291c'
}
NAMES = {
    'AAPL': 'Apple Inc.',       'AMZN': 'Amazon.com Inc.',
    'AVGO': 'Broadcom Inc.',    'GOOG': 'Alphabet Inc.',
    'META': 'Meta Platforms',   'MSFT': 'Microsoft Corp.',
    'NVDA': 'NVIDIA Corp.',     'TSLA': 'Tesla Inc.'
}
SECTORS = {
    'AAPL': 'Technology',       'AMZN': 'Consumer Disc.',
    'AVGO': 'Technology',       'GOOG': 'Comm. Services',
    'META': 'Comm. Services',   'MSFT': 'Technology',
    'NVDA': 'Technology',       'TSLA': 'Consumer Disc.'
}

output = {}

for ticker in TICKERS:
    stock = df[df['Ticker'] == ticker].sort_values('Date').copy()

    # Latest price & day change
    latest      = stock.iloc[-1]['Close']
    prev        = stock.iloc[-2]['Close']
    day_chg     = (latest - prev) / prev * 100

    # 1-year and 5-year returns
    yr1_price   = stock.iloc[-253]['Close'] if len(stock) > 253 else stock.iloc[0]['Close']
    yr5_price   = stock.iloc[-1260]['Close'] if len(stock) > 1260 else stock.iloc[0]['Close']
    yr1         = (latest - yr1_price) / yr1_price * 100
    yr5         = (latest - yr5_price) / yr5_price * 100

    # 30-day avg volume
    vol30       = stock.tail(30)['Volume'].mean() / 1_000_000

    # Monthly closes — last 24 months (end-of-month)
    stock_monthly = (
        stock.set_index('Date')['Close']
             .resample('ME')
             .last()
             .tail(24)
    )
    monthly_vals  = [round(v, 2) for v in stock_monthly.tolist()]

    output[ticker] = {
        'name':    NAMES[ticker],
        'sector':  SECTORS[ticker],
        'color':   COLORS[ticker],
        'price':   round(latest, 2),
        'dayChg':  round(day_chg, 2),
        'yr1':     round(yr1, 1),
        'yr5':     round(yr5, 1),
        'vol30':   round(vol30, 1),
        'monthly': monthly_vals
    }

# ── Output ────────────────────────────────────────────────────────────────────
print("\n// ─── PASTE THIS INTO index.html → STOCKS object ───────────────────")
print("const STOCKS = " + json.dumps(output, indent=2) + ";")
print("\n// Date range in dataset:")
print(f"//   From: {df['Date'].min().date()}")
print(f"//   To:   {df['Date'].max().date()}")
print(f"//   Total rows: {len(df):,}")
```

---

## 📐 Key Design Decisions

### Why normalize the price chart?
Stocks in this dataset have wildly different price levels (NVDA ~$183 vs. META ~$640). Plotting raw prices makes low-priced stocks look flat even if they outperformed. Normalization to 100 shows *relative* return — the real story.

### Why vanilla JS for charts?
No library dependencies means the dashboard loads instantly, works offline, and has zero supply-chain risk. All charts are rendered as inline SVG, which is resolution-independent and fully customizable.

### Why Pearson correlation on monthly returns?
Monthly returns (not prices) remove the trend bias that inflates correlation between stocks that are all going up. Pearson r on returns gives a cleaner signal of co-movement.

---

## 💡 Potential Extensions

- [ ] Connect to a live data API (Yahoo Finance, Alpha Vantage, Polygon.io) to auto-refresh prices
- [ ] Add a date range picker to explore custom time windows
- [ ] Integrate S&P 500 index benchmark line on the price chart
- [ ] Export chart as PNG using `html2canvas`
- [ ] Add RSI / moving average overlays for technical analysis view
- [ ] Build a Python Jupyter notebook version of the correlation and return analysis

---

## 👩‍💼 About This Project

Built by **Marcia Durniat** as part of a marketing analytics portfolio demonstrating:
- Data wrangling and statistical analysis (Python/pandas)
- Financial data literacy — returns, normalization, correlation, volume analysis
- Production-grade data visualization without a BI tool
- Clean, readable code structure for collaborative or client-facing contexts

**LinkedIn:** [linkedin.com/in/marciadurniat](https://linkedin.com/in/marciadurniat)  
**Portfolio:** [mdmstrategy.com](https://mdmstrategy.com)

---

*Dataset is public market data used for educational and portfolio demonstration purposes only. Not financial advice.*
