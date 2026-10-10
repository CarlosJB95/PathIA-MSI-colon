# Plan de refactor — inventario y propuesta (sin implementar)

> **Alcance de este documento.** Es un reconocimiento del repositorio a fecha
> de hoy. **No se modificó ni se ejecutó ningún notebook** para producirlo, y
> no se descargó ningún dato. Todo lo de abajo son propuestas para decidir
> después, no cambios ya hechos.
>
> Reglas permanentes del repo (idioma, qué puede/no puede escribir Claude,
> splits por paciente, reproducibilidad, data cards, datos/pesos fuera del
> repo, flujo por PR, commits para principiante) están en `CLAUDE.md`.

---

## 1. Inventario de notebooks

Los 6 notebooks son de dos meses distintos de la ruta de formación:
Mes 1 = Python/NumPy/pandas/visualización/imagen (Notebooks 01–04); Mes 2 =
WSI real con OpenSlide y primera clasificación (Notebooks 05–06).

> Los números de celda son la posición 0-indexada de la celda dentro del
> notebook (la celda `[0]` es siempre el badge de "Abrir en Colab").

### Notebook_01_Python.ipynb — Python intermedio (Semana 1)

Contenido: `*args`/`**kwargs`, closures/scope, comprensión de listas,
diccionarios, sets/strings, clases y objetos. Todo con datos de juguete
(sin imágenes reales).

| Celda | Función / clase | Qué hace |
|---|---|---|
| 4 | `describir_parche(*coords, **metadatos)` | Arma una línea descriptiva de un parche a partir de coords/metadatos sueltos. Ejercicio de `*args/**kwargs`. |
| 6 | `suma_nucleos(*cores)` | Suma conteos de núcleos. Ejercicio de `*args`. |
| 8 | `meta_op(**metadatos)` | Resume metadatos de WSI tolerando claves ausentes (`.get`). |
| 10 | `resumen_laminilla(*args, **kwargs)` | Resume nº de parches, % tejido medio y escáner. |
| 13 | `crear_filtro_tejido(umbral)` | *Factory* de filtros: devuelve una closure `es_tejido(pct)`. |
| 14 | `es_tejido(pct)` | Versión suelta (no funcional, solo para `help()`), usa `umbral` global — código de demostración, no reutilizable. |
| 25 | `procesar_lote_parches(*parches, umbral_tejido=30, **opciones)` | Filtra parches (dicts) por `tejido_pct` contra un umbral. |
| 45 | `hay_fuga(pacientes_train, pacientes_test)` | Detecta fuga (*leakage*) entre dos conjuntos de pacientes (intersección de sets). |
| 52 | `extraer_patient_id(slide_id)` — v1 | Extrae `patient_id` de un barcode TCGA. **Sin validar formato** (acepta basura en silencio). |
| 55 | `construir_manifest(slides)` | Agrupa laminillas por paciente y suma parches totales. |
| 60 | `extraer_patient_id(slide_id)` — v2 | Redefine la de la celda 52: ahora **valida** el formato y lanza `ValueError` si no es un barcode TCGA. |
| 67 | `class Laminilla` | Objeto-expediente de una laminilla: `patient_id()`, `es_valida()` (QC simple de rango de MPP). |
| 77 | `class Manifiesto` | Colección de `Laminilla`: `resumen()`, `hay_fuga(otro)`, `agregar()`, `a_csv(ruta)`. |
| 77 | `leer_manifest(ruta)` | Lee y muestra un CSV, capturando `FileNotFoundError`. |

### Notebook_02_NumPy&Pandas.ipynb — del parche al manifest (Semana 2)

Contenido: NumPy sobre arrays H&E sintéticos (`uint8`, broadcasting,
reducciones por eje, `np.where`), luego pandas sobre el manifest resultante
(indexado, `groupby`, NaN, EDA). Incluye un anexo de práctica con el dataset
Titanic (no forma parte de la cohorte del proyecto).

| Celda | Función | Qué hace |
|---|---|---|
| 16 | `describir_parche(patch, slide_id, patch_id)` | Calcula `pct_tejido` y firma de color media por canal de un parche; devuelve una fila de manifest. **Mismo nombre que la función de Notebook 01, pero firma y propósito distintos** — ver riesgo R1. |
| 17 | `escribir_manifest(filas, ruta)` | Escribe una lista de dicts a CSV con `csv.DictWriter`; captura `OSError`/`PermissionError` y devuelve `bool`. |
| 25 | `estadisticas_parche(patch, umbral_tejido=220)` | `pct_tejido` + `pct_nucleos` (heurística de color) + firma RGB. |
| 26 | `pct_tejido_lento(patch, umbral=220)` / `pct_tejido_rapido(patch, umbral=220)` | Mismo cálculo de % tejido, versión con bucles vs. vectorizada — ejercicio de rendimiento (`%%timeit`), no pensado para reutilizar. |
| 31 | `filtrar_parche(patch, umbral=220)` | Como `estadisticas_parche`, y además decide `conservar` (bool) contra el umbral. |
| 52 | `explorar_manifest(df)` | Imprime una "radiografía" EDA del manifest (shape, dtypes, describe, % conservables). |
| 63 | `resumir_por_slide(df)` | Agrega el manifest de nivel-parche a nivel-laminilla (`groupby("slide_id")`). |

### Notebook_03_Visualizaciones.ipynb — EDA de cohorte (Semana 3)

Contenido: estilo global de matplotlib, 3 figuras de distribución sobre un
manifest **simulado** (300 laminillas sintéticas, no datos reales), panel
resumen de las 3, y un anexo de práctica (Titanic) + checklist de figura
publicable.

| Celda | Función | Qué hace |
|---|---|---|
| 4 | `aplicar_estilo()` | Fija `plt.rcParams` (dpi, tamaños de fuente, grid, spines) para que todas las figuras del notebook compartan estilo. |

El resto del notebook son celdas de script (no funciones) que generan cada
figura y la guardan en `results/`.

### Notebook_04_Imagen_como_dato.ipynb — Pillow + NumPy (Semana 4)

Contenido: descarga parches reales de ejemplo (mirror parquet de
NCT-CRC-HE-100K en Hugging Face) para interrogarlos como imagen (Pillow),
recortes/tiles fijos, resize y su efecto en la información, y canales
RGB + histogramas con NumPy.

| Celda | Función | Qué hace |
|---|---|---|
| 11 | `recortar_campos(img, tam=112, descartar_borde=True)` | Trocea una `PIL.Image` en tiles cuadrados de lado fijo; devuelve `(sub_imagen, caja)`. Geometría pura, sin criterio de tejido. |
| 21 | `histograma_canales(arr, bins=256)` | Grafica histograma RGB superpuesto y devuelve media/mediana/SD por canal. |
| 25 | `es_fondo(arr, umbral_sd=15)` | Marca un parche como fondo si los 3 canales tienen SD baja (demasiado uniforme). |

### Notebook_05_WSI_OpenSlide.ipynb — WSI real con OpenSlide (Mes 2, Días 21–22)

Contenido: instala/abre una WSI real (`CMU-1.svs`, Aperio, CC0) con
OpenSlide, inspecciona pirámide/MPP, lee regiones (`read_region`), genera
una rejilla de tiles, detecta tejido con Otsu sobre el thumbnail y filtra
tiles por % de tejido, y guarda el manifest final a CSV.

| Celda | Función | Qué hace |
|---|---|---|
| 10 | `fraccion_tejido_en_tile(mascara, x0, y0, w0, h0, fx, fy)` | Traduce la caja de un tile (coords de nivel 0) a coords de la máscara Otsu del thumbnail y devuelve la fracción de tejido. Mezcla geometría de tiling **y** criterio de QC de tejido. |

Ver la sección 4 para la propuesta concreta de cómo separar este notebook.

### Notebook_06_intro_clasificacion.ipynb — primera clasificación (Mes 2, Días 23–24)

Contenido: features de color de un tile (media/SD por canal), demo de
`GroupShuffleSplit` por paciente con datos sintéticos, exploración de
features TUM vs NORM sobre `CRC-VAL-HE-7K`, y un baseline de regresión
logística sobre features ya cacheadas (`nct_dia24.npz`) con matriz de
confusión.

| Celda | Función | Qué hace |
|---|---|---|
| 2 | `features_color(tile_rgb)` | Vector de 6 features: media y SD por canal RGB. |
| 4 | `features_color(tile_rgb)` (idéntica) + `cargar_muestra(clase, n, seed=42)` | Redefine `features_color`; `cargar_muestra` carga N parches por clase desde `CRC-VAL-HE-7K/<clase>/*.tif`. |
| 6 | `features_color(tile_rgb)` (idéntica, 3ª vez) | Se usa para transformar `Xtr`/`Xte` cargados de `nct_dia24.npz`. |

`features_color` y `cargar_muestra` son exactamente el tipo de código que
`CLAUDE.md` (regla 2) marca como objetivo de aprendizaje (extracción de
features) — **no son candidatas a que Claude las mueva o reescriba** salvo
que Carlos lo pida explícitamente.

---

## 2. Funciones candidatas a mover a `src/`

Solo se listan funciones que **ya existen** en los notebooks (mover ≠
escribir código nuevo). Se separan en dos grupos según la regla 2 de
`CLAUDE.md`.

### 2.1 Infraestructura — seguras de mover tal cual

| Función/clase | Origen | Destino propuesto |
|---|---|---|
| `escribir_manifest` | Notebook 02, celda 17 | `src/manifest_io.py` |
| `leer_manifest` | Notebook 01, celda 77 | `src/manifest_io.py` |
| `construir_manifest` | Notebook 01, celda 55 | `src/manifest_io.py` |
| `resumir_por_slide` | Notebook 02, celda 63 | `src/manifest_io.py` |
| `class Manifiesto` (`resumen`, `agregar`, `a_csv`) | Notebook 01, celda 77 | `src/manifest_io.py` |
| `extraer_patient_id` (**solo la v2, celda 60, la que valida**) | Notebook 01 | `src/patient_ids.py` |
| `hay_fuga` | Notebook 01, celda 45 | `src/patient_ids.py` (o `src/splits.py`) |
| `aplicar_estilo` | Notebook 03, celda 4 | `src/viz.py` |
| `recortar_campos` | Notebook 04, celda 11 | `src/tiling.py` (geometría pura, sin criterio de tejido) |

`class Laminilla` (Notebook 01, celda 67) es mayormente infraestructura,
pero su método `es_valida()` aplica un criterio de rango de MPP que es, en
sentido estricto, una regla de QC. Se puede mover junto con el resto, pero
señalando que ese método puntual es "QC ligero" por si Carlos prefiere
excluirlo o revisarlo primero.

### 2.2 Objetivo de aprendizaje — NO mover sin que Carlos lo pida

Estas funciones son features/QC/color por definición (regla 2 de
`CLAUDE.md`). Se listan para que el inventario quede completo, no para
moverlas:

- `describir_parche` (Notebook 02, celda 16) — % tejido + firma de color.
- `estadisticas_parche`, `filtrar_parche` (Notebook 02) — % tejido/núcleos.
- `pct_tejido_lento` / `pct_tejido_rapido` (Notebook 02) — ejercicio de
  vectorización, no una utilidad a reusar.
- `histograma_canales`, `es_fondo` (Notebook 04) — estadística de canal /
  detección de fondo.
- `fraccion_tejido_en_tile` (Notebook 05) — QC de tejido por tile (ver
  sección 4).
- `features_color`, `cargar_muestra` (Notebook 06) — extracción de
  features y muestreo para el modelo.

### 2.3 Nota sobre colisiones de nombre

`describir_parche` existe con **dos firmas distintas** en Notebook 01
(celda 4: texto legible a partir de `*coords/**metadatos`) y Notebook 02
(celda 16: dict de QC a partir de `patch, slide_id, patch_id`). Si algún
día se movieran ambas a `src/`, necesitan nombres distintos — no es un
problema hoy porque ninguna de las dos se está moviendo todavía.

---

## 3. Propuesta de estructura

Las carpetas `src/`, `configs/`, `splits/`, `results/` y `data_cards/`
**ya existen** (con `.gitkeep` y sus propios `README.md`); no hace falta
crearlas. La propuesta es sobre qué contenido va en cada una a medida que
el código deja de ser exploratorio:

- **`src/`** — módulos de la sección 2.1 (`manifest_io.py`,
  `patient_ids.py`, `viz.py`, `tiling.py`, y más adelante `wsi_reader.py`
  — ver sección 4). Cada notebook futuro importaría desde aquí en vez de
  redefinir la función.
- **`configs/`** — ya tiene `config.yaml` con semilla, rutas, umbral de
  tejido y esquema de split. Pendiente: que los notebooks lean estos
  valores de `config.yaml` en vez de fijarlos sueltos en cada notebook
  (ver riesgos R5/R6/R7).
- **`splits/`** — hoy solo tiene su `README.md`. Ahí deberían terminar los
  CSV de partición por paciente (train/val/test del NCT-CRC-HE-100K y el
  CRC-VAL-HE-7K), no en el directorio de trabajo del notebook.
- **`results/`** — ya se usa correctamente en Notebook 03 (figuras
  guardadas ahí). Pendiente: que los manifiestos de Notebooks 04/05/06
  (`manifest_CMU-1.csv`, etc.) también se guarden aquí (o en `splits/` si
  son particiones) en vez de en el directorio de ejecución del notebook.
- **`data_cards/`** — ya tiene la plantilla `tcga_crc.md`. Pendiente:
  crear data cards para los datasets que los notebooks 04–06 ya usan de
  hecho (el mirror de NCT-CRC-HE-100K en Hugging Face, `CRC-VAL-HE-7K`, y
  la WSI de prueba `CMU-1.svs` de OpenSlide) — ver riesgo R9.

---

## 4. Propuesta concreta: separar Notebook 05 en "lector de WSI" vs "tiling"

Hoy el notebook mezcla, en este orden: (a) abrir la WSI y leer su
metadata/regiones, (b) generar una rejilla de coordenadas de tiles, y (c)
decidir qué tiles conservar según tejido (Otsu). Propuesta de separación en
tres capas, para que la parte (c) —que es el objetivo de aprendizaje activo
de Carlos— quede aislada de la infraestructura:

1. **`src/wsi_reader.py`** (infraestructura, movible sin objeción) —
   envolvería únicamente E/S de la WSI, sin ningún criterio de tejido:
   - abrir la WSI (`openslide.OpenSlide(ruta)`),
   - exponer niveles/dimensiones/downsamples/MPP,
   - `get_thumbnail(...)`,
   - `read_region(...)` con el descarte del canal alfa ya resuelto
     (`.convert("RGB")`).

2. **`src/tiling.py`** (infraestructura, la parte de geometría) — solo la
   generación de la rejilla de coordenadas `(x0, y0, ancho, alto)` sobre
   las dimensiones del nivel 0, sin decidir todavía qué tile conservar.
   Reutilizaría el mismo módulo que `recortar_campos` de Notebook 04.

3. **Se queda en el notebook** (objetivo de aprendizaje, QC de tejido) —
   la máscara de Otsu sobre el thumbnail y `fraccion_tejido_en_tile`, que
   deciden qué tiles pasan el umbral. Esta capa consumiría las dos
   anteriores (`wsi_reader` para leer, `tiling` para la rejilla) pero el
   criterio de "cuánto tejido es suficiente" sigue siendo código que
   Carlos escribe y ajusta él mismo.

Esta separación es solo una propuesta para decidir; no se ha creado ningún
archivo nuevo en `src/` como parte de esta tarea.

---

## 5. Riesgos detectados (solo se listan, no se corrigen)

- **R1 — Colisión de nombres.** `describir_parche` está definida dos veces
  en el repo con firmas y propósitos distintos (Notebook 01 celda 4 vs.
  Notebook 02 celda 16). Ver sección 2.3.
- **R2 — Redefinición sin limpiar.** `extraer_patient_id` se define dos
  veces en Notebook 01 (celda 52 sin validar, celda 60 con validación).
  Funciona porque Python usa la última definición, pero un lector nuevo
  puede confundirse sobre cuál es "la buena". `features_color` se repite
  **tres veces**, idéntica, en Notebook 06 (celdas 2, 4 y 6).
- **R3 — Deuda de nomenclatura en `notebooks/README.md`.** El README de
  `notebooks/` describe una convención (`Notebook_Semana_01.ipynb`,
  `Notebook_Semana_02.ipynb`, ...) que ya no corresponde a los archivos
  reales (`Notebook_01_Python.ipynb` ... `Notebook_06_intro_clasificacion.ipynb`,
  organizados por tema/mes, no por semana). Está desactualizado.
- **R4 — Celda dependiente de Colab sin aviso.** Notebook 03, última celda,
  usa `from google.colab import files` para descargar las figuras. Fallará
  fuera de Colab (Jupyter local, CI, etc.) y no está señalada como
  "solo-Colab" más que en un comentario suelto (`#opcional, colab-only`).
- **R5 — Parámetro duplicado y desalineado.** `configs/config.yaml` fija
  `tissue_threshold: 0.25`, pero Notebook 05 usa
  `UMBRAL_TEJIDO = 0.5` (Día 21) hardcodeado en la celda, sin leer el
  config. Si algún día se cambia uno, el otro queda desalineado en
  silencio.
- **R6 — Rutas de dataset hardcodeadas fuera de `config.yaml`.** Notebook 06
  fija `BASE = "CRC-VAL-HE-7K"` como ruta relativa literal; Notebook 04 fija
  `DIR_PARCHES = BASE / "data" / "parches"` dentro del propio repo (en vez
  de `paths.data_root` de `config.yaml`, que apunta fuera del repo,
  `../data/tcga_crc`). Ninguno de los dos lee `configs/config.yaml`.
- **R7 — Ruta interna inconsistente en Notebook 04.** La celda de setup
  define `DIR_HIST = BASE / "results" / "histogramas"`, pero una celda
  posterior (Día 17) **redefine** `DIR_HIST = Path("01_colon_eda/results/histogramas")`
  — una ruta de otra estructura de proyecto que no existe en este repo.
  Parece un resto de copiar/pegar.
- **R8 — Reproducibilidad de red sin verificación.** Notebook 05 descarga
  `CMU-1.svs` por `wget` desde `openslide.cs.cmu.edu` cada vez que se
  ejecuta, sin verificar checksum ni cachear localmente; además usa
  `!apt-get install` / `!pip install` (shell magics), lo que asume Colab o
  un entorno con permisos de root. Notebook 04 descarga tiles reales vía
  `datasets.load_dataset(..., streaming=True)` sin fijar una revisión del
  dataset remoto.
- **R9 — Datasets en uso sin data card.** Ya se usan datos reales de
  **NCT-CRC-HE-100K** (mirror `junyeong-nero/mini-NCTCRCHE100K` en Hugging
  Face, Notebook 04), **CRC-VAL-HE-7K** (Notebook 06) y la WSI de prueba
  **CMU-1.svs** de OpenSlide (Notebook 05), pero `data_cards/` solo tiene
  la plantilla de TCGA-COAD/READ — ninguno de los tres tiene su propio
  data card todavía.
- **R10 — Manifiestos y datos escritos en el directorio de ejecución.**
  Varios notebooks escriben CSV/artefactos en la carpeta desde la que se
  ejecuta el notebook en vez de `results/` o `splits/`: `manifest.csv`,
  `manifest_parches.csv`, `manifest_conservables.csv`,
  `manifest_slide.csv` (Notebooks 01–02) y `manifest_CMU-1.csv`,
  `CMU-1.svs` (Notebook 05). El `.gitignore` los protege de subirse por
  extensión/patrón, pero la ubicación no sigue la convención de carpetas
  del README del proyecto.
- **R11 — Origen no documentado de un artefacto cacheado.** Notebook 06
  carga `nct_dia24.npz` (`Xtr, ytr, Xte, yte`) pero ninguno de los 6
  notebooks contiene el código que genera ese archivo — no es reproducible
  ejecutando "Restart & run all" sobre lo que hay en el repo hoy.
- **R12 — Dependencias sin versión fijada.** `requirements.txt` fija solo
  cotas mínimas (`numpy>=1.26`, etc.), tal como el propio archivo indica
  ("por ahora se fija un piso mínimo"). Además, paquetes que ya se
  importan en Notebooks 04–05 (`datasets`, `Pillow`, `scikit-image`,
  `openslide-python`) no están declarados como dependencia activa (están
  ausentes o comentados en `requirements.txt`).
- **R13 — Split oficial no verificado explícitamente en Notebook 06.** El
  notebook demuestra `GroupShuffleSplit` por paciente solo con datos
  sintéticos (celda 3); el baseline real (celdas 6–9, sobre
  `nct_dia24.npz`) no incluye una verificación tipo `hay_fuga` de que el
  train/test cargado respeta el split oficial (100K vs. 7K) o de qué
  cohorte proviene cada mitad.
