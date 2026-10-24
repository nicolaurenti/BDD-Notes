---
tags:
  - tp4
---

# TP4 - Ejemplo de máximo de palabras por artículo

Implementación y pruebas del máximo de quince asociaciones por artículo. Se conserva la sintaxis del enunciado.

## Ejemplo completo de un máximo por artículo

Se supone que cada fila de CONTIENE representa una asociación única de artículo y palabra. Un control AFTER ROW puede consultar el estado que ya incluye la inserción o el cambio de artículo:

```sql
CREATE OR REPLACE FUNCTION controlar_max_palabras()
RETURNS trigger AS $$
DECLARE
    cantidad INTEGER;
BEGIN
    SELECT COUNT(*) INTO cantidad
    FROM contiene
    WHERE id_articulo = NEW.id_articulo;

    IF cantidad > 15 THEN
        RAISE EXCEPTION 'El articulo supera las 15 palabras';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE 'plpgsql';

CREATE TRIGGER tr_max_palabras
AFTER INSERT OR UPDATE OF id_articulo ON contiene
FOR EACH ROW
EXECUTE PROCEDURE controlar_max_palabras();
```

Con 14 asociaciones, insertar una deja 15 y pasa. Con 15, insertar otra deja 16 y la excepción rechaza la sentencia; AFTER no significa que el cambio ya sea irreversible. Mover una asociación a un artículo con 15 también se rechaza. DELETE disminuye el conteo y no puede infringir este máximo.

El límite depende del momento: en BEFORE INSERT el conteo todavía no incluye la nueva fila, por lo que se rechazaría cuando el conteo previo ya fuera 15. Copiar la misma comparación `> 15` en un BEFORE admitiría indebidamente la número 16.

## Fuente de estudio

Enunciado del TP4 en la carpeta `../../Practicos` y filminas de cátedra `BDI_04 (SQL Procedural).pdf` (carpeta de cursada 2023).
