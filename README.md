# pathia-msi-colon

Predicción reproducible de inestabilidad de microsatélites (**MSI-H vs MSS**) a
partir de histología H&E de cáncer colorrectal, mediante *deep learning* sobre
*whole slide images* (WSI).

> ⚠️ **Alcance — investigación y educación, sin uso clínico.**
> Este proyecto es un trabajo de aprendizaje autodirigido y de investigación
> reproducible. **No es un dispositivo médico, no está validado clínicamente y no
> debe usarse para decisiones diagnósticas ni de tratamiento.** Las conclusiones
> se limitan a factibilidad metodológica.

---

## Objetivo

Evaluar de forma reproducible el uso de *embeddings* de modelos fundacionales de
patología para clasificación a nivel de laminilla/paciente (MSI-H vs MSS) en
tejido colorrectal, con metodología correcta (**split por paciente**, evaluación
a nivel de parche/laminilla/paciente) y reporte según **CLAIM 2024**.

## Datos

Se usan **exclusivamente datos públicos**. **El repositorio no contiene imágenes
ni datos de pacientes** — solo manifiestos (listas de IDs y coordenadas),
configuración y resultados. Cada dataset se documenta en `data_cards/` con su
origen, licencia y forma de obtenerlo, para que cualquiera lo **descargue por su
cuenta** y reproduzca el trabajo.

| Fuente | Uso | Nota |
|---|---|---|
| TCGA (COAD/READ) | Principal | Público; respetar términos de acceso |
| CPTAC | Principal / validación | Público |
| PANDA | Apoyo | Público (Kaggle) |
| PAIP2020, TNBC, Gleason 2019 | Respaldo | Requieren registro/lead time |

## Modelos fundacionales

| Modelo | Rol | Licencia / acceso |
|---|---|---|
| **UNI** | Principal | *Gated* en Hugging Face — **CC-BY-NC-ND-4.0, uso académico no comercial** |
| Phikon-v2 | Comparación | owkin/HistoSSLscaling |
| HIPT | Comparación | mahmoodlab/HIPT (pesos vía Git LFS) |

> Los pesos de los modelos **no se versionan** en este repo (ver `.gitignore`).
> El uso de UNI se limita a investigación académica no comercial conforme a su
> licencia.

## Entorno y reproducibilidad

- **Fases 1–4:** Google Colab · **Fases 5–12:** RTX 4070 local (CUDA 12.x).
- Semilla global fija (`42`) en NumPy / random / frameworks.
- Particiones **por paciente** (nunca por laminilla) — sin *leakage*.
- Versiones ancladas: ver `requirements.txt`.

```bash
# clonar y preparar entorno
git clone https://github.com/<usuario>/pathia-msi-colon.git
cd pathia-msi-colon
pip install -r requirements.txt
```

## Estructura del repositorio

```
pathia-msi-colon/
├── README.md            # este archivo
├── .gitignore           # qué NO se sube (datos, pesos, secretos)
├── requirements.txt     # dependencias con versiones ancladas
├── LICENSE              # licencia del código (MIT)
├── notebooks/           # 01_python..., 02_wsi..., 03_visualizaciones.ipynb ...
├── src/                 # funciones reutilizables (tiling, QC, deconvolución, métricas)
├── configs/             # hiperparámetros y rutas en YAML (no hardcodeados)
├── splits/              # particiones por paciente (solo IDs, sin imágenes)
├── results/             # figuras, métricas, tablas (p. ej. fig_distribuciones_qc.png)
└── data_cards/          # descripción de cada dataset SIN los datos
```

## Estado

Ruta de formación de 12 meses (Mes 1 en curso). El hito del Mes 1 es este
repositorio con los 4 notebooks fundacionales corriendo de punta a punta en Colab.

## Licencia

Código bajo **MIT** (ver `LICENSE`). Los datasets y modelos conservan sus propias
licencias, referidas arriba y en `data_cards/`.

## Autor

Carlos — anatomopatólogo · proyecto de investigación/educación en patología
digital computacional.
