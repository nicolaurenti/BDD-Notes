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
