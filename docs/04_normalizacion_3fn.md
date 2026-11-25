# 04. Normalización hasta tercera forma normal (3FN)

## Objetivo

Demostrar que el diseño relacional de `SolazoDB` elimina redundancias estructurales y anomalías de inserción, actualización y borrado, respetando la semántica real de OpenCEM v1.0.0.

## Primera Forma Normal (1FN)

**Reglas verificadas:**
1. **Identificación única:** En SQLite las tablas `analog_measurements`, `status`, `settings` y `fault_history` carecían de `PRIMARY KEY`. Mediante consultas de agrupación (`HAVING COUNT(*) > 1`) se comprobó que `(inverter, read_ts)` no tiene duplicados ni valores nulos, adoptándose como clave primaria compuesta.
2. **Atomicidad y manejo de estructuras JSON:**
   - Los atributos escalares (`battsoc`, `battvolt`, `pv1power`, etc.) almacenan un único valor atómico por celda.
   - Los campos tipo arreglo (`reserve`, `battsetreserve` y `faulthistoryrecord00..31`) fueron confirmados en el diccionario oficial como payloads binarios/registro codificados en JSON cuya semántica posicional depende del componente (`component-defined`). Para evitar invención de columnas sin respaldo documental, se almacenan como tipos nativos `JSONB` atómicos a nivel de payload.
   - En `historial_fallos`, las 32 parejas `(faulthistoryrecordXX, faulthistoryrecordXX_time)` pueden mantenerse en estructura columnar controlada o normalizarse en una tabla hija `(inverter_id, read_ts, slot_id, payload_jsonb, event_time)` para eliminar los 32 grupos repetitivos y evitar almacenar ~96,7 % de columnas nulas en los slots `03` al `31`.

## Segunda Forma Normal (2FN)

**Reglas verificadas:**
1. Se cumple 1FN en todas las relaciones.
2. **Eliminación de dependencias parciales:** En las tablas de telemetría cuya clave primaria es compuesta `(inverter_id, read_ts)`, todos los atributos no clave (mediciones eléctricas, estados y parámetros) dependen funcionalmente de la combinación completa **inversor + marca de tiempo exacta**.
3. Cualquier atributo propio exclusivamente del equipo físico (como su identificador, descripción o rol en la microrred) se extrae a la entidad maestro `inversores(inverter_id, ...)`, evitando repetir metadatos del equipo en casi 3 millones de filas.

## Tercera Forma Normal (3FN)

**Reglas verificadas:**
1. Se cumple 2FN en todas las relaciones.
2. **Eliminación de dependencias transitivas:**
   - Los atributos no clave dentro de `lecturas_analogicas`, `estados_operativos` y `configuraciones` representan mediciones o parámetros independientes capturados en ese instante; ningún atributo no clave determina funcionalmente a otro atributo no clave.
   - Los indicadores derivados (como balances energéticos, variaciones entre lecturas o promedios móviles) **no se almacenan como columnas redundantes** en las tablas base; se calculan dinámicamente mediante **vistas** y **vistas materializadas** (Línea 4), eliminando dependencias transitivas y anomalías de actualización.

## Resumen de descomposición

| Origen SQLite | Tablas PostgreSQL (3FN) | Dependencias funcionales | Justificación técnica |
|---|---|---|---|
| `analog_measurements` (sin PK) | `inversores` + `lecturas_analogicas` | `inverter_id -> nombre_equipo`<br>`(inverter_id, read_ts) -> {battsoc, battvolt, battcurr, pv1power, ...}` | Se formaliza la PK compuesta, se separa el catálogo de inversores y se delegan los cálculos derivados a vistas. |
| `status` (sin PK) | `estados_operativos` | `(inverter_id, read_ts) -> {startupstatus, outstatus, chgstatus, ...}` | Se desacopla de `lecturas_analogicas` debido a la disponibilidad temporal desigual comprobada en el cruce. |
| `settings` (176 cols, sin PK) | `configuraciones` | `(inverter_id, read_ts) -> {outvoltset, outfreqset, cvvolt, reserve, ...}` | Se conserva la trazabilidad temporal de parámetros y se tipifican los arreglos JSON válidos como `JSONB`. |
| `fault_history` (66 cols repetitivas) | `historial_fallos` (y detalle opcional por slot `00..31`) | `(inverter_id, read_ts) -> {record_00..31, time_00..31}` | Se convierten los timestamps Unix `REAL` a `TIMESTAMPTZ` UTC y los textos JSON a `JSONB` preservando nulos. |
