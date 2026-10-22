---
tags:
  - tp4
---

# TP4 - Resolución 1 - Pruebas de artículos

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
