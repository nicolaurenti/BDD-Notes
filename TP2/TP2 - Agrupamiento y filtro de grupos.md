---
tags:
  - tp2
---

# TP2 - Agrupamiento y filtro de grupos

Consultas y criterios para elegir las columnas de agrupamiento.

## GROUP BY y HAVING

`GROUP BY` reúne filas que tienen los mismos valores en las columnas indicadas.

```sql
SELECT id_institucion, COUNT(*) AS cantidad
FROM voluntario
GROUP BY id_institucion;
```

Como regla general, cada expresión seleccionada debe estar agrupada o calculada mediante una agregación.

#### WHERE y HAVING

- `WHERE`: filtra filas antes de agrupar.
- `HAVING`: filtra grupos después de calcular las agregaciones.

Instituciones con más de diez voluntarios que aportaron más de 1000 horas:

```sql
SELECT id_institucion, COUNT(*) AS cantidad
FROM voluntario
WHERE horas_aportadas > 1000
GROUP BY id_institucion
HAVING COUNT(*) > 10;
```

#### Agrupar por varias columnas

Cantidad de voluntarios por institución y coordinador:

```sql
SELECT
    id_institucion,
    id_coordinador,
    COUNT(*) AS cantidad
FROM voluntario
WHERE id_coordinador IS NOT NULL
GROUP BY id_institucion, id_coordinador
HAVING COUNT(*) < 3;
```

#### Agrupar por identificadores

Agrupar solamente por nombre puede mezclar entidades diferentes que se llaman igual.

Conviene agrupar por la clave y, cuando corresponda, por las columnas descriptivas seleccionadas.

## Fuente de estudio

Enunciado del TP2 en la carpeta `../../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
