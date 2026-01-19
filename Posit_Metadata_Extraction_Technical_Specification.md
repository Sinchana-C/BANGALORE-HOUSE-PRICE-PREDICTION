# Posit Connect Metadata Extraction - Technical Specification Document

## 1. Document Scope

The scope of this document is to define the technical specifications, architectural components, and workflows involved in the Posit Connect metadata extraction solution. This solution enables comprehensive analysis of R Markdown reports to identify redundant content, understand data lineage, and optimize reporting infrastructure.

---

## 2. Target Audience

The intended audience of this document includes:
- Technical SMEs at CHOP and WinWire
- Data Engineering Teams
- Business Intelligence Developers
- DevOps and Infrastructure Teams

---

## 3. In Scope

This document details the technical process for extracting metadata from Posit Connect applications to enable:
- Identification of redundant or similar reports
- Understanding of data sources and connections
- Analysis of report dependencies and lineage
- Documentation of calculated columns and formulas
- Visualization component mapping

The scope covers the extraction mechanisms for:
- **R Markdown (.Rmd) files** - Interactive dashboards and reports
- **Shiny applications** - Dynamic web applications built with R
- Resulting in standardized CSV output artifacts

---

## 4. Approach

The solution utilizes a Python-based extraction engine to parse Posit Connect R Markdown files locally. The system analyzes YAML frontmatter, R code chunks, SQL queries, database connections, and visualization components to normalize complex report logic into standardized CSV artifacts.

**Key Components:**
- **Extractor Module** - Main orchestration engine
- **Logic Module** - Core parsing and transformation logic
- **Configuration** - Environment-based settings via .env file

---

## 5. Posit Connect Inventory

Based on the current deployment analysis, the following inventory exists:

| Content Type | Count | File Format |
|-------------|-------|-------------|
| R Markdown Dashboards | TBD | .Rmd |
| Shiny Applications | TBD | .Rmd (with Shiny runtime) |

**Sample Applications:**
- FY26 Primary Care Well Visits Dashboard
- CLABSI PDSA Dashboard with Risk Analysis
- (Additional applications to be catalogued)

---

## 6. Posit Connect Metadata Extraction

The Posit metadata extraction process consists of three main phases:

### 6.1 Prerequisites

To process Posit Connect applications, the automation engine requires local access to the R Markdown source files and associated metadata.

#### Strategy 1: Posit Connect API (Primary Method)

This method utilizes the Posit Connect REST API to programmatically download content:

- **Required Authentication:**
  - API Key from Posit Connect
  - Server URL (e.g., https://connect.your-domain.com)
  
- **Capabilities:**
  - Authenticating with the Posit Connect server
  - Listing all accessible content
  - Downloading content bundles
  - Retrieving application metadata
  
- **API Documentation:** Available at `https://<your-server>/__docs__/`

**Example API Calls:**
```python
# Authentication header
headers = {
    'Authorization': f'Key {API_KEY}'
}

# List content
GET https://<server>/__api__/v1/content

# Download content bundle
GET https://<server>/__api__/v1/content/{guid}/download
```

#### Strategy 2: Direct File Access (Alternative Method)

If direct server access is available, content can be extracted from the Posit Connect data directory:

- **Default Path:** `/var/lib/rstudio-connect/data`
- **Content Location:** Each application stored in subdirectory by GUID
- **Action:** Locate .Rmd files and associated METADATA.json files

**Required Files per Application:**
- `*.Rmd` - The R Markdown source file
- `METADATA.json` - Package dependencies and runtime configuration
- `manifest.json` - Deployment manifest

---

### 6.2 Environment Configuration

The following environment setup is required to support Python development and metadata extraction:

#### 6.2.1 System Requirements

- **Operating System:** Windows 10/11, Linux, or macOS
- **Python Version:** Python 3.8 or higher
- **IDE:** Visual Studio Code (recommended) - [Download](https://code.visualstudio.com/)

#### 6.2.2 Python Environment Setup

**Step 1: Install Python Dependencies**

```powershell
# Navigate to project directory
cd C:\Users\<YourUsername>\Downloads\PositProjects\Final_metadata_extraction

# Create virtual environment (recommended)
python -m venv venv

# Activate virtual environment
.\venv\Scripts\Activate.ps1

# Install required packages
pip install python-dotenv pyyaml
```

**Required Python Packages:**
- `python-dotenv` - Environment variable management
- `pyyaml` - YAML parsing for R Markdown frontmatter
- Standard library modules: `pathlib`, `csv`, `re`, `json`

**Step 2: Install Windows Registry Support (Windows only)**

The `winreg` module (included in Windows Python) enables DSN lookup from Windows ODBC configuration.

---

### 6.3 Development Setup

This section details the programmatic configuration required to capture report metadata and serialize it into CSV format.

#### 6.3.1 Project Structure

```
Final_metadata_extraction/
├── .env                    # Environment configuration
├── data/                   # Input: Posit application folders
│   ├── {guid1}/
│   │   ├── *.Rmd
│   │   ├── METADATA.json
│   │   └── manifest.json
│   └── {guid2}/
│       └── ...
├── output/                 # Generated CSV artifacts
│   ├── {guid1}/
│   │   ├── Posit_Tables_Metadata.csv
│   │   ├── Posit_Tables_Summary.csv
│   │   ├── Posit_App_Connections.csv
│   │   └── Posit_Measured_Calculated_Columns.csv
│   └── {guid2}/
│       └── ...
└── src/                    # Source code
    ├── __init__.py
    ├── extractor.py        # Main entry point
    ├── logic.py            # Core parsing logic
    └── runner.py           # Alternative runner
```

#### 6.3.2 Environment Configuration (.env)

Create or modify the `.env` file in the project root:

```ini
# Module Configuration
MODULE_NAME=src

# Directory Paths
DATA_DIR=data
OUTPUT_DIR=output

# Process specific folder (leave blank to process all)
REPORT_FOLDER=

# File Patterns
RMD_FILE_PATTERN=*.Rmd

# CSV Output Filenames
TABLES_METADATA_CSV=Posit_Tables_Metadata.csv
TABLES_SUMMARY_CSV=Posit_Tables_Summary.csv
APP_CONNECTIONS_CSV=Posit_App_Connections.csv
MEASURED_CALCULATED_COLUMNS_CSV=Posit_Measured_Calculated_Columns.csv
```

**Configuration Options:**

| Parameter | Description | Default |
|-----------|-------------|---------|
| `MODULE_NAME` | Python module name for imports | `src` |
| `DATA_DIR` | Input directory containing .Rmd files | `data` |
| `OUTPUT_DIR` | Output directory for CSV files | `output` |
| `REPORT_FOLDER` | Specific folder GUID to process (optional) | Empty (process all) |

#### 6.3.3 Data Preparation

**Step 1: Organize Input Files**

Place Posit Connect application exports in the `data/` directory:

```
data/
  └── {application-guid}/
      ├── {report-name}.Rmd
      ├── METADATA.json
      └── manifest.json
```

**Step 2: Verify File Structure**

Each application folder should contain:
- **Exactly one .Rmd file** (or nested in subdirectories)
- **METADATA.json** with package dependencies
- **manifest.json** with deployment information

---

### 6.4 Code Implementation

#### 6.4.1 Main Extractor (extractor.py)

The extractor orchestrates the metadata extraction process:

**Key Functions:**
- Loads environment configuration
- Scans data directory for .Rmd files
- Extracts metadata using logic module
- Generates standardized CSV outputs

**Execution:**

```powershell
# Run the extractor
python src/extractor.py
```

**Output:**
```
Processing single folder: e867918a-73e4-4299-9ad1-00f268b68675
Saved: output\e867918a-73e4-4299-9ad1-00f268b68675\Posit_Tables_Metadata.csv
Saved: output\e867918a-73e4-4299-9ad1-00f268b68675\Posit_Tables_Summary.csv
Saved: output\e867918a-73e4-4299-9ad1-00f268b68675\Posit_App_Connections.csv
Saved: output\e867918a-73e4-4299-9ad1-00f268b68675\Posit_Measured_Calculated_Columns.csv
```

#### 6.4.2 Extraction Logic (logic.py)

The logic module contains the core parsing algorithms:

**Primary Functions:**

| Function | Purpose |
|----------|---------|
| `extract_yaml()` | Parses YAML frontmatter from .Rmd files |
| `extract_main_tabs()` | Identifies main dashboard sections |
| `extract_subtabs()` | Extracts tabset sub-sections |
| `extract_sql_reads()` | Captures SQL queries and data sources |
| `extract_connections()` | Identifies ODBC and Pins connections |
| `get_dsn_details()` | Resolves DSN configuration from Windows Registry |
| `extract_metadata()` | Main orchestration for metadata extraction |
| `rows_from_metadata()` | Converts metadata to CSV rows |
| `visuals_rows()` | Extracts visualization components |

**Supported Data Source Types:**
- SQL queries via `rocqi::run_sql()`
- Pin boards via `pin_read()`
- ODBC connections via `odbc::dbConnect()`
- Posit Connect pins via `board_connect()`

---

### 6.5 Extraction Process Flow

```
┌─────────────────────────────────────────────────────────────┐
│ 1. Scan data/ directory for application folders            │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│ 2. Locate .Rmd file (supports nested directories)          │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│ 3. Parse YAML frontmatter for report metadata              │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│ 4. Extract R code chunks and analyze                        │
│    - SQL queries                                            │
│    - Database connections                                   │
│    - Data transformations                                   │
│    - Pin reads                                              │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│ 5. Parse dashboard structure                                │
│    - Main tabs (identified by ====== headers)              │
│    - Sub-tabs (within {.tabset} sections)                  │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│ 6. Extract visualization metadata                           │
│    - Charts and graphs                                      │
│    - Tables and data grids                                  │
│    - Calculated columns                                     │
│    - Formulas                                               │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│ 7. Resolve database connections                             │
│    - ODBC DSN lookup (Windows Registry)                    │
│    - Extract server, database, driver details              │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│ 8. Generate CSV artifacts in output/{guid}/                │
└─────────────────────────────────────────────────────────────┘
```

---

## 7. Output Artifacts

The extraction process generates four standardized CSV files for each application:

### 7.1 Posit_Tables_Metadata.csv

**Purpose:** Documents data sources and column mappings

**Schema:**

| Column | Description | Example |
|--------|-------------|---------|
| `report_id` | Unique application identifier (GUID) | `e867918a-73e4-4299-9ad1-00f268b68675` |
| `file` | Source .Rmd filename | `CLABSI PDSA Dashboard with Risk.Rmd` |
| `title` | Report title from YAML | `CLABSI PDSA Dashboard` |
| `main_tab` | Main dashboard section | `Overview` |
| `sub_tab` | Sub-section within tabset | `Trend Analysis` |
| `dataset` | R dataframe variable name | `cohort`, `metrics_df` |
| `source_type` | Type of data source | `sql`, `pin`, `derived` |
| `source` | Table/pin name | `dbo.patient_encounters`, `pin:barmar/cohort` |
| `column_name` | Column/field name | `patient_id`, `encounter_date` |

---

### 7.2 Posit_Tables_Summary.csv

**Purpose:** Comprehensive summary including SQL queries

**Schema:**

| Column | Description | Example |
|--------|-------------|---------|
| `report_id` | Application GUID | `e867918a-73e4-4299-9ad1-00f268b68675` |
| `file` | Source filename | `CLABSI PDSA Dashboard with Risk.Rmd` |
| `title` | Report title | `CLABSI PDSA Dashboard` |
| `main_tab` | Main section | `Data Overview` |
| `sub_tab` | Sub-section | `Monthly Trends` |
| `dataset` | Dataframe name | `encounter_data` |
| `source_type` | Source type | `sql` |
| `source` | Source table | `analytics.encounters` |
| `column_name` | Column name | `admit_date` |
| `sql` | Full SQL query | `SELECT patient_id, admit_date FROM analytics.encounters WHERE...` |

**Key Feature:** Full SQL text preserved for query analysis

---

### 7.3 Posit_App_Connections.csv

**Purpose:** Database and API connection inventory

**Schema:**

| Column | Description | Example |
|--------|-------------|---------|
| `report_id` | Application GUID | `e867918a-73e4-4299-9ad1-00f268b68675` |
| `name` | Connection variable name | `conn`, `db_connection` |
| `type` | Connection type | `odbc`, `pins_connect` |
| `dsn` | ODBC DSN name | `Analytics_Prod` |
| `driver` | ODBC driver | `SQL Server Native Client 11.0` |
| `server` | Database server | `sql-prod-01.chop.edu` |
| `host` | Host/server (alternate) | `sql-prod-01.chop.edu` |
| `database` | Database name | `AnalyticsDB` |
| `uid` | Username/account | `svc_reporting` |

**Windows Registry Integration:**
- Automatically resolves DSN details from Windows ODBC configuration
- Extracts server, database, and driver information
- Falls back to connection string parameters if registry unavailable

---

### 7.4 Posit_Measured_Calculated_Columns.csv

**Purpose:** Visualization components and calculated fields

**Schema:**

| Column | Description | Example |
|--------|-------------|---------|
| `report_id` | Application GUID | `e867918a-73e4-4299-9ad1-00f268b68675` |
| `report_name` | Report title | `CLABSI Dashboard` |
| `tab_name` | Main tab | `Risk Analysis` |
| `sub_tab_name` | Sub-tab | `Comparative View` |
| `visual_name` | Visual component identifier | `monthly_trend_chart` |
| `visual_type` | Component type | `plotly`, `DT`, `valueBox`, `ggplot` |
| `column_name` | Column used in visual | `infection_rate` |
| `column_type` | Column classification | `measure`, `dimension`, `calculated` |
| `formula` | Calculation formula | `sum(cases)/sum(patient_days)*1000` |
| `source_dataset` | Source dataframe | `metrics_summary` |
| `source_sql` | Originating SQL | `SELECT ... FROM analytics.metrics...` |
| `x_axis` | X-axis field | `month_year` |
| `y_axis` | Y-axis field | `infection_rate` |

**Supported Visualization Types:**
- **plotly** - Interactive JavaScript charts
- **DT** - DataTables (interactive tables)
- **ggplot2** - Static R graphics
- **valueBox** - KPI metric boxes
- **flexdashboard gauges** - Gauge visualizations

---

## 8. Technical Implementation Details

### 8.1 R Markdown Structure Parsing

The solution understands flexdashboard-specific syntax:

**Main Tab Detection:**
```markdown
Overview
========================================
```
Pattern: Text followed by line of `=` characters

**Sub-Tab Detection:**
```markdown
Row {.tabset}
-------------------------------------
### Trend Analysis
### Risk Breakdown
```
Pattern: `{.tabset}` container with `###` sub-sections

### 8.2 SQL Query Extraction

**Pattern Recognition:**
```r
cohort <- rocqi::run_sql("
  SELECT patient_id, encounter_date
  FROM analytics.patient_encounters
  WHERE ...
", conn = db_conn)
```

**Extracted Information:**
- Dataframe variable name (`cohort`)
- Full SQL text
- Referenced tables (FROM/JOIN clauses)
- Connection variable (`db_conn`)
- Individual columns in SELECT clause

### 8.3 Pin Board Integration

**Pattern Recognition:**
```r
# Posit Connect pin board
board <- board_connect(
  server = "https://connect.chop.edu",
  account = "analytics"
)

# Read pinned data
cohort <- pin_read(board, "barmar/FY26WellVisits_cohort")
```

**Extracted Information:**
- Board connection details
- Server URL
- Account/namespace
- Pin names and references
- Dataframe mappings

### 8.4 Data Lineage Tracking

The system traces data transformations:

```r
# Base data
raw_data <- rocqi::run_sql("SELECT * FROM source_table")

# Transformation
filtered_data <- raw_data %>% 
  filter(year == 2026)

# Aggregation
summary_data <- filtered_data %>%
  group_by(month) %>%
  summarize(total = sum(value))
```

**Lineage Chain:**
`source_table` → `raw_data` → `filtered_data` → `summary_data`

---

## 9. Usage Guide

### 9.1 Process All Applications

```powershell
# Edit .env - ensure REPORT_FOLDER is blank
REPORT_FOLDER=

# Run extraction
python src/extractor.py
```

### 9.2 Process Specific Application

```powershell
# Edit .env - set specific GUID
REPORT_FOLDER=e867918a-73e4-4299-9ad1-00f268b68675

# Run extraction
python src/extractor.py
```

### 9.3 Verify Output

```powershell
# Check generated files
ls output\{guid}\*.csv

# Expected files:
# - Posit_Tables_Metadata.csv
# - Posit_Tables_Summary.csv
# - Posit_App_Connections.csv
# - Posit_Measured_Calculated_Columns.csv
```

---

## 10. Advanced Features

### 10.1 Windows ODBC DSN Resolution

The solution automatically resolves DSN details from Windows Registry:

**Registry Paths Checked:**
1. `HKEY_CURRENT_USER\SOFTWARE\ODBC\ODBC.INI\{DSN}`
2. `HKEY_LOCAL_MACHINE\SOFTWARE\ODBC\ODBC.INI\{DSN}`
3. `HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\ODBC\ODBC.INI\{DSN}`

**Extracted Attributes:**
- Server/Host
- Database name
- Driver name and version
- Additional connection parameters

### 10.2 Recursive Folder Scanning

The extractor supports nested directory structures:

```
data/
  └── {guid}/
      └── nested_folder/
          └── deep_folder/
              └── report.Rmd  ✓ Found
```

Uses `folder.glob("**/*.Rmd")` for recursive discovery.

### 10.3 Error Handling

- **Missing .Rmd:** Skips folder with warning
- **Invalid YAML:** Continues with empty metadata
- **SQL Parse Errors:** Captures partial information
- **Registry Access Denied:** Falls back to connection string

---

## 11. Troubleshooting

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| No CSV files generated | .Rmd file not found | Verify file exists in `data/{guid}/` |
| Empty metadata CSV | No SQL queries found | Check .Rmd for `rocqi::run_sql()` calls |
| Missing DSN details | Registry not accessible | Run as administrator or check DSN config |
| Import errors | Missing dependencies | Run `pip install python-dotenv pyyaml` |
| Module not found | Incorrect Python path | Verify MODULE_NAME in .env |

### Debug Mode

Enable verbose output by modifying `extractor.py`:

```python
# Add debug prints
print(f"Processing: {rmd_path}")
print(f"Metadata: {metadata}")
print(f"Connections: {conns}")
```

---

## 12. Performance Considerations

- **Processing Time:** ~2-5 seconds per application
- **Memory Usage:** Minimal (<100MB for typical reports)
- **Disk Space:** CSV files typically <1MB per application
- **Scalability:** Can process hundreds of applications sequentially

**Optimization Tips:**
- Process specific folders using `REPORT_FOLDER` parameter
- Use SSD storage for data directory
- Increase Python buffer size for large .Rmd files

---

## 13. Future Enhancements

### Planned Features
- [ ] Direct Posit Connect API integration
- [ ] Automated download of content bundles
- [ ] Excel output format option
- [ ] Data quality validation
- [ ] Duplicate report detection
- [ ] SQL query complexity scoring
- [ ] Dependency graph visualization
- [ ] Support for Quarto documents (.qmd)

---

## 14. Appendix

### A. Sample .Rmd Structure

```markdown
---
title: "CLABSI PDSA Dashboard"
runtime: shiny
output:
  flexdashboard::flex_dashboard:
    orientation: rows
---

```{r setup, include=FALSE}
library(flexdashboard)
library(DBI)
library(odbc)

conn <- odbc::dbConnect(odbc::odbc(), dsn = "Analytics")

cohort <- rocqi::run_sql("
  SELECT patient_id, line_days, infections
  FROM analytics.clabsi_data
  WHERE year = 2026
", conn = conn)
```

Overview
========================================

Row
-------------------------------------

### Infection Rate

```{r}
valueBox(
  value = sum(cohort$infections) / sum(cohort$line_days) * 1000,
  caption = "Infections per 1000 Line Days"
)
```
```

### B. Output Sample

**Posit_Tables_Metadata.csv:**
```csv
report_id,file,title,main_tab,sub_tab,dataset,source_type,source,column_name
e867918a-73e4-4299-9ad1-00f268b68675,CLABSI PDSA Dashboard.Rmd,CLABSI Dashboard,Overview,,cohort,sql,analytics.clabsi_data,patient_id
e867918a-73e4-4299-9ad1-00f268b68675,CLABSI PDSA Dashboard.Rmd,CLABSI Dashboard,Overview,,cohort,sql,analytics.clabsi_data,line_days
e867918a-73e4-4299-9ad1-00f268b68675,CLABSI PDSA Dashboard.Rmd,CLABSI Dashboard,Overview,,cohort,sql,analytics.clabsi_data,infections
```

### C. Technology Stack

| Component | Technology | Version |
|-----------|-----------|---------|
| Language | Python | 3.8+ |
| YAML Parser | PyYAML | Latest |
| Environment Config | python-dotenv | Latest |
| Regex Engine | Python re | Built-in |
| File I/O | pathlib | Built-in |
| CSV Writer | csv | Built-in |
| Registry Access | winreg | Built-in (Windows) |

---

## 15. Conclusion

This metadata extraction solution provides comprehensive visibility into Posit Connect applications, enabling data governance, redundancy identification, and infrastructure optimization. The standardized CSV outputs support downstream analytics, reporting consolidation, and migration planning initiatives.

**Key Benefits:**
- ✓ Automated metadata capture
- ✓ Standardized output format
- ✓ SQL query preservation
- ✓ Connection inventory
- ✓ Data lineage tracking
- ✓ Visualization documentation

---

**Document Version:** 1.0  
**Last Updated:** January 19, 2026  
**Prepared By:** CHOP Analytics Team  
**Classification:** Internal Technical Documentation
