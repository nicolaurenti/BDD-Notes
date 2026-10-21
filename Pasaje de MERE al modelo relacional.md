---
tags:
  - tp1
---

# Pasaje de MERE al modelo relacional

Derivar un esquema consiste en representar con tablas la información y las asociaciones del MERE. No alcanza con copiar nombres: hay que conservar qué identifica a cada objeto, con quién puede relacionarse y qué datos son obligatorios. En el TP1, esto termina en columnas, tipos, PK, claves alternativas y FK.

## De entidad a tabla

Una entidad se representa mediante una tabla; sus atributos, mediante columnas; y su identificador elegido, mediante la clave primaria. Una fila representa una instancia, no un atributo.

Por ejemplo, en `PALABRA(cod_p, idioma, descripcion)`, el identificador puede ser `(cod_p, idioma)`. Las filas `(1, 'ES', 'Datos')` y `(1, 'EN', 'Data')` son distintas aunque compartan el código. La fila `(1, 'ES', 'Tablas')` no puede agregarse si ya existe la primera: repetiría la combinación identificadora.

```sql
CREATE TABLE palabra (
    cod_p NUMERIC(4,0),
    idioma CHAR(2),
    descripcion VARCHAR(25),
    CONSTRAINT pk_palabra PRIMARY KEY (cod_p, idioma)
);
```

La PK compuesta no se traduce como dos PK independientes. Una tabla tiene una sola clave primaria, que puede contener varias columnas.

## Claves candidatas y alternativas

Una clave candidata es una combinación mínima capaz de identificar una instancia. Se elige una como primaria; las restantes son alternativas. Si una persona se identifica tanto por `id_persona` como por `documento`, se puede elegir el id como PK y declarar el documento `UNIQUE NOT NULL`.

La minimalidad importa: si `id_persona` ya identifica, `(id_persona, nombre)` no es una clave candidata mínima. Que una combinación no se repita en los cinco datos de prueba tampoco prueba que sea clave: la regla debe valer para todas las instancias permitidas por el modelo.

## Cómo revisar la derivación

Primero listar las entidades y sus identificadores completos. Después derivar cada relación y ubicar sus atributos. Por último, comprobar que las FK apunten a claves completas y que la nulidad respete la participación. Un esquema puede aceptar los datos de ejemplo y, sin embargo, permitir estados que el diagrama prohíbe: por eso se revisan las reglas, además de la carga.

## Fuente de estudio

Enunciado del TP1 en la carpeta `../Practicos` y filminas de cátedra `BDI_01 (Intro - creacion esquemas).pdf` (carpeta de cursada 2023).
