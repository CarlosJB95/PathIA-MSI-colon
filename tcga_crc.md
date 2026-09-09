# Data Card — TCGA-COAD/READ (colon y recto)

> Plantilla para documentar cada dataset **sin incluir los datos**. Copia este
> archivo por cada fuente (p. ej. `tcga_crc.md`, `cptac.md`, `panda.md`) y
> rellena los campos. El objetivo es que otra persona pueda **reproducir el
> trabajo descargando ella misma los datos**.

---

## Identificación
- **Nombre:** TCGA-COAD + TCGA-READ (adenocarcinoma de colon y recto)
- **Versión / fecha de acceso:** _(p. ej. descargado 2026-09-08)_
- **Fuente oficial:** GDC Data Portal — https://portal.gdc.cancer.gov
- **Licencia / términos de uso:** datos públicos NIH/GDC; respetar la
  *Data Use Agreement* de TCGA. **No redistribuir imágenes**; solo se comparten
  IDs y manifiestos.

## Contenido
- **Modalidad:** WSI de histología **H&E** (diagnóstico, `.svs`).
- **Nº de pacientes:** _(rellenar tras la descarga)_
- **Nº de laminillas:** _(rellenar)_
- **Magnificación / MPP:** _(p. ej. 20× / 40×; anotar MPP por slide)_
- **Etiqueta de interés:** estado **MSI** (MSI-H vs MSS) — origen de la etiqueta:
  _(p. ej. anotación clínica MSI-status / MANTIS / archivo de metadatos)_

## Distribución de clases
| Clase | n pacientes | % |
|---|---|---|
| MSI-H | _(rellenar)_ | _()_ |
| MSS   | _(rellenar)_ | _()_ |

> Anotar aquí el **desbalance** observado — es lo que justifica reportar AUPRC.

## Particiones (splits)
- **Nivel de split:** **por paciente** (obligatorio — nunca por laminilla).
- **Esquema:** _(p. ej. train/val/test 70/15/15, estratificado por clase)_
- **Semilla:** 42
- **Archivo de referencia:** `splits/tcga_crc_patientlevel.csv` (solo IDs).

## Preprocesamiento aplicado
- Detección de tejido / umbral: _(p. ej. Otsu sobre intensidad; pct_tejido mín.)_
- Tamaño de parche y nivel: _(p. ej. 224×224 @ 20×)_
- Filtros de QC: _(fondo, artefactos, pliegues — criterio usado)_

## Auditoría de calidad
- **Fugas (leakage):** ¿mismo paciente en más de un split? _(sí/no — verificado)_
- **Duplicados:** _(método de detección)_
- **Casos excluidos y motivo:** _(lista o criterio)_

## Cómo obtener los datos (reproducibilidad)
1. Crear cuenta / aceptar términos en el GDC Data Portal.
2. Descargar el *manifest* de casos COAD/READ con WSI diagnósticas H&E.
3. Usar `gdc-client` con el manifiesto incluido en `data_cards/manifests/`.
4. Verificar `md5` de las descargas.

## Notas
- **Alcance:** investigación/educación, sin uso clínico.
- Ninguna imagen ni dato de paciente se versiona en este repositorio.
