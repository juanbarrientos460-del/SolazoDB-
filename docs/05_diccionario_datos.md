# 05. Diccionario de datos

## Convenciones técnicas

- **Esquema principal:** `solazodb` (datos operativos normalizados).
- **Zona horaria:** Todas las marcas temporales (`TIMESTAMPTZ`) se almacenan en **UTC**, convertidas desde segundos Unix fraccionales (`to_timestamp(read_ts)`).
- **Tratamiento de nulos:** Se preservan los valores `SQL NULL` de la fuente original. No se imputan ceros sobre datos ausentes.

## 1. Tabla: `solazodb.inversores`

| Esquema | Tabla | Columna | Tipo PostgreSQL | PK/FK | Nulo | Unidad / Formato | Descripción |
|---|---|---|---|---|---|---|---|
| `solazodb` | `inversores` | `inverter_id` | `SMALLINT` | PK | NO | Entero (1, 2) | Identificador único del inversor en la microrred OpenCEM. |
| `solazodb` | `inversores` | `codigo_equipo` | `VARCHAR(30)` | — | NO | Texto | Etiqueta descriptiva del inversor (ej. `INV-01`, `INV-02`). |
| `solazodb` | `inversores` | `activo` | `BOOLEAN` | — | NO | Booleano | Indica si el equipo está habilitado en el modelo operativo. |

## 2. Tabla: `solazodb.lecturas_analogicas`

| Esquema | Tabla | Columna | Tipo PostgreSQL | PK/FK | Nulo | Unidad / Formato | Descripción |
|---|---|---|---|---|---|---|---|
| `solazodb` | `lecturas_analogicas` | `inverter_id` | `SMALLINT` | PK, FK | NO | Entero | Referencia a `solazodb.inversores(inverter_id)`. |
| `solazodb` | `lecturas_analogicas` | `read_ts` | `TIMESTAMPTZ` | PK | NO | UTC (ISO-8601) | Marca de tiempo exacta de la lectura convertida desde Unix seconds. |
| `solazodb` | `lecturas_analogicas` | `read_ts_unix` | `DOUBLE PRECISION` | — | NO | Segundos Unix | Marca temporal original en segundos Unix con decimales (trazabilidad). |
| `solazodb` | `lecturas_analogicas` | `battsoc` | `NUMERIC(5,2)` | — | SÍ | % (6 a 100) | Estado de carga de la batería (State of Charge). |
| `solazodb` | `lecturas_analogicas` | `battvolt` | `NUMERIC(6,2)` | — | SÍ | V (0,0 a 57,6) | Voltaje medido en terminales de la batería. |
| `solazodb` | `lecturas_analogicas` | `battcurr` | `NUMERIC(7,2)` | — | SÍ | A | Corriente de carga/descarga de la batería. |
| `solazodb` | `lecturas_analogicas` | `pv1power` | `NUMERIC(8,2)` | — | SÍ | W (0 a 3.933) | Potencia instantánea generada por el arreglo fotovoltaico 1. |
| `solazodb` | `lecturas_analogicas` | `pv1volt` | `NUMERIC(6,2)` | — | SÍ | V | Voltaje de entrada del arreglo fotovoltaico 1. |
| `solazodb` | `lecturas_analogicas` | `pv1curr` | `NUMERIC(7,2)` | — | SÍ | A | Corriente de entrada del arreglo fotovoltaico 1. |
| `solazodb` | `lecturas_analogicas` | `linevolta` | `NUMERIC(6,2)` | — | SÍ | V | Voltaje de línea fase A de la red eléctrica. |
| `solazodb` | `lecturas_analogicas` | `linecurra` | `NUMERIC(7,2)` | — | SÍ | A | Corriente de línea fase A de la red eléctrica. |
| `solazodb` | `lecturas_analogicas` | `gridpowerw_a` | `NUMERIC(8,2)` | — | SÍ | W | Potencia activa de red en fase A. |

## 3. Tabla: `solazodb.estados_operativos`

| Esquema | Tabla | Columna | Tipo PostgreSQL | PK/FK | Nulo | Unidad / Formato | Descripción |
|---|---|---|---|---|---|---|---|
| `solazodb` | `estados_operativos` | `inverter_id` | `SMALLINT` | PK, FK | NO | Entero | Referencia a `solazodb.inversores(inverter_id)`. |
| `solazodb` | `estados_operativos` | `read_ts` | `TIMESTAMPTZ` | PK | NO | UTC (ISO-8601) | Marca de tiempo del registro de estado. |
| `solazodb` | `estados_operativos` | `startupstatus` | `SMALLINT` | — | SÍ | Código discreto | Estado de arranque del inversor (ej. `1`). |
| `solazodb` | `estados_operativos` | `outstatus` | `SMALLINT` | — | SÍ | Código discreto | Estado de salida del inversor. |
| `solazodb` | `estados_operativos` | `chgstatus` | `SMALLINT` | — | SÍ | Código discreto | Estado del cargador de batería. |
| `solazodb` | `estados_operativos` | `paramode` | `SMALLINT` | — | SÍ | Código discreto | Modo de operación en paralelo. |
| `solazodb` | `estados_operativos` | `ecomode` | `SMALLINT` | — | SÍ | Código discreto | Indicador de modo económico. |
| `solazodb` | `estados_operativos` | `sysstateflag` | `INTEGER` | — | SÍ | Bandera de bits | Bandera general de estado del sistema. |
| `solazodb` | `estados_operativos` | `sysalarmflag` | `INTEGER` | — | SÍ | Bandera de bits | Bandera de alarmas activas del sistema. |
| `solazodb` | `estados_operativos` | `derbattstate` | `SMALLINT` | — | SÍ | Código discreto | Estado reportado de la batería distribuida. |
| `solazodb` | `estados_operativos` | `derinverterstate` | `SMALLINT` | — | SÍ | Código discreto | Estado reportado del inversor distribuido. |

## 4. Tabla: `solazodb.configuraciones`

| Esquema | Tabla | Columna | Tipo PostgreSQL | PK/FK | Nulo | Unidad / Formato | Descripción |
|---|---|---|---|---|---|---|---|
| `solazodb` | `configuraciones` | `inverter_id` | `SMALLINT` | PK, FK | NO | Entero | Referencia a `solazodb.inversores(inverter_id)`. |
| `solazodb` | `configuraciones` | `read_ts` | `TIMESTAMPTZ` | PK | NO | UTC (ISO-8601) | Marca de tiempo de la lectura de configuración. |
| `solazodb` | `configuraciones` | `outvoltset` | `NUMERIC(6,2)` | — | SÍ | V (ej. 230.0) | Voltaje nominal de salida configurado. |
| `solazodb` | `configuraciones` | `outfreqset` | `NUMERIC(5,2)` | — | SÍ | Hz (ej. 50.0) | Frecuencia nominal de salida configurada. |
| `solazodb` | `configuraciones` | `maxchgcurrset` | `NUMERIC(6,2)` | — | SÍ | A (ej. 80.0) | Corriente máxima de carga configurada. |
| `solazodb` | `configuraciones` | `battype` | `SMALLINT` | — | SÍ | Código (ej. 4) | Código de tipo de batería configurado (95,9 % SQL NULL en origen). |
| `solazodb` | `configuraciones` | `cvvolt` | `NUMERIC(6,2)` | — | SÍ | V (ej. 57.6) | Voltaje configurado de carga a voltaje constante. |
| `solazodb` | `configuraciones` | `floatvolt` | `NUMERIC(6,2)` | — | SÍ | V (ej. 57.6) | Voltaje configurado de flotación de batería. |
| `solazodb` | `configuraciones` | `batsoclowalarm` | `SMALLINT` | — | SÍ | % (ej. 15) | Umbral de alarma por bajo estado de carga. |
| `solazodb` | `configuraciones` | `batstopsoc` | `SMALLINT` | — | SÍ | % (ej. 5) | Umbral de parada por descarga de batería. |
| `solazodb` | `configuraciones` | `battfullsoc` | `SMALLINT` | — | SÍ | % (95 o 100) | Umbral configurado de batería llena. |
| `solazodb` | `configuraciones` | `inv2linesoc` | `SMALLINT` | — | SÍ | % (ej. 10) | Umbral SOC de transferencia de inversor a red. |
| `solazodb` | `configuraciones` | `lineback2invsoc` | `SMALLINT` | — | SÍ | % (ej. 100) | Umbral SOC de retorno de red a inversor. |
| `solazodb` | `configuraciones` | `reserve` | `JSONB` | — | SÍ | Arreglo JSON (15 pos.) | Payload de registros reservados (`component-defined`). |
| `solazodb` | `configuraciones` | `battsetreserve` | `JSONB` | — | SÍ | Arreglo JSON (8 pos.) | Payload de reserva de configuración de batería (`component-defined`). |

## 5. Tabla: `solazodb.historial_fallos`

| Esquema | Tabla | Columna | Tipo PostgreSQL | PK/FK | Nulo | Unidad / Formato | Descripción |
|---|---|---|---|---|---|---|---|
| `solazodb` | `historial_fallos` | `inverter_id` | `SMALLINT` | PK, FK | NO | Entero | Referencia a `solazodb.inversores(inverter_id)`. |
| `solazodb` | `historial_fallos` | `read_ts` | `TIMESTAMPTZ` | PK | NO | UTC (ISO-8601) | Marca de tiempo de captura del bloque de historial. |
| `solazodb` | `historial_fallos` | `faulthistoryrecord00` .. `31` | `JSONB` | — | SÍ | Arreglo JSON (16 pos.) | Payloads decodificados de historial de fallos (slots `00` a `31`). |
| `solazodb` | `historial_fallos` | `faulthistoryrecord00_time` .. `31_time` | `TIMESTAMPTZ` | — | SÍ | UTC (ISO-8601) | Tiempo calibrado del evento asociado a cada slot `00` a `31`. |
