# 03. Modelo entidad-relación

## Propósito

Documentar las entidades, atributos, identificadores, relaciones y cardinalidades que se deriven del dataset real y de las reglas de negocio.

## Criterios de diseño

- Cada entidad debe tener una identidad clara.
- Las relaciones deben representar reglas comprobables.
- Los atributos deben ser atómicos.
- Las claves primarias y foráneas deben quedar identificadas.
- Las lecturas de telemetría deben conservar su referencia temporal y su procedencia.
- No se deben crear entidades o atributos solo por conveniencia si la fuente no los respalda.

## Entidades candidatas

La lista siguiente es preliminar y debe confirmarse con la fuente:

| Entidad candidata | Responsabilidad | Estado |
|---|---|---|
| Sistema o microrred | Identificar el sistema monitoreado | Por validar |
| Dispositivo o componente | Identificar equipo medido | Por validar |
| Tipo de medición | Describir magnitud medida | Por validar |
| Lectura de telemetría | Registrar valor y marca temporal | Por validar |
| Unidad de medida | Documentar la unidad física | Por validar |

## Diagrama

Guardar el diagrama conceptual en `diagrams/modelo_conceptual.png` y el modelo relacional en `diagrams/modelo_relacional.png`.

## Relaciones y cardinalidades

Pendiente de definir a partir del análisis de fuente y reglas de negocio.
