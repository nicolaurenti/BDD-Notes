---
tags:
  - tp1
---

# TP1 - Ejemplo de creación de Artículo, Palabra y Contiene

El ejemplo reúne el esquema de artículos y palabras y muestra el orden de creación y las dos formas de declarar referencias.

## Creación de tablas

Ejemplo de artículos y palabras, conservando las obligaciones de título, autor, fecha y descripción y la unicidad de título que muestran las filminas:

```sql
CREATE TABLE articulo (
    id_articulo NUMERIC(4,0),
    titulo      VARCHAR(120) NOT NULL UNIQUE,
    autor       VARCHAR(30) NOT NULL,
    fecha_pub   DATE NOT NULL,

    CONSTRAINT pk_articulo
        PRIMARY KEY (id_articulo)
);

CREATE TABLE palabra (
    cod_p       NUMERIC(4,0),
    idioma      CHAR(2),
    descripcion VARCHAR(25) NOT NULL,

    CONSTRAINT pk_palabra
        PRIMARY KEY (cod_p, idioma)
);

CREATE TABLE contiene (
    id_articulo NUMERIC(4,0),
    cod_p       NUMERIC(4,0),
    idioma      CHAR(2),

    CONSTRAINT pk_contiene
        PRIMARY KEY (id_articulo, cod_p, idioma),

    CONSTRAINT fk_contiene_articulo
        FOREIGN KEY (id_articulo)
        REFERENCES articulo (id_articulo),

    CONSTRAINT fk_contiene_palabra
        FOREIGN KEY (cod_p, idioma)
        REFERENCES palabra (cod_p, idioma)
);
```

### Orden de creación

1. Crear las tablas referenciadas.
2. Crear las tablas que contienen las FK.

Si existen referencias circulares, se pueden crear primero las tablas y agregar alguna FK posteriormente con `ALTER TABLE`.

## Modificación de la estructura con ALTER TABLE

Permite modificar una tabla existente, por ejemplo agregando una restricción:

```sql
ALTER TABLE contiene
ADD CONSTRAINT fk_contiene_articulo
FOREIGN KEY (id_articulo)
REFERENCES articulo (id_articulo);
```

> [!warning]
> Este ejemplo es una alternativa a declarar esa FK dentro de `CREATE TABLE`. No corresponde agregarla nuevamente si ya existe.

## Fuente de estudio

Enunciado del TP1 en la carpeta `../Practicos` y filminas de cátedra `BDI_01 (Intro - creacion esquemas).pdf` (carpeta de cursada 2023).
