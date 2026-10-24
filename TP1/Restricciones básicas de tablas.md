---
tags:
  - tp1
---

# Restricciones básicas de tablas

## Restricciones básicas

| Restricción | Función |
|---|---|
| `PRIMARY KEY` | Identifica cada fila. Exige unicidad y ausencia de nulos. |
| `UNIQUE` | Impide duplicados según las reglas de nulidad del motor. |
| `NOT NULL` | Impide valores nulos. |
| `FOREIGN KEY` | Exige que la referencia exista, salvo los casos permitidos de nulidad. |

> [!note]
> En PostgreSQL, `UNIQUE` permite por defecto varios valores `NULL`. Para representar una clave alternativa se utiliza `UNIQUE` junto con `NOT NULL`.

### Clave compuesta

```sql
CONSTRAINT pk_palabra
    PRIMARY KEY (cod_p, idioma)
```

La combinación debe ser única. No se exige que cada columna sea única por separado.

## Comprobación de restricciones

El motor verifica las restricciones al insertar o modificar datos.

Una operación puede rechazarse por:

- Repetir una PK.
- Colocar `NULL` en una columna que no lo permite.
- Repetir una clave alternativa.
- Referenciar una fila inexistente.
- Modificar o borrar una clave referenciada, según las acciones referenciales.

## Distinguir identificación, obligatoriedad y referencia

En `persona(id_persona, documento, nombre, id_institucion)`, se pueden establecer reglas diferentes:

```sql
CREATE TABLE persona_ejemplo (
    id_persona INTEGER PRIMARY KEY,
    documento VARCHAR(20) NOT NULL UNIQUE,
    nombre VARCHAR(60) NOT NULL,
    id_institucion INTEGER REFERENCES institucion (id_institucion)
);
```

El id identifica internamente. Documento es una clave alternativa porque no se repite y no admite NULL. Nombre es obligatorio, pero dos personas pueden llamarse igual. La institución puede omitirse; si se informa, debe existir en la tabla referenciada. El ejemplo supone que INSTITUCION ya fue creada con su PK.

No agregar UNIQUE a cada atributo solo porque todos los valores de la pequeña carga de prueba sean distintos. Cada restricción debe provenir de una regla del modelo. Tampoco hace falta repetir NOT NULL sobre una columna que ya integra la PK: la clave primaria ya lo exige.

Los nombres de restricciones, como `pk_persona` o `fk_persona_institucion`, ayudan a interpretar los errores y a modificar el esquema. No cambian la regla que se comprueba.

## Fuente de estudio

Enunciado del TP1 en la carpeta `../../Practicos` y filminas de cátedra `BDI_01 (Intro - creacion esquemas).pdf` (carpeta de cursada 2023).
