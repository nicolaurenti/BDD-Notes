---
tags:
  - tp4
---

# TP4 - Resolución 2 - Triggers de la alternativa STATEMENT

Cada tabla necesita preparar su conteo, acumular sus filas y registrar el resultado. Instalar estas cabeceras después de las tres funciones.

```sql
CREATE TRIGGER tp4_iniciar_entrega
BEFORE INSERT OR UPDATE OR DELETE ON entrega
FOR EACH STATEMENT EXECUTE PROCEDURE tp4_iniciar_conteo();

CREATE TRIGGER tp4_contar_entrega
AFTER INSERT OR UPDATE OR DELETE ON entrega
FOR EACH ROW EXECUTE PROCEDURE tp4_contar_fila();

CREATE TRIGGER tp4_registrar_entrega
AFTER INSERT OR UPDATE OR DELETE ON entrega
FOR EACH STATEMENT EXECUTE PROCEDURE tp4_registrar_sentencia();

CREATE TRIGGER tp4_iniciar_renglon
BEFORE INSERT OR UPDATE OR DELETE ON renglon_entrega
FOR EACH STATEMENT EXECUTE PROCEDURE tp4_iniciar_conteo();

CREATE TRIGGER tp4_contar_renglon
AFTER INSERT OR UPDATE OR DELETE ON renglon_entrega
FOR EACH ROW EXECUTE PROCEDURE tp4_contar_fila();

CREATE TRIGGER tp4_registrar_renglon
AFTER INSERT OR UPDATE OR DELETE ON renglon_entrega
FOR EACH STATEMENT EXECUTE PROCEDURE tp4_registrar_sentencia();
```

Para una actualización de tres entregas: reinicio una vez, tres incrementos y un registro final con cantidad 3. Para cero entregas: reinicio, ningún incremento y un registro final con cantidad 0.
