# Quality KPI Measures (DAX)

This file contains reusable DAX patterns for quality-style KPI reporting, including
base KPI calculation, percent normalization, time intelligence, rolling trends,
and variance analysis.

Note: Table and column names are intentionally anonymized for portfolio use.

---------------------------------------------------------------------

KPI % (Base)
Definition:
Calculates the core KPI as a weighted percentage using numerator and denominator
values. This ensures accurate rollups across Region, Plant, and KPI category.

KPI % =
DIVIDE (
    SUM ( FactKPI[Numerator] ),
    SUM ( FactKPI[Denominator] )
)

---------------------------------------------------------------------

KPI % (Normalized)
Definition:
Corrects inconsistent percent storage by scaling values greater than 1.
Prevents issues where values such as 54 are interpreted as 5400%.

KPI % (Normalized) =
VAR v = [KPI %]
RETURN
    IF ( v > 1, v / 100, v )

---------------------------------------------------------------------

KPI MTD (Month-to-Date)
Definition:
Returns the month-to-date KPI value based on the current date context.
Used for evaluating partial month performance.

KPI MTD =
CALCULATE (
    [KPI % (Normalized)],
    DATESMTD ( DimDate[Date] )
)

---------------------------------------------------------------------

KPI YTD (Weighted)
Definition:
Calculates a weighted year-to-date KPI using cumulative numerator and
denominator values. Best suited for overall YTD performance reporting.

KPI YTD (Weighted) =
CALCULATE (
    [KPI % (Normalized)],
    DATESYTD ( DimDate[Date] )
)

---------------------------------------------------------------------

KPI YTD Avg (Monthly)
Definition:
Calculates an unweighted average of monthly KPI values year-to-date.
Each month contributes equally, matching common executive-style “YTD Avg” cards.

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

---------------------------------------------------------------------

KPI Rolling 3M
Definition:
Calculates the KPI over a rolling three-month window to smooth short-term
variability and highlight trends.

KPI Rolling 3M =
VAR EndDate =
    MAX ( DimDate[Date] )
RETURN
    CALCULATE (
        [KPI % (Normalized)],
        DATESINPERIOD ( DimDate[Date], EndDate, -3, MONTH )
    )

---------------------------------------------------------------------

KPI Prior Month
Definition:
Returns the KPI value for the previous month.
Used as a baseline for month-over-month comparisons.

KPI Prior Month =
CALCULATE (
    [KPI % (Normalized)],
    DATEADD ( DimDate[Date], -1, MONTH )
)

---------------------------------------------------------------------

KPI MoM Change (Percentage Points)
Definition:
Calculates the month-over-month change in KPI performance expressed
in percentage points rather than percent change.

KPI MoM Change (pp) =
[KPI % (Normalized)] - [KPI Prior Month]

---------------------------------------------------------------------

KPI Target (Generic)
Definition:
Represents a generic performance target used for variance analysis
and conditional formatting logic.

KPI Target =
0.90

---------------------------------------------------------------------

KPI Variance vs Target (Percentage Points)
Definition:
Calculates the difference between actual KPI performance and the
defined target, expressed in percentage points.

KPI Variance vs Target (pp) =
[KPI % (Normalized)] - [KPI Target]
