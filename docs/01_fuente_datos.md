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


## Perfil de valores nulos en variables principales de analog_measurements

Se ejecutó una consulta de conteo de valores nulos para seis variables de medición. Sobre un total de **2.989.759 registros**, se obtuvieron los siguientes resultados:

| Variable | Valores nulos | Porcentaje sobre el total |
|---|---:|---:|
| `battsoc` | 57.931 | 1,94 % |
| `battvolt` | 58.141 | 1,95 % |
| `battcurr` | 58.217 | 1,95 % |
| `pv1power` | 58.660 | 1,96 % |
| `pv1volt` | 59.379 | 1,99 % |
| `pv1curr` | 58.001 | 1,94 % |

Los porcentajes se calcularon respecto al total de filas de `analog_measurements` y se redondearon a dos decimales. Se confirma la presencia de valores faltantes en las seis variables revisadas. Este hallazgo no demuestra por sí solo que los registros sean inválidos: antes de definir reglas de limpieza o restricciones `NOT NULL`, se debe revisar la distribución de nulos por inversor y su coincidencia entre variables. No se reemplazan ni eliminan valores en esta etapa.


### Perfil de valores nulos en variables eléctricas de `analog_measurements`

Se evaluaron los valores nulos de seis variables relevantes de batería y generación fotovoltaica sobre los 2.989.759 registros de `analog_measurements`.

| Variable | Nulos | Porcentaje del total |
|---|---:|---:|
| `battsoc` | 57.931 | 1,938 % |
| `battvolt` | 58.141 | 1,945 % |
| `battcurr` | 58.217 | 1,947 % |
| `pv1power` | 58.660 | 1,962 % |
| `pv1volt` | 59.379 | 1,986 % |
| `pv1curr` | 58.001 | 1,940 % |

#### Distribución por inversor

| Variable | Inversor 1: nulos | Inversor 2: nulos |
|---|---:|---:|
| `battsoc` | 809 | 57.122 |
| `battvolt` | 771 | 57.370 |
| `battcurr` | 854 | 57.363 |
| `pv1power` | 1.233 | 57.427 |
| `pv1volt` | 1.428 | 57.951 |
| `pv1curr` | 790 | 57.211 |

Los resultados muestran una concentración marcada de nulos en el inversor 2. En las seis variables, los porcentajes de nulos por inversor son aproximadamente 0,05–0,09 % para el inversor 1 y 4,14–4,20 % para el inversor 2. Esta diferencia es un hallazgo de perfilamiento que requiere investigación; por sí sola no demuestra una falla física ni permite determinar la causa de los valores ausentes.

Los valores nulos se conservarán como ausentes durante la preparación y migración. No se sustituirán automáticamente por cero, porque cero puede ser una medición válida y no equivale a un dato desconocido.


### Coincidencia de valores nulos entre seis variables

Se evaluó si los nulos de `battsoc`, `battvolt`, `battcurr`, `pv1power`, `pv1volt` y `pv1curr` aparecen simultáneamente en una misma fila de `analog_measurements`.

| Inversor | Total de registros | Seis variables nulas simultáneamente | Registros con al menos un nulo |
|---|---:|---:|---:|
| 1 | 1.610.276 | 1 | 5.289 |
| 2 | 1.379.483 | 55.657 | 65.606 |

En el inversor 2, 55.657 de los 65.606 registros con al menos un nulo tienen las seis variables nulas simultáneamente. En el inversor 1 solo se observó un registro con esa condición. El patrón indica que gran parte de los valores ausentes del inversor 2 se presentan conjuntamente, lo que podría relacionarse con condiciones de adquisición o disponibilidad de mediciones. La causa no se puede establecer únicamente con esta consulta y queda pendiente de investigación.

Los registros se conservarán sin imputar automáticamente los valores ausentes.


### Distribución diaria de registros con seis variables nulas

Se agruparon por día los registros de `analog_measurements` en los que las seis variables `battsoc`, `battvolt`, `battcurr`, `pv1power`, `pv1volt` y `pv1curr` son nulas simultáneamente. Los 15 días con mayor cantidad reportada pertenecen al inversor 2 y se concentran entre el 13 de mayo y el 2 de junio de 2026.

| Fecha | Registros con seis variables nulas |
|---|---:|
| 2026-05-20 | 1.450 |
| 2026-05-25 | 1.450 |
| 2026-05-13 | 1.449 |
| 2026-05-18 | 1.449 |
| 2026-05-19 | 1.449 |
| 2026-05-22 | 1.449 |
| 2026-05-23 | 1.449 |
| 2026-05-26 | 1.449 |
| 2026-05-27 | 1.449 |
| 2026-05-28 | 1.449 |
| 2026-05-29 | 1.449 |
| 2026-05-30 | 1.449 |
| 2026-05-31 | 1.449 |
| 2026-06-01 | 1.449 |
| 2026-06-02 | 1.449 |

La similitud de los conteos diarios sugiere un patrón recurrente de ausencia de mediciones en ese periodo. El resultado por sí solo no permite atribuir una causa física u operativa. Se requiere contrastar estos días con el total diario de lecturas y con las demás tablas antes de concluir que hubo interrupciones de adquisición.


## Análisis exploratorio de la tabla fault_history

La tabla `fault_history` contiene **2.129.517 registros**. En la inspección de su estructura se identificaron 32 columnas de historial (`faulthistoryrecord00` a `faulthistoryrecord31`) y 32 columnas de tiempo asociadas (`faulthistoryrecord00_time` a `faulthistoryrecord31_time`).

### Identificación de registros y valores

- La consulta de nulos en `inverter` y `read_ts` devolvió cero valores nulos.
- La consulta de duplicados agrupando por `(inverter, read_ts)` no encontró grupos duplicados.
- Por tanto, `(inverter, read_ts)` es una **clave candidata** para la tabla de origen. Su adopción como clave primaria en PostgreSQL queda sujeta a validación durante la migración.
- Una consulta inicial de cinco filas mostró que algunos registros tienen los campos de historial vacíos; por ello, una muestra pequeña no representa necesariamente el contenido de toda la tabla.

### Estructura JSON observada

Los campos `faulthistoryrecord00` y `faulthistoryrecord01` se inspeccionaron mediante funciones JSON de SQLite:

- `faulthistoryrecord00` tiene tipo `text`, es reconocido como JSON válido y en la muestra consultada presentó arreglos de 16 elementos y arreglos de un elemento.
- Para `faulthistoryrecord00`, el perfil de longitudes dio 56.909 arreglos de un elemento y 2.066.726 arreglos de 16 elementos, sobre 2.123.635 valores no nulos.
- Una consulta adicional confirmó que los 56.909 arreglos de un elemento de `faulthistoryrecord00` son exactamente `[null]`; los 56.909 tienen su campo `faulthistoryrecord00_time` nulo.
- Para `faulthistoryrecord01`, el perfil de longitudes dio 56.855 arreglos de un elemento y 2.067.589 arreglos de 16 elementos, sobre 2.124.444 valores no nulos. No se verificó todavía el contenido exacto de todos los arreglos de un elemento de este campo.
- En los ejemplos consultados, los arreglos de 16 elementos contienen valores numéricos y su campo de tiempo asociado contiene una marca de tiempo REAL. Algunos registros muestran arreglos repetidos en distintas filas con diferentes valores de `read_ts`; este comportamiento requiere interpretación con la documentación técnica.

Los 16 elementos no se interpretan todavía como 16 fallos independientes. No se conoce el significado de cada posición, por lo que no se asignan nombres ni se descompone el arreglo en columnas inventadas. Se debe conservar el contenido original durante la migración hasta confirmar su semántica.

### Perfil de valores nulos en los campos de tiempo

Se consultaron los valores nulos de los 32 campos de tiempo. El total de filas de la tabla fue 2.129.517. Los resultados obtenidos para cada grupo fueron:

| Campo de tiempo | Valores nulos |
|---|---:|
| `faulthistoryrecord00_time` | 84.395 |
| `faulthistoryrecord01_time` | 83.489 |
| `faulthistoryrecord02_time` | 83.827 |
| `faulthistoryrecord03_time` | 2.057.367 |
| `faulthistoryrecord04_time` | 2.057.377 |
| `faulthistoryrecord05_time` | 2.057.563 |
| `faulthistoryrecord06_time` | 2.057.574 |
| `faulthistoryrecord07_time` | 2.057.538 |
| `faulthistoryrecord08_time` | 2.058.008 |
| `faulthistoryrecord09_time` | 2.057.977 |
| `faulthistoryrecord10_time` | 2.057.943 |
| `faulthistoryrecord11_time` | 2.057.959 |
| `faulthistoryrecord12_time` | 2.057.960 |
| `faulthistoryrecord13_time` | 2.057.935 |
| `faulthistoryrecord14_time` | 2.057.950 |
| `faulthistoryrecord15_time` | 2.058.183 |
| `faulthistoryrecord16_time` | 2.058.135 |
| `faulthistoryrecord17_time` | 2.058.148 |
| `faulthistoryrecord18_time` | 2.058.012 |
| `faulthistoryrecord19_time` | 2.058.051 |
| `faulthistoryrecord20_time` | 2.058.064 |
| `faulthistoryrecord21_time` | 2.058.049 |
| `faulthistoryrecord22_time` | 2.057.619 |
| `faulthistoryrecord23_time` | 2.057.623 |
| `faulthistoryrecord24_time` | 2.057.589 |
| `faulthistoryrecord25_time` | 2.057.658 |
| `faulthistoryrecord26_time` | 2.057.674 |
| `faulthistoryrecord27_time` | 2.057.668 |
| `faulthistoryrecord28_time` | 2.062.295 |
| `faulthistoryrecord29_time` | 2.059.580 |
| `faulthistoryrecord30_time` | 2.057.681 |
| `faulthistoryrecord31_time` | 2.057.653 |

Los tres primeros campos de tiempo tienen aproximadamente entre 3,9 % y 4,0 % de valores nulos. Los campos 03–31 tienen aproximadamente entre 96,6 % y 96,9 % de valores nulos. La distribución es desigual y debe considerarse al diseñar la transformación; los valores ausentes se conservarán como ausentes, sin sustituirlos automáticamente por cero ni eliminar registros.

### Decisión provisional de modelado

La estructura de origen presenta grupos de columnas repetidas y arreglos JSON, por lo que debe evaluarse una transformación que permita consultar los registros de historial de forma relacional. Sin embargo, todavía no se define una tabla hija con una fila por elemento: primero se debe confirmar en la documentación de OpenCEM qué representa cada posición del arreglo, cómo se relacionan los 32 campos entre sí y qué significa cada marca de tiempo. La migración debe preservar los valores originales y permitir rastrear cada dato hasta su fila de origen.

Estas consultas se ejecutaron en DBeaver sobre la conexión SQLite del archivo original. Los resultados son parte del perfilamiento exploratorio y no constituyen por sí solos una interpretación funcional de los códigos.


### Confirmación documental de los arreglos de `fault_history`

La ficha del diccionario de columnas para `fault_history.faulthistoryrecord00` confirma que el campo conserva un payload de múltiples registros en un arreglo JSON codificado como texto UTF-8. Su descripción indica que la semántica de los elementos depende del componente (`component-defined`), por lo que no se deben asignar nombres o significados a las 16 posiciones sin documentación específica del componente.

La ficha también establece que los valores ya están convertidos y no requieren escalado adicional de registros; que `SQL NULL` representa información ausente o no disponible y no debe imputarse; y que el campo aplica a ambos inversores cuando hay valores disponibles, aunque los canales estructurales inactivos fueron depurados.

**Implicación para el diseño:** conservar el payload original como JSON (evaluar `JSONB` en PostgreSQL) y mantener su campo de tiempo asociado. No descomponer sus posiciones en atributos relacionales con semántica supuesta. Esta decisión se debe revisar si el diccionario aporta posteriormente una definición explícita de los elementos.


### Semántica de la marca de tiempo de `faulthistoryrecord00`

El diccionario de columnas define `fault_history.faulthistoryrecord00_time` como un valor `REAL` que contiene el tiempo calibrado del evento decodificado desde el payload histórico `faulthistoryrecord00`. Está expresado como timestamp Unix fraccional en segundos desde 1970-01-01 UTC. El valor ya está convertido y no requiere escalado adicional. `SQL NULL` representa un tiempo ausente o no disponible y no debe imputarse.

**Implicación para PostgreSQL:** se puede transformar a `TIMESTAMPTZ` mediante conversión explícita desde segundos Unix, manteniendo la zona horaria UTC. Este campo debe conservar su asociación con el payload `faulthistoryrecord00`; no debe confundirse con la hora de inserción de la fila.


### Estructura común de las 32 marcas de tiempo de `fault_history`

La revisión del diccionario confirma que las 32 columnas `faulthistoryrecord00_time` a `faulthistoryrecord31_time` (posiciones 34 a 65) comparten la misma definición: tipo SQLite `REAL`, segundos Unix fraccionales en UTC desde 1970-01-01, tiempo de evento calibrado y decodificado desde el payload correspondiente. Los valores ya están convertidos y no requieren escalado adicional. `SQL NULL` significa tiempo ausente o no disponible y no debe imputarse. La descripción de cada columna cambia únicamente para referirse al payload `faulthistoryrecordXX` correspondiente.

**Implicación para el diseño:** aplicar una regla uniforme de conversión a tipo temporal con zona horaria en PostgreSQL para las 32 marcas, manteniendo cada tiempo asociado a su payload correspondiente. La conversión debe tratar el valor original como segundos Unix, no como milisegundos, y preservar los valores nulos.


### Análisis de la tabla `settings`

La tabla `settings` contiene 176 columnas. Su estructura incluye la identificación temporal (`read_ts`, tipo `REAL`, obligatorio) y el inversor (`inverter`, tipo `INTEGER`, obligatorio), además de parámetros de configuración eléctrica, prioridades de operación, batería, protecciones de red, programación de carga/descarga y controles de potencia.

El conteo por inversor en SQLite arrojó 1.280.052 registros para el inversor 1 y 1.004.723 para el inversor 2, para un total de 2.284.775. El periodo observado fue:
- Inversor 1: `read_ts` mínimo 1752327095.265811 y máximo 1783919633.451776.
- Inversor 2: `read_ts` mínimo 1752472412.761199 y máximo 1782106524.235112.

La consulta de duplicados agrupando por `(inverter, read_ts)` y filtrando grupos con más de una fila no devolvió resultados. Por tanto, esa combinación es una **clave primaria compuesta candidata** para `settings`; su adopción definitiva queda sujeta a confirmar la semántica y las relaciones con las demás tablas.

**Observación de modelado:** los campos `reserve` y `battsetreserve` son de tipo SQLite `TEXT`; su contenido y formato deben inspeccionarse antes de decidir su representación en PostgreSQL. No se deben asumir unidades o significados de los parámetros únicamente a partir de sus nombres.


### Validación de los campos `reserve` y `battsetreserve` en `settings`

Se ejecutó una validación con las funciones `json_valid` y comprobación de `IS NULL` en SQLite. Los resultados fueron:

| Campo | Total de filas | SQL NULL | JSON válido no nulo | JSON no válido |
|---|---:|---:|---:|---:|
| `reserve` | 2.284.775 | 6.297 | 2.278.478 | 0 |
| `battsetreserve` | 2.284.775 | 4.819 | 2.279.956 | 0 |

Todos los valores no nulos de ambos campos son JSON válido. En muestras se observaron arreglos de 15 posiciones en `reserve` y de 8 posiciones en `battsetreserve`, además de arreglos como `[null]`. Estos arreglos JSON no deben confundirse con `SQL NULL`: el primero es un valor JSON presente y el segundo representa ausencia SQL.

**Implicación para PostgreSQL:** ambos campos son candidatos para conservarse como `JSONB`, preservando los valores JSON y los SQL NULL. Antes de interpretar o normalizar sus posiciones, se debe consultar la descripción y semántica de ambos campos en el diccionario oficial de columnas.


### Definición oficial del campo `settings.reserve`

El diccionario de columnas define `settings.reserve` (posición 77) como tipo SQLite `TEXT`, codificado como arreglo JSON en texto UTF-8. El campo conserva el payload original de múltiples registros; la semántica de sus elementos depende del componente (`component-defined`). Los valores ya están convertidos y no requieren escalado adicional. `SQL NULL` significa que el valor está ausente o no disponible y no debe imputarse. El campo aplica a ambos inversores cuando hay valores disponibles; los canales estructurales inactivos fueron depurados.

Esta definición confirma que no se deben asignar significados a las posiciones del arreglo sin documentación específica del componente. Se mantiene la recomendación de preservar el contenido como JSON (candidato a `JSONB` en PostgreSQL), diferenciando SQL NULL de valores JSON como `[null]`.


### Definición oficial del campo `settings.battsetreserve`

El diccionario de columnas define `settings.battsetreserve` (posición 118) como tipo SQLite `TEXT`, codificado como arreglo JSON en texto UTF-8. Conserva el payload original de múltiples registros y la semántica de sus elementos depende del componente (`component-defined`). Los valores ya están convertidos y no requieren escalado adicional. `SQL NULL` representa un valor ausente o no disponible y no debe imputarse. El campo aplica a ambos inversores cuando hay valores disponibles; los canales estructurales inactivos fueron depurados.

Al igual que `reserve`, no se deben asignar significados supuestos a las posiciones del arreglo. Se recomienda preservar el contenido como JSON (candidato a `JSONB` en PostgreSQL), distinguiendo SQL NULL de valores JSON como `[null]`.


### Validación de columnas JSON en `settings` (reserva)

Se verificaron las columnas `reserve` y `battsetreserve`:

| Columna | Registros totales | SQL NULL | JSON válido no nulo | JSON inválido |
|---|---:|---:|---:|---:|
| `reserve` | 2.284.775 | 6.297 | 2.278.478 | 0 |
| `battsetreserve` | 2.284.775 | 4.819 | 2.279.956 | 0 |

Todos los valores no nulos de ambas columnas son JSON válido. Se mantiene la distinción entre SQL `NULL` (dato ausente/no disponible) y arreglos JSON como `[null]`; no se deben reemplazar automáticamente por cero ni interpretar las posiciones sin documentación específica del componente. Para el modelo PostgreSQL se propone conservar ambos arreglos como `JSONB`, sujeto a la definición final del esquema.


### Análisis de valores nulos en parámetros de `settings`

Consulta de seis parámetros sobre los 2.284.775 registros de la tabla:

| Columna | Valores SQL NULL |
|---|---:|
| `outvoltset` | 58.317 |
| `outfreqset` | 58.409 |
| `maxchgcurrset` | 58.290 |
| `battype` | 2.190.892 |
| `cvvolt` | 58.504 |
| `floatvolt` | 58.709 |

La tabla contiene dos inversores. Los seis parámetros presentan una proporción de valores ausentes muy diferente: `outvoltset`, `outfreqset`, `maxchgcurrset`, `cvvolt` y `floatvolt` tienen aproximadamente entre 2,55 % y 2,57 % de valores SQL NULL; `battype` presenta aproximadamente 95,90 %.

Este patrón se registra como hallazgo de calidad y disponibilidad, no como prueba de corrupción. Se conservarán los valores ausentes como SQL `NULL`; antes de definir restricciones de obligatoriedad para estos atributos se debe revisar su distribución por inversor y la definición del diccionario oficial.


### Distribución de valores nulos de `settings` por inversor

Se desglosaron los valores ausentes de cuatro parámetros por inversor:

| Inversor | Registros | `battype` NULL | `outvoltset` NULL | `cvvolt` NULL | `floatvolt` NULL |
|---:|---:|---:|---:|---:|---:|
| 1 | 1.280.052 | 1.232.422 | 1.296 | 1.424 | 1.570 |
| 2 | 1.004.723 | 958.470 | 57.021 | 57.080 | 57.139 |

El atributo `battype` presenta una proporción muy alta de valores SQL NULL en ambos inversores (aproximadamente 96,3 % en el inversor 1 y 95,4 % en el inversor 2). En cambio, los parámetros `outvoltset`, `cvvolt` y `floatvolt` tienen pocos nulos en el inversor 1 (alrededor de 0,10–0,12 %) y muchos más en el inversor 2 (alrededor de 5,7 %).

La diferencia entre inversores se conserva como una observación descriptiva de disponibilidad de datos. No se infiere su causa. Se recomienda mantener estos campos anulables en el modelo hasta verificar sus definiciones y reglas de captura en la documentación de OpenCEM.


### Valores observados de `settings.battype`

Al agrupar los valores no nulos por inversor, el único código observado fue:

| Inversor | Código observado | Registros |
|---:|---:|---:|
| 1 | 4 | 47.630 |
| 2 | 4 | 46.253 |

En total se observaron 93.883 registros no nulos, todos con el valor `4`. El significado técnico de ese código no se interpreta sin una correspondencia explícita en la documentación oficial. El atributo conserva sus valores originales y se mantiene anulable, dado el alto porcentaje de SQL NULL.
