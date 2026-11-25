# SolazoDB

Base de datos relacional en PostgreSQL para analizar la telemetría de una microrred híbrida **solar + batería + red eléctrica**, a partir del conjunto de datos OpenCEM.

## Enfoque del equipo: línea 4

**SolarDB: vistas, vistas materializadas y funciones de ventana para indicadores energéticos.**

El proyecto contempla analizar la fuente de datos, diseñar un modelo entidad-relación, normalizarlo hasta tercera forma normal (3FN), documentar el diccionario de datos, implementar el esquema en PostgreSQL desde DBeaver, cargar al menos un millón de lecturas y construir consultas de negocio.

## Objetivos

- Documentar el origen, estructura, cobertura y calidad del dataset OpenCEM utilizado.
- Diseñar un modelo relacional trazable a los datos reales.
- Justificar la normalización hasta 3FN.
- Implementar tablas, claves, restricciones e índices en PostgreSQL.
- Preparar y ejecutar una carga reproducible de al menos 1.000.000 de lecturas.
- Construir vistas, vistas materializadas y consultas con funciones de ventana para indicadores energéticos.
- Versionar documentación, scripts y decisiones técnicas en GitHub.

## Indicadores a estudiar

La selección y las fórmulas definitivas se establecerán después de inspeccionar las columnas, unidades, granularidad y cobertura temporal del dataset. Como candidatos se consideran:

- Generación fotovoltaica.
- Energía consumida por la carga.
- Carga y descarga de la batería.
- Energía importada desde la red y exportada hacia la red.
- Balance energético por intervalo.
- Indicadores agregados por hora, día y periodo.
- Picos, promedios móviles y variaciones temporales.

No se deben asumir nombres de columnas, unidades ni fórmulas hasta documentar la fuente real.

## Estructura del repositorio

```text
SolazoDB-/
├── README.md
├── .gitignore
├── docs/                 # Fuente, requisitos, modelo, 3FN y diccionario
├── diagrams/             # Diagramas conceptual y relacional
├── data/
│   ├── raw/               # Datos originales (no versionar archivos pesados)
│   ├── processed/         # Datos transformados
│   └── samples/           # Muestras pequeñas y anonimizadas si aplica
├── sql/                   # DDL, vistas, vistas materializadas y consultas
├── scripts/               # Descarga, transformación y carga
└── tests/                 # Validaciones de datos y consultas
```

## Orden de trabajo

1. Identificar y documentar la fuente exacta de OpenCEM.
2. Perfilar los archivos y validar columnas, tipos, unidades, fechas y calidad.
3. Diseñar el modelo conceptual y relacional.
4. Normalizar y completar el diccionario de datos.
5. Implementar el DDL y las restricciones en PostgreSQL.
6. Preparar la carga y comprobar el volumen mínimo exigido.
7. Crear y probar vistas, vistas materializadas y funciones de ventana.
8. Ejecutar pruebas, documentar resultados y preparar la sustentación.

## Entorno

- PostgreSQL.
- DBeaver para administrar la conexión y ejecutar los scripts SQL.
- Git y GitHub para control de versiones.
- Python, si resulta necesario para descargar, perfilar o transformar los datos.

## Estado inicial

Este repositorio contiene la estructura y las plantillas iniciales. El modelo físico, las fórmulas de los indicadores, la carga y los resultados se completarán después de verificar el dataset seleccionado. **La estructura inicial no significa que la base de datos ya esté implementada ni que se haya cargado el millón de lecturas.**

## Equipo

- Integrantes: SEBAS CEBALLOS JUAN LOPEZ  CAMILO LOPEZ 
- Asignatura: BASE DE DATOS  I 
- Institución: PASCUAL 
