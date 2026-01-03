# OIPA Statistics Canada - Power BI Model Documentation

---

## Model Overview

This Power BI model analyzes graduate statistics from Statistics Canada, tracking educational outcomes across:
- Geographic regions (provinces)
- Fields of study
- Educational qualifications
- Gender demographics
- Age groups
- Population groups
- Time periods (years)

---

## Tables

### DataFileRecordLayout
**Type:** Fact Table  
**Hidden:** No  
**Description:** Main fact table containing graduate statistics from Statistics Canada

#### Columns (21 total)

##### Key Columns
- **RowNumber-2662979B-1795-4F74-8F37-6A1BA8059B61** (Int64) - *Hidden, Primary Key*
- **VALUE** (Int64) - The core metric value (graduate counts)

##### Dimension Columns
- **REF_DATE** (String) - Reference date/year for the statistic
- **GEO** (String) - Geographic location (province/territory)
- **DGUID** (String) - Unique geographic identifier
- **Educational qualification** (String) - Type of degree or qualification
- **Field of study** (String) - Academic field or discipline
- **Gender** (String) - Gender classification
- **Age group** (String) - Age range grouping
- **Population group** (String) - Population demographic category
- **Indicator** (String) - Type of indicator/metric being measured

##### Metadata Columns
- **UOM** (String) - Unit of measure
- **UOM_ID** (Int64) - Unit of measure identifier
- **SCALAR_FACTOR** (String) - Scaling factor for the value
- **SCALAR_ID** (Int64) - Scaling factor identifier
- **VECTOR** (String) - Statistics Canada vector identifier
- **COORDINATE** (String) - Coordinate reference
- **STATUS** (String) - Data status flag
- **SYMBOL** (String) - Data notation symbol
- **TERMINATED** (String) - Termination indicator
- **DECIMALS** (Int64) - Number of decimal places

---

## Measures (21 total)

### Core Metrics

#### Total Graduates
```dax
SUM('DataFileRecordLayout'[VALUE])
```
**Format:** General Number  
**Description:** Base aggregation of graduate counts

---

### Gender Analysis Measures

#### Female Graduates
```dax
CALCULATE(
    SUM('DataFileRecordLayout'[VALUE]),
    'DataFileRecordLayout'[Gender] = "Woman"
)
```
**Format:** 0  
**Description:** Total graduates filtered for women - Optimized to use direct SUM

#### Male Graduates
```dax
CALCULATE(
    SUM('DataFileRecordLayout'[VALUE]),
    'DataFileRecordLayout'[Gender] = "Man"
)
```
**Format:** 0  
**Description:** Total graduates filtered for men

#### Female Percentage
```dax
DIVIDE(
    [Female Graduates],
    [Total Graduates],
    0
)
```
**Format:** General Number  
**Description:** Percentage of female graduates

#### Male Percentage
```dax
DIVIDE(
    [Male Graduates],
    [Total Graduates],
    BLANK()
)
```
**Format:** 0.00%  
**Description:** Percentage of male graduates

#### Gender Balance Ratio
```dax
VAR FemaleCount = [Female Graduates]
VAR MaleCount = [Male Graduates]
VAR MinValue = MIN(FemaleCount, MaleCount)
VAR MaxValue = MAX(FemaleCount, MaleCount)
RETURN
    DIVIDE(MinValue, MaxValue, BLANK())
```
**Format:** 0.00  
**Description:** Gender diversity ratio (closer to 1.0 means more balanced)

---

### Geographic Measures

#### Number of Provinces
```dax
DISTINCTCOUNT('DataFileRecordLayout'[GEO])
```
**Format:** 0  
**Description:** Count of distinct geographic regions

#### Avg Graduates per Province
```dax
DIVIDE(
    [Total Graduates],
    [Number of Provinces],
    BLANK()
)
```
**Format:** #,##0  
**Description:** Average graduates per province

#### Province Rank by Graduates
```dax
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
**Format:** 0  
**Description:** Rank provinces by number of graduates (1=highest)

---

### Time-Based Measures

#### First Year
```dax
MIN('DataFileRecordLayout'[REF_DATE])
```
**Format:** General  
**Description:** Earliest year in the dataset

#### Last Year
```dax
MAX('DataFileRecordLayout'[REF_DATE])
```
**Format:** General  
**Description:** Most recent year in the dataset

#### Latest Year Graduates
```dax
VAR MaxYear = MAX('DataFileRecordLayout'[REF_DATE])
RETURN
    CALCULATE(
        [Total Graduates],
        'DataFileRecordLayout'[REF_DATE] = MaxYear
    )
```
**Format:** #,##0  
**Description:** Graduates in the most recent year available - Fixed to avoid placeholder errors

#### Cumulative Graduates
```dax
CALCULATE(
    [Total Graduates],
    FILTER(
        ALLSELECTED('DataFileRecordLayout'[REF_DATE]),
        'DataFileRecordLayout'[REF_DATE] <= MAX('DataFileRecordLayout'[REF_DATE])
    )
)
```
**Format:** #,##0  
**Description:** Running total of graduates over time

---

### Growth & Change Measures

#### Graduates YoY Change (%)
```dax
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
**Format:** 0.00%  
**Description:** Year-over-year percentage change - Optimized with direct SUM and better error handling

#### Graduates YoY Change (Absolute)
```dax
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
**Format:** +#,##0;-#,##0;0  
**Description:** Year-over-year absolute change in graduates

#### Total Period Growth %
```dax
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
**Format:** 0.00%  
**Description:** Total growth from first to last year - Fixed to avoid placeholder errors

#### CAGR
```dax
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
**Format:** 0.00%  
**Description:** Compound annual growth rate over the entire period - Fixed to avoid placeholder errors

#### Growth Trend Indicator
```dax
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
**Format:** Text  
**Description:** Visual indicator: Up arrow, down arrow, or neutral based on YoY change

#### 3-Year Moving Avg
```dax
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
**Format:** #,##0  
**Description:** Moving average of graduates over 3 years

---

### Comparative Measures

#### % of Total Graduates
```dax
DIVIDE(
    [Total Graduates],
    CALCULATE([Total Graduates], ALL('DataFileRecordLayout')),
    BLANK()
)
```
**Format:** 0.00%  
**Description:** Percentage of current selection vs total across all contexts

---

### Metadata Measures

#### Number of Fields of Study
```dax
DISTINCTCOUNT('DataFileRecordLayout'[Field of study])
```
**Format:** 0  
**Description:** Number of distinct fields of study

---

## Relationships

**Note:** Currently, this model has **no relationships** defined. The model consists of a single fact table.

---

## Model Optimization Notes

### Optimized Measures
The following measures have been optimized for performance:
1. **Female Graduates** - Uses direct SUM instead of measure chaining
2. **Graduates YoY Change (%)** - Caches current value to avoid recalculation
3. **Latest Year Graduates** - Uses variable instead of measure in filter
4. **Total Period Growth %** - Uses variables instead of measures in filters
5. **CAGR** - Uses variables for all calculations

### Key Optimization Patterns Used
- Direct aggregation functions (SUM, MIN, MAX) instead of measure dependencies
- Variable caching to avoid recalculation
- BLANK() instead of 0 for better null handling
- Proper use of ISINSCOPE for context-aware calculations
- ALLSELECTED for filter context preservation

---

## Usage Recommendations

### Dashboard KPIs
- Latest Year Graduates
- Growth Trend Indicator
- CAGR
- Gender Balance Ratio

### Time Series Analysis
- Total Graduates (with REF_DATE)
- 3-Year Moving Avg
- Cumulative Graduates
- Graduates YoY Change (%)

### Geographic Comparisons
- Province Rank by Graduates
- Avg Graduates per Province
- % of Total Graduates (by GEO)

### Gender Equity Analysis
- Gender Balance Ratio
- Female Percentage / Male Percentage
- Female Graduates / Male Graduates trends over time

---

## Data Quality Considerations

1. **REF_DATE is String type** - Consider converting to Date type for better time intelligence
2. **No Date Table** - Consider creating a dedicated date dimension table
3. **No Relationships** - Single table model limits advanced analytics
4. **Missing Data Handling** - All measures use BLANK() for error handling

---

## Future Enhancement Opportunities

1. **Create a Date Dimension** - Proper date table for time intelligence
2. **Add Calculated Columns** - Year, Quarter, Month from REF_DATE
3. **Parameter Tables** - For dynamic measure selection
4. **Calculation Groups** - For time intelligence patterns
5. **Field Hierarchies** - Geography hierarchy (Country > Province)
6. **Data Categories** - Mark columns appropriately (Geographic, Date, etc.)

---

## Version Control

Save this file as: `OIPA_StatisticsCanada_Model_v1.0.md`

For Git version control:
- Track this file in your repository
- Update version number with each schema change
- Document measure changes in commit messages
- Consider exporting to TMDL folder structure for full version control

---
