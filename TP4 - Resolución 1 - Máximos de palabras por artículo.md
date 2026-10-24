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

[[TP4 - Resolución 1 - Pruebas de artículos|Pruebas válidas, inválidas y en el límite]].

Fuente: `Practico 4 - Acciones Procedurales.docx`, TP3 y esquema correspondiente de la cátedra. Se usa `EXECUTE PROCEDURE`.
