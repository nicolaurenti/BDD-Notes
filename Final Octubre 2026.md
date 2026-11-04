
# Ej 1

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



R2: Es un ASSERTION, no es soportado por PostgreSQL.

Evento critico:
- INSERT: Al agregar un pedido se debe corroborar la condicion
- UPDATE: Al actualizar cantidad, id_pedido o precio_unitario en Detalle_Pedido o id_pedido, importe_total en Pedido

R3: Tiene un CHECK con subconsulta, no es soportado por PostgreSQL.
- INSERT: Al agregar un pedido se debe corroborar la condicion
- UPDATE: fecha_entrega, estado de Pedido

