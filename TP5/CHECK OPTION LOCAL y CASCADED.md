---
tags:
  - tp5
---

# CHECK OPTION LOCAL y CASCADED


- **LOCAL:** agrega el control de la vista actual; los chequeos que ya tengan las vistas inferiores siguen aplicándose.
- **CASCADED:** exige las condiciones de la vista y de todas las vistas subyacentes, tengan o no WCO propio.
- Sin especificar modalidad, `WITH CHECK OPTION` usa **CASCADED**.

Ejemplo del ejercicio 4:

```sql
CREATE VIEW tarea10000hs AS
SELECT * FROM tarea
WHERE max_horas > 10000
WITH LOCAL CHECK OPTION;

CREATE VIEW tarea10000rep AS
SELECT * FROM tarea10000hs
WHERE id_tarea LIKE '%REP%'
WITH LOCAL CHECK OPTION;
```

| Inserción | Resultado por WCO |
|---|---|
| MGR, máximo 20000, en tarea10000rep | Rechaza: el id no contiene REP |
| REPA, máximo 5500, en tarea10000hs | Rechaza: máximo insuficiente |
| CC_REP, máximo 9000, en tarea10000rep | Rechaza: incumple el WCO de la vista inferior |
| ROM, máximo 12000, en tarea10000hs | Pasa los filtros; confirmar las demás restricciones |

Si la vista inferior no tuviera WCO, LOCAL en la superior no agregaría por sí solo el control del filtro inferior. CASCADED sí lo haría.

## Comparar combinaciones de chequeos

La diferencia se observa cuando la vista inferior no tiene WCO propio. Si ya lo tiene, LOCAL en la superior no desactiva el control inferior.

[[TP5 - Ejemplos de diferencias entre LOCAL y CASCADED|Ejemplos de diferencias entre LOCAL y CASCADED]]

## Fuente de estudio

Enunciado del TP5 en la carpeta `../../Practicos` y filminas de cátedra `BDI_05 (Vistas).pdf` (carpeta de cursada 2023).
