---
tags:
  - tp3
---

# CHECK y ASSERTION

## CHECK y lógica de tres valores

```sql
ALTER TABLE articulo
ADD CONSTRAINT ck_nacionalidad
CHECK (nacionalidad IN ('Argentina', 'Español', 'Inglesa', 'Alemana', 'Chilena'));

ALTER TABLE articulo
ADD CONSTRAINT ck_fecha_publicacion
CHECK (fecha_pub >= DATE '2010-01-01');

ALTER TABLE producto
ADD CONSTRAINT ck_descripcion_presentacion
CHECK (descripcion IS NOT NULL OR presentacion IS NOT NULL);
```

Un `CHECK` se incumple cuando da **FALSE**. Si da `UNKNOWN` por un nulo, no lo rechaza: agregar `NOT NULL` cuando la información sea obligatoria. Diferencia importante con `WHERE`, que solo conserva `TRUE`.

Una implicación «si A, entonces B» se expresa como `NOT A OR B`:

```sql
-- Después de 2020: desde 2021 inclusive.
CHECK (fecha_pub < DATE '2021-01-01' OR nacionalidad = 'Argentina')
```

Si la nacionalidad debe estar informada siempre, agregar `NOT NULL`. Si solo es obligatoria para artículos posteriores a 2020, la segunda rama debe incluir `nacionalidad IS NOT NULL`.

Para un prefijo literal `S_`:

```sql
CHECK (cod_suc LIKE 'S!_%' ESCAPE '!')
```

El `_` sin escape significa «cualquier carácter»: `S_%` también admitiría SA123. El escape obliga a que el segundo carácter sea el guion bajo literal.

## Restricciones con conteos y ASSERTION

Para prohibir infracciones se escribe una consulta que las detecte y se exige NOT EXISTS. El alcance depende de las filas consultadas, no solo del nombre CHECK o ASSERTION.

### Ejemplos de conteos con CHECK de tabla y ASSERTION

Planteos declarativos SQL-1999: máximo de palabras y horas dentro de los límites de tarea. Se indica lo que PostgreSQL no admite.

Patrón: **no debe existir un caso infractor**.

```sql
-- SQL estándar del ejercicio; PostgreSQL no admite CREATE ASSERTION.
CREATE ASSERTION max_15_palabras
CHECK (NOT EXISTS (
    SELECT id_articulo
    FROM contiene
    GROUP BY id_articulo
    HAVING COUNT(*) > 15
));
```

Se supone una fila de `contiene` por palabra clave asociada. Si el modelo permite repeticiones, definir qué se cuenta y usar la clave completa correspondiente.

Para las horas dentro de la tarea, el caso infractor es:

```sql
SELECT v.nro_voluntario
FROM voluntario v
JOIN tarea t ON t.id_tarea = v.id_tarea
WHERE v.horas_aportadas < t.min_horas
   OR v.horas_aportadas > t.max_horas;
```

La aserción exige que esa consulta esté vacía. Si los nulos están prohibidos, controlarlos además: las comparaciones ordinarias no detectan un nulo como infracción.

En el apartado PostgreSQL, no escribir subconsultas dentro de CHECK ni CREATE ASSERTION como si fueran recursos disponibles. Si FK o UNIQUE expresan la regla, usarlos; de lo contrario, indicar que no se puede implementar con los recursos declarativos vistos en este práctico.

#### CHECK de tabla según SQL-1999

La cátedra permite plantear una subconsulta en un CHECK de tabla cuando el control necesita varias filas de esa tabla. Por ejemplo, para el límite de asociaciones:

```sql
-- Planteo declarativo SQL-1999; no es ejecutable en PostgreSQL.
ALTER TABLE contiene
ADD CONSTRAINT ck_max_palabras
CHECK (NOT EXISTS (
    SELECT id_articulo
    FROM contiene
    GROUP BY id_articulo
    HAVING COUNT(*) > 15
));
```

La consulta devuelve artículos infractores. Con 15 asociaciones no devuelve ese artículo; con 16, sí. NOT EXISTS invierte esa existencia para expresar la condición válida. La misma condición puede escribirse como ASSERTION, pero para clasificarla sigue siendo de tabla porque solo consulta CONTIENE.

Para «solo se pueden publicar artículos argentinos con hasta diez palabras», separar las posibles lecturas: exigir nacionalidad argentina a todos los artículos y, además, el máximo diez no es lo mismo que imponer el máximo únicamente cuando el artículo es argentino. Declarar la interpretación elegida antes de construir la negación de los infractores.

## Construir una implicación sin exigir demasiado

«Los artículos publicados después de 2020 deben ser argentinos» solo impone nacionalidad a los posteriores a esa fecha. Exigir `fecha_pub >= DATE '2021-01-01' AND nacionalidad = 'Argentina'` sería incorrecto: excluiría todos los artículos anteriores.

En `NOT A OR B`, A es «publicado después de 2020» y B es «argentino». Para un artículo de 2019, NOT A es verdadero y el control se satisface sea cual sea su nacionalidad. Para uno de 2022, NOT A es falso y se necesita B. Con nacionalidad NULL, B da UNKNOWN: el CHECK no lo rechaza salvo que se exija información con NOT NULL u otra condición explícita.

## CHECK agregado según SQL-1999

El ejemplo de CHECK de tabla permite contrastar el planteo del estándar con su falta de soporte para subconsultas en PostgreSQL.

## Fuente de estudio

Enunciado del TP3 en la carpeta `../../Practicos` y filminas de cátedra `BDI_03.P1 (Restricciones-1).pdf; BDI_03.P1 (Restricciones-2).pdf` (carpeta de cursada 2023).
