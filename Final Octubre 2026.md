![[Pasted image 20261104115312.png]]
# Ej 1

![[Pasted image 20261104115324.png]]
## a)

## R1
```postgresql
	ALTER TABLE Pedido
	ADD CONSTRAINT constr_pedidos_cancelados_r1
	CHECK NOT EXISTS (
		SELECT 1
		FROM Pedido p
		WHERE p.estado_pedido = 'cancelado'
		GROUP BY (p.id_direccion, p.cliente_id, p.fecha_entrega)
		HAVING COUNT(*)>3
	)
```

## R2
```postgresql
CREATE ASSERTION ass_importe_consistente_r2
CHECK (
	NOT EXISTS (
		SELECT 1
		FROM Pedido p
		WHERE p.importe_total = (
			SELECT count(dp.cantidad * dp.precio_unitario)
			FROM Detalle_Pedido
			WHERE dp.id_pedido = p.id_pedido
		)
	)
)
```

## R3
```postgresql
ALTER TABLE Pedido
ADD CONSTRAINT const_fecha_entrega_r3
CHECK NOT EXISTS (
	SELECT 1 
	FROM Pedido p
					WHERE p.fecha_entrega IS NOT NULL AND (p.estado_pedido <> 'en camino' OR p.estado_pedido <> 'entregado')
)
```

## b)

R1: Tiene un CHECK con subconsulta, no es soportado por PostgreSQL.

Evento critico: 
- INSERT: Al agregar un pedido se debe corroborar la condicion
- UPDATE: Se podria actualizar un pedido cambiando el estado o la direccion de entrega o la fecha

```postgresql
CREATE TRIGGER tr_pedidos_cancelados
AFTER INSERT OR UPDATE OF estado_pedido, id_direccion, fecha_entrega ON Pedido
FOR EACH ROW
EXECUTE PROCEDURE fn_pedidos_cancelados;
```


R2: Es un ASSERTION, no es soportado por PostgreSQL.

Evento critico:
- INSERT: Al agregar un pedido se debe corroborar la condicion
- UPDATE: Al actualizar cantidad, id_pedido o precio_unitario en Detalle_Pedido o id_pedido, importe_total en Pedido

```postgresql
CREATE TRIGGER tr_control_importe_total
AFTER INSERT OR UPDATE OF cantidad, id_pedido o precio_unitario ON Detalle_pedido
FOR EACH ROW
EXECUTE PROCEDURE fn_control_importe_total;
```


R3: Tiene un CHECK con subconsulta, no es soportado por PostgreSQL.

Evento critico:
- INSERT: Al agregar un pedido se debe corroborar la condicion
- UPDATE: fecha_entrega, estado de Pedido

```postgresql
CREATE TRIGGER tr_control_fecha_estado
AFTER INSERT OR UPDATE OF fecha_entrega, estado ON Pedido
FOR EACH ROW
EXECUTE PROCEDURE fn_control_fecha_estado;
```

## Ej 2

![[Pasted image 20261104115340.png]]


Objetivo: 
	Completar tabla informe_clientes con: 
De la tabla Cliente
-  id_cliente
- nombre
- apellido
De la tabla Pedido
- Cantidad de estos pedidos realizados 
- Importe total promedio
- Fecha de entrega del mas reciente

Recurso a utilizar: Como el ejercicio pide completar una tabla y no involucra acciones (**INSERT**, **UPDATE** o **DELETE**) se debe utilizar **stored procedure**

Restriccion:
- el cliente posee al menos un pedido entregado (estado_pedido ='entregado' en Pedido) de mas de N productos distintos realizado a partir de la fecha F (N y F parametros)



```postgresql
CREATE OR REPLACE PROCEDURE sp_completar_inf_clientes (n INT, f DATE) 
LANGUAGE 'plpgsql'
AS $$
BEGIN
    DELETE FROM informe_clientes;

    INSERT INTO informe_clientes (
    id_cliente, 
    apellido, 
    nombre,
    cantidad_pedidos, 
    importe_promedio, 
    ultima_fecha_entrega
    )
    SELECT c.id_cliente,
           c.apellido,
           c.nombre,
           COUNT(*),
           AVG(p.importe_total),
           MAX(p.fecha_entrega)
    FROM cliente c
    JOIN pedido p ON p.cliente_id = c.id_cliente
    WHERE p.estado_pedido = 'entregado'
      AND p.fecha_pedido >= f
      AND (
          SELECT COUNT(*)
          FROM detalle_pedido d
          WHERE d.id_pedido = p.id_pedido
      ) > n
    GROUP BY c.id_cliente, c.apellido, c.nombre;
END;
$$;

CALL sp_completar_inf_clientes(5, DATE '2026-10-01')
```

## Ej 3

![[Pasted image 20261104115356.png]]

a)

```postgresql
CREATE VIEW v1_electronica AS
SELECT p.*
FROM producto p
WHERE p.categoria = 'electrónica'
  AND p.stock > 0
  AND EXISTS (
      SELECT 1
      FROM detalle_pedido dp
      JOIN proveedor pv ON pv.id_proveedor = dp.id_proveedor
      JOIN pedido ped ON ped.id_pedido = dp.id_pedido
      JOIN direccion_entrega de
        ON de.id_direccion = ped.id_direccion
       AND de.cliente_id = ped.cliente_id
      WHERE dp.cod_producto = p.cod_producto
        AND dp.id_proveedor = p.id_proveedor
        AND pv.puntuacion > 3
        AND ped.estado_pedido = 'entregado'
        AND ped.fecha_entrega > CURRENT_DATE - 30
        AND ped.fecha_entrega <= CURRENT_DATE
        AND de.ciudad = 'Tandil'
  );
```

**Por qué es automáticamente actualizable.** El `FROM` principal tiene una sola tabla base, `producto`, y las columnas de la vista son las de `p.*`. El `EXISTS` filtra las filas visibles. PostgreSQL puede aplicar un `UPDATE` sobre la vista a la fila correspondiente de `producto`.

Supongamos que el producto `(cod_producto, id_proveedor) = (10, 3)` aparece en la vista y tiene stock positivo. Comparamos esta sentencia desde el mismo estado inicial en ambas variantes.

```postgresql
UPDATE v1_electronica
SET stock = 0
WHERE cod_producto = 10 AND id_proveedor = 3;
```

| Variante | Resultado |
|---|---|
| Sin opción de chequeo | El `UPDATE` guarda `stock = 0` en `producto`. La fila deja de aparecer en la vista porque deja de cumplir `stock > 0`. |
| Con `WITH CHECK OPTION` | PostgreSQL comprueba el nuevo estado de la fila. Como con `stock = 0` quedaría fuera de la vista, rechaza la sentencia y el stock conserva su valor anterior. |

Para definir la segunda variante se agrega `WITH CHECK OPTION` al final del `CREATE VIEW`, antes del punto y coma. En la vista ya creada también se puede activar así.

```postgresql
ALTER VIEW v1_electronica SET (check_option = 'local');
```

`LOCAL` y `CASCADED` dan el mismo resultado acá porque la vista se define sobre una tabla base.

b)

```postgresql
CREATE VIEW v2_prov_ped AS
SELECT pv.id_proveedor,
       pv.nombre AS nombre_proveedor,
       pv.email,
       pv.telefono,
       pv.puntuacion,
       p.cod_producto,
       p.nombre AS nombre_producto,
       p.precio_actual,
       p.stock,
       p.categoria
FROM proveedor pv
JOIN producto p ON p.id_proveedor = pv.id_proveedor
WHERE NOT EXISTS (
    SELECT 1
    FROM detalle_pedido dp
    JOIN pedido ped ON ped.id_pedido = dp.id_pedido
    WHERE dp.cod_producto = p.cod_producto
      AND dp.id_proveedor = p.id_proveedor
      AND ped.estado_pedido NOT IN ('cancelado', 'devuelto')
);
```

**Qué filas aparecen.** La subconsulta busca pedidos con un estado distinto de cancelado o devuelto para el producto identificado por su clave completa. Si el producto nunca se pidió, la subconsulta queda vacía. También queda vacía cuando todos sus pedidos están cancelados o devueltos. Un solo pedido en otro estado alcanza para excluirlo.

**Cómo se propaga el `DELETE`.** El `JOIN` entre `proveedor` y `producto` en el `FROM` principal impide la actualización automática en PostgreSQL. Un trigger `INSTEAD OF DELETE FOR EACH ROW` define qué significa eliminar cada fila visible. Se interpreta como dar de baja el producto identificado por `(OLD.cod_producto, OLD.id_proveedor)`. El proveedor puede ofrecer otros productos, por lo que su fila se conserva.

**Decisión sobre el historial.** Para que también se puedan eliminar productos referenciados, se asume que está permitido retirar sus líneas de pedidos cancelados o devueltos. Primero se eliminan esas líneas y después el producto. Los pedidos se conservan. Se supone que la FK del detalle hacia producto impide eliminar un producto con referencias restantes (`NO ACTION` inmediato o `RESTRICT`). Si aparece una referencia que impide la baja, la FK rechaza el borrado y se revierten los cambios de esa sentencia. Si el negocio exige conservar todos los detalles históricos, habría que usar una baja lógica para los productos referenciados o rechazar su borrado físico.

```postgresql
CREATE OR REPLACE FUNCTION fn_eliminar_producto_v2()
RETURNS trigger
LANGUAGE plpgsql
AS $$
BEGIN
    DELETE FROM detalle_pedido dp
    USING (
        SELECT ped.id_pedido
        FROM pedido ped
        JOIN detalle_pedido ref ON ref.id_pedido = ped.id_pedido
        WHERE ref.cod_producto = OLD.cod_producto
          AND ref.id_proveedor = OLD.id_proveedor
          AND ped.estado_pedido IN ('cancelado', 'devuelto')
        FOR SHARE OF ped
    ) AS elegibles
    WHERE dp.id_pedido = elegibles.id_pedido
      AND dp.cod_producto = OLD.cod_producto
      AND dp.id_proveedor = OLD.id_proveedor;

    DELETE FROM producto p
    WHERE p.cod_producto = OLD.cod_producto
      AND p.id_proveedor = OLD.id_proveedor;

    IF NOT FOUND THEN
        RETURN NULL;
    END IF;
    RETURN OLD;
END;
$$;

CREATE TRIGGER tr_eliminar_producto_v2
INSTEAD OF DELETE ON v2_prov_ped
FOR EACH ROW
EXECUTE PROCEDURE fn_eliminar_producto_v2();
```

`FOR SHARE OF ped` bloquea cambios de estado en los pedidos elegibles mientras dura la operación. `RETURN OLD` indica que el trigger procesó la fila borrada. Si el producto no tiene detalles, el primer `DELETE` afecta cero filas y la baja del producto se realiza igual.

```postgresql
DELETE FROM v2_prov_ped
WHERE cod_producto = 10 AND id_proveedor = 3;
```

Si el producto aparece en la vista, se ejecuta el trigger. Si aparece en un pedido entregado, la vista lo excluye y la sentencia afecta cero filas.

## Ej 4

![[Pasted image 20261104115419.png]]![[Pasted image 20261104115435.png]]
