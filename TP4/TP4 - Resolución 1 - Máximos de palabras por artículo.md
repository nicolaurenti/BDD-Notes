---
tags:
  - tp4
---

# TP4 - Resolución 1 - Máximos de palabras por artículo

Se implementan conjuntamente 3.d y 3.e. La interpretación de 3.e sigue la consulta de las filminas: **si el artículo es argentino, puede tener hasta diez palabras**. No exige que todos los artículos sean argentinos; eso anularía las demás nacionalidades admitidas por 3.a.

Se cuentan asociaciones de CONTIENE. Su clave evita repetir la misma palabra e idioma en un artículo. El diagrama del TP3 usa `cod_palabra`; los controles no dependen del nombre de esa columna porque cuentan filas.

```sql
-- TP4, ejercicio 1: TP3 3.d y 3.e.
-- Tablas del diagrama: articulo, contiene(id_articulo, idioma, cod_palabra, nro_seccion).
-- Ejecutar una sola vez en un esquema de práctica con las tablas base creadas.
CREATE OR REPLACE FUNCTION tp4_controlar_palabras()
RETURNS trigger AS $$
BEGIN
    IF EXISTS (
        SELECT id_articulo
        FROM contiene
        GROUP BY id_articulo
        HAVING COUNT(*) > 15
    ) THEN
        RAISE EXCEPTION 'RI_3D: un articulo supera las 15 palabras';
    END IF;

    IF EXISTS (
        SELECT a.id_articulo
        FROM articulo a JOIN contiene c
          ON c.id_articulo = a.id_articulo
        WHERE a.nacionalidad = 'Argentina'
        GROUP BY a.id_articulo
        HAVING COUNT(*) > 10
    ) THEN
        RAISE EXCEPTION 'RI_3E: un articulo argentino supera las 10 palabras';
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE 'plpgsql';

CREATE TRIGGER tp4_palabras_contiene
AFTER INSERT OR UPDATE OF id_articulo ON contiene
FOR EACH STATEMENT
EXECUTE PROCEDURE tp4_controlar_palabras();

CREATE TRIGGER tp4_palabras_articulo
AFTER UPDATE OF nacionalidad ON articulo
FOR EACH STATEMENT
EXECUTE PROCEDURE tp4_controlar_palabras();
```

Ambos controles observan el estado posterior: con quince palabras no argentinas, agregar la número dieciséis se rechaza; con diez argentinas, agregar la número once también. Cambiar nacionalidad a Argentina puede fallar aunque no se modifique CONTIENE.

## Pruebas válidas, inválidas y en el límite

Los casos parten de una instancia válida con estos artículos, todos publicados en 2019: A1 español con quince palabras, A2 argentino con diez, A3 español con once y A4 español sin palabras. Existen las palabras ES/1…ES/50. A1 contiene 1…15, A2 contiene 16…25 y A3 contiene 26…36.

Usar los identificadores 1, 2, 3 y 4. Cada caso parte otra vez del estado inicial. En DataGrip, iniciar una transacción, ejecutar el caso y terminar con ROLLBACK; si falla la sentencia, ejecutar ROLLBACK antes del siguiente caso.

```sql
-- CONTIENE: palabra 16 de un artículo no argentino. Rechaza RI_3D.
INSERT INTO contiene VALUES (1, 'ES', 37, 1);

-- CONTIENE: palabra 11 de un artículo argentino. Rechaza RI_3E.
INSERT INTO contiene VALUES (2, 'ES', 37, 1);

-- CONTIENE: mover una palabra a un artículo argentino lleno. Rechaza RI_3E.
UPDATE contiene SET id_articulo = 2
WHERE id_articulo = 3 AND cod_palabra = 26;

-- CONTIENE: sustitución sin superar quince. Se acepta la pareja de sentencias.
DELETE FROM contiene
WHERE id_articulo = 1 AND cod_palabra = 1;
INSERT INTO contiene VALUES (1, 'ES', 37, 1);
```

Pruebas del trigger de ARTICULO:

```sql
-- Once palabras al pasar a argentino: rechaza RI_3E.
UPDATE articulo SET nacionalidad = 'Argentina' WHERE id_articulo = 3;

-- Quince palabras al pasar a argentino: rechaza RI_3E.
UPDATE articulo SET nacionalidad = 'Argentina' WHERE id_articulo = 1;

-- Cero palabras al pasar a argentino: se acepta.
UPDATE articulo SET nacionalidad = 'Argentina' WHERE id_articulo = 4;
```

Los tres CHECK de nacionalidad, fecha mínima y condición posterior a 2020 se mantuvieron instalados en la prueba final. Para no infringirlos al preparar los casos, los artículos no argentinos son de 2019.

El INSERT sin lista de columnas corresponde al orden del fixture: `(id_articulo, idioma, cod_palabra, nro_seccion)`. En tus tablas conviene explicitar esas columnas; si se llamó `cod_p` al código en TP1, adaptar el nombre de las pruebas, no los controles.

Fuente: `Practico 4 - Acciones Procedurales.docx`, TP3 y esquema correspondiente de la cátedra. Se usa `EXECUTE PROCEDURE`.
