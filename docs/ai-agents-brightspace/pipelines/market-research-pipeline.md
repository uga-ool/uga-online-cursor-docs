# UGA Market Research Pipelines

This directory contains data pipelines for the UGA Market Research project, including ingestion, transformation, and export workflows.

## How It Works

- **Poetry Environment**: Scripts run using the poetry environment from `uga-online-packages`
- **Script Runner**: `run.sh` handles environment setup and python path configuration
- **Data Storage**: Downloaded/processed data is stored in `../data_raw/` and `../data_products/`
- **Two Interfaces**: Use either Make (simple tasks) or CLI (complex workflows)

## Quick Start

### Using Make (Recommended for Common Tasks)

```bash
make help                    # Show all available commands
make ipeds-metadata          # Fetch IPEDS metadata only
make ipeds-download          # Download all IPEDS data
make ipeds-download-recent   # Download IPEDS data for 2020-2024
make clean                   # Remove Python cache files
```

### Using CLI (Recommended for Advanced Usage)

```bash
# Show all commands
./run.sh src.cli --help

# IPEDS ingestion commands
./run.sh src.cli ingest ipeds --help
./run.sh src.cli ingest ipeds --metadata-only
./run.sh src.cli ingest ipeds --years 2023 --years 2024
./run.sh src.cli ingest ipeds --sleep 3 --force-metadata-refresh

# Future command groups (coming soon)
./run.sh src.cli transform --help
./run.sh src.cli export --help
```

## Available Commands

### Ingest

| Command | Description | Make | CLI |
|---------|-------------|------|-----|
| IPEDS Metadata | Fetch metadata only | `make ipeds-metadata` | `./run.sh src.cli ingest ipeds --metadata-only` |
| IPEDS All Years | Download all available data | `make ipeds-download` | `./run.sh src.cli ingest ipeds` |
| IPEDS Recent | Download 2020-2024 only | `make ipeds-download-recent` | `./run.sh src.cli ingest ipeds --years 2020 2021 2022 2023 2024` |
| IPEDS Custom Years | Download specific years | N/A | `./run.sh src.cli ingest ipeds --years YYYY YYYY` |

### Transform

| Command | Description | Make | CLI |
|---------|-------------|------|-----|
| Conferrals Pipeline | Build IPEDS conferrals data product | `make conferrals-pipeline` | `./run.sh src.cli transform conferrals` |

### Utilities

| Command | Description | Make | CLI |
|---------|-------------|------|-----|
| Clean | Remove Python cache files | `make clean` | N/A |

## Adding New Commands

### Adding a New Data Source

1. Create a new source directory: `src/sources/newsource/`
2. Add the following files:
   - `__init__.py` - Module docstring and exports
   - `download.py` - Download/ingestion logic
   - `extract_*.py` - Extraction scripts for different data types

3. Add CLI commands in `src/cli.py`:

```python
@ingest.command(name="newsource")
@click.option("--option", help="Description")
def newsource_download(option):
    """Download from new source."""
    from src.sources.newsource.download import main
    # ... call main with args
```

4. Use shared utilities from `src/common/`:

```python
from src.common import setup_logger, ProjectPaths, read_csv_flexible

logger = setup_logger(__name__)
paths = ProjectPaths(project_root)
raw_dir = paths.source_raw("newsource")
```

### Adding a Make Task

Edit `Makefile` and add a new target:

```makefile
my-new-task:
	./run.sh src.sources.newsource.download --some-flag
```

### Adding a Data Product

For products that combine multiple sources, add to `src/products/`:

```python
# src/products/combined_analysis.py
from src.common import setup_logger, ProjectPaths

def build_combined_product(project_root, logger):
    paths = ProjectPaths(project_root)
    ipeds_data = load_from(paths.source_raw("ipeds"))
    other_data = load_from(paths.source_raw("othersource"))
    return combine_and_transform(ipeds_data, other_data)
```

## Direct Script Execution

You can also run scripts directly using the module syntax:

```bash
./run.sh src.sources.ipeds.download --metadata-only
./run.sh src.products.ipeds_conferrals --project-root /path/to/project
```

## Environment Variables

- `PROJECT_ROOT`: Set automatically to `/home/jcastle/local_dev/uga-online-market-research`
- `PYTHONPATH`: Set automatically to include the pipelines directory
- `MONOREPO_ROOT`: Points to `/home/jcastle/local_dev/uga-online-packages` for poetry

## Frontend Dashboard

The project includes an interactive web dashboard built with **Vite + React + TypeScript** that analyzes IPEDS conferrals data using **DuckDB-WASM** for client-side SQL queries on parquet files.

### Key Features

- **Three Views**:
  1. **CIP Analysis**: Compare UGA against peer groups (All, Distance, R1, R1+Distance) for selected CIP codes
  2. **School Lookup**: View yearly conferrals for a specific school and CIP code over time
  3. **School Programs**: See all CIP programs offered by a school in a specific year

- **Multi-CIP Selection**: Analyze multiple CIP codes simultaneously with aggregated results
- **Client-Side Querying**: All data processing happens in the browser using DuckDB-WASM
- **Growth Metrics**: CAGR, total percentage change, and absolute change calculations
- **Interactive Visualizations**: Time series charts showing conferrals trends
- **CSV Export**: Download analysis results for further processing
- **Brightspace Ready**: Configured for deployment to Brightspace Course Files

### Architecture

```
frontend/
├── src/
│   ├── components/          # React components
│   │   ├── Filters.tsx      # Multi-CIP code selection with search
│   │   ├── ComparisonCards.tsx  # UGA vs peer group metrics
│   │   ├── TopNRankings.tsx     # Top schools by conferrals
│   │   ├── ConferralsChart.tsx  # Time series visualization
│   │   ├── SchoolLookup.tsx     # Yearly conferrals by school+CIP
│   │   ├── SchoolPrograms.tsx   # All programs at a school
│   │   └── ViewTabs.tsx         # View navigation
│   ├── hooks/
│   │   ├── useIPEDSData.ts      # DuckDB query execution hook
│   │   └── useCIPMetrics.ts     # 19-query orchestration for CIP analysis
│   ├── lib/
│   │   ├── duckdb.ts            # DuckDB-WASM initialization and queries
│   │   ├── queryBuilder.ts      # SQL query generation (9 builders)
│   │   └── csvExport.ts         # Export functionality
│   └── types/
│       └── ipeds.ts             # TypeScript interfaces and constants
└── public/
    └── data_products/           # Symlink to ../../data_products for dev

```

### Data Flow

1. **Pipeline generates parquet**: `pipelines/src/cli.py transform conferrals` creates `data_products/ipeds_conferrals/conferrals_by_cip.parquet` (2.8M rows, 16.4 MB)
2. **DuckDB loads in browser**: On page load, DuckDB-WASM fetches and registers the parquet file
3. **User selects filters**: CIP codes (multiple), years (2015-2024), award levels (13 types)
4. **Queries execute**: SQL queries run client-side against the parquet data
5. **Results display**: Metrics, rankings, charts, and tables render with growth calculations

### Key Technical Details

- **Single Parquet Load**: File loaded once, all subsequent queries use cached data
- **Manual Query Execution**: "Run Query" button prevents queries on every filter change
- **TypeScript Strict Mode**: `verbatimModuleSyntax` enabled for type safety
- **BigInt/TypedArray Handling**: Custom conversion in `duckdb.ts` to handle DuckDB numeric types
- **SQL Query Builders**: 9 specialized builders generate optimized SQL with proper escaping
- **Growth Calculations**: CAGR formula: `((lastYear/firstYear)^(1/years) - 1) * 100`
- **Peer Group Logic**: 
  - UGA: `unitid = 139959`
  - All: No filter
  - Distance: `programs_any_distance = true`
  - R1: `carnegie_class = 15` (uses C15BASIC for 2015-2017)
  - R1+Distance: Both conditions

### Development

```bash
cd frontend
npm install                  # Install dependencies
npm run dev                  # Start dev server (http://localhost:5173)
npm run build                # Build for production (creates dist/ and ZIP)
```

### Deployment to Brightspace

```bash
cd frontend
npm run build                # Creates conferrals_dashboard.zip
```

Upload `conferrals_dashboard.zip` to Brightspace Course Files and extract. The dashboard runs entirely in the browser with no server required.

### Configuration

- **Base Path**: `base: './'` in vite.config.ts for relative paths
- **No File Hashing**: Disabled for predictable filenames in Brightspace
- **Parquet Location**: `public/data_products` symlink for dev, packaged in ZIP for production
- **Year Range**: `MIN_YEAR=2015`, `MAX_YEAR=2024` in types/ipeds.ts
- **UGA UNITID**: `139959` (constant)
- **R1 Carnegie Code**: `15` (constant)

## Directory Structure

```
├── pipelines/
│   ├── README.md              # This file
│   ├── run.sh                 # Script runner with poetry environment
│   ├── Makefile               # Common tasks for quick access
│   └── src/
│       ├── cli.py             # Click CLI interface
│       ├── common/            # Shared utilities
│       │   ├── __init__.py    # Re-exports: setup_logger, ProjectPaths, etc.
│       │   ├── logging.py     # Centralized logging setup
│       │   ├── paths.py       # Project path conventions (ProjectPaths)
│       │   └── io.py          # I/O utilities (read_csv_flexible, standardize_cip_code)
│       ├── sources/           # Source-specific ingestion and extraction
│       │   └── ipeds/         # IPEDS data source
│       │       ├── download.py           # Download from NCES Data Center
│       │       ├── cip_codes.py          # CIP code reference extraction
│       │       ├── extract_conferrals.py # Conferrals data extraction
│       │       ├── extract_institutions.py # Institutions data extraction
│       │       └── extract_distance_ed.py  # Distance education extraction
│       ├── products/          # Data product builders
│       │   └── ipeds_conferrals.py  # Build integrated conferrals product
│       └── validate/          # Data validation
│           └── ipeds_conferrals.py  # Validate conferrals product
├── frontend/                  # React + TypeScript dashboard
│   ├── src/                   # Source code (components, hooks, lib)
│   ├── public/                # Static assets + data symlink
│   ├── dist/                  # Build output (gitignored)
│   └── vite.config.ts         # Build configuration
└── data_products/             # Generated data files
    └── ipeds_conferrals/
        ├── conferrals_by_cip.parquet  # Main data file (16.4 MB)
        ├── institutions.parquet       # Institution reference data
        ├── manifest.json              # Product metadata
        └── validation_report.json     # Validation results
```

## Troubleshooting

**Module not found errors**: Ensure you're running scripts through `run.sh` or `make`, not directly with python.

**Poetry environment issues**: Check that poetry is installed in `uga-online-packages`:
```bash
cd /home/jcastle/local_dev/uga-online-packages
poetry env info
```

**Permission denied on run.sh**: Make it executable:
```bash
chmod +x run.sh
```
