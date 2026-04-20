# Quality KPI Measures (DAX)

This file contains reusable DAX patterns for quality-style KPI reporting:
base KPI calculation, normalization, time intelligence, and variance metrics.

> **Note:** Table/column names are anonymized for portfolio use.
>
> ## 1) KPI % (Base)

**Definition:**  
Calculates the core KPI as a weighted percentage using numerator and denominator
values. This approach ensures accurate rollups across Region, Plant, and KPI level.

KPI % =
DIVIDE (
    SUM ( FactKPI[Numerator] ),
    SUM ( FactKPI[Denominator] )
)

2) KPI % (Normalized)
Definition:
Corrects inconsistent percent storage by scaling values greater than 1.
This prevents scenarios where values such as 54 are mistakenly treated as 5400%.

KPI % (Normalized) =
VAR v = [KPI %]
RETURN
    IF ( v > 1, v / 100, v )
3) KPI MTD (Month-to-Date)
Definition:
Returns the month-to-date KPI value based on the current date context.
Used for evaluating partial month performance.

KPI MTD =
CALCULATE (
    [KPI % (Normalized)],
    DATESMTD ( DimDate[Date] )
)
4) KPI YTD (Weighted)
Definition:
Calculates a weighted year-to-date KPI using cumulative numerator and
denominator values. Best suited for overall YTD performance reporting.
KPI YTD (Weighted) =
CALCULATE (
    [KPI % (Normalized)],
    DATESYTD ( DimDate[Date] )
)
5) KPI YTD Avg (Monthly)
Definition:
Calculates an unweighted average of monthly KPI values year-to-date.
Each month contributes equally, matching many executive-style “YTD Avg” cards.
KPI YTD Avg (Monthly) =
VAR MonthsYTD =
    CALCULATETABLE (
        VALUES ( DimDate[MonthStart] ),
        DATESYTD ( DimDate[Date] )
    )
RETURN
    AVERAGEX (
        MonthsYTD,
        CALCULATE ( [KPI % (Normalized)] )
    )
6) KPI Rolling 3M
Definition:
Calculates the KPI over a rolling three-month window.
Used to smooth short-term volatility and highlight trends.
KPI Rolling 3M =
VAR EndDate =
    MAX ( DimDate[Date] )
RETURN
    CALCULATE (
        [KPI % (Normalized)],
        DATESINPERIOD ( DimDate[Date], EndDate, -3, MONTH )
    )
7) KPI Prior Month
Definition:
Returns the KPI value for the previous month.
Acts as a reference measure for month-over-month comparisons.
KPI Prior Month =
CALCULATE (
    [KPI % (Normalized)],
    DATEADD ( DimDate[Date], -1, MONTH )
)
8) KPI MoM Change (Percentage Points)
Definition:
Calculates the month-over-month change in KPI performance expressed
in percentage points rather than percent change.
KPI MoM Change (pp) =
[KPI % (Normalized)] - [KPI Prior Month]
9) KPI Target (Generic)
Definition:
Represents a generic KPI performance target used for variance and
conditional formatting logic. Value is intentionally generalized.
KPI Target =
0.90
10) KPI Variance vs Target (Percentage Points)
Definition:
Calculates the difference between actual KPI performance and the
defined target, expressed in percentage points.
KPI Variance vs Target (pp) =
[KPI % (Normalized)] - [KPI Target]
