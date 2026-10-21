---
tags:
  - tp2
---

# Joins y conteos de filas relacionadas

## Diferencias entre joins

| Join | Filas que conserva |
|---|---|
| `INNER JOIN` | Solo las que encuentran coincidencia. |
| `LEFT JOIN` | Todas las de la izquierda y sus coincidencias. |
| `RIGHT JOIN` | Todas las de la derecha y sus coincidencias. |
| `FULL JOIN` | Todas las de ambos lados. |
| `CROSS JOIN` | Todas las combinaciones posibles. |
| `NATURAL JOIN` | Combina usando todas las columnas con el mismo nombre. |

Cuando un outer join no encuentra coincidencia, completa las columnas del lado faltante con `NULL`.

### INNER y LEFT con instituciones

INNER conserva coincidencias; LEFT conserva además las filas izquierdas sin pareja, completando el otro lado con NULL.

[[TP2 - Ejemplos básicos de INNER JOIN, LEFT JOIN y USING|Ejemplos básicos de INNER JOIN, LEFT JOIN y USING]]

### Filtros en un LEFT JOIN

ON decide qué coincidencias se forman. WHERE filtra después de la unión y puede quitar las filas extendidas con NULL que se querían conservar.

[[TP2 - LEFT JOIN con filtros en ON y WHERE|LEFT JOIN con filtros en ON y WHERE]]

### Relacionar una tabla consigo misma

Un autojoin usa alias distintos para representar roles diferentes de una misma tabla, como voluntario y coordinador.

[[TP2 - Autojoins, coordinadores y parejas de filas|Autojoins, coordinadores y parejas de filas]]

### USING y nombres comunes

USING permite indicar la columna común cuando tiene el mismo nombre. NATURAL JOIN usa todas las columnas con nombres coincidentes, por lo que debe revisarse cuidadosamente.

[[TP2 - Ejemplos básicos de INNER JOIN, LEFT JOIN y USING|Ejemplos básicos de INNER JOIN, LEFT JOIN y USING]]

### Cantidad de parejas de un autojoin

El join produce todas las parejas que satisfacen ON, incluso la pareja de una fila consigo misma si no se excluye.

[[TP2 - Autojoins, coordinadores y parejas de filas|Autojoins, coordinadores y parejas de filas]]

## Contar filas relacionadas

Para contar entidades después de unir, identificar qué representa cada fila. En LEFT JOIN se cuenta la PK del lado opcional para obtener cero cuando no hay coincidencias.

[[TP2 - Conteos de entidades relacionadas y ausentes|Conteos de entidades relacionadas y ausentes]]

## Resultado trabajado de LEFT JOIN

El desarrollo muestra las filas concretas antes y después de filtrar, y por qué COUNT(*) difiere de contar la PK opcional.

[[TP2 - LEFT JOIN con filtros en ON y WHERE|LEFT JOIN con filtros en ON y WHERE]]

## Fuente de estudio

Enunciado del TP2 en la carpeta `../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
