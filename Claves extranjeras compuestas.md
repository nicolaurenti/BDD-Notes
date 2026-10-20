---
tags:
  - tp1
---

# Claves extranjeras compuestas


Cuando se referencia una clave compuesta, la FK debe incluir **todas sus columnas**, con tipos compatibles y en el orden correspondiente.

```sql
FOREIGN KEY (cod_p, idioma)
    REFERENCES palabra (cod_p, idioma)
```

Esto exige que exista **esa combinación** en `PALABRA`.

> [!warning]
> Dos FK independientes no equivalen a una FK compuesta: no garantizan que los valores pertenezcan a la misma fila referenciada.
