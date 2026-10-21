---
tags:
  - tp2
---

# Agregaciones, agrupamiento y porcentajes

## Funciones de agregación

| Función | Resultado |
|---|---|
| `COUNT(*)` | Cantidad de filas. |
| `COUNT(columna)` | Cantidad de valores no nulos de la columna. |
| `COUNT(DISTINCT columna)` | Cantidad de valores diferentes y no nulos. |
| `SUM(columna)` | Suma de valores no nulos. |
| `AVG(columna)` | Promedio de valores no nulos. |
| `MIN(columna)` | Menor valor no nulo. |
| `MAX(columna)` | Mayor valor no nulo. |
| `STDDEV(columna)` | Desviación estándar muestral de los valores no nulos. |

#### Diferencias cuando todos los valores son NULL

Si existen cuatro filas y todas tienen `puntos = NULL`:

| Expresión | Resultado |
|---|---|
| `COUNT(*)` | `4` |
| `COUNT(puntos)` | `0` |
| `AVG(puntos)` | `NULL` |
| `SUM(puntos)` | `NULL` |

Sin filas de entrada y sin `GROUP BY`, `COUNT` devuelve `0`; las demás agregaciones de esta tabla devuelven `NULL`.

## Promedios y reemplazo de NULL

Reemplazar el resultado nulo de AVG no equivale a convertir cada entrada nula en cero: cambia el denominador del promedio.

[[TP2 - COALESCE antes y después del promedio|COALESCE antes y después del promedio]]

## GROUP BY y HAVING

GROUP BY define qué filas se resumen juntas. WHERE decide cuáles entran y HAVING cuáles grupos permanecen. Agrupar por institución no equivale a agrupar por institución y coordinador.

[[TP2 - Agrupamiento y filtro de grupos|Agrupamiento y filtro de grupos]]

## Cálculos sobre subconjuntos y totales

Un promedio mide valores; un porcentaje relaciona cantidades. Elegir el subconjunto del numerador y el total del denominador es parte de traducir la consigna.

[[TP2 - Promedios, porcentajes y comparación con totales|Promedios, porcentajes y comparación con totales]]

## Contar lo que pide el enunciado

Si un voluntario tiene tres registros históricos con tareas A, A y B, `COUNT(*)` da 3 y `COUNT(DISTINCT id_tarea)` da 2. La primera cantidad representa registros o períodos; la segunda, tareas diferentes. Ninguna demuestra por sí sola cuántas veces cambió de tarea: para hablar de cambios debe estar definido cómo cada registro representa un período o una transición.

En una institución con voluntarios `(coordinador 10, persona 1)`, `(10, 2)` y `(20, 3)`, agrupar solo por institución produce cantidad 3. Agrupar por institución y coordinador produce los grupos de tamaños 2 y 1. Para «coordinadores con menos de 3 voluntarios dentro de cada institución», la clave del grupo debe contener ambas columnas.

## Fuente de estudio

Enunciado del TP2 en la carpeta `../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
