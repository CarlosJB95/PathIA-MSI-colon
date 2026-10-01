# Data card — Baseline NCT-CRC-HE (TUM vs NORM)

> **Material de aprendizaje — sin uso clínico.**

## 1. Procedencia
- **Fuente:** Kather, J. N., Halama, N., & Marx, A. (2018). *100,000 histological images of human
  colorectal cancer and healthy tissue* (Version v0.1) [Dataset]. Zenodo.
  https://doi.org/10.5281/zenodo.1214456
- **Espejo usado:** Hugging Face `1aurent/NCT-CRC-HE` (parquet).
- **Licencia:** Creative Commons Attribution 4.0 International (CC BY 4.0).
- **Obtención de los parches:** extraídos manualmente de laminillas H&E de muestras FFPE (según
  Zenodo). La página no especifica quién delineó las regiones.

## 2. Espécimen y técnica
- **Tejido:** colorrectal, tinción H&E, muestras FFPE; parches de 224×224 px a 0.5 µm/px (MPP).
- **Clase TUM:** incluye tumor primario de CRC **y** tejido tumoral de metástasis hepáticas de CRC.
- **Clase NORM:** aumentada con regiones no tumorales de especímenes de **gastrectomía** para
  incrementar la variabilidad → no es exclusivamente mucosa colónica.
- **Normalización de color:** 100K → método de Macenko (DOI 10.1109/ISBI.2009.5193250).
  7K → [COMPLETAR]
- **Clases usadas:** TUM vs NORM; 2 de las 9 clases del dataset.

## 3. Cohortes y muestreo
| Uso   | Cohorte          | Institución                        | Pacientes | Parches (TUM + NORM) |
|:------|:-----------------|:-----------------------------------|:---------:|:--------------------:|
| Train | NCT-CRC-HE-100K  | NCT Biobank (Heidelberg) + archivo UMM (Mannheim) | 86 | 600 + 600 |
| Test  | CRC-VAL-HE-7K    | Banco de tejidos del NCT (Heidelberg) | 50     | 300 + 300            |

- **Semilla:** 42 · **Caché:** `Downloads/nct_dia24.npz` (Drive, PathIA 3.0).
- **Separación por paciente:** garantizada **por cohorte**: Zenodo declara que el 7K no comparte
  pacientes con el 100K. No reverificable por parche.

## 4. Modelo y resultados (test)
- **Modelo:** regresión logística sobre 6 features de color (`features_color`, notebook 06).
- **AUC:** 0.761 (métrica principal; independiente del umbral y de la prevalencia).
- **@0.50:** sens 0.687 · espec 0.757 · VPP 0.738 · VPN 0.707.
- **Umbral:** no se declara punto de operación. Sens ≥ 0.90 exige umbral 0.08 con espec 0.213.

## 5. Limitaciones
- Sin `patient_id` por parche: la ausencia de fuga depende de la separación por cohorte.
- Umbrales explorados en test (sin conjunto de validación) → estimaciones optimistas.
- Solo 6 features de color: sensibilidad de tamizaje solo a costa de especificidad inaceptable.
- Submuestreo pequeño (33 TUM en el análisis de prevalencia 10%) → alta varianza; falta IC 95%.

## 6. Historial
- 2026-09-30 · v0.1 · Creación (Día 26). Autor: Carlos.
