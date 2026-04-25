# DAX Measures — Marketing Spend Dashboard
## Paste all measures into your `_Measure` table in Power BI

---

## SECTION 1 — Core Totals

```dax
Total Budget =
SUM ( Budget[Budget - Amount (in AED Mn) ] )
```

```dax
Total Actual =
SUM ( Budget[Actual Spends] )
```

```dax
Overall Budget =
SUM ( Overall[Budget] )
```

---

## SECTION 2 — Variance & Utilisation

```dax
Variance AED =
[Total Actual] - [Total Budget]
```

```dax
Variance % =
DIVIDE (
    [Total Actual] - [Total Budget],
    [Total Budget],
    0
)
```

```dax
Utilisation % =
DIVIDE ( [Total Actual], [Total Budget], 0 )
```

---

## SECTION 3 — Budget Efficiency (for heatmap matrix)

```dax
Efficiency % =
DIVIDE ( [Total Actual], [Total Budget], 0 )
```

```dax
Efficiency Label =
VAR _eff = [Efficiency %]
RETURN
    SWITCH (
        TRUE (),
        _eff = 0,        "No Spend",
        _eff < 0.5,      "Low (" & FORMAT( _eff, "0%" ) & ")",
        _eff < 0.9,      "Under (" & FORMAT( _eff, "0%" ) & ")",
        _eff <= 1.05,    "On Track (" & FORMAT( _eff, "0%" ) & ")",
        _eff <= 1.2,     "Over (" & FORMAT( _eff, "0%" ) & ")",
        "Excess (" & FORMAT( _eff, "0%" ) & ")"
    )
```

---

## SECTION 4 — Year-on-Year Comparison

```dax
Budget PY =
CALCULATE (
    [Total Budget],
    SAMEPERIODLASTYEAR ( Budget[Year] )
)
```

> Note: Budget[Year] is an integer column. If SAMEPERIODLASTYEAR does not work,
> use the CALCULATE + FILTER pattern below instead:

```dax
Budget PY (Alt) =
VAR _currentYear = SELECTEDVALUE ( Budget[Year] )
RETURN
    CALCULATE (
        [Total Budget],
        Budget[Year] = _currentYear - 1
    )
```

```dax
Actual PY =
VAR _currentYear = SELECTEDVALUE ( Budget[Year] )
RETURN
    CALCULATE (
        [Total Actual],
        Budget[Year] = _currentYear - 1
    )
```

```dax
Budget YoY % =
DIVIDE (
    [Total Budget] - [Budget PY (Alt)],
    [Budget PY (Alt)],
    0
)
```

```dax
Actual YoY % =
DIVIDE (
    [Total Actual] - [Actual PY],
    [Actual PY],
    0
)
```

---

## SECTION 5 — Marketing UAE Transaction Spend
> UAE amount column is numeric: AMOUNT (IN AED)

```dax
UAE Actual Spend =
SUM ( 'Marketing UAE'[AMOUNT (IN AED)] )
```

```dax
UAE Spend No VAT =
SUM ( 'Marketing UAE'[AMOUNT (NO VAT)] )
```

```dax
UAE Monthly Spend =
CALCULATE (
    [UAE Actual Spend],
    ALLEXCEPT ( 'Marketing UAE', 'Marketing UAE'[JOB MONTH], 'Marketing UAE'[JOB YEAR] )
)
```

---

## SECTION 6 — Marketing Siniya Transaction Spend
> Siniya amount column is numeric: AMOUNT (IN AED)

```dax
Siniya Actual Spend =
SUM ( 'Marketing Siniya'[AMOUNT (IN AED)] )
```

---

## SECTION 7 — Marketing UAQ Transaction Spend
> UAQ amount column is numeric: AMOUNT (IN AED)

```dax
UAQ Actual Spend =
SUM ( 'Marketing UAQ'[AMOUNT (IN AED)] )
```

---

## SECTION 8 — Marketing Dubai & AUH Transaction Spend
> IMPORTANT: Dubai and AUH store amounts as TEXT with commas e.g. "8,190.00"
> You must convert the text to a number using VALUE() and SUBSTITUTE()

```dax
Dubai Actual Spend =
SUMX (
    'Marketing Dubai',
    VAR _clean = SUBSTITUTE ( 'Marketing Dubai'[AMOUNT(IN AED)], ",", "" )
    RETURN
        IF ( ISNUMBER ( VALUE ( _clean ) ), VALUE ( _clean ), 0 )
)
```

```dax
AUH Actual Spend =
SUMX (
    'Marketing AUH',
    VAR _clean = SUBSTITUTE ( 'Marketing AUH'[AMOUNT(IN AED)], ",", "" )
    RETURN
        IF ( ISNUMBER ( VALUE ( _clean ) ), VALUE ( _clean ), 0 )
)
```

---

## SECTION 9 — Combined Total Across All Locations
> Combines all 4 marketing location tables into one number (AED)

```dax
Total Txn Spend (All Locations) =
[UAE Actual Spend]
    + [Siniya Actual Spend]
    + [UAQ Actual Spend]
    + [Dubai Actual Spend]
    + [AUH Actual Spend]
```

---

## SECTION 10 — Supplier Analysis

```dax
Supplier Spend UAE =
CALCULATE (
    [UAE Actual Spend],
    ALLEXCEPT ( 'Marketing UAE', 'Marketing UAE'[SUPPLIER NAME] )
)
```

```dax
Supplier Spend Rank =
RANKX (
    ALL ( 'Marketing UAE'[SUPPLIER NAME] ),
    [UAE Actual Spend],
    ,
    DESC,
    DENSE
)
```

```dax
Top 5 Supplier Flag =
IF ( [Supplier Spend Rank] <= 5, "Top 5", "Other" )
```

---

## SECTION 11 — Category & Tagging Breakdown

```dax
Spend by Category =
CALCULATE (
    [Total Actual],
    ALLEXCEPT ( Budget, Budget[Marketing Heads] )
)
```

```dax
Category Budget Share % =
DIVIDE (
    [Total Budget],
    CALCULATE ( [Total Budget], ALL ( Budget[Marketing Heads] ) ),
    0
)
```

```dax
Category Actual Share % =
DIVIDE (
    [Total Actual],
    CALCULATE ( [Total Actual], ALL ( Budget[Marketing Heads] ) ),
    0
)
```

---

## SECTION 12 — Emirate Budget Split (for Donut Chart)

```dax
Emirate Budget Share % =
DIVIDE (
    [Overall Budget],
    CALCULATE ( [Overall Budget], ALL ( Overall[Emirate] ) ),
    0
)
```

---

## SECTION 13 — Month Sort Helper (Calculated Column)
> Add this as a CALCULATED COLUMN in Marketing UAE table
> Then use "Sort by Column" → Month Number to fix line chart axis

```dax
Month Number =
SWITCH (
    'Marketing UAE'[JOB MONTH],
    "JAN", 1,
    "FEB", 2,
    "MAR", 3,
    "APR", 4,
    "MAY", 5,
    "JUN", 6,
    "JUL", 7,
    "AUG", 8,
    "SEP", 9,
    "OCT", 10,
    "NOV", 11,
    "DEC", 12,
    0
)
```

> Apply the same calculated column to: Marketing Siniya, Marketing Dubai,
> Marketing AUH, Marketing UAQ — replacing the table name in each.

---

## SECTION 14 — KPI Status Labels (for Conditional Formatting)

```dax
Budget Status =
VAR _var = [Variance %]
RETURN
    SWITCH (
        TRUE (),
        _var > 0.2,   "Critical Over",
        _var > 0.05,  "Over Budget",
        _var > -0.05, "On Target",
        _var > -0.15, "Under Budget",
        "Significantly Under"
    )
```

```dax
Budget Status Color =
VAR _var = [Variance %]
RETURN
    SWITCH (
        TRUE (),
        _var > 0.2,   "#A32D2D",
        _var > 0.05,  "#E24B4A",
        _var > -0.05, "#1D9E75",
        _var > -0.15, "#0F6E56",
        "#888780"
    )
```

---

## SECTION 15 — Forecast (Simple Linear Projection)
> Uses last 2 years of Budget data to project next year

```dax
Projected Budget Next Year =
VAR _lastYear = MAXX ( ALL ( Budget[Year] ), Budget[Year] )
VAR _budgetLastYear =
    CALCULATE ( [Total Budget], Budget[Year] = _lastYear )
VAR _budgetPrevYear =
    CALCULATE ( [Total Budget], Budget[Year] = _lastYear - 1 )
VAR _growth =
    DIVIDE ( _budgetLastYear - _budgetPrevYear, _budgetPrevYear, 0 )
RETURN
    _budgetLastYear * ( 1 + _growth )
```

```dax
Projected Actual Next Year =
VAR _lastYear = MAXX ( ALL ( Budget[Year] ), Budget[Year] )
VAR _actualLastYear =
    CALCULATE ( [Total Actual], Budget[Year] = _lastYear )
VAR _actualPrevYear =
    CALCULATE ( [Total Actual], Budget[Year] = _lastYear - 1 )
VAR _growth =
    DIVIDE ( _actualLastYear - _actualPrevYear, _actualPrevYear, 0 )
RETURN
    _actualLastYear * ( 1 + _growth )
```

---

## SECTION 16 — Dynamic Title Measures (for Visual Titles)

```dax
Selected Year Title =
VAR _yr = SELECTEDVALUE ( Budget[Year], "All Years" )
RETURN "Marketing Spend — " & _yr
```

```dax
Selected Emirate Title =
VAR _em = SELECTEDVALUE ( Overall[Emirate], "All Emirates" )
RETURN "Emirate: " & _em
```

```dax
Variance Summary Text =
VAR _bud = [Total Budget]
VAR _act = [Total Actual]
VAR _var = [Variance %]
VAR _dir = IF ( _var >= 0, " over budget", " under budget" )
RETURN
    "Actual is " & FORMAT ( ABS ( _var ), "0.0%" ) & _dir
    & " (" & FORMAT ( ABS ( _act - _bud ), "#,##0.0" ) & " AED Mn)"
```

---

## QUICK REFERENCE — Which Measure Goes in Which Visual

| Visual | Measures to Use |
|---|---|
| Card 1 — Total Budget | `Total Budget` |
| Card 2 — Total Actual | `Total Actual` |
| Card 3 — Variance % | `Variance %` + `Budget Status Color` |
| Card 4 — Utilisation | `Utilisation %` |
| Chart 5 — Budget vs Actual bar | `Total Budget`, `Total Actual` by `Budget[Year]` |
| Chart 6 — Monthly trend line | `UAE Actual Spend` by `JOB MONTH` + `JOB YEAR` |
| Chart 7 — Category bar | `Total Actual`, `Total Budget` by `Budget[Marketing Heads]` |
| Chart 8 — Emirate donut | `Overall Budget`, `Emirate Budget Share %` by `Overall[Emirate]` |
| Chart 9 — Top suppliers | `UAE Actual Spend`, `Supplier Spend Rank` by `SUPPLIER NAME` |
| Chart 10 — Heatmap matrix | `Efficiency %` rows: `Marketing Heads`, cols: `Year` |
| Dynamic titles | `Selected Year Title`, `Variance Summary Text` |
| Forecast cards | `Projected Budget Next Year`, `Projected Actual Next Year` |
