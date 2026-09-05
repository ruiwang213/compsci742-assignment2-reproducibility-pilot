# COMPSCI 742 Assignment 2 — Supplementary Reproducibility Study

## Project status

This repository contains a completed supplementary reproducibility study for the COMPSCI 742 Assignment 2 group project:

> **Can Large Language Models Detect, Categorise and Explain Network Traffic Anomalies?**

The repository was developed independently by Rui Wang to validate an end-to-end experimental workflow before integration with the team study.

It provides supplementary and contingency evidence for:

* binary Benign-versus-DoS detection;
* deterministic-text versus structured-JSON input representation;
* comparison with conventional tabular models; and
* agreement between LLM-cited features and instance-level model attributions.

This repository does **not** replace the main experiments conducted by Jack, Sampath, and the wider team. In particular, the team's visual, multimodal, flag-based, and multiclass experiments remain separate from this 46-feature supplementary pipeline.

The computational workflow, statistical analyses, figures, manifests, and result inventories in this repository have been completed and verified.

---

## Scope within the group research questions

The wider team study considers four research questions. This supplementary repository contributes to three of them.

### RQ1 — Input representation and binary detection

How does input representation affect an LLM's ability to distinguish Benign from DoS network traffic?

This repository compares two information-matched representations:

1. structured JSON; and
2. deterministic natural-language text.

Both representations contain the same locked 46 feature names and values. Only the presentation format changes.

### RQ2 — Attack-category classification

How accurately can an LLM distinguish Benign traffic from individual attack categories?

This question is **outside the scope of this repository**.

The present pipeline evaluates binary Benign-versus-DoS detection only. Multiclass attack-category experiments are conducted separately in the wider team study.

### RQ3 — Explanation quality and attribution agreement

How closely do the features cited by the LLM agree with instance-level Logistic Regression and Random Forest attributions for the same network-flow record?

This repository addresses the **feature-attribution agreement component** of RQ3.

It does not claim that agreement with a conventional model is complete ground truth for explanation quality. Factuality, usefulness, causal validity, and human interpretability require additional evaluation.

### RQ4 — Conventional-model comparison

How does LLM binary anomaly-detection performance compare with conventional tabular models evaluated on the same frozen network-flow records?

This repository provides the binary comparison using leakage-safe Logistic Regression and Random Forest baselines.

It does not provide the wider team's multiclass conventional-model comparison.

---

## Dataset

The dataset used in this study is:

**NF-UNSW-NB15-v3**

The dataset contains NetFlow records with:

* a binary `Label` field;
* an `Attack` category field;
* flow timestamps;
* addressing and protocol information;
* traffic-volume and packet-count features;
* duration and throughput measurements;
* inter-arrival-time measurements; and
* additional NetFlow-v3 fields.

The raw dataset is stored locally under:

```text
data/raw/nf_unsw_nb15_v3/
```

Raw data are excluded from Git because of their size. This repository does not redistribute the original dataset.

The target `Label` and `Attack` fields are never included in the LLM input. They are retained separately for evaluation only.

---

## Final experimental design

### Frozen sample

The completed experiment uses:

* **200 frozen network-flow records**
* **46 locked original input features**
* **2 input representations**
* **400 planned LLM requests**

The two representation conditions are:

1. `deterministic_text`
2. `structured`

The deterministic-text representation is produced using fixed Python templates. No additional LLM is used to summarise, interpret, select, or rewrite the feature values.

### Response validation

Of the 400 planned LLM responses:

* **359 responses passed individual validation**
* **178 records had a complete valid response pair**
* **356 validated responses entered the paired analysis**

The complete paired population contains:

* **92 Benign records**
* **86 DoS records**

A record enters the paired representation comparison only when both its deterministic-text response and structured-JSON response pass validation.

Incomplete response pairs are not imputed or replaced.

### Controlled factors

Across the two representation conditions, the experiment holds constant:

* evaluation records;
* 46-feature input policy;
* feature values;
* units and numerical precision;
* binary label definition;
* LLM and model version;
* inference settings;
* decision instructions;
* output schema;
* validation procedure; and
* evaluation procedure.

The intended experimental difference is therefore the input representation rather than the underlying information supplied to the LLM.

---

## Notebook workflow

The complete workflow is implemented across ten documented notebooks.

| Notebook                           | Purpose                                                                                                                |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `01_data_audit.ipynb`              | Audits the raw NF-UNSW-NB15-v3 data, schema, labels, missing values, duplicates, distributions, and potential leakage. |
| `02_data_preparation.ipynb`        | Defines the preprocessing policy and prepares reproducible analysis-ready records.                                     |
| `03_pilot_sampling.ipynb`          | Creates and verifies the frozen 200-record evaluation sample.                                                          |
| `04_input_representation.ipynb`    | Produces information-matched structured-JSON and deterministic-text inputs using the locked 46-feature policy.         |
| `05_llm_pilot_inference.ipynb`     | Tests the prompt, response schema, parsing logic, and pilot inference workflow.                                        |
| `06_llm_backend_validation.ipynb`  | Validates the selected LLM backend and confirms that saved responses satisfy the required interface.                   |
| `07_llm_batch_inference.ipynb`     | Runs the 400 planned LLM requests with checkpoints and reproducible manifests.                                         |
| `08_llm_response_validation.ipynb` | Audits, validates, and freezes the LLM responses without using ground-truth labels during generation.                  |
| `09_detection_evaluation.ipynb`    | Evaluates binary detection, compares the two input representations, and constructs LR/RF out-of-fold baselines.        |
| `10_explanation_alignment.ipynb`   | Evaluates LLM-cited features against held-out LR/RF attributions and performs paired and mismatch-based inference.     |

---

## Binary detection results

### Input-representation comparison

The primary paired RQ1 analysis uses the 178 records with valid responses under both conditions.

| Input representation | Binary accuracy |
| -------------------- | --------------: |
| Deterministic text   |          0.4213 |
| Structured JSON      |          0.5000 |

The paired accuracy difference is:

```text
Structured JSON − deterministic text = +0.0787
```

The paired McNemar test produced:

```text
p = 0.0436
```

Within this supplementary 46-feature experiment, structured JSON therefore achieved approximately 7.9 percentage points higher binary accuracy than deterministic text.

This result concerns the present model, prompt, sample, and feature policy. It should not be generalised automatically to the team's visual, multimodal, flag-based, or multiclass conditions.

---

## Conventional-model baselines

Two conventional tabular models were evaluated using leakage-safe five-fold stratified cross-validation:

* Logistic Regression
* Random Forest

Preprocessing was fitted separately inside each training fold. Held-out predictions were reconstructed in the original 200-record sample order.

The final out-of-fold binary accuracies were:

| Model               | Out-of-fold binary accuracy |
| ------------------- | --------------------------: |
| Logistic Regression |                       0.975 |
| Random Forest       |                       1.000 |

The conventional models substantially outperformed both LLM representation conditions in this supplementary binary experiment.

These results show that the selected 46 tabular features contain sufficient information for highly accurate conventional binary classification. They do not imply that the conventional models solve multiclass attack-category classification.

---

## Explanation–attribution alignment design

Notebook 10 compares the five features cited in each validated LLM response with local feature attributions from two reference models.

### Logistic Regression reference

Logistic Regression contributions are evaluated on the:

```text
DoS log-odds scale
```

### Random Forest reference

Random Forest contributions are calculated using held-out TreeSHAP values on the:

```text
DoS probability scale
```

One-hot encoded features are aggregated back to the locked 46 original fields.

Because the LR and RF attribution values use different scales, their absolute magnitudes must not be compared directly with each other.

### Alignment metrics

Two confirmatory metrics are used.

#### Top-five overlap fraction

The proportion of the LLM's five cited features that also appear among the reference model's five largest absolute local attributions.

#### Attribution-mass coverage

The proportion of the reference model's total absolute local attribution captured by the five features cited by the LLM.

These metrics measure feature-attribution agreement. They do not establish causality or complete explanation correctness.

---

## Explanation–attribution alignment results

Four confirmatory paired comparisons were performed:

1. Logistic Regression × top-five overlap fraction
2. Logistic Regression × attribution-mass coverage
3. Random Forest × top-five overlap fraction
4. Random Forest × attribution-mass coverage

For every comparison, the structured-JSON and deterministic-text results were paired using the same network-flow record.

The difference was defined as:

```text
Structured JSON − deterministic text
```

The primary results were:

| Reference model     |          Alignment metric | Deterministic-text mean | Structured-JSON mean | Paired difference |            95% CI | Holm-adjusted p |
| ------------------- | ------------------------: | ----------------------: | -------------------: | ----------------: | ----------------: | --------------: |
| Logistic Regression | Top-five overlap fraction |                  0.1101 |               0.0966 |           −0.0135 | [−0.0326, 0.0056] |          0.8272 |
| Logistic Regression | Attribution-mass coverage |                  0.1008 |               0.0924 |           −0.0084 | [−0.0216, 0.0047] |          0.8272 |
| Random Forest       | Top-five overlap fraction |                  0.0517 |               0.0472 |           −0.0045 | [−0.0225, 0.0146] |          1.0000 |
| Random Forest       | Attribution-mass coverage |                  0.1070 |               0.1075 |           +0.0005 | [−0.0079, 0.0087] |          1.0000 |

None of the four Holm-adjusted tests rejected the null hypothesis at the 0.05 level.

The experiment therefore found **no statistically significant evidence** that structured JSON changed LLM–model attribution agreement relative to deterministic text.

This non-significant result is not proof that the two representations are equivalent.

---

## Supporting same-sample mismatch analysis

A class-stratified mismatch analysis tests whether an LLM explanation agrees more closely with the attribution profile for the correct record than with attribution profiles from different records of the same true class.

The mismatch null preserves:

* the Benign/DoS class;
* representation condition;
* reference model;
* global feature preferences; and
* the number of cited features.

Each of the eight model–condition–metric combinations uses 50,000 strict mismatch randomisations.

After Holm correction:

* **6 of 8 comparisons showed significantly greater same-sample alignment**
* both Logistic Regression metrics passed under both representations;
* Random Forest attribution-mass coverage passed under both representations; and
* Random Forest exact top-five overlap did not pass under either representation.

This supporting result indicates that the LLM explanations were not entirely explained by repeatedly citing globally common features.

However, correspondence with the Random Forest's exact local top-five feature set remained weak.

---

## Global reference-model attribution profiles

Reference attributions were retained for all 200 held-out records:

```text
200 samples × 46 features × 2 models = 18,400 attribution rows
```

The five largest global mean absolute Logistic Regression attributions were:

1. `MIN_IP_PKT_LEN`
2. `MAX_TTL`
3. `MIN_TTL`
4. `SERVER_TCP_FLAGS`
5. `DST_TO_SRC_AVG_THROUGHPUT`

The five largest global mean absolute Random Forest TreeSHAP attributions were:

1. `MIN_IP_PKT_LEN`
2. `MAX_TTL`
3. `MIN_TTL`
4. `SHORTEST_FLOW_PKT`
5. `SERVER_TCP_FLAGS`

The recurrence of packet-length, TTL, and TCP-flag fields suggests that these were important signals for the fitted binary reference models.

These results describe model reliance, not causal relationships.

---

## Main conclusions

This supplementary experiment supports four main conclusions.

1. **Structured JSON improved LLM binary detection accuracy in the paired 46-feature experiment.**
   Structured JSON achieved 0.5000 accuracy compared with 0.4213 for deterministic text, with a paired difference of +0.0787 and McNemar `p = 0.0436`.

2. **Both LLM conditions were substantially weaker than the conventional tabular baselines.**
   Logistic Regression achieved 0.975 out-of-fold accuracy and Random Forest achieved 1.000.

3. **Input representation did not detectably change explanation–attribution agreement.**
   None of the four confirmatory LR/RF alignment comparisons was significant after Holm correction.

4. **The LLM explanations contained some observation-specific attribution signal, but overall agreement was modest.**
   Six of eight strict mismatch tests were significant, while Random Forest top-five overlap remained weak.

In plain language:

> Presenting the same 46 network-flow features as structured JSON helped the LLM make somewhat more accurate binary predictions than presenting them as deterministic text. However, neither LLM condition approached the accuracy of Logistic Regression or Random Forest. Changing the representation also did not reliably change how closely the LLM's cited features matched the conventional models' local attributions. The explanations were not completely generic, but their agreement with conventional-model explanations was limited.

---

## Interpretation boundaries

The results must be interpreted within the following boundaries:

* This repository evaluates binary Benign-versus-DoS detection only.
* It does not evaluate nine-category attack classification.
* It does not evaluate visual, multimodal, image-plus-flag, image-plus-JSON, or flags-only conditions.
* Agreement with LR or RF attribution is not complete ground truth for explanation quality.
* Model attribution describes model reliance rather than causal importance.
* A non-significant difference is not evidence of equivalence.
* Logistic Regression and Random Forest attribution magnitudes use different scales.
* The complete paired LLM analysis contains 178 rather than all 200 records.
* The findings are specific to the frozen sample, prompt, LLM backend, and locked 46-feature policy.
* This study is supplementary to the wider group experiment.

---

## Saved results

Notebook 10 saves and verifies 16 explanation-alignment artifacts under:

```text
results/explanation_alignment/primary_46/
```

These include:

* feature-level alignment tables;
* response-level alignment summaries;
* confirmatory inference results;
* paired-effect forest plots in PNG and PDF;
* class-stratified mismatch results;
* mismatch figures in PNG and PDF;
* reference-attribution tables;
* global attribution summaries;
* global LR/RF attribution figures in PNG and PDF;
* machine-readable manifests;
* SHA-256 digests;
* a final Notebook 10 summary; and
* a verified artifact inventory.

The saved figures include publication-ready 300-dpi PNG files and vector PDF files.

---

## Repository structure

```text
compsci742-rui-pilot/
├── configs/                         Experiment and prompt configuration
├── data/
│   ├── raw/                         Original immutable data; excluded from Git
│   ├── interim/                     Intermediate data; excluded from Git
│   ├── processed/                   Processed data; excluded from Git
│   └── manifests/                   Reproducible sample identifiers
├── notebooks/
│   ├── 01_data_audit.ipynb
│   ├── 02_data_preparation.ipynb
│   ├── 03_pilot_sampling.ipynb
│   ├── 04_input_representation.ipynb
│   ├── 05_llm_pilot_inference.ipynb
│   ├── 06_llm_backend_validation.ipynb
│   ├── 07_llm_batch_inference.ipynb
│   ├── 08_llm_response_validation.ipynb
│   ├── 09_detection_evaluation.ipynb
│   └── 10_explanation_alignment.ipynb
├── results/
│   ├── inference/primary_46/
│   ├── response_validation/primary_46/
│   ├── detection_evaluation/primary_46/
│   └── explanation_alignment/primary_46/
├── reports/                         Decision logs and experiment records
├── src/                             Reusable Python functions
├── environment.yml
├── environment-lock.yml
└── README.md
```

---

## Environment

The project uses a dedicated Conda environment:

```bash
conda env create -f environment.yml
conda activate compsci742-a2
jupyter lab
```

`environment.yml` records the primary requested dependencies.

`environment-lock.yml` provides a more detailed environment snapshot for reproducibility.

---

## Reproducibility principles

The workflow follows these principles:

* preserve the original raw data without modification;
* use fixed random seeds for sampling, model fitting, bootstrap inference, and randomisation tests;
* record sample identifiers rather than relying on row position;
* lock the feature policy before formal evaluation;
* version prompts and deterministic representation templates;
* keep ground-truth labels out of all LLM inputs;
* preserve raw LLM responses and checkpoints;
* record invalid, missing, repaired, or incomplete responses;
* use leakage-safe out-of-fold conventional-model predictions;
* fit preprocessing only inside each cross-validation training fold;
* compare representation conditions on the same paired records;
* apply multiplicity correction to confirmatory hypothesis families;
* save figures in reproducible raster and vector formats;
* record artifact sizes and SHA-256 digests;
* exclude raw datasets, API keys, and local secrets from Git;
* distinguish confirmatory analyses from supporting analyses; and
* distinguish statistical association from causal interpretation.

---

## Relationship to the wider team study

This repository should be interpreted as a supplementary, independently reproducible study.

The wider team study includes additional input representations, visual and multimodal experiments, domain-derived flag conditions, and multiclass attack-category evaluation that are not reproduced here.

## Results from this repository may be integrated into the final paper where they provide:

* a controlled structured-JSON versus deterministic-text binary comparison;
* a leakage-safe LR/RF binary benchmark;
* a reproducible feature-attribution agreement analysis; and
* a contingency analysis supporting the robustness and transparency of the wider project
  
---

## Feature-selection rationale

The NF-UNSW-NB15-v3 NetFlow schema contained 53 candidate input fields. Seven fields were excluded before modelling, producing the final locked set of 46 features. FLOW_START_MILLISECONDS and FLOW_END_MILLISECONDS were excluded to reduce temporal leakage and dependence on dataset-specific collection periods. IPV4_SRC_ADDR and IPV4_DST_ADDR were excluded because these high-cardinality identifiers could allow models to memorise particular hosts or network environments rather than learn generalisable traffic behaviour. DNS_QUERY_ID was excluded because it is a transient request–response identifier without stable security meaning. Finally, SRC_TO_DST_SECOND_BYTES and DST_TO_SRC_SECOND_BYTES were removed to maintain consistency with the main experimental feature definition and because they are derived directional-rate variables that overlap with retained byte-count, duration, and throughput features. Label and Attack were retained only as outcome variables and were never provided as model inputs.
