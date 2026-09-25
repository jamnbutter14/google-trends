"""
daily_kpop_trends.py

Pulls Google Trends "interest over time" data for a fixed set of topics
(Knowledge Graph entity IDs — the /g/... codes from your Explore URL),
for a fixed geo + timeframe, and saves both a CSV of the raw numbers and
a PNG chart with a FIXED filename.

Fixed filenames matter: whatever consumes this later (a GitHub repo file,
a Notion embed, etc.) should always point at the same path, and only the
*content* changes each run.

Run once a day (cron / GitHub Actions / Task Scheduler) — see notes below.
"""

import sys
from datetime import datetime, timezone

import pandas as pd
import matplotlib
matplotlib.use("Agg")  # render to file, no display needed
import matplotlib.pyplot as plt
from pytrends.request import TrendReq

# ---- Configuration -------------------------------------------------------

# Knowledge Graph topic IDs -> display names (from your Explore link)
TOPICS = {
    "/g/11yjly_225": "James",
    "/g/11xt4k_q7r": "Martin",
    "/g/11xt4srq2w": "Juhoon",
    "/g/11xvlz7chy": "Seonghyeon",
    "/g/11xt00ktl_": "Keonho",
}

TIMEFRAME = "now 7-d"   # rolling 7-day window, matches your Explore link
GEO = ""                # "" = Worldwide in pytrends/Trends terms
CATEGORY = 0            # 0 = All categories

OUT_CSV = "trend_data.csv"
OUT_PNG = "trend.png"

# ---- Fetch -----------------------------------------------------------------

def fetch_interest_over_time() -> pd.DataFrame:
    pytrends = TrendReq(
        hl="en-US",
        tz=0,
        timeout=(10, 25),
        retries=3,
        backoff_factor=0.5,
    )

    kw_list = list(TOPICS.keys())
    if len(kw_list) > 5:
        raise ValueError("Google Trends only accepts up to 5 terms per request.")

    pytrends.build_payload(
        kw_list=kw_list,
        cat=CATEGORY,
        timeframe=TIMEFRAME,
        geo=GEO,
    )

    df = pytrends.interest_over_time()

    if df.empty:
        raise RuntimeError(
            "Google Trends returned no data — check the topic IDs, "
            "timeframe, or geo, or you may be temporarily rate-limited (429)."
        )

    if "isPartial" in df.columns:
        df = df.drop(columns=["isPartial"])

    return df.rename(columns=TOPICS)


# ---- Plot --------------------------------------------------------------

def render_chart(df: pd.DataFrame, out_path: str) -> None:
    plt.figure(figsize=(12, 6))
    for name in TOPICS.values():
        if name in df.columns:
            plt.plot(df.index, df[name], label=name, linewidth=2)

    plt.title(f"Interest over time — Worldwide — {TIMEFRAME}")
    plt.xlabel("Date")
    plt.ylabel("Relative interest (0-100)")
    plt.legend(loc="upper left")
    plt.grid(alpha=0.3)
    plt.tight_layout()

    generated = datetime.now(timezone.utc).strftime("%Y-%m-%d %H:%M UTC")
    plt.figtext(0.99, 0.01, f"Generated {generated}", ha="right", fontsize=8, color="gray")

    plt.savefig(out_path, dpi=150)
    plt.close()


# ---- Main ----------------------------------------------------------------

def main() -> None:
    print("Fetching Google Trends data...")
    df = fetch_interest_over_time()

    df.to_csv(OUT_CSV)
    print(f"Saved raw data to {OUT_CSV}")

    render_chart(df, OUT_PNG)
    print(f"Saved chart to {OUT_PNG}")


if __name__ == "__main__":
    try:
        main()
    except Exception as e:
        print(f"ERROR: {e}", file=sys.stderr)
        sys.exit(1)
