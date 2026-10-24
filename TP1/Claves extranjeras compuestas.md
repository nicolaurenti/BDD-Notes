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

## Por qué debe coincidir la combinación

Supongamos que PALABRA contiene solo `(1, 'ES')` y `(2, 'EN')`. Una fila de CONTIENE con `(1, 'EN')` debe rechazarse: el código 1 existe y el idioma EN existe, pero no existe esa palabra. La FK compuesta comprueba ambos valores en **una misma fila padre**.

Las columnas se corresponden por posición: la primera de FOREIGN KEY con la primera de REFERENCES. Los nombres podrían diferir; deben conservar el significado y tener tipos compatibles.

```sql
CREATE TABLE contiene (
    id_articulo NUMERIC(4,0),
    cod_p NUMERIC(4,0),
    idioma CHAR(2),
    PRIMARY KEY (id_articulo, cod_p, idioma),
    FOREIGN KEY (id_articulo) REFERENCES articulo (id_articulo),
    FOREIGN KEY (cod_p, idioma) REFERENCES palabra (cod_p, idioma)
);
```

La PK de CONTIENE tiene tres columnas; la FK hacia PALABRA tiene dos. No hay obligación de que una FK coincida con toda la PK de la hija: debe coincidir con toda la clave que identifica al padre referenciado. Para probar el diseño, cargar una asociación válida y otra con valores existentes por separado pero sin la combinación; solo la primera debe ser admisible.

## Fuente de estudio

Enunciado del TP1 en la carpeta `../../Practicos` y filminas de cátedra `BDI_01 (Intro - creacion esquemas).pdf` (carpeta de cursada 2023).
