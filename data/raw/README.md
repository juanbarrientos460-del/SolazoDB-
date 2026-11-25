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


### Cruce exploratorio con `status`

Se cruzaron las tres lecturas con `battvolt = 0` con la tabla `status` por `inverter` y `read_ts`. El registro del 2026-05-12 10:16:17 presentó varios campos de estado; el del 2026-05-15 04:10:28 no presentó valores de estado útiles en la coincidencia exacta; y el del 2026-06-19 04:35:37 solo presentó `startupstatus = 1`. El resultado indica disponibilidad desigual de datos de estado, pero no prueba una causa física ni un error. Se documenta una siguiente revisión de registros cercanos en el tiempo.


## Perfilamiento de valores nulos

En la tabla original `analog_measurements` se identificaron valores nulos en seis variables: `battsoc` (57.931), `battvolt` (58.141), `battcurr` (58.217), `pv1power` (58.660), `pv1volt` (59.379) y `pv1curr` (58.001), sobre 2.989.759 registros.

La distribución por inversor evidencia que los nulos se concentran en el inversor 2: entre 57.122 y 57.951 nulos por variable, frente a entre 771 y 1.428 en el inversor 1. Es un hallazgo de calidad pendiente de explicación, no una prueba de avería. No reemplazar los nulos por cero sin una regla de negocio justificada.


## Coincidencia de nulos entre variables

Una consulta conjunta de seis variables (`battsoc`, `battvolt`, `battcurr`, `pv1power`, `pv1volt`, `pv1curr`) encontró:

| Inversor | Total de registros | Las seis variables nulas | Al menos una variable nula |
|---|---:|---:|---:|
| 1 | 1.610.276 | 1 | 5.289 |
| 2 | 1.379.483 | 55.657 | 65.606 |

La concentración de nulos simultáneos en el inversor 2 es un patrón relevante para el análisis de calidad. No se conoce aún su causa; no se deben reemplazar esos valores por cero de forma automática.
