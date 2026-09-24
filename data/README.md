# data/

**Los datos NO se versionan en git.** Esta carpeta existe en el repositorio
solo con este `README.md` y un `.gitkeep`; todo lo demás que pongas aquí lo
ignora `.gitignore` (regla `data/*`).

Motivos:

- **Tamaño:** NCT-CRC-HE-100K pesa ~11–15 GB y una WSI de TCGA suele pesar
  cientos de MB; git no está hecho para archivos binarios de ese tamaño.
- **Licencias:** cada dataset tiene sus propios términos de redistribución.
  Se comparte cómo *obtenerlos*, no los archivos.
- **Privacidad:** aunque las fuentes sean públicas y anonimizadas, la regla del
  proyecto es no subir nunca imágenes de pacientes.

## Qué sí se versiona

| Qué | Dónde |
|---|---|
| Descripción, licencia y forma de descarga de cada dataset | `data_cards/` |
| Manifests (IDs, coordenadas, etiquetas, split) — sin píxeles | `splits/` |
| Figuras y métricas curadas | `results/` |

## Organización local sugerida

```
data/
├── raw/          # descargas originales, sin tocar (zip, .svs, .tif)
├── tiles/        # parches extraídos (si se guardan a disco)
└── embeddings/   # features del modelo fundacional (.h5), uno por laminilla
```

> ⚠️ Los embeddings calculados con UNI son "datasets created from the UNI
> models" según sus términos de uso: **no se publican ni se suben al repo**.

En Colab, los datos suelen vivir en Google Drive (`/content/drive/MyDrive/...`);
apunta a esa ruta desde `configs/config.yaml`, no desde el código.
