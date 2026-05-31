# Sentiment Zone Oscillator Alert — MQL4 Script

A MetaTrader 4 script that computes a **Sentiment Zone Oscillator (SZO)** by summing positive and negative close-to-close price changes over a rolling `SZOPeriod` window and expressing the directional balance as a `(sumPos − sumNeg) / (sumPos + sumNeg)` ratio bounded between `−1.0` and `+1.0`, and fires threshold-crossing alerts when the SZO transitions from below `BullishThreshold` to above it (bullish sentiment extreme entered) or from above `BearishThreshold` to below it (bearish sentiment extreme entered) — using a `szoPrevious` global to enforce strict first-cross detection.

---

## Overview

The Sentiment Zone Oscillator is a custom momentum indicator that measures the balance of positive and negative price momentum over a configurable window, expressing it as a normalized score from −1 to +1. Unlike RSI — which uses exponentially smoothed gains and losses — the SZO uses raw sums of unsigned price changes, giving it a more direct relationship to the actual magnitude of the period's price activity. A value approaching +1 indicates that nearly all price movement within the period has been upward — extreme bullish sentiment. A value approaching −1 indicates dominant downward movement — extreme bearish sentiment. The threshold crossings monitored by this script fire at the moment the SZO first exceeds the configurable `BullishThreshold` (default `+0.7`) from below, or first falls below `BearishThreshold` (default `−0.7`) from above — capturing the point where sentiment transitions from neutral into statistically extreme territory, which historically correlates with high-momentum continuation moves or potential reversal zones depending on context.

> **Note on file naming:** This file is distributed as `ZigZag_with_Fibonacci_Levels_001.mq4` but implements a Sentiment Zone Oscillator. The README documents the actual implemented logic.

---

## Features

- **SZO formula** — `CalculateSZO()` iterates `i = 1` to `SZOPeriod`; `change = iClose(i) − iClose(i+1)`; `sumPos` accumulates positive changes; `sumNeg` accumulates `MathAbs` of negative changes; `szo = (sumPos − sumNeg) / (sumPos + sumNeg)` — division-by-zero safe since zero `sumPos + sumNeg` implies flat market
- **`iBars() < period` sufficient-data guard** — `CalculateSZO()` prints a warning and returns `0.0` if insufficient bars are available
- **Strict first-cross detection** — `szoCurrent >= BullishThreshold && szoPrevious < BullishThreshold` → **Bullish Sentiment Extreme Detected**; `szoCurrent <= BearishThreshold && szoPrevious > BearishThreshold` → **Bearish Sentiment Extreme Detected** — equality on the threshold side ensures the first bar that reaches or exceeds the threshold fires the alert
- **`szoPrevious` persistent state** — updated to `szoCurrent` unconditionally at cycle end, maintaining accurate prior-cycle reading
- **Rich alert message** — `AlertSZO()` formats with `"SZO Value: %.3f"` for three-decimal precision context
- **Three notification channels:** sound alert, email, and mobile push
- **Lightweight loop** — polls once per minute (`Sleep(60000)`)

---

## How It Works

1. Every minute, `CalculateSZO()` accumulates positive/negative changes across `SZOPeriod` bars; returns normalized ratio
2. Two threshold crossing conditions evaluated:
   - `szoCurrent >= BullishThreshold && szoPrevious < BullishThreshold` → **Bullish Sentiment Extreme Detected**
   - `szoCurrent <= BearishThreshold && szoPrevious > BearishThreshold` → **Bearish Sentiment Extreme Detected**
3. `szoPrevious = szoCurrent` updated at cycle end

---

## Input Parameters

| Parameter             | Type            | Default     | Description                                                          |
|-----------------------|-----------------|-------------|----------------------------------------------------------------------|
| `TradeSymbol`         | string          | `EURUSD`    | Symbol for analysis                                                  |
| `Timeframe`           | ENUM_TIMEFRAMES | `PERIOD_H1` | Timeframe for SZO calculation                                        |
| `SZOPeriod`           | int             | `14`        | Lookback period for positive/negative change accumulation            |
| `BullishThreshold`    | double          | `0.7`       | SZO value at or above which a bullish extreme alert fires            |
| `BearishThreshold`    | double          | `-0.7`      | SZO value at or below which a bearish extreme alert fires            |
| `EnableAlerts`        | bool            | `true`      | Fire an on-screen/sound alert                                        |
| `EnableEmail`         | bool            | `false`     | Send an email notification                                           |
| `EnablePush`          | bool            | `false`     | Send a mobile push notification                                      |

---

## Alert Message Format

```
Bullish Sentiment Extreme Detected detected on EURUSD (Timeframe: PERIOD_H1)
SZO Value: 0.724
```

---

## Installation

1. Copy `ZigZag_with_Fibonacci_Levels_001.mq4` to `MQL4/Scripts/` in your MT4 data folder
2. Compile in MetaEditor (F7)
3. Drag onto any chart from Navigator → Scripts
4. Configure inputs and click **OK**

---

## Requirements

- MetaTrader 4 (`#property strict` compatible build)
- MQL4 compiler (MetaEditor)

---

## License

MIT License

Copyright (c) 2026

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
