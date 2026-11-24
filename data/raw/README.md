# Datos originales

Esta carpeta está reservada para los archivos originales descargados desde la fuente OpenCEM seleccionada.

## Fuente registrada

- Dataset: OpenCEM, versión 1.0.0.
- Registro: https://zenodo.org/records/21436223
- DOI: https://doi.org/10.5281/zenodo.21436223
- Archivo SQLite utilizado para la inspección: `opencem-v1.0.0.sqlite`.
- Archivo comprimido de origen: `opencem-v1.0.0.sqlite.zst`.
- Archivo complementario: `opencem-v1.0.0-support.zip`.
- Archivo de checksum: `opencem-v1.0.0-SHA256SUMS.txt`.
- Licencia: revisar `LICENSE-DATA.txt` y registrar sus condiciones antes de redistribuir los datos.

## Reglas de almacenamiento

- Los archivos pesados se mantienen localmente y se excluyen del control de versiones.
- No modificar los archivos originales.
- No subir la base SQLite ni los datos crudos al repositorio GitHub.
- Registrar y verificar el checksum SHA-256 para comprobar la integridad del archivo descargado.
- Documentar cualquier transformación en `docs/01_fuente_datos.md`.

## Estado de inspección

La base SQLite fue abierta en DBeaver. Se identificaron ocho tablas principales y se contaron sus registros. El inventario, periodo de mediciones, distribución de lecturas por inversor y primeras validaciones de calidad están documentados en `docs/01_fuente_datos.md`.


## Hallazgo adicional de calidad

En la revisión de las filas de `analog_measurements` con `battvolt = 0` se identificaron tres registros, todos del inversor 2 (fechas UTC: 2026-05-12 10:16:17, 2026-05-15 04:10:28 y 2026-06-19 04:35:37). En los tres también aparecen en cero `battcurr`, `pv1volt`, `pv1curr`, `pv1power`, `linevolta`, `linecurra` y `gridpowerw_a`, mientras `battsoc` registra 87. El patrón queda documentado como caso pendiente de interpretación; no se clasifica como error ni se modifica el dato original.
