---
tags:
  - tp5
---

# Triggers INSTEAD OF sobre vistas

## Ejemplo de una política de actualización

La política del ejemplo permite cambiar solo cantidad y rechaza los cambios de clave o título. El trigger traduce la escritura de la vista en un UPDATE de RENGLON_ENTREGA.

### Ejemplo de trigger INSTEAD OF para cambiar cantidad

Ejemplo completo de la vista, su función y su trigger con EXECUTE PROCEDURE.

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
EXECUTE PROCEDURE actualizar_cantidad_vista();
```

Se filtra la tabla base por la clave completa de `OLD`. Este ejemplo habilita UPDATE con esa política; no implementa INSERT ni DELETE. Los filtros requeridos por el negocio deben controlarse explícitamente cuando se escribe mediante triggers: no asumir que WCO resuelve automáticamente la política definida en el trigger.

## Por qué aquí se modifica la tabla base

El trigger reemplaza la operación solicitada sobre la vista por el código que se escribió. NEW contiene los valores que el usuario quiere ver y OLD los de la fila seleccionada. El UPDATE de la función indica explícitamente qué tabla y columnas cambian: el motor no tiene que adivinarlo desde el join.

Con una fila `(10, P1, 5, 'Titulo A')`, pedir cantidad 8 actualiza el renglón cuya clave anterior es `(10, P1)`. Filtrar solo por película podría modificar varios renglones; usar toda la clave de OLD identifica el original incluso en políticas que admitieran cambiar una parte de la clave.

El IF del ejemplo rechaza cambios de clave y título porque la política elegida solo habilita cantidad. Si se quisiera admitir modificaciones del título, habría que declarar que se actualiza PELICULA y aceptar su efecto sobre todas las filas que la referencian. El enunciado exige justificar esa propagación; «agregar INSTEAD OF» no define por sí solo una política correcta.

RETURN NEW indica que el trigger atendió la actualización. RETURN NULL haría que se considere no procesada esa fila de la vista. Para INSERT y DELETE se requieren sus propias operaciones y retornos adecuados; este ejemplo no los habilita. Se conserva EXECUTE PROCEDURE, igual que en el TP4.

## Fuente de estudio

Enunciado del TP5 en la carpeta `../../Practicos` y filminas de cátedra `BDI_05 (Vistas).pdf` (carpeta de cursada 2023).
