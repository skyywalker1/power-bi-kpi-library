# Quality KPI Measures (DAX)

This file contains reusable DAX patterns for quality-style KPI reporting:
base KPI calculation, normalization, time intelligence, and variance metrics.

> **Note:** Table/column names are anonymized for portfolio use.

---

## Model Assumptions (Generic)
- Fact table: `FactKPI`
  - `FactKPI[Numerator]` (e.g., completed / achieved count)
  - `FactKPI[Denominator]` (e.g., planned / total count)
- Date table: `DimDate`
  - `DimDate[Date]` (daily date)
  - Mark `DimDate` as a Date table in Power BI

---

## Measures

### 1) KPI % (Base)
**Definition:** Calculates the KPI as a weighted percentage using numerator/denominator.

```DAX
KPI % =
DIVIDE (
    SUM ( FactKPI[Numerator] ),
    SUM ( FactKPI[Denominator] )
)
