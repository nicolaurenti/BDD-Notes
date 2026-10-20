---
tags:
  - tp2
---

# Valores NULL y lógica de tres valores

`NULL` representa un valor desconocido o ausente.

Las condiciones pueden resultar:

- Verdaderas.
- Falsas.
- Desconocidas: `UNKNOWN`.

`WHERE` y `HAVING` conservan únicamente los resultados **verdaderos**.

```sql
-- Correcto.
WHERE telefono IS NULL

-- Incorrecto para buscar nulos.
WHERE telefono = NULL
```

### Negar una condición no incluye automáticamente los nulos

```sql
WHERE NOT (porcentaje BETWEEN 0.15 AND 0.30)
```

Esta condición tampoco selecciona filas cuyo porcentaje sea `NULL`.

Para incluirlas:

```sql
WHERE porcentaje NOT BETWEEN 0.15 AND 0.30
   OR porcentaje IS NULL
```

### Reemplazar un NULL con COALESCE

Devuelve el primer argumento no nulo:

```sql
SELECT COALESCE(telefono, 'Sin teléfono')
FROM voluntario;
```

### Comparaciones y negación

`NULL` no equivale a `0` ni a una cadena vacía (`''`). Incluso comparar dos nulos devuelve desconocido:

| Expresión | Resultado |
|---|---|
| `NULL = NULL` | `UNKNOWN` |
| `NULL <> NULL` | `UNKNOWN` |
| `NOT UNKNOWN` | `UNKNOWN` |
| `NULL IS NULL` | `TRUE` |
| `NULL IS NOT NULL` | `FALSE` |

Una condición desconocida no es falsa, pero tampoco permite que la fila pase un `WHERE`. En un `INNER JOIN`, `ON` también exige `TRUE` para formar una pareja.

### Ejercicio 2.1: cuatro equipos con puntos y descripción nulos

Datos de partida: cuatro equipos con identificadores 1, 2, 3 y 4; tanto `puntos` como `descripcion` son `NULL`.

| Consulta o expresión | Resultado | Motivo |
|---|---|---|
| `AVG(puntos), COUNT(puntos), COUNT(*)` | `NULL, 0, 4` | No hay puntos no nulos, pero hay cuatro filas. |
| `SELECT DISTINCT puntos FROM equipo` | Una fila con `NULL` | `DISTINCT` reúne los nulos en un único resultado. |
| `'Su descripción es ' || descripcion` | `NULL` en cada fila | La concatenación con `||` propaga el nulo. |
| `WHERE puntos NOT IN (SELECT DISTINCT puntos FROM equipo)` | Ninguna fila | `NULL NOT IN (NULL)` es `UNKNOWN`. |
| Autojoin con `ON e1.puntos = e2.puntos` | Ninguna fila | Todas las comparaciones son `NULL = NULL`, que da `UNKNOWN`. |

> [!important]
> La consulta 2 de la imagen también contiene el filtro `NOT IN`. Por eso la consulta completa no devuelve filas. Solo si se quita ese filtro aparecen los cuatro identificadores con la descripción concatenada en `NULL`.

### Puntos para repasar

- [ ] Distinguir valor nulo, cero y texto vacío.
- [ ] Evaluar comparaciones y negaciones con `UNKNOWN`.
- [ ] Reconocer que `WHERE`, `HAVING` y las coincidencias de `ON` requieren `TRUE`.
- [ ] Distinguir `COUNT(*)` de `COUNT(columna)` y explicar un promedio sin valores.
- [ ] Explicar por qué `DISTINCT` reúne nulos sin que `NULL = NULL` sea verdadero.
- [ ] Detectar el efecto de los nulos en `NOT IN` y en la concatenación.
- [ ] Diferenciar reemplazar el resultado de una agregación de reemplazar sus datos de entrada.
- [ ] Contar todas las parejas de un join y la fila sin coincidencia de un `LEFT JOIN`.

Ver Funciones de agregación, DISTINCT, Subconsultas, Manejo de strings, Diferencias entre joins y Conteos con joins.
