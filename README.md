# BEDCA Food Composition Dataset

[![DOI](https://zenodo.org/badge/1339567893.svg)](https://doi.org/10.5281/zenodo.22017000)

Food composition data from the Spanish Food Composition Database,
retrieved from <https://www.bedca.net/> (the database app at
`https://www.bedca.net/bdpub/`) in August 2026 via the site's public XML
query interface, at a polite rate (one sequential request every 2.5 s).

## Download

- **GitHub release** — `bedca-dataset-2026.zip`: the complete dataset
  (README, LICENSE, `output/` CSV files, manifest). Verify with the
  published SHA-256 checksum in `bedca-dataset-2026.zip.sha256`:
  `sha256sum -c bedca-dataset-2026.zip.sha256`
- **Zenodo archive** — <https://doi.org/10.5281/zenodo.22017000>: a
  citable, versioned record of this repository. The concept DOI
  `10.5281/zenodo.22017001` always resolves to the latest version.

## Attribution

**Source: AESAN/BEDCA Base de Datos Española de Composición de Alimentos v1.0 (2010)**

This attribution must accompany any reproduction or publication of this
data, including in application credits.

## Terms of use (summary)

Per the AESAN/BEDCA conditions of use (mirrored in `LICENSE`, original in
`terms-of-use-txt`):

- The data may be reproduced only with clear indication of the original
  source (see the attribution string above).
- The data may not be modified, nor may its original meaning be altered.
  All values in this dataset are therefore stored **verbatim**, exactly as
  returned by the database (no rounding, no unit conversion, no
  normalization).
- Use is limited to personal, educational, or **non-commercial** purposes.
  Reproduction, translation, or any other use of the data in electronic
  format requires the express authorization of AESAN/BEDCA.
- Errors in the data should be reported to bedca.adm@gmail.com.

## Contents (`output/`)

| File | Format | Description |
|---|---|---|
| `bedca_foods.csv` | wide | 1 row per food. Food metadata, then one value + one unit column per nutrient component (e.g. `ENERC`, `ENERC_unit`). Values verbatim. UTF-8 with BOM, CRLF. |
| `bedca_foodvalues.csv` | long | 1 row per (food, component) — the full-fidelity record, including per-value `stdv`, `min`, `max`, `v_n`, scientific `citation`/`ref_id`, and method provenance. |
| `bedca_units.csv` | long | Distinct component/unit combinations (`v_unit`, `u_descripcion`, `mu_descripcion`) — the unit reference for downstream merging. |
| `bedca_index.csv` | long | Food list snapshot: `f_id`, Spanish/English names, LanguaL codes, `f_origen`, `edible_portion`. |
| `manifest.json` | JSON | Fetch log per food (status, bytes, retries), failures, and the parse report. |

Field definitions come from the EuroFIR documentation referenced by
AESAN/BEDCA; key ones: `best_location` = the reported value, `v_unit` =
unit of the value, `moex` = method of expression (`W` = per 100 g of edible
portion), `value_type` = `AR` as reported / `BE` best estimate,
`edible_portion` = edible fraction as stored by the source database.

## Usage

The CSV files are UTF-8 with BOM + CRLF. Load them in pandas with
`encoding="utf-8-sig"` — this strips the BOM and reads Spanish diacritics
correctly:

```python
import pandas as pd

foods = pd.read_csv("output/bedca_foods.csv", encoding="utf-8-sig")
foodvalues = pd.read_csv("output/bedca_foodvalues.csv", encoding="utf-8-sig")
```

`bedca_foods.csv` has one row per food (one value + one unit column per
nutrient); `bedca_foodvalues.csv` has one row per measured component with
full provenance. See the table above for column details.

## Known data characteristics

- The public database returned by the site's list query contains 969
  foods (public subset, `publico=1`), more than the commonly cited ~500.
- **957 foods have composition data** (31,166 component values); **12
  foods** are listed in the index but return an empty `<food/>` element
  from the detail endpoint — they have no public data. Their ids are
  recorded in `manifest.json` → `empty` (e.g. 2412, 2509, 2702–2710,
  2714).
- 47 of the catalog's components occur in the data; per-food coverage
  varies (roughly 30 components per food), and a food may carry a
  component row with an empty value.
- `f_origen` contains both `BEDCA` and `BEDCA2` values — a property of the
  source database's own filtering, reproduced verbatim.
- Five components appear with more than one unit across foods (`BIOT`,
  `CHO`, `ENERC`, `FIBT`, `WATER`; see `manifest.json` →
  `parse.multi_unit_components`). Values and units are stored as-is;
  downstream merging logic decides how to handle that.
- Run `python bedca_verify.py` to re-run the full verification suite.
