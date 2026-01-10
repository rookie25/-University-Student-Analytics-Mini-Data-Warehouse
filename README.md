# 📊 University First-Year Retention Analytics

A comprehensive data analytics project analyzing first-year student retention across 6,000+ U.S. universities using IPEDS 2020 data, demonstrating end-to-end analytics workflow from raw data to interactive Power BI dashboard.

![Project Status](https://img.shields.io/badge/Status-Complete-success)
![Data Source](https://img.shields.io/badge/Data-IPEDS%202020-blue)
![Tools](https://img.shields.io/badge/Tools-SQL%20Server%20%7C%20Power%20BI%20%7C%20DAX-orange)

---

## 📑 Table of Contents

- [Overview](#overview)
- [Project Objectives](#project-objectives)
- [Data Source](#data-source)
- [Architecture](#architecture)
- [Methodology](#methodology)
- [Key Performance Indicators](#key-performance-indicators)
- [Dashboard Components](#dashboard-components)
- [Key Insights](#key-insights)
- [Technical Implementation](#technical-implementation)
- [Skills Demonstrated](#skills-demonstrated)
- [Deliverables](#deliverables)
- [How to Use](#how-to-use)

---

## 🎯 Overview

This project provides a deep dive into first-year student retention patterns across U.S. higher education institutions. By analyzing IPEDS 2020 data covering over 6,000 universities, the project reveals critical insights into retention performance, enrollment demographics, and the relationship between institutional size and student success.

### Key Highlights
- **6,000+** universities analyzed
- **72.60%** average retention rate
- **74.50%** enrollment-weighted retention rate
- **49.21%** of institutions achieve ≥70% retention

---

## 🎯 Project Objectives

The primary objectives of this analytics project are to:

1. **Analyze Retention Patterns** - Understand how first-year retention varies across different types of institutions
2. **Identify Performance Drivers** - Examine the relationship between enrollment size and retention success
3. **Demographic Analysis** - Explore enrollment composition across race and gender categories
4. **Benchmark Performance** - Calculate key metrics including enrollment-weighted retention rates
5. **Data Storytelling** - Create an interactive dashboard for stakeholder insights

---

## 🗂️ Data Source

### IPEDS 2020 Dataset
**Source:** Integrated Postsecondary Education Data System (IPEDS)  
**Publisher:** U.S. Department of Education's National Center for Education Statistics  
**Year:** 2020 Academic Year

### Dataset Attributes
| Attribute | Description |
|-----------|-------------|
| `unitid` | Unique institution identifier |
| `institution_name` | Name of the university |
| `total_enrollment` | Total fall enrollment count |
| `retention_rate` | First-year retention rate (%) |
| `race` | Race/ethnicity category code |
| `sex` | Gender identifier (1=Male, 2=Female, 99=Other) |
| `year` | Academic year |

---

## 🏗️ Architecture

The project follows a structured data pipeline architecture:

```
┌─────────────────────┐
│   Raw CSV Files     │
│   (IPEDS 2020)      │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│   SQL Server        │
│ • Staging Tables    │
│ • Data Cleaning     │
│ • Type Conversion   │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Star Schema Model  │
│ • Fact Tables       │
│ • Dimension Tables  │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Data Aggregation    │
│ • Summary Views     │
│ • SQL Validation    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│   Power BI          │
│ • Data Model        │
│ • DAX Measures      │
│ • Visualizations    │
└─────────────────────┘
```

### Database Schema

#### Fact Tables
- **`fact_retention`** - Retention rates by institution and year
- **`fact_enrollment`** - Enrollment data by demographics

#### Dimension Tables
- **`dim_institution`** - Institution master data
- **`dim_term`** - Academic year/term information

#### Views
- **`vw_enrollment_retention_summary`** - Aggregated institution-year grain data

---

## 🔬 Methodology

### Step 1: Data Loading & Cleaning
```sql
-- Example: Loading with type conversion and NULL handling
CREATE TABLE staging_retention (
    unitid INT,
    retention_rate DECIMAL(5,2),
    year INT
);

-- Data cleaning using TRY_CONVERT
UPDATE staging_retention
SET retention_rate = TRY_CONVERT(DECIMAL(5,2), retention_rate_raw)
WHERE TRY_CONVERT(DECIMAL(5,2), retention_rate_raw) IS NOT NULL;
```

**Key Actions:**
- Imported raw CSV files into SQL Server staging tables
- Applied `TRY_CONVERT` for safe data type conversions
- Handled NULL and invalid values explicitly
- Removed duplicates and inconsistencies

### Step 2: Data Modeling (Star Schema)

**Why Star Schema?**
- Reduces data duplication
- Optimizes query performance
- Separates facts from dimensions
- Enables scalable analytics

**Implementation:**
- Created fact tables for retention and enrollment metrics
- Built dimension tables for institutions and terms
- Established foreign key relationships using `unitid`

### Step 3: Data Standardization

**Challenge:** Raw IPEDS data contained multiple rows per institution due to demographic breakdowns, causing inflated counts.

**Solution:** Aggregated data to institution-year grain
```sql
CREATE VIEW vw_enrollment_retention_summary AS
SELECT 
    unitid,
    year,
    SUM(enrollment) AS total_enrollment,
    AVG(retention_rate) AS retention_rate
FROM fact_enrollment e
JOIN fact_retention r ON e.unitid = r.unitid AND e.year = r.year
GROUP BY unitid, year;
```

### Step 4: SQL Validation

All KPIs were calculated and validated in SQL before Power BI implementation to ensure accuracy.

**Example: Enrollment-Weighted Retention**
```sql
SELECT
    SUM(total_enrollment * retention_rate) / NULLIF(SUM(total_enrollment), 0) AS weighted_retention,
    AVG(retention_rate) AS simple_average,
    COUNT(DISTINCT unitid) AS total_universities
FROM vw_enrollment_retention_summary
WHERE total_enrollment IS NOT NULL
  AND retention_rate IS NOT NULL;
```

**Results:**
- Weighted Retention: 74.50%
- Simple Average: 72.60%
- Total Universities: 6,000+

### Step 5: Power BI Development

1. **Data Import** - Connected to SQL Server views
2. **Relationships** - Established relationships using `unitid` and `year`
3. **DAX Measures** - Created calculated measures for KPIs
4. **Visualizations** - Built interactive dashboard components
5. **Formatting** - Applied professional styling and color schemes

---

## 📊 Key Performance Indicators

| KPI | Value | Description |
|-----|-------|-------------|
| **Total Universities** | 6K | Distinct higher education institutions analyzed |
| **Average Retention Rate** | 72.60% | Simple arithmetic mean of retention rates |
| **Enrollment-Weighted Retention** | 74.50% | Retention weighted by institutional size |
| **High Performers (≥70%)** | 49.21% | Percentage of universities meeting 70% benchmark |

### DAX Measure Examples

#### 1. Enrollment-Weighted Retention (Global Context)
```dax
Enrollment Weighted Retention (Global) = 
DIVIDE(
    SUMX(
        ALL(vw_enrollment_retention_summary),
        vw_enrollment_retention_summary[total_enrollment] 
        * vw_enrollment_retention_summary[retention_rate]
    ),
    CALCULATE(
        SUM(vw_enrollment_retention_summary[total_enrollment]),
        ALL(vw_enrollment_retention_summary)
    ),
    0
)
```

#### 2. Retention Performance Categories
```dax
Retention Category = 
SWITCH(
    TRUE(),
    [retention_rate] < 0.30, "Low (<30%)",
    [retention_rate] < 0.50, "Medium (30-50%)",
    [retention_rate] < 0.70, "Good (50-70%)",
    "Excellent (≥70%)"
)
```

#### 3. High Retention Universities Percentage
```dax
High Retention % = 
DIVIDE(
    CALCULATE(
        COUNTROWS(vw_enrollment_retention_summary),
        vw_enrollment_retention_summary[retention_rate] >= 0.70
    ),
    COUNTROWS(vw_enrollment_retention_summary),
    0
) * 100
```

---

## 📈 Dashboard Components

### 1. KPI Cards (Top Metrics)
Four color-coded cards displaying key metrics:
- **Purple-Blue Gradient:** Total Universities (6K)
- **Pink-Red Gradient:** Average Retention Rate (72.60%)
- **Cyan-Blue Gradient:** Weighted Retention (74.50%)
- **Green-Turquoise Gradient:** High Performers (49.21%)

**Design Features:**
- Bold, large font for values (42pt)
- Descriptive labels with clear context
- Shadow effects for visual depth
- Consistent sizing and spacing

### 2. Retention Rate Distribution Chart
**Type:** Vertical Column Chart  
**Purpose:** Shows the distribution of universities across retention rate ranges

**Key Features:**
- Color-coded bars from red (low) to purple (high retention)
- Clear axis labels and gridlines
- Data labels on each bar
- 7 retention brackets (0-10% through 60-70%)

**Insights:**
- Most universities (1,101) fall in the 60-70% range
- Only 65 universities have critically low (<10%) retention
- Distribution skews toward higher retention rates

### 3. Total Enrollment by Demographics
**Type:** Clustered Column Chart  
**Purpose:** Compare enrollment across race and gender categories

**Key Features:**
- Side-by-side bars for Male (Blue) and Female (Red)
- Clear category labels on X-axis
- Y-axis scaled in millions
- Legend positioned at top

**Insights:**
- Race Category 1 has highest enrollment (9.5M total)
- Female enrollment consistently exceeds male across most categories
- Significant demographic concentration in top 3 categories

### 4. Enrollment Size vs Retention Rate Scatter Plot
**Type:** Scatter Plot with Color Coding  
**Purpose:** Analyze correlation between institutional size and retention

**Key Features:**
- X-axis: Total Fall Enrollment
- Y-axis: First-Year Retention Rate (0-100%)
- Color coding by performance:
  - 🔴 Red: Low (<30%)
  - 🟠 Orange: Medium (30-50%)
  - 🔵 Blue: Good (50-70%)
  - 🟢 Green: Excellent (≥70%)
- Trend line showing positive correlation

**Insights:**
- Larger institutions tend to have higher retention rates
- Smaller schools show greater variability in performance
- Clear positive correlation between size and retention

---

## 💡 Key Insights

### 1. **Size Matters for Retention**
Larger universities demonstrate consistently higher retention rates, likely due to:
- More comprehensive student support services
- Greater resource availability
- Diverse academic programs and extracurricular options
- Established retention infrastructure

### 2. **Weighted vs Unweighted Metrics**
- **Enrollment-weighted retention (74.50%)** exceeds the **simple average (72.60%)**
- This indicates that larger, better-performing institutions educate a disproportionate share of students
- Policy implications: Focus on improving smaller institutions

### 3. **Performance Distribution**
- Nearly **half of all universities (49.21%)** meet or exceed the 70% retention benchmark
- However, a significant tail of underperforming institutions exists
- 65 universities with <10% retention require urgent intervention

### 4. **Demographic Concentration**
- Top 3 race categories account for majority of enrollment
- Female enrollment exceeds male in most categories
- Category 99 (Other/Unknown) represents a small portion, suggesting good data quality

### 5. **Opportunities for Improvement**
- Universities in the 40-60% retention range represent the greatest opportunity for targeted interventions
- Best practices from high-performing institutions could be scaled
- Data-driven retention strategies should be prioritized

---

## 🛠️ Technical Implementation

### Tools & Technologies

| Tool | Purpose | Key Features Used |
|------|---------|-------------------|
| **SQL Server** | Data warehouse & ETL | Staging tables, Views, Aggregations, Validation queries |
| **Power BI Desktop** | Business intelligence | Data modeling, Relationships, DAX measures, Visualizations |
| **DAX** | Business logic | Calculated columns, Measures, Context manipulation, SUMX/CALCULATE |
| **IPEDS Database** | Data source | Public education statistics, Institution-level data |

### SQL Components

**Key Tables:**
- `staging_retention` - Raw retention data
- `staging_enrollment` - Raw enrollment data
- `fact_retention` - Cleaned retention facts
- `fact_enrollment` - Cleaned enrollment facts
- `dim_institution` - Institution dimension
- `dim_term` - Academic term dimension
- `vw_enrollment_retention_summary` - Analytics-ready view

**Sample Validation Query:**
```sql
-- Verify KPIs match between SQL and Power BI
SELECT 
    COUNT(DISTINCT unitid) AS total_universities,
    AVG(retention_rate) * 100 AS avg_retention_pct,
    SUM(total_enrollment * retention_rate) / SUM(total_enrollment) * 100 AS weighted_retention_pct,
    COUNT(CASE WHEN retention_rate >= 0.70 THEN 1 END) * 100.0 / COUNT(*) AS high_retention_pct
FROM vw_enrollment_retention_summary
WHERE retention_rate IS NOT NULL;
```

### Power BI Components

**Data Model:**
- Star schema with fact and dimension tables
- One-to-many relationships on `unitid`
- Optimized for performance with aggregated views

**Key DAX Patterns:**
- Context transition with `CALCULATE`
- Iterator functions (`SUMX`, `AVERAGEX`)
- Context removal with `ALL`
- Safe division with `DIVIDE`
- Conditional logic with `SWITCH`

**Visualization Best Practices:**
- Consistent color palette across all visuals
- Clear, descriptive titles and subtitles
- Appropriate chart types for data patterns
- Professional formatting with rounded corners and shadows
- Responsive layout for different screen sizes

---

## 🎓 Skills Demonstrated

### Data Engineering
- ETL pipeline design and implementation
- Star schema data modeling
- Data quality validation and cleaning
- Type conversion and NULL handling
- View creation for aggregated analytics

### SQL Development
- Complex aggregation queries
- Window functions and CTEs
- Performance optimization
- Data validation and testing
- View and stored procedure development

### Business Intelligence
- Power BI data modeling
- DAX measure creation and optimization
- Advanced calculated columns
- Context manipulation in DAX
- Relationship management

### Data Visualization
- Dashboard design principles
- Color theory and visual hierarchy
- Chart type selection
- KPI card design
- Interactive filtering and drill-down

### Analytical Thinking
- KPI definition and validation
- Business metric interpretation
- Statistical analysis (weighted averages, distributions)
- Insight generation from data
- Stakeholder communication

---

## 📦 Deliverables

### 1. Power BI Dashboard (.pbix)
- Interactive dashboard with 4 KPI cards and 4 visualizations
- Fully documented DAX measures
- Professional formatting and styling
- Responsive design

### 2. SQL Scripts
- Database schema creation scripts
- Data loading and transformation queries
- Validation queries for all KPIs
- View definitions for analytics

### 3. Documentation
- Comprehensive README (this document)
- Data dictionary
- DAX measure documentation
- Methodology explanation
- Insights and recommendations

### 4. Data Validation Report
- SQL vs Power BI KPI comparison
- Data quality metrics
- Null value analysis
- Duplicate record checks

---

## 🚀 How to Use

### Prerequisites
- SQL Server 2016 or later
- Power BI Desktop (latest version)
- IPEDS 2020 dataset (CSV files)

### Setup Instructions

#### 1. Database Setup
```sql
-- Create database
CREATE DATABASE UniversityRetentionDB;
GO

-- Run schema creation scripts
-- (Located in /sql/01_schema.sql)

-- Load data from CSV files
-- (Use SQL Server Import/Export Wizard or BULK INSERT)

-- Create views
-- (Located in /sql/02_views.sql)

-- Validate data
-- (Run queries in /sql/03_validation.sql)
```

#### 2. Power BI Setup
1. Open Power BI Desktop
2. Connect to SQL Server database
3. Import the following views:
   - `vw_enrollment_retention_summary`
   - `dim_institution`
   - `dim_term`
4. Establish relationships in Model view
5. Create DAX measures (documented in /dax/measures.txt)
6. Build visualizations following dashboard design

#### 3. Validation
1. Run SQL validation queries
2. Compare KPIs between SQL and Power BI
3. Verify all visualizations display correct data
4. Test filters and interactivity

### Usage Tips
- Use the scatter plot to identify outlier institutions
- Filter by retention category to focus on specific performance groups
- Hover over visualizations for detailed tooltips
- Export visuals for presentations or reports

---

## 📊 Sample Outputs

### KPI Summary
```
Total Universities: 6,000+
Average Retention Rate: 72.60%
Enrollment-Weighted Retention: 74.50%
High Performers (≥70%): 49.21%
```

### Retention Distribution
```
0-10%:   65 universities
10-20%:  27 universities
20-30%:  48 universities
30-40%: 109 universities
40-50%: 221 universities
50-60%: 656 universities
60-70%: 1,101 universities
```

---

## 🤝 Contributing

This project is part of a portfolio demonstration. For questions or suggestions:
- Open an issue for bugs or feature requests
- Fork the repository for your own analysis
- Cite this project if using in academic work

---

## 📄 License

This project uses publicly available IPEDS data from the U.S. Department of Education. The data is in the public domain. Analysis and code are provided as-is for educational and portfolio purposes.

---

## 📧 Contact

For questions about this project or collaboration opportunities, please reach out via:
- LinkedIn: [Your Profile]
- Email: [Your Email]
- Portfolio: [Your Website]

---

## 🙏 Acknowledgments

- **U.S. Department of Education** - For providing comprehensive IPEDS data
- **National Center for Education Statistics** - For maintaining the IPEDS database
- **Power BI Community** - For DAX patterns and best practices
- **SQL Server Community** - For data modeling guidance

---

**⭐ If you found this project helpful, please consider giving it a star!**

---

*Last Updated: January 2026*
