# COMPSCI 742 Assignment 2 — Reproducibility Pilot

## Project status

This repository is a personal working environment supporting the
COMPSCI 742 Assignment 2 group project. It is used to independently
validate the experimental workflow and produce reproducible components
for later integration into the team repository.

It is intended to validate the complete experimental workflow on a
small development sample before the methodology is frozen or the
implementation is integrated into the team repository.

The current contents are provisional and should not be interpreted as
final experimental results.

## Overall research topic

**Can Large Language Models Detect, Categorise and Explain Network
Traffic Anomalies?**

## Current research questions

### RQ1 — Binary anomaly detection

How does changing the input representation of network-flow data affect
an LLM's ability to distinguish anomalous from benign traffic?

The LLM prediction will be compared with the binary `Label` field in
the dataset.

### RQ2 — Attack-category classification

How accurately can an LLM assign a network flow to one of the attack
categories defined by the dataset?

The LLM category prediction will be compared with the `Attack` field
in the dataset.

### RQ3 — Anomaly explanation

How well can an LLM explain its anomaly decision?

A small, pre-specified subset of explanations will be evaluated using
a documented manual-review rubric. This analysis is exploratory
because the dataset does not provide ground-truth explanations.

### RQ4 — Conventional-model comparison (optional)

How does LLM anomaly detection compare with conventional tabular
models such as Logistic Regression and Random Forest?

These models may also provide independent feature-attribution
references. TreeSHAP produces feature attributions, not attack
categories.

## Dataset

The current dataset is:

**NF-UNSW-NB15-v3**

The dataset contains NetFlow records with:

- a binary `Label` field;
- an `Attack` category field;
- flow timestamps;
- addressing and protocol information;
- traffic-volume and packet-count features;
- duration, throughput and inter-arrival-time features; and
- additional NetFlow-v3 features.

The original dataset is stored locally under:

`data/raw/nf_unsw_nb15_v3/`

Raw data are intentionally excluded from Git because of their size.
The repository does not redistribute the dataset.

## Initial representation conditions

The initial controlled pilot will prioritise:

1. structured feature values; and
2. deterministic template-based natural-language descriptions.

For the deterministic text condition, the same fields, values, units
and numerical precision will be converted into prose using fixed
Python templates. No separate LLM will summarise or interpret the
record.

Visual and combined representation conditions remain provisional.
They will be implemented only after the analysis unit and
information-equivalence rules are clearly defined.

## Scientific controls

Across representation conditions, the experiment should hold constant:

- evaluation records;
- feature set;
- feature values;
- units and numerical precision;
- label definitions;
- model and model version;
- inference settings;
- decision instructions;
- output schema; and
- evaluation procedure.

The target record's `Label` and `Attack` values must never be included
in the LLM input. They are retained only as ground truth for scoring.

## Development stages

### Stage 1 — Data audit

Audit the complete raw dataset without modifying it:

- file integrity;
- dimensions and schema;
- data types;
- class distribution;
- missing and non-finite values;
- duplicate records;
- label consistency;
- timestamp coverage; and
- potential leakage features.

### Stage 2 — Smoke pilot

Create a reproducible, stratified development sample of approximately
200 records, provisionally 20 records from each of the ten classes,
subject to the data audit.

This sample is for pipeline development only and will not be treated
as the final evaluation set.

### Stage 3 — Expanded development pilot

After the smoke pilot works end to end, create a larger development
sample for preprocessing and conventional baseline development.

### Stage 4 — Frozen evaluation

Define and freeze the final data split, feature set, prompts, model
settings, sample sizes and statistical analysis before producing
reportable results.

## Repository structure

```text
compsci742-rui-pilot/
├── configs/       Experiment and prompt configuration
├── data/
│   ├── raw/       Original immutable data; excluded from Git
│   ├── interim/   Intermediate data; excluded from Git
│   ├── processed/ Processed data; excluded from Git
│   └── manifests/ Reproducible sample identifiers
├── notebooks/     Documented Jupyter notebooks
├── outputs/
│   ├── figures/   Generated figures
│   ├── metrics/   Evaluation summaries
│   └── llm_responses/ Raw responses; excluded from Git
├── reports/       Decision log and experiment records
├── src/           Reusable Python functions
├── environment.yml
└── environment-lock.yml

## Planned notebooks
01_data_audit.ipynb
02_pilot_sampling.ipynb
03_representation_generation.ipynb
04_llm_pilot.ipynb
05_baseline_models.ipynb
06_evaluation.ipynb
Environment

The project uses a dedicated Conda environment:
conda env create -f environment.yml
conda activate compsci742-a2
jupyter lab

environment.yml records the main requested dependencies.
environment-lock.yml provides a more detailed environment snapshot.

## Reproducibility principles
Preserve the original data without modification.
Use fixed random seeds for sampling and model fitting.
Record sample identifiers rather than relying on row position alone.
Version prompts and representation templates.
Separate development samples from the frozen final evaluation set.
Record invalid, repaired, timed-out and missing LLM outputs.
Do not commit datasets, API keys or local secret files.
Clearly distinguish exploratory findings from confirmatory results.
