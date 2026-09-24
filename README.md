# pathia-msi-colon

Predicción reproducible de inestabilidad de microsatélites (**MSI-H vs MSS**) a
partir de histología H&E de cáncer colorrectal, mediante *Multiple Instance
Learning* (MIL) sobre parches codificados por un **modelo fundacional de
patología congelado**.

> ⚠️ **Material de aprendizaje — sin uso clínico.**
> Este proyecto es un trabajo de aprendizaje autodirigido y de investigación
> reproducible. **No es un dispositivo médico, no está validado clínicamente y no
> debe usarse para decisiones diagnósticas ni de tratamiento.** Las conclusiones
> se limitan a factibilidad metodológica.

---

## Objetivo

Construir y evaluar, con rigor metodológico y de forma completamente
reproducible, un pipeline que prediga el estado **MSI-H vs MSS** a nivel de
paciente a partir de H&E:

1. **Tiling** de la laminilla con detección de tejido y control de calidad.
2. **Extracción de embeddings** por parche con un modelo fundacional
   **congelado** (sin *fine-tuning*).
3. **Agregación MIL** (p. ej. ABMIL) de los embeddings de cada laminilla/paciente
   en una predicción única, entrenada solo con la etiqueta de paciente
   (supervisión débil).
4. **Evaluación** por paciente (AUROC, AUPRC, intervalos de confianza) con
   particiones sin *data leakage*.

El reporte seguirá la guía **CLAIM 2024** (*Checklist for Artificial Intelligence
in Medical Imaging*), con miras a un preprint en medRxiv.

## Sustrato morfológico del MSI-H

Un modelo sobre H&E no *mide* inestabilidad de microsatélites —eso es genómica—,
pero el fenotipo MSI-H deja **huellas histológicas reconocibles**. Esa es la
premisa que hace plausible predecir MSI-H desde morfología.

Rasgos asociados al fenotipo MSI-H en cáncer colorrectal:

| Rasgo | Descripción | Valor discriminante |
|---|---|---|
| **TILs intraepiteliales** | Linfocitos infiltrando los nidos tumorales (no solo peritumorales) | El más robusto |
| **Reacción Crohn-like** | Agregados linfoides peritumorales en el frente de invasión | Alto |
| **Subtipos especiales** | Medular (casi patognomónico), mucinoso, poco diferenciado, células en anillo de sello | Alto |
| **Ausencia de "dirty necrosis"** | La necrosis luminal eosinofílica con detritos es típica de MSS; su **escasez** orienta a MSI-H | Rasgo negativo |
| **Bordes expansivos (pushing)** | Margen de invasión no infiltrativo; frecuente localización en colon derecho | Moderado |

**La paradoja MSI-H:** histología a menudo "de alto grado" (poco diferenciada)
con comportamiento clínico **favorable** — la apariencia y el pronóstico divergen.

### Por qué es aprendible

Ningún rasgo es 100 % sensible ni específico por sí solo; por eso el ojo humano
acierta solo **parcialmente**. La señal está en la **combinación** de estas
texturas, distribuida sobre miles de parches y a menudo por debajo del umbral de
la inspección visual caso por caso. Un modelo puede aprender esa combinación —que
es justo donde puede complementar, no sustituir, al patólogo.

Este es el precedente que el proyecto busca reproducir con rigor metodológico:

> Kather, J.N., Pearson, A.T., Halama, N. et al. *Deep learning can predict
> microsatellite instability directly from histology in gastrointestinal cancer.*
> **Nat Med** 25, 1054–1056 (2019). https://doi.org/10.1038/s41591-019-0462-y

> ⚠️ **Alcance:** esta sección justifica la *plausibilidad biológica* del objetivo.
> No implica validez clínica: las conclusiones del proyecto se limitan a
> factibilidad metodológica.

## Datos

Se usan **exclusivamente datos públicos**. **El repositorio no contiene imágenes
ni datos de pacientes** — solo manifests (listas de IDs y coordenadas),
configuración y resultados (ver [`data/README.md`](data/README.md)). Cada dataset
se documenta en `data_cards/` con su origen, licencia y forma de obtenerlo, para
que cualquiera lo **descargue por su cuenta** y reproduzca el trabajo.

### NCT-CRC-HE-100K

| Campo | Valor |
|---|---|
| Contenido | 100 000 parches H&E no superpuestos, 224×224 px a 0.5 µm/px |
| Origen | 86 laminillas FFPE — NCT Biobank (Heidelberg) y archivo de patología UMM (Mannheim) |
| Etiquetas | 9 **clases de tejido**: ADI, BACK, DEB, LYM, MUC, MUS, NORM, STR, TUM |
| Normalización | Macenko (existe la variante `NCT-CRC-HE-100K-NONORM` sin normalizar) |
| Validación externa | `CRC-VAL-HE-7K` (7 180 parches, 50 pacientes, sin solapamiento) |
| Licencia | CC-BY 4.0 |
| Fuente | Kather, Halama & Marx (2018), Zenodo — https://doi.org/10.5281/zenodo.1214456 |

> ⚠️ **Limitación a resolver antes de modelar MSI.** NCT-CRC-HE-100K **no trae
> etiqueta MSI** (sus etiquetas son tipos de tejido) y sus parches **no traen
> identificador de paciente ni de laminilla**, por lo que por sí solo no permite
> entrenar un clasificador MSI-H/MSS ni construir la jerarquía
> paciente→laminilla→parche descrita abajo. Usos naturales dentro del proyecto:
> sondeo lineal (*linear probe*) para validar el extractor congelado, y un
> clasificador de tejido para seleccionar parches tumorales (TUM) antes del MIL.
> La etiqueta MSI a nivel de paciente vendrá de una cohorte con estado MSI, p.
> ej. **TCGA-COAD/READ** (ya documentada en `data_cards/tcga_crc.md`) o los
> parches MSI vs MSS de Kather et al. derivados de TCGA (Zenodo
> 10.5281/zenodo.2530835).

### Otras fuentes consideradas

| Fuente | Uso | Nota |
|---|---|---|
| TCGA (COAD/READ) | Cohorte MSI principal | Público; respetar términos de acceso GDC |
| CPTAC | Validación externa | Público |
| PANDA | Apoyo | Público (Kaggle) |
| PAIP2020, TNBC, Gleason 2019 | Respaldo | Requieren registro/lead time |

## Modelos fundacionales

Se usan como **extractores de características congelados**: los pesos del
modelo fundacional no se modifican; solo se entrena el agregador MIL encima de
sus embeddings.

| Modelo | Rol | Arquitectura | Acceso | Licencia |
|---|---|---|---|---|
| **UNI** (`MahmoodLab/UNI`) | Principal | ViT-L/16, DINOv2, 1024-d | *Gated* en Hugging Face (registro + aceptación de términos) | CC-BY-NC-ND 4.0 + términos de MahmoodLab |
| **Phikon-v2** (`owkin/phikon-v2`) | Respaldo / comparación | ViT-L/16, DINOv2, 1024-d | Público en Hugging Face | Owkin Non-Commercial License |
| HIPT | Comparación (opcional) | ViT jerárquico | mahmoodlab/HIPT | Ver repositorio |

> Los pesos de los modelos **no se versionan** en este repo (ver `.gitignore`);
> cada usuario los descarga con su propia cuenta de Hugging Face.

## Licencias

### Código de este repositorio — MIT

El **código** propio (notebooks, `src/`, `tests/`, configuración) se distribuye
bajo licencia **MIT** (ver [`LICENSE`](LICENSE)). La licencia MIT cubre
**únicamente el código**; no otorga ningún derecho sobre los datasets, los
modelos fundacionales ni lo que se derive de ellos, que conservan sus propias
licencias:

### UNI — CC-BY-NC-ND 4.0 + términos de uso de MahmoodLab

- Solo **investigación académica no comercial**, con atribución.
- **ND (*No Derivatives*):** no se permite distribuir versiones modificadas del
  modelo.
- **Extensión de MahmoodLab:** queda prohibido el uso comercial, venta o
  monetización de UNI **y de sus derivados, que incluyen modelos entrenados
  sobre los outputs de UNI y datasets creados a partir de UNI**, salvo
  aprobación previa.
- Al descargarlo, el usuario se compromete a **no distribuir, publicar ni
  reproducir una copia** del modelo; cada persona debe registrarse
  individualmente en Hugging Face. No se permite intentar reidentificar los
  datos usados para entrenarlo.
- Texto oficial: https://huggingface.co/MahmoodLab/UNI y
  https://github.com/mahmoodlab/UNI#license-and-terms-of-tuse

### Phikon-v2 — Owkin Non-Commercial License

- Distribuido bajo la **licencia no comercial de Owkin** (no es una licencia
  permisiva tipo Apache/MIT): también limita el uso a fines no comerciales.
- Texto oficial: https://huggingface.co/owkin/phikon-v2/blob/main/LICENSE.pdf
- Ventaja práctica frente a UNI: **no es *gated*** (descarga directa, sin
  aprobación), lo que lo hace un buen respaldo para Colab y para terceros que
  quieran reproducir el trabajo.

### Política de redistribución de este proyecto

- ❌ **No se redistribuirán pesos entrenados derivados de UNI**: ni los pesos
  de UNI, ni los agregadores MIL entrenados sobre embeddings de UNI, ni
  checkpoints de ningún modelo cuyo entrenamiento haya consumido outputs de UNI.
- ❌ **No se publicarán los embeddings/features extraídos con UNI** (cuentan
  como "datasets creados a partir de UNI").
- ✅ Sí se publica: código (MIT), manifests (solo IDs y coordenadas),
  configuración, métricas agregadas y figuras.
- Cualquier artefacto derivado de Phikon-v2 seguirá igualmente sus términos no
  comerciales.

## Convención de manifests (prevención de *data leakage*)

Los datos se describen con **manifests** (tablas CSV sin píxeles, versionadas en
`splits/`) organizados en una jerarquía estricta:

```
paciente (patient_id)            ← aquí vive la ETIQUETA MSI y el SPLIT
   └── laminilla (slide_id)      ← hereda etiqueta y split del paciente
          └── parche (tile_id)   ← hereda etiqueta y split de su laminilla
```

**Principio:** la unidad de independencia estadística es el **paciente**. Dos
parches —o dos laminillas— del mismo paciente comparten biología, tinción y
escáner; si uno cae en entrenamiento y otro en prueba, el modelo "reconoce al
paciente" y las métricas se inflan artificialmente.

### Tres tablas, una fuente de verdad por dato

| Archivo | Una fila por | Columnas mínimas |
|---|---|---|
| `patients.csv` | paciente | `patient_id`, `label_msi` (`MSI-H`/`MSS`), `label_fuente`, `fuente`, `split` |
| `slides.csv` | laminilla | `slide_id`, `patient_id`, `archivo`, `MPP`, `magnificacion`, `fuente` |
| `tiles.csv` | parche | `tile_id`, `slide_id`, `patient_id`, `x`, `y`, `nivel`, `w`, `h`, `MPP`, `pct_tejido`, `qc_flag`, `semilla` |

Las columnas de `tiles.csv` extienden el manifest del Notebook 05 (`slide_id`,
`x`, `y`, `nivel`, `w`, `h`, `MPP`, `pct_tejido`, `fuente`, `semilla`) con los
identificadores de la jerarquía.

### Reglas

1. **El split se asigna solo en `patients.csv`.** `slides.csv` y `tiles.csv`
   **no** llevan columna `split` propia: la obtienen por *join* con
   `patient_id`. Así es imposible que dos niveles se contradigan.
2. **Nunca se particiona a nivel de parche ni de laminilla.** Validación cruzada
   con agrupación por paciente (p. ej. `StratifiedGroupKFold` con
   `groups=patient_id`), estratificada por `label_msi`, semilla fija (`42`).
3. **La etiqueta MSI es del paciente.** Los parches no tienen etiqueta propia;
   es supervisión débil y por eso se usa MIL (una "bolsa" = una laminilla o un
   paciente).
4. **IDs estables y trazables.** `patient_id` = identificador de paciente de la
   fuente (en TCGA, los primeros 12 caracteres del barcode, p. ej.
   `TCGA-AA-3544`); `slide_id` = nombre del archivo sin extensión;
   `tile_id` = `{slide_id}_x{x}_y{y}_l{nivel}`.
5. **Todo lo que se "aprende" de los datos se ajusta solo con train:** umbrales
   de QC, parámetros de normalización de tinción, estandarización de features.
6. **Sitio de origen:** en TCGA, el *tissue source site* (`TCGA-XX-…`) puede
   actuar como atajo (*batch effect*). Se reportará su distribución por split y,
   cuando sea factible, un esquema que separe sitios.
7. **Chequeos automáticos en `tests/`:** ningún `patient_id` en más de un split;
   toda laminilla pertenece a un paciente de `patients.csv`; todo parche
   pertenece a una laminilla de `slides.csv`; el `patient_id` de cada parche
   coincide con el de su laminilla.
8. **Los manifests son inmutables por versión.** Si cambia un split, se crea un
   archivo nuevo (p. ej. `patients_v2.csv`) y se documenta el motivo; el test set
   no se consulta hasta la evaluación final.

## Entorno y reproducibilidad

- **Fases 1–4:** Google Colab · **Fases 5–12:** RTX 4070 local.
- Python **3.11 o 3.12**. Todas las dependencias con versión exacta en
  `requirements.txt` (verificado que resuelven sin conflictos en ambas
  versiones).
- Semilla global fija (`42`) en NumPy / random / PyTorch.
- Particiones **por paciente** (ver convención de manifests).

### Local (Linux / Windows / macOS)

```bash
git clone https://github.com/CarlosJB95/PathIA-MSI-colon.git
cd PathIA-MSI-colon

python -m venv .venv
source .venv/bin/activate            # Windows: .venv\Scripts\activate

pip install --upgrade pip
pip install -r requirements.txt
```

> **GPU local (RTX 4070):** si `torch.cuda.is_available()` devuelve `False`, la
> build de PyTorch por defecto no coincide con tu driver/CUDA. Instala primero
> `torch==2.12.1` y `torchvision==0.27.1` desde el índice que indique el
> selector de https://pytorch.org/get-started/locally/ y después
> `pip install -r requirements.txt`.

### Google Colab

```python
!git clone https://github.com/CarlosJB95/PathIA-MSI-colon.git
%cd PathIA-MSI-colon
!pip install -q -r requirements.txt
# Reinicia el entorno de ejecución después de instalar (Runtime → Restart session)
```

> Colab trae su propia versión de PyTorch preinstalada; `requirements.txt` la
> reemplaza por la versión fijada para que los resultados sean comparables.

### Acceso a los modelos fundacionales

1. Crea una cuenta en Hugging Face y solicita acceso a
   https://huggingface.co/MahmoodLab/UNI (Phikon-v2 no requiere aprobación).
2. Genera un *token* de lectura en https://huggingface.co/settings/tokens.
3. Autentícate con `hf auth login` (local; CLI de huggingface_hub 1.x) o con
   `huggingface_hub.login()` usando los *Secrets* de Colab.
   **Nunca** escribas el token en un notebook ni lo subas al repo (`.env` y
   `.huggingface/` están en `.gitignore`).

### Notebooks limpios

```bash
nbstripout --install   # una vez por clon: elimina salidas de los .ipynb al hacer commit
```

## Estructura del repositorio

```
PathIA-MSI-colon/
├── README.md            # este archivo
├── LICENSE              # licencia del código (MIT)
├── .gitignore           # qué NO se sube (datos, pesos, embeddings, secretos)
├── requirements.txt     # dependencias con versiones exactas
├── data/                # datos locales — NO versionados (solo README y .gitkeep)
├── data_cards/          # descripción de cada dataset SIN los datos
├── splits/              # manifests paciente→laminilla→parche (solo IDs)
├── configs/             # hiperparámetros y rutas en YAML (no hardcodeados)
├── notebooks/           # notebooks de aprendizaje y experimentos
├── src/                 # funciones reutilizables (tiling, QC, métricas)
├── tests/               # pruebas con pytest (integridad de manifests, leakage)
├── docs/                # checklist CLAIM 2024, decisiones, diagramas
└── results/             # figuras, métricas y tablas curadas
```

## Estado

Ruta de formación de 12 meses (Mes 2 en curso: lectura de WSI con OpenSlide,
tiling dirigido por tejido y primer manifest).

## Autor

Carlos — anatomopatólogo · proyecto de investigación/educación en patología
digital computacional.
