# Drawdown Enhancements to Reporting Script

## Summary of Changes

The reporting script has been enhanced to calculate and report:

1. ✅ **Maximum Drawdown for Entire Strategy** (already existed)
2. ✅ **Maximum Drawdown for Each Year** (newly added)

## What Was Already There

### Overall Maximum Drawdown
- **Function**: `compute_drawdown()` (line 370)
- **Calculated in**: `compute_performance_metrics()` (line 417-418)
- **Output file**: `performance_summary.csv`
- **Column name**: `max_drawdown`

Example `performance_summary.csv`:
```
start_date,end_date,trading_days,start_equity,end_equity,total_return,cagr,ann_vol,sharpe,max_drawdown
2020-01-02,2023-12-29,1015,100000,145000,0.45,0.1324,0.1856,0.713,-0.1245
```

The `max_drawdown` column shows -12.45%, meaning the worst peak-to-trough decline was 12.45%.

## What Was Added

### New Function: `compute_yearly_drawdowns()`
```python
def compute_yearly_drawdowns(portfolio: pd.DataFrame) -> pd.DataFrame:
    """
    Compute maximum drawdown for each calendar year.
    
    Parameters:
    -----------
    portfolio : DataFrame with 'date' and 'equity' columns
    
    Returns:
    --------
    DataFrame with columns: year, max_drawdown
    """
```

This function:
1. Groups portfolio data by calendar year
2. Calculates the maximum drawdown within each year
3. Returns a DataFrame with one row per year

### New Output File: `yearly_drawdowns.csv`

Example output:
```
year,max_drawdown
2020,-0.0856
2021,-0.0432
2022,-0.1567
2023,-0.0723
```

This shows:
- 2020: worst drawdown was -8.56%
- 2021: worst drawdown was -4.32%
- 2022: worst drawdown was -15.67% (worst year)
- 2023: worst drawdown was -7.23%

## Output Files Summary

After running the enhanced script, you will have:

### Overall Performance
- `performance_summary.csv` - Contains CAGR, overall max drawdown, Sharpe ratio, etc.

### Returns
- `yearly_returns.csv` - Return for each calendar year
- `monthly_returns.csv` - Monthly return matrix

### Drawdowns (NEW!)
- `yearly_drawdowns.csv` - Maximum drawdown for each calendar year

### Other Files
- `rolling_annualized_volatility.csv` - 2-month rolling volatility
- `daily_account_ledger.csv` - Daily equity, margin, free cash
- `trade_impact_ledger.csv` - Trade-by-trade impact analysis
- Charts (if enabled)

## How Drawdown is Calculated

Drawdown at time t is:
```
DD(t) = Equity(t) / RunningMax(Equity[0:t]) - 1.0
```

Maximum drawdown is the minimum value (most negative) over the period.

For example:
- Start with $100,000
- Peak at $120,000
- Drop to $108,000
- Drawdown = $108,000 / $120,000 - 1.0 = -0.10 or -10%

The yearly maximum drawdown finds the worst such decline within each calendar year.

## Usage

Simply run the enhanced script as before:
```bash
python 6-reporting_script_with_yearly_drawdowns.py
```

The script will automatically generate all the files including the new `yearly_drawdowns.csv`.

## Integration with Daily Statement Report

If you want to also display yearly drawdowns in your daily statement PDF report, you can:
1. Load `yearly_drawdowns.csv` in the daily statement script
2. Add a summary page showing yearly statistics (returns, volatility, drawdowns)
3. Display the max drawdown for the specific year on each daily statement page


# Summary of Changes to Daily Report

Added load_summary_statistics() function - Loads performance metrics from CSV files
Added draw_summary_statistics_page() function - Renders the summary page with all statistics
Modified write_daily_statement_pdf() signature - Added summary_stats parameter
Modified PDF generation - Summary stats page is now first, then volatility chart, then daily statements
Modified main() function - Loads summary stats and passes them to PDF writer

The summary page will now show:

CAGR
Total Return
Sharpe Ratio
Annual Volatility
Overall Maximum Drawdown
Table of yearly returns and drawdowns

All formatted cleanly on the first page of the PDF report!