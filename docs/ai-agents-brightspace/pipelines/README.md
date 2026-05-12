# Pipelines

Offline data processing pipelines for agents that consume large or pre-processed datasets.

## Convention

Each agent with data processing needs gets a subfolder:

```
pipelines/<agent-name>/
├── Makefile            # Build targets for the pipeline
├── run.sh              # Runner script (sets PYTHONPATH, etc.)
├── requirements.txt    # Python dependencies (or pyproject.toml)
├── src/                # Pipeline source code
│   ├── sources/        # Data download / extraction
│   ├── products/       # Data product builders
│   ├── validate/       # Validation checks
│   └── common/         # Shared utilities (paths, logging, io)
├── data_raw/           # Raw downloaded data (gitignored)
└── data_products/      # Processed output files (gitignored)
```

## Key Principles

- **Pipelines are offline/CI processes.** They are never run during `npm run build:agent`. The framework build system only packages their output.
- **`data_raw/` and `data_products/` are gitignored** by default. Large binary files should not be committed. Use LFS or external storage for sharing raw data across machines.
- **Each pipeline is self-contained.** It manages its own Python (or other language) dependencies independently of the Node.js frontend workspace.
- **Output convention:** Processed files go to `data_products/`. The agent's `.env` declares a `DATA_PRODUCTS_PATH` pointing here, which the build system uses for `--with-data` builds and the Vite dev server uses for local serving.

## Connecting a Pipeline to an Agent

1. Create your pipeline under `pipelines/<agent-name>/`.
2. In the agent's `.env` (`frontend/src/agents/<agent-name>/.env`), set:
   ```
   DATA_PRODUCTS_PATH=../../pipelines/<agent-name>/data_products
   ```
3. Run `npm run dev:agent -- <agent-name>` — the dev server will serve `data_products/` from the pipeline output directory.
4. Build with data: `npm run build:agent -- <agent-name> --with-data` produces a full ZIP including data.
5. Build without data: `npm run build:agent -- <agent-name>` produces an app-only ZIP (deploy data separately).

## Existing Pipelines

- **market-research** — IPEDS and NC-SARA data ingestion, transformation, and parquet product generation (Python + Polars)
