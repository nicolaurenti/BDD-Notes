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

[[TP3 - Ejemplos de conteos con CHECK de tabla y ASSERTION|Ejemplos de conteos con CHECK de tabla y ASSERTION]]

## Construir una implicación sin exigir demasiado

«Los artículos publicados después de 2020 deben ser argentinos» solo impone nacionalidad a los posteriores a esa fecha. Exigir `fecha_pub >= DATE '2021-01-01' AND nacionalidad = 'Argentina'` sería incorrecto: excluiría todos los artículos anteriores.

En `NOT A OR B`, A es «publicado después de 2020» y B es «argentino». Para un artículo de 2019, NOT A es verdadero y el control se satisface sea cual sea su nacionalidad. Para uno de 2022, NOT A es falso y se necesita B. Con nacionalidad NULL, B da UNKNOWN: el CHECK no lo rechaza salvo que se exija información con NOT NULL u otra condición explícita.

## CHECK agregado según SQL-1999

El ejemplo de CHECK de tabla permite contrastar el planteo del estándar con su falta de soporte para subconsultas en PostgreSQL.

[[TP3 - Ejemplos de conteos con CHECK de tabla y ASSERTION|Ejemplos de conteos con CHECK de tabla y ASSERTION]]

## Fuente de estudio

Enunciado del TP3 en la carpeta `../../Practicos` y filminas de cátedra `BDI_03.P1 (Restricciones-1).pdf; BDI_03.P1 (Restricciones-2).pdf` (carpeta de cursada 2023).
