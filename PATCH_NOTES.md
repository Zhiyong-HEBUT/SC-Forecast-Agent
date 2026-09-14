# SmartSCM Agents - Forecasting Patch Notes

## What was fixed

1. **Real recursive 14-day forecasting**
   - Removed the old behavior that predicted once and copied the same value 14 times.
   - Each future day now recalculates calendar, lag_7, lag_14, rollmean_7 and rollmean_28.
   - Each prediction is fed back into the history for the next step.
   - Negative demand predictions are clipped to zero.
   - Future sell_price uses the latest known price as an explicit baseline assumption.

2. **Demand Analyst chain fixed**
   - DemandInsight now contains:
     - daily forecast + dates
     - recent 28-day actual average
     - change vs recent average
     - forecast standard deviation
     - coefficient of variation (CV)
     - demand level
     - trend direction
     - volatility level
   - Demand Analyst receives the real daily forecast instead of only an average.
   - Dashboard and CLI both pass the same structured metrics to the agent.
   - Agent prompt is instructed not to invent promotion/holiday/price explanations.

3. **Zero-sales metric explosion fixed**
   - Removed sklearn MAPE from training output.
   - Added RMSE, MAE, WMAPE, sMAPE and Forecast Bias.
   - Individual zero-sales observations no longer cause division-by-zero explosions.
   - If total actual demand for an entire evaluation split is zero, aggregate percentage metrics return NaN instead of a misleading huge number.

## Main modified files

- src/forecasting/forecast_service.py
- src/forecasting/train_baseline.py
- src/agents/tools.py
- src/agents/domain_agents.py
- src/app/run_agents_planning.py
- src/app/dashboard.py

## Run

```bash
pip install -r requirements.txt
python -m src.forecasting.train_baseline
python -m src.app.demo_one_item
python -m src.app.run_daily_planning --top_n 10
```

For LLM agents, set OPENAI_API_KEY and then:

```bash
python -m src.app.run_agents_planning --top_n 10
streamlit run src/app/dashboard.py
```

## Demand classification rules

- Demand level:
  - HIGH: forecast average >= 20% above recent 28-day average
  - LOW: forecast average <= 20% below recent 28-day average
  - NORMAL: otherwise
- Trend:
  - UP: second-half forecast average >= 10% above first-half average
  - DOWN: <= 10% below
  - STABLE: otherwise
- Volatility:
  - LOW: CV < 0.20
  - MEDIUM: 0.20 <= CV < 0.50
  - HIGH: CV >= 0.50
