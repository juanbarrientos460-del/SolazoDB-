# 06. Diseño de indicadores energéticos — Línea 4

## Enfoque

Construir vistas, vistas materializadas y consultas con funciones de ventana en PostgreSQL. Los indicadores definitivos dependen de las variables, unidades y granularidad presentes en el dataset.

## Catálogo preliminar

| Indicador candidato | Nivel temporal | Definición / fórmula | Campos fuente | Estado |
|---|---|---|---|---|
| Generación solar | Intervalo / hora / día | Por determinar según unidad y granularidad | Por validar | Pendiente |
| Consumo de carga | Intervalo / hora / día | Por determinar | Por validar | Pendiente |
| Carga de batería | Intervalo | Por determinar | Por validar | Pendiente |
| Descarga de batería | Intervalo | Por determinar | Por validar | Pendiente |
| Importación de red | Intervalo / periodo | Por determinar | Por validar | Pendiente |
| Exportación a red | Intervalo / periodo | Por determinar | Por validar | Pendiente |
| Balance energético | Intervalo / periodo | Por determinar | Por validar | Pendiente |

## Vistas

Definir vistas lógicas para simplificar la lectura de las mediciones y exponer indicadores reutilizables.

## Vistas materializadas

Seleccionar agregaciones útiles por hora, día u otro periodo que esté respaldado por la granularidad de los datos. Documentar:
- Consulta de definición.
- Frecuencia y procedimiento de actualización.
- Índices aplicables.
- Comparación de tiempo de consulta antes y después, cuando sea posible.

## Funciones de ventana

Casos de uso a evaluar:
- `LAG` y `LEAD` para comparar lecturas consecutivas.
- `SUM() OVER` para acumulados por periodo.
- `AVG() OVER` para promedios móviles.
- `ROW_NUMBER` o funciones de ranking para máximos y clasificación.
- `PARTITION BY` para separar por dispositivo, tipo de medición o periodo, si el modelo lo requiere.

## Validaciones

Las fórmulas deben respetar unidades, intervalos, signos y reglas físicas documentadas. No sumar potencias como si fueran energía: cualquier integración temporal debe considerar la duración real del intervalo y la unidad correspondiente.
