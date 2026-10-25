---
tags:
  - tp6
---

# REVOKE, RESTRICT y CASCADE

```sql
REVOKE SELECT ON TABLE instalacion FROM adm CASCADE;

REVOKE GRANT OPTION FOR UPDATE
ON TABLE instalacion FROM adm CASCADE;
```

- La primera retira SELECT a adm y las concesiones dependientes de ese camino.
- La segunda quita su capacidad de delegar UPDATE, pero conserva su UPDATE propio; las delegaciones dependientes se revisan con CASCADE.
- RESTRICT rechaza la revocación si existen concesiones dependientes que requieran retirarse.
- CASCADE no elimina indiscriminadamente todos los permisos del receptor: pueden persistir concesiones independientes.

## Un permiso de tabla no se resta revocando una columna

Si se otorgó `UPDATE ON instalacion`, revocar `UPDATE(tarea)` no elimina el permiso sobre esa columna que sigue cubierto por el UPDATE de toda la tabla.

Para dejar acceso solo a ciertas columnas, retirar la concesión amplia correspondiente y conceder el subconjunto permitido, revisando permisos alternativos y dependencias. Por eso, en el ejercicio 1, no asumir que revocar la columna `tarea` a adm elimina el UPDATE de tabla que recibió al comienzo.

## Comparar tres revocaciones sobre la misma cadena

RESTRICT rechaza dependencias que quedarían sin fundamento. CASCADE retira concesiones dependientes. GRANT OPTION FOR puede quitar delegación conservando el permiso de uso.

### Ejemplo comparado de RESTRICT, CASCADE y GRANT OPTION FOR

Cadena A→B→C para comparar los tres resultados sobre la misma situación inicial.

Partiendo de `A → B con SELECT y WGO → C con SELECT`, sin otros caminos:

| Revocación de A a B | B después | C después |
|---|---|---|
| SELECT con RESTRICT | Se rechaza por la concesión dependiente; conserva lo anterior | Conserva lo anterior |
| SELECT con CASCADE | Pierde SELECT y su delegación | Pierde el SELECT dependiente |
| GRANT OPTION FOR SELECT con CASCADE | Conserva SELECT, pierde la delegación | Pierde el SELECT dependiente |

En el primer caso no se retira primero el permiso y se deja a C «colgado»: se rechaza la operación que produciría esa dependencia inválida. En CASCADE se siguen solo los caminos afectados; una concesión independiente puede mantener el acceso.

## Tabla y columna no son permisos negativos

Si adm recibió UPDATE de toda INSTALACION, revocar `UPDATE(tarea)` no introduce una prohibición sobre esa columna. El permiso amplio sigue cubriéndola. Para analizar el ejercicio 1, registrar el nivel exacto de la concesión original antes de interpretar la revocación.

Si el objetivo fuera dejar a adm solo cantHoras, haría falta retirar el permiso amplio correspondiente y volver a conceder el restringido, considerando las delegaciones que dependían de él. Una revocación cambia concesiones: no sustituye esa secuencia por una regla de «denegación de columna».

## Fuente de estudio

Enunciado del TP6 en la carpeta `../../Practicos` y filminas de cátedra `BDI_06 (Seguridad).pdf` (carpeta de cursada 2023).
