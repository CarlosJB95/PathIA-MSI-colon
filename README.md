# PathIA-MSI-colon

Ruta de formación de 12 meses en IA aplicada a patología digital computacional (colon y mama), con meta final de predecir MSI-H a partir de histología H&E. Este repositorio documenta los notebooks, data cards y resultados de cada etapa.

> **Material de aprendizaje — sin uso clínico.** Ningún resultado de este repositorio debe usarse para diagnóstico ni decisiones clínicas.

## Estado actual · `v0.2-mes2-sem2` (Mes 2, Semana 2)

**Notebook `06_intro_clasificacion`:** estudio de factibilidad TUM (epitelio tumoral) vs NORM (mucosa normal) a nivel de parche, con features hechas a mano y regresión logística.

| Modelo | Features | AUC (test) | Sens / Espec @0.50 |
|:--|:--|:--:|:--:|
| Solo color | 6 (media y DE por canal RGB) | 0.752 | 0.673 / 0.750 |
| Color + textura | 11 (+ 5 descriptores de Haralick, GLCM) | **0.936** | 0.887 / 0.877 |

Comparación controlada: mismo split por cohorte, mismo pipeline (`StandardScaler` + `LogisticRegression`), semilla 42; único cambio, el vector de features; test evaluado una sola vez. **Sin intervalos de confianza todavía** (bootstrap pendiente).

## Datos

- **Fuente:** Kather, J. N., Halama, N., & Marx, A. (2018). *100,000 histological images of human colorectal cancer and healthy tissue* (Version v0.1) [Dataset]. Zenodo. https://doi.org/10.5281/zenodo.1214456
- **Licencia:** Creative Commons Attribution 4.0 International (CC BY 4.0).
- **Espejo usado:** Hugging Face `1aurent/NCT-CRC-HE` (parquet).
- **Split por cohorte:** train = NCT-CRC-HE-100K (86 pacientes; NCT Biobank + archivo UMM) · test = CRC-VAL-HE-7K (50 pacientes; NCT). Zenodo declara que las cohortes no comparten pacientes; no hay `patient_id` por parche, así que no es reverificable parche por parche.
- **Submuestreo:** 600 TUM + 600 NORM (train) y 300 + 300 (test), semilla 42.
- Procedencia completa y limitaciones: [`data_cards/nct_crc_baseline.md`](data_cards/nct_crc_baseline.md) (v0.2).

## Cómo correr

1. Abrir `06_intro_clasificacion.ipynb` en Google Colab.
2. *Ejecutar todo*. La primera celda monta Google Drive y **se detiene si el montaje falla** (`assert os.path.ismount`), para no escribir en una carpeta local que desaparece.
3. Los datos se cargan desde caché en Drive; solo si no existe el caché se descarga el subconjunto de Hugging Face (una vez).

**Ruta del proyecto en Drive:** `MyDrive/DigiPath/IA Docs/PathIA 3.0/`

| Caché | Contenido |
|:--|:--|
| `Downloads/nct_dia24.npz` | Parches RGB 224×224: `Xtr` (1200), `Xte` (600) y etiquetas |
| `Downloads/nct_textura_d1-3_n32.npz` | Features de textura `Xtr_tex` (1200, 5), `Xte_tex` (600, 5) + sus parámetros (distancias 1 y 3 px, 32 niveles); se recalcula si cambian |

Los cachés **no** se versionan en el repositorio (son datos).

## Estructura y convenciones

```
notebooks/     06_intro_clasificacion.ipynb
data_cards/    nct_crc_baseline.md        ← generado desde el notebook (no editar a mano)
results/       06_*.png                   ← figuras
```

- **Prefijo de figuras = número del notebook:** `results/06_matriz_confusion.png`, `results/06_roc_comparacion.png`, etc.
- **Cachés nombrados por contenido**, no por día.
- El data card se escribe **al final** del notebook (§9) con los números insertados desde variables, para que nunca se desincronice de los resultados.

## Limitaciones principales

- **Imágenes normalizadas con Macenko** en NCT-CRC-HE-100K; para CRC-VAL-HE-7K la normalización no está declarada explícitamente en Zenodo.
- **Test de una sola institución** (NCT): no evalúa generalización entre centros.
- **Desplazamiento de distribución** documentado entre 100K y 7K: la brecha train→test puede reflejar diferencias entre cohortes, no solo capacidad del modelo.
- **Procedencia de las clases:** TUM incluye metástasis hepáticas de CRC; NORM fue aumentada con regiones no tumorales de gastrectomía.
- La textura podría captar también **artefactos de adquisición o de procedencia**, no solo morfología.
- **Sin punto de operación declarado:** los umbrales se exploraron en test con fines ilustrativos; un umbral definitivo requiere conjunto de validación independiente, uso clínico definido y prevalencia de la población destino.
- Sin IC 95% (bootstrap pendiente).

## Versiones

| Tag | Contenido |
|:--|:--|
| `v0.1-mes2-sem1` | Baseline de color: tabla 2×2, prevalencia, ROC/AUC, punto de operación ilustrativo, data card v0.1 |
| `v0.2-mes2-sem2` | Textura GLCM/Haralick, comparación controlada (AUC 0.752 → 0.936), data card v0.2, QC de reproducibilidad |

## Autor

Carlos J Beltrán — Médico Anatomopatólogo, miembro del Consejo Mexicano de Médicos Anatomopatólogos, A.C.
