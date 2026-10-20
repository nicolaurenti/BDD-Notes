---
tags:
  - tp4
---

# Auditoría por fila y por sentencia


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
