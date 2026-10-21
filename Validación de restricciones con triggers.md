---
tags:
  - tp4
---

# Validación de restricciones con triggers

## Eventos críticos de una restricción

Ejemplo del TP: proveedor y sucursal deben ser de la misma localidad.

| Tabla | INSERT | UPDATE relevante | DELETE |
|---|---|---|---|
| PROVEE | Verificar la relación nueva | Si cambia proveedor o sucursal | Quita una posible infracción; considerar otras RI |
| PROVEEDOR | No crea relación por sí solo | Si cambia localidad, revisar sus relaciones | Depende de FK y cascadas |
| SUCURSAL | No crea relación por sí sola | Si cambia localidad, revisar sus relaciones | Depende de FK y cascadas |

Para cada restricción, listar **todas las operaciones capaces de romperla**. Ejemplos:

- Hasta 15 palabras: INSERT o cambio de artículo en `contiene`.
- Horas dentro de tarea: cambio de horas/tarea del voluntario **y** de los límites de la tarea.
- Horas no superiores al coordinador: cambio del coordinado **y** disminución de horas del coordinador.

Un trigger que cuenta filas puede fallar ante dos transacciones concurrentes que ven el mismo estado. Para garantizar límites globales se debe coordinar el acceso, por ejemplo bloqueando una fila padre común en todos los caminos relevantes, o usando aislamiento serializable con reintentos. Ver TP8 - Concurrencia y recuperación.

## Reinicio de horas y control de variación

Este ejemplo cubre el primer requisito del ejercicio 5:

```sql
CREATE OR REPLACE FUNCTION reiniciar_horas_tarea()
RETURNS trigger
LANGUAGE plpgsql
AS $$
BEGIN
    IF NEW.id_tarea IS DISTINCT FROM OLD.id_tarea THEN
        NEW.horas_aportadas := 0;
    END IF;
    RETURN NEW;
END;
$$;

CREATE TRIGGER tr_reiniciar_horas_tarea
BEFORE UPDATE OF id_tarea ON voluntario
FOR EACH ROW
EXECUTE FUNCTION reiniciar_horas_tarea();
```

`IS DISTINCT FROM` detecta cambios incluso si interviene NULL. `UPDATE OF` se activa cuando la columna figura en el SET, aunque su valor no cambie: el IF verifica el cambio real.

Modificar `NEW` evita ejecutar otro UPDATE sobre la misma tabla, que puede producir recursión. Las filminas usan `EXECUTE PROCEDURE` en cabeceras de trigger; la forma moderna es `EXECUTE FUNCTION`, y el objeto llamado sigue siendo una función `RETURNS trigger`.

El segundo requisito del ejercicio 5 tiene redacción ambigua. Si se interpreta como «la variación máxima es ±10 %», para horas anteriores no negativas:

```sql
-- Fragmento dentro de la función, con política de nulidad definida.
IF NEW.horas_aportadas < OLD.horas_aportadas * 0.90
   OR NEW.horas_aportadas > OLD.horas_aportadas * 1.10 THEN
    RAISE EXCEPTION 'La variación de horas supera el 10 por ciento';
END IF;
```

Explicitar la interpretación. Si cambia tarea, priorizar el reinicio con `IF ... ELSIF ...`: de otro modo el control porcentual podría rechazar el cero obligatorio. Si las horas anteriores son cero, ±10 % solo admite cero. Controlar nulos explícitamente si no se permiten.

## Comprobación de triggers

Para cada regla, preparar al menos un caso válido, uno que deba rechazarse y uno de borde. Probar también un UPDATE de varias filas, nulos y cambios en la tabla relacionada. Comprobar el estado final, no solo el mensaje del motor.

Los ejemplos de esta nota son material de estudio: no fueron instalados en las bases existentes.
