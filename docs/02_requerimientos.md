# 02. Requerimientos del proyecto

## Requerimientos funcionales

1. Identificar y documentar la fuente OpenCEM utilizada.
2. Representar los datos mediante un modelo entidad-relación.
3. Transformar el diseño hasta 3FN y justificar las decisiones.
4. Implementar el esquema relacional en PostgreSQL.
5. Definir claves primarias, foráneas, restricciones e índices.
6. Cargar al menos 1.000.000 de lecturas válidas, de acuerdo con el alcance del taller.
7. Crear vistas para consultar indicadores energéticos.
8. Crear vistas materializadas para resultados agregados que justifiquen su persistencia.
9. Utilizar funciones de ventana para análisis temporal y comparaciones.
10. Documentar y probar consultas de negocio desde DBeaver.

## Requerimientos no funcionales

- Integridad referencial y consistencia de datos.
- Reproducibilidad de la carga.
- Consultas comprensibles y documentadas.
- Trazabilidad de las transformaciones.
- Control de versiones mediante GitHub.
- Separación entre datos originales, procesados, scripts y documentación.

## Restricciones y decisiones pendientes

- El diseño final depende de la estructura real del dataset seleccionado.
- Los nombres de campos, unidades, frecuencia temporal y reglas de negocio deben comprobarse.
- Los archivos originales grandes no se deben subir al repositorio sin revisar tamaño y licencia.
- El volumen de un millón de lecturas debe demostrarse mediante una consulta de conteo y evidencia de ejecución.
