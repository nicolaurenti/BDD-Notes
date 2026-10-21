---
tags:
  - tp2
---

# Subconsultas y condiciones de ausencia

## Subconsultas

Una subconsulta es una consulta incluida dentro de otra.

#### IN

Comprueba si un valor pertenece al resultado:

```sql
SELECT *
FROM voluntario
WHERE nro_voluntario IN (
    SELECT nro_voluntario
    FROM historico
);
```

#### EXISTS

Comprueba si la subconsulta devuelve al menos una fila:

```sql
SELECT *
FROM institucion i
WHERE EXISTS (
    SELECT 1
    FROM voluntario v
    WHERE v.id_institucion = i.id_institucion
);
```

#### NOT EXISTS

Comprueba que no exista ninguna fila:

```sql
SELECT *
FROM institucion i
WHERE NOT EXISTS (
    SELECT 1
    FROM voluntario v
    WHERE v.id_institucion = i.id_institucion
);
```

#### Subconsulta correlacionada

Hace referencia a una fila de la consulta exterior.

En los ejemplos anteriores, la condición:

```sql
v.id_institucion = i.id_institucion
```

relaciona la subconsulta con la institución que se está evaluando.

#### Riesgo de NOT IN con NULL

```sql
WHERE nro_voluntario NOT IN (
    SELECT id_coordinador
    FROM voluntario
)
```

Si la subconsulta devuelve algún `NULL`, puede impedir que se seleccionen filas que se esperaban obtener.

Alternativas:

- Excluir los nulos de la subconsulta.
- Expresar la ausencia mediante `NOT EXISTS`.

#### Ejemplo de NOT IN con nulos

Si `SELECT DISTINCT puntos FROM equipo` devuelve una fila con `NULL`, para una fila exterior con puntos nulos:

```sql
NULL IN (NULL)     -- UNKNOWN
NULL NOT IN (NULL) -- UNKNOWN
```

`NOT` no transforma lo desconocido en verdadero. Por eso el `WHERE` no conserva esa fila. Tampoco se trata de que dos valores nulos sean iguales.

Para un valor no nulo, `5 NOT IN (5, NULL)` es `FALSE`, mientras que `7 NOT IN (5, NULL)` es `UNKNOWN`: el nulo impide confirmar que no hay coincidencia.

Filtrar nulos de la subconsulta o usar `NOT EXISTS` requiere decidir también qué se pretende hacer con los nulos de la consulta exterior. No son reemplazos equivalentes automáticamente en presencia de nulos.

El bloque de NULL del TP2 desarrolla la lógica de tres valores.

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

## Evaluar una correlación sin memorizar la plantilla

Supongamos instituciones 20, 50 y 80. Solo hay voluntarios en 20 y 50. Al evaluar el NOT EXISTS de la nota, para la institución 20 la subconsulta encuentra una fila y NOT EXISTS da FALSE. Para la 80 no encuentra ninguna y da TRUE. El `SELECT 1` no cuenta voluntarios ni devuelve el identificador de la institución: indica que el contenido de la fila encontrada no interesa.

Si se omite `v.id_institucion = i.id_institucion`, la subconsulta deja de preguntar por cada institución y pregunta si hay algún voluntario en toda la tabla. Como lo hay, NOT EXISTS sería falso para **todas** las instituciones, incluso para la 80.

## “Alguno”, “ninguno” y “solo”

«Tiene alguno que cumple P» se expresa con EXISTS sobre P. «No tiene ninguno que cumple P» se expresa con NOT EXISTS sobre P. «Solo tiene voluntarios que cumplen P» exige que no exista un voluntario que incumpla P; si además debe tener voluntarios, se agrega EXISTS. Esa última condición evita aceptar por ausencia de casos a una institución vacía.

Por ejemplo, si una institución tiene una persona con 500 horas y otra con 2000, encontrar a la primera no demuestra «todos aportan menos de 1000». Para demostrarlo hay que buscar a quien alcance o supere 1000 y exigir que no exista. Si hay NULL, decidir si falta de información cuenta como incumplimiento y agregar el caso al predicado.

## Fuente de estudio

Enunciado del TP2 en la carpeta `../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
