# Posit Connect Metadata Extraction - Technical Specification Document

## 1. Document Scope
The scope of this document is to define the technical specifications, architectural components, and workflows involved in the Posit Connect metadata extraction solution.

## 2. Target Audience
The intended audience of this document includes the Technical SMEs at CHOP and WinWire.

## 3. In Scope
This document details the technical process for extracting metadata to enable the identification of redundant or similar reports. The scope covers the extraction mechanisms for R Markdown (.Rmd) files and Shiny applications, resulting in standardized output generation.

## 4. Approach

The solution utilizes a Python application to parse R Markdown files locally and extract metadata including SQL queries, database connections, dashboard structure, and visualization components. The workflow normalizes complex report logic into standardized CSV artifacts.

## 5. Posit Connect Inventory from Snowflake

Based on the details from snowflake, we have found below inventory for Posit Connect:

•	R Markdown Dashboards – TBD
•	Shiny Applications – TBD


## 5.1 R Markdown Metadata Extraction
This R Markdown metadata extraction is executed in below steps.

1.	Prerequisite
2.	Environment Configuration
3.	Development Implementation

### 5.1.1 Prerequisite
To process report metadata recursively, the automation engine requires physical access to .Rmd source files. Two strategies have been identified for retrieving these files from the Posit Connect Server.

**Strategy 1: Posit Connect API (Primary Method)**
This method utilizes the Posit Connect REST API to programmatically connect to the server and download content.
•	Required Components: API Key, Server URL
•	Capabilities:
    o	Authenticating with the Posit Connect server
    o	Navigating the content library
    o	Downloading content bundles
    o	Retrieving application metadata
•	API Documentation: Available at https://<your-server>/__docs__/

**Strategy 2: Direct File Access (Alternative Workaround)**
If possible, a direct extraction from the server's data directory is a viable alternative.

•	Path: /var/lib/rstudio-connect/data or C:\ProgramData\RStudio\rstudio-connect
•	Action: Verify the availability of .Rmd files in application directories and transfer to the local development environment.

### 5.1.2 Environment Configuration
The following environment setup is required to support the Python development and metadata capture process:
•	IDE: Install Visual Studio Code - [Link](https://code.visualstudio.com/)
•	Runtime: Install Python 3.8 or higher - [Link](https://www.python.org/downloads/)
•	Required Libraries: Install python-dotenv and pyyaml
•	Input Data Source: Place the .Rmd files exported from Posit Connect in the data/ folder and utilize this path as an input to read the files.

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

### 5.1.3 Development Setup
This section details the programmatic configuration required to capture report metadata and serialize it into CSV format.

•	Project Setup:
    o	Create project folder: Final_metadata_extraction
    o	Create subdirectories: data/, output/, src/
    o	Place .Rmd files in data/{guid}/ folders
    
•	Environment Configuration:
    o	Create .env file in project root
    o	Set configuration parameters:
 
```ini
MODULE_NAME=src
DATA_DIR=data
OUTPUT_DIR=output
REPORT_FOLDER=
```

•	Install Dependencies:
```powershell
pip install python-dotenv pyyaml
```

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
•	Code Implementation:
    o	The src/extractor.py file orchestrates the extraction process
    o	The src/logic.py file contains parsing logic for:
        - YAML frontmatter extraction
        - SQL query parsing
        - Database connection extraction
        - Dashboard structure analysis
        - Visualization metadata capture
    o	IMPORTANT: Update the .env file to specify which folder to process
    
    o	Validation: Upon execution, the extractor generates CSV artifacts in the designated output folder containing the parsed metadata for further analysis.

## Output Artifacts
The script processes the .Rmd files and serializes the data into structured CSV files, organized by application GUID.

•	Output Path: /output/{id}/

| File Name | Description | Key Columns |
|-----------|-------------|-------------|
| Posit_Tables_Metadata.csv | Data sources and column mappings | report_id, file, title, main_tab, sub_tab, dataset, source_type, source, column_name |
| Posit_Tables_Summary.csv | Comprehensive summary with SQL queries | All columns from Tables_Metadata plus sql (full query text) |
| Posit_App_Connections.csv | Database connection inventory | report_id, name, type, dsn, driver, server, database, uid |
| Posit_Measured_Calculated_Columns.csv | Visualization components and formulas | report_id, report_name, tab_name, sub_tab_name, visual_name, visual_type, column_name, column_type, formula, source_dataset, source_sql, x_axis, y_axis |
