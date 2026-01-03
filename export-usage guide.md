# OIPA Statistics Canada Model - Export Package Guide

**Version:** 1.0.0  
**Date:** 2026-01-03  
**Model:** OIPA_StatisticsCanada

---

## 📦 Package Contents

This export package contains 5 different formats of your Power BI model documentation:

1. **TMDL Folder Structure** - Version control friendly format
2. **Individual Measure Files** - Organized DAX files by category
3. **Change Log** - Complete history of optimizations and changes
4. **Data Dictionary (CSV)** - Business-friendly documentation
5. **JSON Export** - Machine-readable format for automation

---

## 📁 Recommended Folder Structure

```
OIPA_StatisticsCanada/
├── README.md                          # This file
├── CHANGELOG.md                       # Change log
├── Data_Dictionary.csv                # Business documentation
├── Model_Export.json                  # JSON schema
├── database.tmdl                      # Root TMDL file
│
├── tables/
│   └── DataFileRecordLayout/
│       ├── DataFileRecordLayout.tmdl  # Table definition
│       └── measures.tmdl              # All measures in TMDL format
│
├── measures/                          # Individual DAX files
│   ├── core/
│   │   └── Total_Graduates.dax
│   ├── gender/
│   │   ├── Female_Graduates.dax
│   │   ├── Male_Graduates.dax
│   │   ├── Female_Percentage.dax
│   │   ├── Male_Percentage.dax
│   │   └── Gender_Balance_Ratio.dax
│   ├── geographic/
│   │   ├── Number_of_Provinces.dax
│   │   ├── Avg_Graduates_per_Province.dax
│   │   └── Province_Rank_by_Graduates.dax
│   ├── time/
│   │   ├── First_Year.dax
│   │   ├── Last_Year.dax
│   │   ├── Latest_Year_Graduates.dax
│   │   └── Cumulative_Graduates.dax
│   ├── growth/
│   │   ├── Graduates_YoY_Change_Percent.dax
│   │   ├── Graduates_YoY_Change_Absolute.dax
│   │   ├── Three_Year_Moving_Avg.dax
│   │   ├── Total_Period_Growth_Percent.dax
│   │   ├── CAGR.dax
│   │   └── Growth_Trend_Indicator.dax
│   ├── comparative/
│   │   └── Percent_of_Total_Graduates.dax
│   └── metadata/
│       └── Number_of_Fields_of_Study.dax
│
└── docs/
    ├── OIPA_StatisticsCanada_Model_v1.0.md  # Complete documentation
    └── screenshots/                          # Add your screenshots here
```

---

## 🚀 Getting Started

### Step 1: Create the Folder Structure

```bash
# Create base directory
mkdir OIPA_StatisticsCanada
cd OIPA_StatisticsCanada

# Create subdirectories
mkdir -p tables/DataFileRecordLayout
mkdir -p measures/{core,gender,geographic,time,growth,comparative,metadata}
mkdir -p docs/screenshots
```

### Step 2: Copy All Artifact Files

Copy each artifact to its respective location:

- `database.tmdl` → Root directory
- `tables/DataFileRecordLayout.tmdl` → tables/DataFileRecordLayout/
- `tables/DataFileRecordLayout/measures.tmdl` → tables/DataFileRecordLayout/
- All individual `.dax` files → measures/ subdirectories
- `CHANGELOG.md` → Root directory
- `Data_Dictionary.csv` → Root directory
- `Model_Export.json` → Root directory
- Complete documentation → docs/

### Step 3: Initialize Git Repository

```bash
git init
git add .
git commit -m "Initial commit: OIPA Statistics Canada model v1.0.0"
```

---

## 📖 How to Use Each Format

### 1. TMDL Folder Structure

**Purpose:** Version control, collaboration, deployment

**Use Cases:**
- Track changes in Git with readable diffs
- Deploy to Fabric/Analysis Services via Tabular Editor
- Collaborate with team members on model changes
- Automate model deployments with CI/CD

**Tools:**
- Tabular Editor 3
- ALM Toolkit
- Git for version control

**Example Workflow:**
```bash
# Make changes to measure files
git diff measures/growth/CAGR.dax

# Commit changes
git add measures/growth/CAGR.dax
git commit -m "Optimized CAGR calculation for better performance"

# Deploy to dev environment
tabular-editor deploy --folder . --server dev-server --database TestModel
```

---

### 2. Individual Measure Files

**Purpose:** Easy editing, searching, and reviewing individual measures

**Use Cases:**
- Quick reference for specific measure logic
- Copy-paste measures between models
- Code review of individual measures
- Search across all measures with grep/IDE

**Example:**
```bash
# Find all measures using CALCULATE
grep -r "CALCULATE" measures/

# Find measures with YoY in the name
find measures/ -name "*YoY*"
```

---

### 3. Change Log (CHANGELOG.md)

**Purpose:** Track all model changes over time

**Use Cases:**
- Understand what changed and why
- Track optimization history
- Audit trail for compliance
- Onboard new team members

**Best Practices:**
- Update with every model change
- Include before/after comparisons
- Document impact and benefits
- Reference Git commit hashes

**Example Entry Format:**
```markdown
## [v1.1.0] - 2026-01-15

### Added
- New measure: Retention Rate

### Changed
- Optimized Female Graduates (15% faster)

### Fixed
- Bug in CAGR when first year = 0
```

---

### 4. Data Dictionary (CSV)

**Purpose:** Business-friendly documentation for non-technical users

**Use Cases:**
- Training new analysts
- Documentation for business users
- Metadata repository
- Data governance compliance

**How to Use:**
```
1. Open in Excel or Google Sheets
2. Filter by Category to find measures
3. Search by Object Name
4. Share with business stakeholders
```

**Columns Explained:**
- **Object Type:** Column or Measure
- **Category:** Functional grouping
- **Description:** Business meaning
- **Example Values:** Sample data
- **Business Rules:** Calculation logic

---

### 5. JSON Export (Model_Export.json)

**Purpose:** Machine-readable format for automation and analysis

**Use Cases:**
- Import into documentation tools
- Automated testing scripts
- Model comparison utilities
- Custom analysis tools

**Example Python Usage:**
```python
import json

# Load model metadata
with open('Model_Export.json', 'r') as f:
    model = json.load(f)

# Get all optimized measures
optimized = [m for m in model['tables'][0]['measures'] 
             if m.get('isOptimized', False)]
             
print(f"Found {len(optimized)} optimized measures")

# Get all measures by category
from collections import defaultdict
by_category = defaultdict(list)

for measure in model['tables'][0]['measures']:
    by_category[measure['category']].append(measure['name'])

for category, measures in by_category.items():
    print(f"\n{category}: {len(measures)} measures")
```

---

## 🔄 Version Control Best Practices

### Git Workflow

```bash
# Feature branch for new measures
git checkout -b feature/add-retention-measures

# Make changes
# ... edit measure files ...

# Stage and commit
git add measures/retention/
git commit -m "Add retention rate measures"

# Push and create PR
git push origin feature/add-retention-measures
```

### Commit Message Convention

```
<type>: <subject>

<body>

<footer>
```

**Types:**
- `feat:` New measure or feature
- `fix:` Bug fix in measure
- `perf:` Performance optimization
- `docs:` Documentation update
- `refactor:` Code restructuring

**Example:**
```
perf: optimize CAGR calculation

- Use variables to cache year values
- Remove measure dependencies
- Add better null handling

Reduces query time by 20%
```

---

## 📊 Deployment Strategies

### Development → Test → Production

1. **Development**
   - Make changes in local .pbix file
   - Export to TMDL folder
   - Commit to Git (feature branch)

2. **Test**
   - Deploy to test workspace
   - Run automated tests
   - Validate with stakeholders

3. **Production**
   - Merge to main branch
   - Tag release (v1.1.0)
   - Deploy to production workspace

### Using Tabular Editor

```powershell
# Deploy from TMDL folder
TabularEditor.exe `
  -folder "C:\OIPA_StatisticsCanada" `
  -deploy "powerbi://api.powerbi.com/v1.0/myorg/Production" `
  -database "OIPA_StatisticsCanada"
```

---

## 🔍 Quality Assurance

### Pre-Commit Checklist

- [ ] All measures have descriptions
- [ ] Format strings are specified
- [ ] No placeholder errors
- [ ] CHANGELOG.md updated
- [ ] Data dictionary updated (if new objects)
- [ ] JSON export regenerated
- [ ] Tests pass (if applicable)

### Testing Measures

Create a `tests/` folder with validation queries:

```dax
// tests/validate_totals.dax
// Ensure all gender measures sum to total
EVALUATE
SUMMARIZECOLUMNS(
    'DataFileRecordLayout'[REF_DATE],
    "Total", [Total Graduates],
    "Female + Male", [Female Graduates] + [Male Graduates],
    "Difference", [Total Graduates] - ([Female Graduates] + [Male Graduates])
)
```

---

## 🤝 Collaboration Tips

### Code Review Process

1. Create feature branch
2. Make changes
3. Update documentation
4. Create pull request
5. Team reviews DAX code
6. Address feedback
7. Merge to main

### Documentation Standards

- **Every measure MUST have:**
  - Description
  - Category comment
  - Format string
  - Example values in data dictionary

- **Complex measures SHOULD have:**
  - Inline comments explaining logic
  - Performance notes
  - Edge case handling documentation

---

## 🛠️ Maintenance

### Regular Tasks

**Monthly:**
- Review and update CHANGELOG.md
- Regenerate exports if model changed
- Check for deprecated measures
- Update data dictionary examples

**Quarterly:**
- Audit measure usage in reports
- Identify optimization opportunities
- Clean up unused measures
- Update documentation screenshots

**Yearly:**
- Major version bump
- Comprehensive model review
- Update compatibility level if needed
- Archive old versions

---

## 📞 Support

### Getting Help

- **Model Questions:** Check Data_Dictionary.csv
- **Measure Logic:** Review individual .dax files
- **Change History:** See CHANGELOG.md
- **Technical Details:** Refer to Model_Export.json

### Contributing

To contribute improvements:

1. Fork the repository
2. Create feature branch
3. Make changes following standards
4. Update all documentation
5. Submit pull request

---

## 📝 License

Internal use - OIPA Statistics Canada Project

---

## 📚 Additional Resources

- [DAX Guide](https://dax.guide/)
- [SQLBI Best Practices](https://www.sqlbi.com/articles/best-practices/)
- [Tabular Editor Documentation](https://docs.tabulareditor.com/)
- [Power BI Documentation](https://docs.microsoft.com/power-bi/)

---

*Last Updated: 2026-01-03*  
*Version: 1.0.0*  
*Maintained by: Data Analytics Team*