---
tags:
  - tp2
---

# TP2 - COALESCE antes y después del promedio

Compara los resultados sobre cuatro nulos, sobre 10 y NULL, y sobre una entrada vacía.

## COALESCE: reemplazar el resultado o los datos de entrada

Para mostrar cero cuando el promedio es nulo (ejercicio 2.1.b):

```sql
SELECT COALESCE(AVG(puntos), 0) AS promedio,
       COUNT(puntos) AS cantidad_puntos,
       COUNT(*) AS cantidad_filas
FROM equipo;
```

Con cuatro filas cuyos puntos son nulos devuelve `0 | 0 | 4`. Los conteos ya devuelven cero cuando no hay valores para contar; no necesitan `COALESCE`.

Estas expresiones tienen significados diferentes:

```sql
COALESCE(AVG(puntos), 0) -- Sustituye el resultado si el promedio es NULL.
AVG(COALESCE(puntos, 0)) -- Convierte cada punto NULL en cero antes de promediar.
```

Con puntos `10, NULL`, la primera devuelve **10** y la segunda **5**. Sobre una entrada sin filas, la segunda sigue devolviendo `NULL`, porque no hay filas sobre las que aplicar `COALESCE`.

Si se necesita un blanco en la salida, hay que convertir el promedio a texto:

```sql
SELECT COALESCE(CAST(AVG(puntos) AS text), '') AS promedio
FROM equipo;
```

El bloque de NULL del TP2 desarrolla la lógica de tres valores.

## Fuente de estudio

Enunciado del TP2 en la carpeta `../../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
