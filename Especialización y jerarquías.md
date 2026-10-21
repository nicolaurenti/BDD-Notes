---
tags:
  - tp1
---

# Especialización y jerarquías


Una forma de representar una especialización es crear:

- Una tabla para el **supertipo**, con los atributos comunes.
- Una tabla para cada **subtipo**, con sus atributos específicos.

### Ejemplo de grabaciones

| Tabla | Contenido |
|---|---|
| `GRABACION` | Atributos comunes de todas las grabaciones. |
| `GRAB_NO_PROPIA` | Atributos específicos, como duración. |
| `GRAB_COMERCIAL` | Atributos específicos y referencia al equipo. |

La clave de cada subtipo es simultáneamente:

- **PK** de su tabla.
- **FK** hacia la tabla del supertipo.

```sql
nro_grabacion INTEGER PRIMARY KEY
    REFERENCES grabacion (nro_grabacion)
```

### Totalidad y disyunción

En el diagrama del práctico, la especialización es:

- **Total:** toda grabación debe pertenecer a algún subtipo.
- **Disjunta:** una grabación no puede pertenecer a ambos subtipos.

> [!important]
> Las PK y FK de este diseño no garantizan por sí solas la totalidad ni la disyunción. Agregar una columna `tipo` tampoco basta: hacen falta controles adicionales para mantener la correspondencia.

## Ejemplo de la representación

```sql
CREATE TABLE grabacion (
    nro_grabacion INTEGER PRIMARY KEY
    -- Los demás atributos comunes se toman del diagrama.
);

CREATE TABLE grab_no_propia (
    nro_grabacion INTEGER PRIMARY KEY,
    duracion INTEGER,
    FOREIGN KEY (nro_grabacion)
        REFERENCES grabacion (nro_grabacion)
);
```

Estos fragmentos muestran la estructura de la jerarquía; no reemplazan las columnas y tipos completos del ejercicio. Una grabación no propia se almacena en GRABACION y en GRAB_NO_PROPIA con la misma clave. No son dos objetos: la fila específica amplía la fila general.

La FK impide crear un subtipo sin supertipo. La PK del subtipo impide tener dos filas específicas para la misma grabación. Sin embargo, todavía sería posible dejar una grabación solo en la tabla general, o insertar su clave en ambas tablas de subtipos. Esos casos muestran por qué **totalidad y disyunción no se deducen de las FK**.

Para revisar una jerarquía, probar mentalmente cuatro estados: solo supertipo, un subtipo, otro subtipo y ambos subtipos. Comparar cuáles permite el diagrama con cuáles permiten las restricciones declaradas. No afirmar que una condición está implementada solo porque se explicó en el texto del modelo.

## Fuente de estudio

Enunciado del TP1 en la carpeta `../Practicos` y filminas de cátedra `BDI_01 (Intro - creacion esquemas).pdf` (carpeta de cursada 2023).
