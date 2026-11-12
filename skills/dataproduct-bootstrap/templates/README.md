# {{DATA_PRODUCT_NAME}}

dbt data product `{{DATA_PRODUCT_ID}}` on Amazon Redshift. Published to [Entropy Data](https://entropy-data.com).

## Install

Project Python deps (dbt, the `dbt-redshift` adapter, `openlineage-dbt`, `datacontract`, `entropy-data`):

```bash
uv sync
```

`uv sync` creates `.venv/` with everything from `pyproject.toml`'s `[dependency-groups].dev`. All invocations below use the venv via `uv run` — no activation needed.

## Configure

Copy `profiles.yml.example` to `~/.dbt/profiles.yml` (or merge it in). The profile logs in with a database user and reads the password from the environment:

```bash
export REDSHIFT_PASSWORD=<your-redshift-password>
```

To use IAM instead of a password, switch the profile to `method: iam_role` (see the commented lines) and sign in to AWS, e.g. `aws sso login`.

Set the Entropy Data API key for OpenLineage transport:

```bash
export OPENLINEAGE__TRANSPORT__AUTH__APIKEY=<your-entropy-data-api-key>
```

## Run

```bash
uv run dbt-ol run
uv run dbt test
```

## Layout

```
models/
├── input_ports/      # external sources you read from
├── staging/          # 1:1 cleaned views over input ports
├── intermediate/     # joined / shaped views
└── output_ports/v1/  # published tables — one per output port
```

Output ports are versioned (`v1`, `v2`, ...). Each version directory contains the SQL models plus the ODCS data contract that governs the schema (`<contract-id>.odcs.yaml`). Cached input-port contracts live in `models/input_ports/` next to their `.source.yaml`.

## Publishing

Run the contract test locally with the same login:

```bash
export DATACONTRACT_REDSHIFT_USERNAME=<your-redshift-user>
export DATACONTRACT_REDSHIFT_PASSWORD=$REDSHIFT_PASSWORD
uv run datacontract test models/output_ports/v1/<contract-file>.odcs.yaml --server production
```

CI in `.github/workflows/data-product.yml` runs `dbt run`, `dbt test`, publishes the data product spec and contract to Entropy Data, and runs the data contract test.
