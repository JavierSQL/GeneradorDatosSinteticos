# Synthetic Data Generator for Microsoft Fabric

📖 [Versión en español disponible](README.md)

Two notebooks for Microsoft Fabric that work in sequence:

1. **`GeneradorDatosSinteticosPD.ipynb`** — Generates synthetic datasets from YAML configurations and writes them as Parquet files to the Lakehouse.
2. **`CargadorTablas.ipynb`** — Reads those Parquet files and registers them as Delta tables in the schema of your choice.

---

## Who is this for?

**Demo and content creators** — If you build Power BI reports or Fabric solutions and need sample data that looks real (names, dates, amounts, categories with realistic distributions), this generator lets you create a complete dataset in minutes without relying on production data.

**Professionals working with sensitive data** — When the real dataset contains personal or confidential information, synthetic data lets you share, publish, or deliver your solution without exposing anything. The generated data is statistically plausible but completely fictional.

**Testers and QA** — Simulate specific volumes (100 rows for development, 1 million for load testing), edge cases (nulls, extreme values, skewed distributions), or concrete business scenarios — all from a configuration file, without touching the code.

---

> ⚠️ **Required platform**: Microsoft Fabric with an attached Lakehouse. Not compatible with Databricks or local Spark in this version.

---

## Step by step

### 1. Download the notebooks

On this GitHub page, download the two `.ipynb` files:

- Click `GeneradorDatosSinteticosPD.ipynb` → **Download raw file**
- Repeat for `CargadorTablas.ipynb`

---

### 2. Import into your Fabric workspace

1. Open your workspace at [Microsoft Fabric](https://app.fabric.microsoft.com)
2. Click **+ New item** → **Import notebook**
3. Select the two downloaded `.ipynb` files
4. Make sure the notebook has a **Lakehouse attached** (left panel of the notebook)

---

### 3. Install dependencies

Open `GeneradorDatosSinteticosPD.ipynb` and run the first installation cell:

```python
%pip install faker faker-commerce pyyaml tqdm
```

> Restart the kernel after installation if Fabric prompts you to.

---

### 4. Create your YAML configuration

Start from one of the examples in `Ejemplos/comunidad/` and adapt it to your use case.
The minimum structure is:

```yaml
dataset:
  name: my_table
  description: "Dataset description"
  rows: 1000
  schema: MySchema          # Destination schema in the Lakehouse
  output: "Files/Data/MyFolder"
  columns:
    - name: ID
      type: int
      sequence: {start: 1, step: 1}
    - name: Name
      type: string
      faker: name
```

See the **Available generators** section for all options.

---

### 5. Validate the YAML syntax

Before uploading the file, paste its content into **[yamllint.com](https://www.yamllint.com/)** and verify there are no indentation or formatting errors. YAML errors are silent and hard to debug inside the notebook.

---

### 6. Upload the YAML to the Lakehouse

1. In the Lakehouse panel, navigate to the folder where you want to store your configurations (e.g., `Files/config/`)
2. Click **...** → **Upload files**
3. Select your `.yml` file

---

### 7. Configure and run the generator

In `GeneradorDatosSinteticosPD.ipynb`, edit **Cell 2**:

```python
FAKER_LOCALE      = "es_ES"              # Language: es_ES, en_US, fr_FR, etc.
RANDOM_SEED       = 42                   # None = random on every run
BASE_OUTPUT_PATH  = "Files"

YAML_FILES = [
    "Files/config/my_table.yml",         # Path to your YAML in the Lakehouse
]
```

Run all cells with **Run all**. Parquet files are written to the `output` folder defined in your YAML.

---

### 8. Load as Delta tables (optional)

If you want the data available as tables in the Lakehouse, open `CargadorTablas.ipynb` and edit **Cell 1**:

```python
DESTINATION_SCHEMA = "dbo"              # Default destination schema
                                        # The YAML 'schema' field overrides this per dataset
WRITE_MODE         = "overwrite"        # "overwrite" replaces | "append" adds rows
YAML_FILES         = [
    "Files/config/my_table.yml",        # Same list as in the generator
]
```

Run all cells. Each Parquet file is registered as `{schema}.{name}` in the Lakehouse.

---

## Available generators

| Generator | Description | YAML example |
|-----------|-------------|-----------------|
| `sequence` | Sequential values | `sequence: {start: 1, step: 1}` |
| `template` | Template with formatted sequence | `template: "SKU-{sequence:04d}"` |
| `faker` | Realistic fake data (names, emails, etc.) | `faker: name` / `faker: email` |
| `values` | Randomly picks from a fixed list | `values: ["A", "B", "C"]` |
| `range` | Value within a range (int, decimal, or date) | `range: [1, 100]` / `range: ["2020-01-01", "2024-12-31"]` |
| `weighted_values` | List with probabilities per value | `weighted_values: [{value: "X", weight: 0.7}, ...]` |
| `conditional` | Value that depends on another column | `conditional: {based_on: Dept, mappings: {...}}` |
| `calculated` | Python formula using already-generated columns | `calculated: {based_on: [A, B], formula: "A * B"}` |
| `fk` | Foreign Key from an existing Parquet | `fk: {source: "Files/Data/stores.parquet", column: StoreKey}` |
| `nullable` | Percentage of NULLs in the column | `nullable: 0.30` (30% NULL) |

**Supported column types**: `int`, `string`, `decimal`, `bool`, `date`, `datetime`

---

## Included examples

Located in `Ejemplos/comunidad/` — all independent, ready to use:

| File | Domain | Rows | Featured generators |
|------|--------|-----:|---------------------|
| `employees.yml` | Human Resources | 500 | sequence, faker, conditional, weighted_values, bool, nullable |
| `products_catalog.yml` | E-commerce | 300 | template, calculated (gross margin) |
| `support_tickets.yml` | SaaS / Support | 1,000 | weighted_values, conditional, nullable (open tickets) |
| `iot_readings.yml` | IoT / Industry | 10,000 | calculated (Heat Index), weighted_values (alerts) |

---

## Generate YAMLs with AI

The community examples are a great starting point for asking Claude Code (or another LLM) to generate a YAML tailored to your industry or use case.

**How to do it:**

1. Download this `README-EN.md` and one or more YAMLs from `Ejemplos/comunidad/` as reference
2. Open Claude Code (or your preferred LLM) and use this prompt:

```
I have a synthetic data generator that uses YAML configuration files.
Here is the complete generator documentation: [paste README-EN.md content]
Here is an example YAML: [paste employees.yml or any other example]

Generate a YAML for [industry or domain, e.g., "a medical clinic" / "a restaurant chain" / "a lending fintech"].
The dataset should represent [brief description, e.g., "medical visit history with 2000 rows"].
Include realistic columns for the domain and distributions that make business sense (e.g., more records in frequent categories, some nulls where appropriate).
```

The LLM will have the full reference — available generators, exact syntax, and real examples — to produce a ready-to-use YAML or something very close to it.

---

## Adapting paths to your Lakehouse

The YAMLs use paths relative to the Lakehouse root (`Files/`). Adjust `output` to match your folder structure:

```yaml
dataset:
  output: "Files/Data/MyFolder"   # ← adapt to your folder
```

For FK lookups, also adjust `fk.source`:

```yaml
fk:
  source: "Files/Data/MyFolder/my_table.parquet"  # ← adapt
  column: MyKey
```

---

## Historical datasets (SCD Type 2)

To generate historical changes over an existing dataset, use `source_parquet` instead of `rows`:

```yaml
dataset:
  name: product_changes
  source_parquet: "Files/Data/Products/products_base.parquet"
  change_generation:
    change_ratio: 0.15              # 15% of records will have changes
    num_changes_per_product: [1, 3] # Between 1 and 3 changes per record
  columns:
    - name: ProductKey
      type: int
      from_source: "ProductKey"     # Copied from the source parquet
    - name: ListPrice
      type: decimal
      from_source: "ListPrice"
      variation: [-0.20, 0.20]      # ±20% of the original value
```

---

## Advanced YAML validation

The file `Ejemplos/schema_dataset.yaml` defines the valid structure of a configuration YAML. To validate locally with [yamale](https://github.com/23andMe/Yamale):

```bash
pip install yamale
yamale -s Ejemplos/schema_dataset.yaml Ejemplos/comunidad/employees.yml
```

---

## License

MIT — see [LICENSE](LICENSE)
