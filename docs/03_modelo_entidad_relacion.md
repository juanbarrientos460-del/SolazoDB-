# 03. Modelo entidad-relación

## Propósito

Documentar las entidades, atributos, identificadores, relaciones y cardinalidades derivadas del análisis empírico del dataset OpenCEM v1.0.0 (`opencem-v1.0.0.sqlite`) para su implementación en PostgreSQL (`SolazoDB`).

## Criterios de diseño

- Cada entidad cuenta con una clave primaria explícita validada sobre los datos reales (en SQLite las tablas de telemetría carecían de `PRIMARY KEY` formal).
- Se respeta la regla de trazabilidad: no se descomponen arreglos JSON (`component-defined`) ni se asignan significados no documentados a códigos internos.
- Se preservan los valores `SQL NULL` originales sin imputación automática.
- Las marcas de tiempo Unix fraccionales en segundos (`read_ts`, `*_time`) se estandarizan a `TIMESTAMPTZ` (UTC).

## Entidades definidas

| Entidad | Tabla PostgreSQL | Descripción y responsabilidad | Clave Primaria (PK) | Estado |
|---|---|---|---|---|
| **Inversor** | `inversores` | Catálogo maestro de los inversores identificados en la microrred híbrida (IDs 1 y 2). | `inverter_id` | Validado |
| **Lectura Analógica** | `lecturas_analogicas` | Telemetría eléctrica y operativa por intervalo (batería, generación fotovoltaica PV1 y red eléctrica). | `(inverter_id, read_ts)` | Validado |
| **Estado Operativo** | `estados_operativos` | Banderas de arranque, modos de operación y alarmas del sistema por marca temporal. | `(inverter_id, read_ts)` | Validado |
| **Configuración** | `configuraciones` | Parámetros eléctricos, protecciones, programación horaria y reservas JSON (`reserve`, `battsetreserve`). | `(inverter_id, read_ts)` | Validado |
| **Registro de Fallos** | `historial_fallos` | Cabecera temporal por lectura que agrupa los payloads históricos JSON (`record_00` a `record_31`) y sus tiempos calibrados. | `(inverter_id, read_ts)` | Validado |

## Relaciones y cardinalidades

1. **`inversores` (1) — (N) `lecturas_analogicas`**
   - **Regla:** Un inversor genera múltiples lecturas analógicas a lo largo del tiempo; cada lectura pertenece obligatoriamente a un único inversor.
   - **Clave foránea:** `lecturas_analogicas.inverter_id` referencia a `inversores.inverter_id`.

2. **`inversores` (1) — (N) `estados_operativos`**
   - **Regla:** Un inversor reporta múltiples registros de estado operativo; cada registro pertenece a un único inversor.
   - **Clave foránea:** `estados_operativos.inverter_id` referencia a `inversores.inverter_id`.

3. **`inversores` (1) — (N) `configuraciones`**
   - **Regla:** Un inversor registra múltiples instantáneas de configuración; cada registro pertenece a un único inversor.
   - **Clave foránea:** `configuraciones.inverter_id` referencia a `inversores.inverter_id`.

4. **`inversores` (1) — (N) `historial_fallos`**
   - **Regla:** Un inversor almacena múltiples registros de historial de fallos; cada registro pertenece a un único inversor.
   - **Clave foránea:** `historial_fallos.inverter_id` referencia a `inversores.inverter_id`.

> **Nota sobre relación entre tablas de telemetría:** Aunque `lecturas_analogicas` y `estados_operativos` comparten la clave `(inverter_id, read_ts)`, el cruce exploratorio demostró que no existe coincidencia temporal exacta al 100 % entre ambas tablas. Por ello, ambas se relacionan directamente con la entidad padre `inversores` en lugar de forzar una clave foránea 1:1 entre ellas.

## Diagramas

Exportar y guardar los diagramas en la carpeta `diagrams/`:
- `diagrams/modelo_conceptual.png`
- `diagrams/modelo_relacional.png`
