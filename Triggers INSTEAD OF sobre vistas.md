---
tags:
  - tp5
---

# Triggers INSTEAD OF sobre vistas


Ejemplo: permitir únicamente cambiar `cantidad` de un renglón a través de una vista que también muestra el título.

```sql
CREATE VIEW entregas_kp_3 AS
SELECT re.nro_entrega, re.codigo_pelicula, re.cantidad, p.titulo
FROM renglon_entrega re
JOIN pelicula p ON p.codigo_pelicula = re.codigo_pelicula;

CREATE OR REPLACE FUNCTION actualizar_cantidad_vista()
RETURNS trigger
LANGUAGE plpgsql
AS $$
BEGIN
    IF NEW.nro_entrega IS DISTINCT FROM OLD.nro_entrega
       OR NEW.codigo_pelicula IS DISTINCT FROM OLD.codigo_pelicula
       OR NEW.titulo IS DISTINCT FROM OLD.titulo THEN
        RAISE EXCEPTION 'Solo se permite modificar cantidad';
    END IF;

    UPDATE renglon_entrega
    SET cantidad = NEW.cantidad
    WHERE nro_entrega = OLD.nro_entrega
      AND codigo_pelicula = OLD.codigo_pelicula;

    RETURN NEW;
END;
$$;

CREATE TRIGGER tr_actualizar_cantidad_vista
INSTEAD OF UPDATE ON entregas_kp_3
FOR EACH ROW
EXECUTE FUNCTION actualizar_cantidad_vista();
```

Se filtra la tabla base por la clave completa de `OLD`. Este ejemplo habilita UPDATE con esa política; no implementa INSERT ni DELETE. Los filtros requeridos por el negocio deben controlarse explícitamente cuando se escribe mediante triggers: no asumir que WCO resuelve automáticamente una política customizada.
