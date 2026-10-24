---
tags:
  - tp2
---

# TP2 - Ausencia de incumplimientos con NOT EXISTS

Caso de instituciones y horas frente al máximo de la tarea, incluyendo entidades vacías y nulos.

## Excluir entidades cuando alguna fila relacionada incumple una condición

#### Patrón general

**No existe una fila relacionada que incumpla la condición.**

Ejemplo: instituciones que no poseen voluntarios cuyas horas alcancen o superen el máximo de su tarea:

```sql
SELECT i.*
FROM institucion i
WHERE NOT EXISTS (
    SELECT 1
    FROM voluntario v
    JOIN tarea t
        ON t.id_tarea = v.id_tarea
    WHERE v.id_institucion = i.id_institucion
      AND v.horas_aportadas >= t.max_horas
);
```

> [!warning]
> Buscar una fila que cumpla la condición no garantiza que todas la cumplan. Una institución puede tener simultáneamente voluntarios que cumplen y otros que incumplen.

#### Entidades sin relacionados

`NOT EXISTS` también acepta instituciones sin voluntarios, porque no existe ningún incumplimiento.

Si se exige al menos uno, agregar:

```sql
AND EXISTS (
    SELECT 1
    FROM voluntario v
    WHERE v.id_institucion = i.id_institucion
)
```

#### Tratamiento de NULL

Una comparación como:

```sql
v.horas_aportadas >= t.max_horas
```

no resulta verdadera si alguno de los valores es `NULL`.

Si la falta de datos debe considerarse incumplimiento, hay que incluirla explícitamente en la condición que detecta las filas inválidas.

## Fuente de estudio

Enunciado del TP2 en la carpeta `../../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
