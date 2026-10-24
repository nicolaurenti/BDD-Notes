---
tags:
  - tp2
---

# TP2 - LEFT JOIN con filtros en ON y WHERE

Compará las dos consultas y seguí sus resultados con instituciones 20 y 50.

## Diferencia entre ON y WHERE

Para conservar todas las instituciones y mostrar solamente sus voluntarios con más de 1000 horas:

```sql
SELECT i.id_institucion, v.nro_voluntario
FROM institucion i
LEFT JOIN voluntario v
    ON v.id_institucion = i.id_institucion
   AND v.horas_aportadas > 1000;
```

Si esa condición se coloca en `WHERE`, desaparecen las filas sin una coincidencia que la cumpla:

```sql
SELECT i.id_institucion, v.nro_voluntario
FROM institucion i
LEFT JOIN voluntario v
    ON v.id_institucion = i.id_institucion
WHERE v.horas_aportadas > 1000;
```

## Una tabla pequeña para seguir el LEFT JOIN

Sean instituciones 20 y 50, y dos voluntarios de 20: `(1, 500 horas)` y `(2, 2000 horas)`. El LEFT JOIN sin otros filtros produce:

| Institución | Voluntario | Horas |
|---|---|---:|
| 20 | 1 | 500 |
| 20 | 2 | 2000 |
| 50 | NULL | NULL |

Si `horas > 1000` está en ON, las coincidencias se buscan con ese requisito: salen `(20, 2, 2000)` y `(50, NULL, NULL)`. Si está en WHERE, se filtra después de unir: de la tabla anterior solo queda `(20, 2, 2000)`, porque NULL > 1000 no es TRUE.

Al agrupar el join original por la clave de la institución, COUNT(*) produce 2 para 20 y 1 para 50. COUNT de la PK del voluntario produce 2 y 0. La diferencia proviene de la fila que LEFT JOIN conserva para representar la institución sin coincidencias.

En las uniones por departamento de Películas, usar `(id_distribuidor, id_departamento)`. Igualar solo id_departamento puede unir departamentos diferentes que comparten número y multiplicar filas sin que corresponda al modelo.

## Fuente de estudio

Enunciado del TP2 en la carpeta `../../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
