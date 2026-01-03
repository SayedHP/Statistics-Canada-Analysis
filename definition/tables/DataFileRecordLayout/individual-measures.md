# Individual Measure Export Files

Save each of these as separate `.dax` files in your measures folder.

---

## measures/core/Total_Graduates.dax
```dax
// Total Graduates
// Base aggregation of graduate counts

SUM('DataFileRecordLayout'[VALUE])
```

---

## measures/gender/Female_Graduates.dax
```dax
// Female Graduates
// Total graduates filtered for women - Optimized to use direct SUM
// Format: 0

CALCULATE(
    SUM('DataFileRecordLayout'[VALUE]),
    'DataFileRecordLayout'[Gender] = "Woman"
)
```

---

## measures/gender/Male_Graduates.dax
```dax
// Male Graduates
// Total graduates filtered for men
// Format: 0

CALCULATE(
    SUM('DataFileRecordLayout'[VALUE]),
    'DataFileRecordLayout'[Gender] = "Man"
)
```

---

## measures/gender/Female_Percentage.dax
```dax
// Female Percentage
// Percentage of female graduates

DIVIDE(
    [Female Graduates],
    [Total Graduates],
    0
)
```

---

## measures/gender/Male_Percentage.dax
```dax
// Male Percentage
// Percentage of male graduates
// Format: 0.00%

DIVIDE(
    [Male Graduates],
    [Total Graduates],
    BLANK()
)
```

---

## measures/gender/Gender_Balance_Ratio.dax
```dax
// Gender Balance Ratio
// Gender diversity ratio (closer to 1.0 means more balanced)
// Format: 0.00

VAR FemaleCount = [Female Graduates]
VAR MaleCount = [Male Graduates]
VAR MinValue = MIN(FemaleCount, MaleCount)
VAR MaxValue = MAX(FemaleCount, MaleCount)
RETURN
    DIVIDE(MinValue, MaxValue, BLANK())
```

---

## measures/geographic/Number_of_Provinces.dax
```dax
// Number of Provinces
// Count of distinct geographic regions
// Format: 0

DISTINCTCOUNT('DataFileRecordLayout'[GEO])
```

---

## measures/geographic/Avg_Graduates_per_Province.dax
```dax
// Avg Graduates per Province
// Average graduates per province
// Format: #,##0

DIVIDE(
    [Total Graduates],
    [Number of Provinces],
    BLANK()
)
```

---

## measures/geographic/Province_Rank_by_Graduates.dax
```dax
// Province Rank by Graduates
// Rank provinces by number of graduates (1=highest)
// Format: 0

IF(
    ISINSCOPE('DataFileRecordLayout'[GEO]),
    RANKX(
        ALLSELECTED('DataFileRecordLayout'[GEO]),
        [Total Graduates],
        ,
        DESC,
        DENSE
    ),
    BLANK()
)
```

---

## measures/time/First_Year.dax
```dax
// First Year
// Earliest year in the dataset

MIN('DataFileRecordLayout'[REF_DATE])
```

---

## measures/time/Last_Year.dax
```dax
// Last Year
// Most recent year in the dataset

MAX('DataFileRecordLayout'[REF_DATE])
```

---

## measures/time/Latest_Year_Graduates.dax
```dax
// Latest Year Graduates
// Graduates in the most recent year available - Fixed to avoid placeholder errors
// Format: #,##0

VAR MaxYear = MAX('DataFileRecordLayout'[REF_DATE])
RETURN
    CALCULATE(
        [Total Graduates],
        'DataFileRecordLayout'[REF_DATE] = MaxYear
    )
```

---

## measures/time/Cumulative_Graduates.dax
```dax
// Cumulative Graduates
// Running total of graduates over time
// Format: #,##0

CALCULATE(
    [Total Graduates],
    FILTER(
        ALLSELECTED('DataFileRecordLayout'[REF_DATE]),
        'DataFileRecordLayout'[REF_DATE] <= MAX('DataFileRecordLayout'[REF_DATE])
    )
)
```

---

## measures/growth/Graduates_YoY_Change_Percent.dax
```dax
// Graduates YoY Change (%)
// Year-over-year percentage change - Optimized with direct SUM and better error handling
// Format: 0.00%

VAR CurrentValue = SUM('DataFileRecordLayout'[VALUE])
VAR CurrentYear = SELECTEDVALUE('DataFileRecordLayout'[REF_DATE])
VAR PriorYearValue = 
    CALCULATE(
        SUM('DataFileRecordLayout'[VALUE]),
        'DataFileRecordLayout'[REF_DATE] = FORMAT(VALUE(CurrentYear) - 1, "0")
    )
RETURN
    DIVIDE(
        CurrentValue - PriorYearValue,
        PriorYearValue,
        BLANK()
    )
```

---

## measures/growth/Graduates_YoY_Change_Absolute.dax
```dax
// Graduates YoY Change (Absolute)
// Year-over-year absolute change in graduates
// Format: +#,##0;-#,##0;0

VAR CurrentValue = [Total Graduates]
VAR CurrentYear = SELECTEDVALUE('DataFileRecordLayout'[REF_DATE])
VAR PriorYearValue = 
    CALCULATE(
        [Total Graduates],
        'DataFileRecordLayout'[REF_DATE] = FORMAT(VALUE(CurrentYear) - 1, "0")
    )
RETURN
    CurrentValue - PriorYearValue
```

---

## measures/growth/Three_Year_Moving_Avg.dax
```dax
// 3-Year Moving Avg
// Moving average of graduates over 3 years
// Format: #,##0

VAR CurrentYear = SELECTEDVALUE('DataFileRecordLayout'[REF_DATE])
VAR YearTable = 
    FILTER(
        ALLSELECTED('DataFileRecordLayout'[REF_DATE]),
        VALUE('DataFileRecordLayout'[REF_DATE]) >= VALUE(CurrentYear) - 2 &&
        VALUE('DataFileRecordLayout'[REF_DATE]) <= VALUE(CurrentYear)
    )
VAR YearsCount = COUNTROWS(YearTable)
RETURN
    IF(
        YearsCount >= 2,
        CALCULATE(
            [Total Graduates],
            YearTable
        ) / YearsCount,
        BLANK()
    )
```

---

## measures/growth/Total_Period_Growth_Percent.dax
```dax
// Total Period Growth %
// Total growth from first to last year - Fixed to avoid placeholder errors
// Format: 0.00%

VAR MinYear = MIN('DataFileRecordLayout'[REF_DATE])
VAR MaxYear = MAX('DataFileRecordLayout'[REF_DATE])
VAR FirstYearValue = 
    CALCULATE(
        [Total Graduates],
        'DataFileRecordLayout'[REF_DATE] = MinYear
    )
VAR LastYearValue = 
    CALCULATE(
        [Total Graduates],
        'DataFileRecordLayout'[REF_DATE] = MaxYear
    )
RETURN
    DIVIDE(
        LastYearValue - FirstYearValue,
        FirstYearValue,
        BLANK()
    )
```

---

## measures/growth/CAGR.dax
```dax
// CAGR
// Compound annual growth rate over the entire period - Fixed to avoid placeholder errors
// Format: 0.00%

VAR MinYear = MIN('DataFileRecordLayout'[REF_DATE])
VAR MaxYear = MAX('DataFileRecordLayout'[REF_DATE])
VAR FirstYearValue = 
    CALCULATE(
        [Total Graduates],
        'DataFileRecordLayout'[REF_DATE] = MinYear
    )
VAR LastYearValue = 
    CALCULATE(
        [Total Graduates],
        'DataFileRecordLayout'[REF_DATE] = MaxYear
    )
VAR YearsCount = VALUE(MaxYear) - VALUE(MinYear)
RETURN
    IF(
        YearsCount > 0 && FirstYearValue > 0,
        POWER(LastYearValue / FirstYearValue, 1 / YearsCount) - 1,
        BLANK()
    )
```

---

## measures/growth/Growth_Trend_Indicator.dax
```dax
// Growth Trend Indicator
// Visual indicator: Up arrow, down arrow, or neutral based on YoY change

VAR YoYChange = [Graduates YofY Change (%)]
RETURN
    SWITCH(
        TRUE(),
        ISBLANK(YoYChange), "–",
        YoYChange > 0.02, "↑ Increase",
        YoYChange < -0.02, "↓ Decrease",
        "→ Stable"
    )
```

---

## measures/comparative/Percent_of_Total_Graduates.dax
```dax
// % of Total Graduates
// Percentage of current selection vs total across all contexts
// Format: 0.00%

DIVIDE(
    [Total Graduates],
    CALCULATE([Total Graduates], ALL('DataFileRecordLayout')),
    BLANK()
)
```

---

## measures/metadata/Number_of_Fields_of_Study.dax
```dax
// Number of Fields of Study
// Number of distinct fields of study
// Format: 0

DISTINCTCOUNT('DataFileRecordLayout'[Field of study])
```

---

## Folder Structure
```
measures/
├── core/
│   └── Total_Graduates.dax
├── gender/
│   ├── Female_Graduates.dax
│   ├── Male_Graduates.dax
│   ├── Female_Percentage.dax
│   ├── Male_Percentage.dax
│   └── Gender_Balance_Ratio.dax
├── geographic/
│   ├── Number_of_Provinces.dax
│   ├── Avg_Graduates_per_Province.dax
│   └── Province_Rank_by_Graduates.dax
├── time/
│   ├── First_Year.dax
│   ├── Last_Year.dax
│   ├── Latest_Year_Graduates.dax
│   └── Cumulative_Graduates.dax
├── growth/
│   ├── Graduates_YoY_Change_Percent.dax
│   ├── Graduates_YoY_Change_Absolute.dax
│   ├── Three_Year_Moving_Avg.dax
│   ├── Total_Period_Growth_Percent.dax
│   ├── CAGR.dax
│   └── Growth_Trend_Indicator.dax
├── comparative/
│   └── Percent_of_Total_Graduates.dax
└── metadata/
    └── Number_of_Fields_of_Study.dax
```