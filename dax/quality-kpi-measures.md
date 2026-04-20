# Quality KPI Measures (DAX)

This file contains reusable DAX patterns for quality-style KPI reporting:
base KPI calculation, normalization, time intelligence, and variance metrics.

> **Note:** Table/column names are anonymized for portfolio use.

KPI % =
DIVIDE (
    SUM ( FactKPI[Numerator] ),
    SUM ( FactKPI[Denominator] )
)

KPI % (Normalized) =
VAR v = [KPI %]
RETURN
    IF ( v > 1, v / 100, v )

KPI MTD =
CALCULATE (
    [KPI % (Normalized)],
    DATESMTD ( DimDate[Date] )
)

KPI YTD (Weighted) =
CALCULATE (
    [KPI % (Normalized)],
    DATESYTD ( DimDate[Date] )
)

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

KPI Rolling 3M =
VAR EndDate =
    MAX ( DimDate[Date] )
RETURN
    CALCULATE (
        [KPI % (Normalized)],
        DATESINPERIOD ( DimDate[Date], EndDate, -3, MONTH )
    )

KPI Prior Month =
CALCULATE (
    [KPI % (Normalized)],
    DATEADD ( DimDate[Date], -1, MONTH )
)

KPI MoM Change (pp) =
[KPI % (Normalized)] - [KPI Prior Month]

KPI Target =
0.90

KPI Variance vs Target (pp) =
[KPI % (Normalized)] - [KPI Target]
``
