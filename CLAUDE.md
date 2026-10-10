# CLAUDE.md — reglas permanentes de este repositorio

> **Contexto.** `PathIA-MSI-colon` es material de aprendizaje de un
> anatomopatólogo que está aprendiendo Python y machine learning desde cero,
> como parte de una ruta de formación de 12 meses hacia la predicción de
> inestabilidad de microsatélites (MSI) en cáncer colorrectal a partir de
> histología H&E. **Sin uso clínico.**

Estas reglas aplican a cualquier tarea que Claude Code realice en este
repositorio, no solo a la tarea que las creó.

## 1. Idioma

Comentarios de código, documentación (READMEs, data cards, este archivo) y
mensajes de commit van siempre **en español**.

## 2. Límite de autoría — qué SÍ y qué NO escribe Claude

El objetivo del repo es que Carlos aprenda programando él mismo. Por eso:

- **No escribas código nuevo** que sea objetivo de aprendizaje del autor:
  extracción de features, modelos (entrenamiento/arquitectura), métricas,
  control de calidad (QC) de tejido/parches, o normalización de tinción.
- **Tu papel** es refactorizar código **ya existente**, infraestructura
  (lectura/escritura de archivos, manejo de rutas, splits, empaquetado en
  `src/`), pruebas y documentación.
- Si una tarea pedida requiere escribir código de una de las categorías
  prohibidas arriba, **detente y pregunta** antes de escribirlo — no lo
  redactes "de paso" ni como parte de un refactor.

## 3. Splits por paciente o cohorte

- Toda partición de datos es **por paciente o por cohorte**, nunca por
  parche o laminilla suelta (evita fuga de información / *leakage*).
- El split **oficial** del proyecto es: entrenar con **NCT-CRC-HE-100K**,
  evaluar en **CRC-VAL-HE-7K** (cohortes de pacientes distintas).
- Cualquier validación interna dentro de NCT-CRC-HE-100K (por ejemplo, un
  train/val por parche dentro del mismo 100K) se etiqueta explícitamente
  como **"optimista"** — no sustituye a la evaluación oficial en el 7K.

## 4. Reproducibilidad

- **Semilla fija** (42) en todo código que use aleatoriedad (NumPy, random,
  scikit-learn, frameworks de DL).
- **Rutas relativas** al repositorio — nunca rutas absolutas ni específicas
  de una máquina o de Colab (p. ej. nada de `/content/...` fuera de celdas
  claramente marcadas como "solo Colab").
- **Dependencias con versión fijada** en `requirements.txt` (evitar rangos
  abiertos tipo `>=` una vez que el entorno se estabilice).

## 5. Data cards

Todo `data_card` en `data_cards/` incluye la leyenda:
**"material de aprendizaje — sin uso clínico"**.

## 6. Nada de datos ni pesos en el repositorio

No subir imágenes, WSI, parches ni pesos/checkpoints de modelos — en
especial nada derivado de **UNI** (licencia CC-BY-NC-ND-4.0, uso académico
no comercial). El `.gitignore` ya bloquea estas extensiones y carpetas; no
debilitarlo ni añadir excepciones sin que Carlos lo pida explícitamente.

## 7. Flujo de trabajo en Git

- Trabajar siempre en una **rama nueva**, nunca directamente en `main`.
- Entregar los cambios por **pull request**; nunca hacer push directo a
  `main` ni fusionar (merge) el propio PR.

## 8. Commits explicados para principiante

Cada commit explica el cambio en **lenguaje sencillo**, pensado para alguien
que recién está aprendiendo Python/Git: qué cambió y por qué, evitando
jerga de ingeniería de software cuando una explicación simple baste.
