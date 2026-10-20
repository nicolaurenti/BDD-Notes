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
