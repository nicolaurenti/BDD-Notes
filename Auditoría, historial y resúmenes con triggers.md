---
tags:
  - tp4
---

# Auditoría, historial y resúmenes con triggers

## Auditoría por fila y por sentencia

En `HIS_ENTREGA`, guardar como mínimo identificador de registro, fecha, operación, cantidad de filas y usuario. Conviene agregar la tabla de origen para distinguir ENTREGA de RENGLON_ENTREGA.

| Diseño | Sentencia que afecta 5 filas |
|---|---|
| AFTER ROW | 5 registros de auditoría, normalmente con cantidad 1 |
| AFTER STATEMENT | 1 registro con cantidad 5, si se implementa ese conteo |

No usar `COUNT(*) FROM entrega` como «filas afectadas»: cuenta la tabla completa. Tampoco `GET DIAGNOSTICS ROW_COUNT` representa automáticamente las filas de la sentencia disparadora.

Para contar en triggers de sentencia pueden usarse tablas de transición en triggers AFTER compatibles:

```sql
-- Fragmento de cabecera para UPDATE; la función se define aparte.
-- AFTER UPDATE ON entrega
-- REFERENCING OLD TABLE AS anteriores NEW TABLE AS nuevas
-- FOR EACH STATEMENT EXECUTE FUNCTION auditar_update_entrega();
```

La función cuenta `nuevas` para UPDATE. INSERT y DELETE requieren sus propias relaciones de transición y cabeceras. Analizar también los eventos que provocan las cascadas de FK. La auditoría normal participa en la misma transacción: si esta se revierte, sus registros también.

## Historial, resúmenes y carga inicial

Para permanencias por departamento, guardar ingreso y cada intervalo de asignación; identificar el departamento con su clave completa `(id_distribuidor, id_departamento)`. Al cambiar de departamento, cerrar el intervalo anterior y abrir el siguiente. Decidir si un intervalo abierto participa del promedio y cómo se calcula su duración.

Para `TextosporAutor`:

1. Carga inicial con `INSERT ... SELECT autor, COUNT(*), MAX(fecha_pub) FROM articulo GROUP BY autor` sobre una tabla resumen inicialmente vacía.
2. Mantenimiento ante INSERT, UPDATE y DELETE.
3. En un cambio de autor, actualizar el autor anterior y el nuevo.
4. Si se borra el artículo más reciente, recalcular el máximo; no basta con restar uno al conteo.
5. Definir qué hacer con autores sin artículos y cómo evitar duplicados al repetir una carga.

Un trigger instalado hoy no conoce fechas históricas que nunca se registraron. Un total de permanencia que crece con el tiempo no se mantiene exacto únicamente con triggers de escritura; puede calcularse al consultar los intervalos.
