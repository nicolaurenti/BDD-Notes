---
tags:
  - tp1
---

# Pasaje de MERE al modelo relacional


Una **entidad** se transforma en una tabla y sus atributos se convierten en columnas. Su identificador se representa mediante una **clave primaria**.

- **Clave primaria simple:** formada por una columna.
- **Clave primaria compuesta:** formada por varias columnas. La combinación completa identifica cada fila; cada columna por separado puede repetirse.
- **Clave alternativa:** identificador posible que no se eligió como clave primaria. Se representa mediante `UNIQUE` y `NOT NULL`.

**Ejemplo:** una palabra se identifica por `(cod_p, idioma)`, porque el código por sí solo no alcanza para identificarla.
