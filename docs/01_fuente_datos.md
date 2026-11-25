# 01. Análisis de la fuente de datos

## Identificación de la fuente

- **Dataset:** OpenCEM: Multimodal photovoltaic and battery microgrid dataset with context records and sky imagery.
- **Versión:** 1.0.0.
- **Autores:** Bartels, T. S., Wu, R., Lu, X., Chen, Y., Lu, Y., Xia, F., Yang, H., Chen, Y., & Li, T.
- **Repositorio de descarga:** https://zenodo.org/records/21436223
- **DOI:** https://doi.org/10.5281/zenodo.21436223
- **Archivo SQLite inspeccionado:** `opencem-v1.0.0.sqlite`, obtenido del archivo comprimido `opencem-v1.0.0.sqlite.zst`.
- **Archivo complementario:** `opencem-v1.0.0-support.zip`.
- **Archivo de checksums:** `opencem-v1.0.0-SHA256SUMS.txt` (verificación pendiente de documentar).
- **Fecha de consulta/descarga:** pendiente de registrar.
- **Licencia:** revisar y registrar las condiciones de `LICENSE-DATA.txt` antes de redistribuir datos.
- **Alcance:** telemetría de una microrred híbrida con mediciones asociadas a inversores. Las imágenes de cielo no forman parte del alcance actual de SolarDB.

Referencia bibliográfica:

> Bartels, T. S., Wu, R., Lu, X., Chen, Y., Lu, Y., Xia, F., Yang, H., Chen, Y., & Li, T. (2026). *OpenCEM: Multimodal photovoltaic and battery microgrid dataset with context records and sky imagery* (Version 1.0.0) [Dataset]. Zenodo. https://doi.org/10.5281/zenodo.21436223

## Inventario inicial de tablas SQLite

La base SQLite fue abierta en DBeaver y se identificaron ocho tablas principales. Los conteos corresponden a la inspección realizada sobre la copia local del archivo:

| Tabla | Registros |
|---|---:|
| `analog_measurements` | 2.989.759 |
| `context` | 25.178 |
| `fault_history` | 2.129.517 |
| `settings` | 2.284.775 |
| `specs` | 1.333.174 |
| `statistics` | 2.285.904 |
| `status` | 2.240.052 |
| `totals` | 2.279.918 |

La tabla `analog_measurements` contiene 2.989.759 lecturas, por lo que el volumen disponible supera el requisito académico de cargar al menos un millón de lecturas. El cumplimiento definitivo se demostrará después de cargar y validar los datos en PostgreSQL.

## Periodo y granularidad temporal observada

Para `analog_measurements`, el campo `read_ts` almacena marcas de tiempo Unix con decimales. La consulta de mínimo y máximo devolvió:

- Primera lectura: `1752327095.265811` = **2025-07-12 13:31:35 UTC**.
- Última lectura: `1783919633.451776` = **2026-07-13 05:13:53 UTC**.
- Total: **2.989.759 lecturas**.

La granularidad o intervalo esperado entre lecturas todavía debe determinarse; no se debe asumir una frecuencia fija.

## Distribución por inversor

La consulta agrupada por `inverter` identificó dos inversores:

| Inversor | Registros | Primera lectura (Unix) | Última lectura (Unix) |
|---|---:|---:|---:|
| 1 | 1.610.276 | 1752327095.265811 | 1783919633.451776 |
| 2 | 1.379.483 | 1752472412.761199 | 1782106524.235112 |
| **Total** | **2.989.759** | | |

El inversor 1 representa aproximadamente el 53,86 % de las lecturas y el inversor 2 el 46,14 %. La diferencia entre ambos es de 230.793 registros. Sus periodos de actividad no son idénticos.

## Estructura original y observaciones preliminares

La inspección del DDL de las ocho tablas permitió observar:

- `analog_measurements` es una tabla ancha que reúne numerosas variables eléctricas y operativas en columnas.
- `fault_history` contiene grupos de columnas repetidas (`faulthistoryrecord00` a `faulthistoryrecord31`) y sus campos de tiempo asociados. Este diseño debe evaluarse al elaborar el modelo lógico.
- `settings` agrupa parámetros de configuración del inversor.
- `statistics` contiene campos con tipos numéricos y campos declarados como texto; se debe investigar su contenido antes de definir conversiones.
- En el DDL inspeccionado, `context` declara una columna `id` como clave primaria. Las demás tablas revisadas no declaran una clave primaria explícita en su sentencia CREATE TABLE.
- Los campos `read_ts` e `inverter` aparecen como NOT NULL en las tablas de telemetría revisadas.

Estas observaciones describen la estructura de origen; no demuestran por sí solas que existan errores en los datos.

## Validaciones de calidad realizadas en analog_measurements

### Identificación y duplicados

- Registros con `inverter` nulo: **0**.
- Registros con `read_ts` nulo: **0**.
- La consulta agrupada por `inverter, read_ts` con `HAVING COUNT(*) > 1` no devolvió filas. No se encontraron duplicados para esa combinación en la tabla inspeccionada.
- Por lo anterior, la combinación `(inverter, read_ts)` es candidata a clave primaria compuesta para las lecturas analógicas. Debe volver a validarse en la migración y en el diseño final.

### Rangos observados

| Variable | Rango global observado |
|---|---:|
| `battsoc` | 6 a 100 |
| `battvolt` | 0,0 a 57,6 |
| `pv1power` | 0 a 3.933 |

Comparación por inversor:

| Variable | Inversor 1 | Inversor 2 |
|---|---:|---:|
| `battsoc` mínimo | 6 | 10 |
| `battsoc` máximo | 100 | 100 |
| `battvolt` mínimo | 29,0 | 0,0 |
| `battvolt` máximo | 57,5 | 57,6 |
| `pv1power` mínimo | 0 | 0 |
| `pv1power` máximo | 3.902 | 3.933 |

Las unidades de estas variables deben confirmarse en la documentación técnica del dataset; no se asignan unidades basándose únicamente en los nombres de columna.

### Lecturas con voltaje de batería igual a cero

La consulta de conteo de `battvolt = 0` devolvió:

| Inversor | Lecturas con `battvolt = 0` |
|---|---:|
| 2 | 3 |

No se encontraron filas para el inversor 1. Las tres lecturas del inversor 2 representan aproximadamente el 0,00010 % de las 2.989.759 lecturas analógicas. Se registran como casos que requieren revisión contextual; **no se clasifican todavía como errores ni se eliminan**.

## Pendientes de análisis

- Verificar el checksum SHA-256 del archivo descargado.
- Registrar fecha de descarga y revisar la licencia de datos.
- Consultar la documentación de OpenCEM para confirmar significado y unidades de las variables.
- Analizar intervalos temporales faltantes y frecuencia de muestreo.
- Investigar las tres lecturas con `battvolt = 0` mediante su fecha, estado y demás variables.
- Perfilar valores nulos y rangos en otras columnas y tablas.
- Diseñar el modelo conceptual y lógico a partir de la estructura y semántica verificadas.
- Migrar a PostgreSQL y demostrar conteos e integridad después de la carga.

## Evidencias y consultas

Las consultas de conteo, periodo, distribución por inversor, duplicados, valores nulos, rangos y lecturas con voltaje cero se ejecutaron en DBeaver sobre la conexión SQLite `opencem-v1.0.0.sqlite`. Conservar los resultados o capturas de estas consultas como evidencia del análisis de la fuente.


### Revisión contextual de las tres lecturas con voltaje de batería cero

Se consultaron las filas de `analog_measurements` donde `battvolt = 0`. Las tres corresponden al inversor 2:

| Fecha UTC | `battvolt` | `battsoc` | `battcurr` | `pv1volt` | `pv1curr` | `pv1power` | `linevolta` | `linecurra` | `gridpowerw_a` |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 2026-05-12 10:16:17 | 0,0 | 87 | 0,0 | 0,0 | 0,0 | 0 | 0,0 | 0,0 | 0 |
| 2026-05-15 04:10:28 | 0,0 | 87 | 0,0 | 0,0 | 0,0 | 0 | 0,0 | 0,0 | 0 |
| 2026-06-19 04:35:37 | 0,0 | 87 | 0,0 | 0,0 | 0,0 | 0 | 0,0 | 0,0 | 0 |

En las tres filas también se observaron `linevolta = 0`, `linecurra = 0` y `gridpowerw_a = 0`. El estado de carga `battsoc` se mantuvo en 87. El patrón muestra que el voltaje de batería cero coincide con ceros en varias otras variables eléctricas de la misma lectura.

La interpretación sigue siendo **pendiente de validación**: podría relacionarse con inactividad, desconexión o una condición de adquisición/registro, pero estas posibilidades no pueden confirmarse solo con las columnas consultadas. No se eliminan ni se convierten estos valores durante esta etapa exploratoria.


### Cruce con la tabla status en la misma marca temporal

Se realizó un LEFT JOIN entre `analog_measurements` y `status` utilizando simultáneamente `inverter` y `read_ts`, limitado a las tres lecturas con `battvolt = 0`. Los resultados fueron:

| Fecha UTC | `startupstatus` | `outstatus` | `chgstatus` | Otros campos de estado |
|---|---:|---:|---:|---|
| 2026-05-12 10:16:17 | 1 | 0 | 0 | Se obtuvieron valores en `paramode`, `ecomode`, `sysstateflag`, `sysalarmflag` y `derbattstate`; `derinverterstate` quedó vacío. |
| 2026-05-15 04:10:28 | Sin coincidencia completa | Sin coincidencia completa | Sin coincidencia completa | Los campos escalares aparecen vacíos y los arreglos de estado aparecen como `[null]`. |
| 2026-06-19 04:35:37 | 1 | Sin dato | Sin dato | Los demás campos consultados aparecen vacíos. |

El cruce permite observar que la primera lectura sí tiene varios campos de estado disponibles en la misma marca temporal; la segunda no presenta datos de estado útiles en el resultado; y la tercera solo presenta `startupstatus = 1`. Por tanto, la información de `status` es incompleta o variable para estos tres casos.

Este resultado no permite concluir por sí solo si hubo desconexión, inactividad o un problema de captura. La ausencia de valores en el cruce exacto tampoco demuestra que no existan registros cercanos en el tiempo. Como siguiente validación se buscarán las filas de `status` temporalmente más cercanas a cada lectura, midiendo la diferencia en segundos. Se conservan los datos originales sin alteración.
