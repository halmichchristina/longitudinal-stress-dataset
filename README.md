# A six-week longitudinal dataset of wearable and self-reported stress measurements in working adults: Validation Code
This repository contains the Jupyter notebooks used for data preparation and
technical validation of the RELAX longitudinal stress dataset, as described in:

> Halmich, C., Jung, O., Schmoigl-Tonis, M., Schranz, C., Kremser, W., Kunas, B.,
> & Laireiter, A.-R. (2026). *A six-week longitudinal dataset of wearable and
> self-reported stress measurements in working adults.* Sci Data (2026).

The dataset is publicly available on Zenodo:

> Halmich, C., Jung, O., Schmoigl-Tonis, M., Schranz, C., Kremser, W., Kunas, B.,
> & Laireiter, A.-R. (2026). *A six-week longitudinal dataset of wearable and
> self-reported stress measurements in working adults* (Version 0) [Data set].
> Salzburg Research Forschungsgesellschaft mbH.
> https://doi.org/10.5281/zenodo.18693288

---

## Repository contents

| File | Description |
|------|-------------|
| `technical_validation.ipynb` | Validation procedures reported in the Technical Validation section of the Data Descriptor. Requires the published dataset. |
| `prepare_data.ipynb` | Raw data processing pipeline used to produce the published Parquet files from internal InfluxDB exports. Provided for transparency; requires non-public raw data. |

---

## Requirements

The notebooks were developed with Python 3.13. The following packages are required:

```
pandas
numpy
matplotlib
fastparquet
pathlib
```

All dependencies are listed in `requirements.txt`, generated from the development environment via `pip freeze`.

Install dependencies into a virtual environment:

```bash
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

---

## Quickstart — running the technical validation

1. Download and unzip the dataset from Zenodo:
   https://doi.org/10.5281/zenodo.18693288

2. Place the extracted folder so that the directory structure matches:

```
RELAXDataset/
└── data/
    ├── <participant_id>/
    │   ├── acc_data.parquet
    │   └── ibi_data.parquet
    └── ...
```

3. Set `DATA_PATH` in `technical_validation.ipynb` to point to the `data/`
   directory, e.g.:

```python
DATA_PATH = "../RELAXDataset/data/"
```

4. Run all cells in `technical_validation.ipynb`.

---

## Notebook descriptions

### `technical_validation.ipynb`

Implements the validation procedures described in the Technical Validation
section of the Data Descriptor:

- **Structural integrity** — verifies expected files, column names, and data types
  for all 31 participants.
- **Timestamp integrity** — checks for strictly increasing timestamps, duplicates,
  and phase boundary compliance for both ACC and IBI modalities.
- **ACC sampling rate** — confirms stable ~52 Hz acquisition across participant-days.
- **IBI plausibility** — applies the physiological plausibility range filter
  (300–2000 ms) and inspects device-reported error estimates (`ibi_errorEstimate`)
  and motion quality flags (`ibi_blocker`).
- **Data completeness** — computes daily and weekly coverage per participant for
  ACC and IBI modalities using a time-based method (inter-sample intervals ≤ 5 s
  are counted as valid recording time).

### `prepare_data.ipynb`

Documents the processing pipeline used to produce the released dataset from
internal raw data exports. Raw data were stored in InfluxDB and exported as
per-week, per-signal CSV files. The pipeline:

1. Parses per-week CSV exports and merges signals into a single ACC and IBI
   DataFrame per participant.
2. Concatenates all weeks and saves intermediate CSV files.
3. Converts intermediate CSVs to Apache Parquet (SNAPPY compression) with
   corrected column types (`timestamp` as UTC datetime, `ibi_blocker` as boolean,
   `ibi_ppi` and `ibi_errorEstimate` as int32).

> **Note:** This notebook requires the non-public raw InfluxDB CSV exports and
> cannot be run from the published Zenodo dataset. It is provided for
> methodological transparency only.

---

## Dataset structure

```
RELAXDataset/
├── data/
│   ├── <participant_id>/
│   │   ├── acc_data.parquet   # Triaxial accelerometer (ACC), ~52 Hz
│   │   └── ibi_data.parquet   # Interbeat intervals (IBI), event-based
│   └── ...
├── questionnaire_responses.xlsx
└── metadata/
    ├── questionnaires.xlsx
    └── README.md
```

**ACC columns:** `timestamp` (UTC datetime), `acc_x`, `acc_y`, `acc_z` (float, g)

**IBI columns:** `timestamp` (UTC datetime), `ibi_ppi` (int32, ms),
`ibi_errorEstimate` (int32, ms), `ibi_blocker` (boolean)

---

## Known data issues

The following phase-level data absences were present in the original raw data
and are faithfully reflected in the released dataset. No imputation was applied.

| Participant | Modality | Affected period |
|-------------|----------|-----------------|
| 27 | ACC, IBI | 2024-04-21 to 2024-04-28 |
| 57 | IBI | 2024-03-03 to 2024-03-17 |
| 63 | ACC, IBI | 2024-02-25 to 2024-03-03 and 2024-04-21 to 2024-04-28 |

---

## License

The code in this repository is released under the
[MIT License](https://opensource.org/licenses/MIT).

The dataset is released under the
[Creative Commons Attribution 4.0 International License (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/).

---

## Contact
Christina Halmich — christina.halmich@salzburgresearch.at
