# Ej 1

a)
Suponiendo:

Plan:

| Id_area | cod_plan | ... |
| ------- | -------- | --- |
| 1       | 1        | ... |
| 1       | 2        | ... |
| 2       | 3        | ... |
Gestion

| id_area | cod_plan | usuario | ... |
| ------- | -------- | ------- | --- |
| 1       | 1        | 2       | ... |
| 2       | 3        | 1       |     |
| 1       | 1        | 1       |     |
V1

| id_area | cod_plan | usuario | ... |
| ------- | -------- | ------- | --- |
| 1       | 1        | 1       |     |
| 1       | 1        | 2       |     |
| 2       | 3        | 1       |     |

El `NATURAL JOIN` compara `idArea` y `cod_plan`. La clave de GESTION es **(usuario, nro_gestion)**.

1. **Falsa.** Un mismo PLAN puede tener varias gestiones, por lo que puede aparecer en varias filas de la vista.
    
2. **Falsa.** Un PLAN_PROMO puede tener varias gestiones; su clave no limita el resultado a una fila por plan promocional.
    
3. **Falsa.** Un PLAN_TRAD puede tener varias gestiones; su clave no limita el resultado a una fila por plan tradicional.
    
4. **Verdadera.** Cada gestión puede coincidir con un solo PLAN como máximo, porque la unión utiliza la clave completa de PLAN.
    
5. **Falsa.** Solo se incluyen las combinaciones que coinciden en `idArea` y `cod_plan`, no todas las combinaciones posibles.
    
6. **Verdadera.** `(usuario, nro_gestion)` identifica cada fila de la vista: ninguna gestión se multiplica al unirse con PLAN.
    
7. **Falsa.** `(idArea, cod_plan)` puede repetirse en la vista cuando existen varias gestiones para el mismo plan.
    
8. **Falsa.** La clave de PLAN_PROMO coincide con la de PLAN y puede repetirse por las distintas gestiones del mismo plan.
    
9. **Falsa.** La clave de PLAN_TRAD coincide con la de PLAN y puede repetirse por las distintas gestiones del mismo plan.
    
10. **Falsa.** Concatenar ambas claves agrega atributos innecesarios: la clave de GESTION ya identifica cada fila; la concatenación no es mínima.
    
11. **Falsa.** Solo GESTION preserva su clave; no se permite propagar automáticamente modificaciones sobre cualquier atributo de ambas tablas.
    
12. **Falsa.** Según el criterio indicado, sí pueden propagarse modificaciones sobre los atributos correspondientes a GESTION, cuya clave se preserva.
    
13. **Falsa.** La propagación no se limita a los atributos clave: también abarca atributos no clave de GESTION, respetando las restricciones.
    
14. **Verdadera.** GESTION es la tabla que propaga su clave a la vista; sus atributos son los modificables según ese criterio.
    

**Distinción:** las opciones 11–14 preguntan por el estándar. En PostgreSQL, esta vista con JOIN **no es automáticamente actualizable**.

b) La vista devuelve todas las tuplas de plan cuyo idArea y cod_plan coincida con tuplas en la tabla gestion

c)Variaran ya que un FULL JOIN agregaria todas las tuplas de ambas tablas y en caso de que una tupla de una tabla no encuentre coincidencia en la otra tabla se completaran las columnas con null 

# Ej 2

```postgresql
CREATE VIEW v2 AS (
SELECT u.* 
FROM usuario u
WHERE NOT EXISTS (
	SELECT 1 
	FROM gestion g
	WHERE 
	u.nroUsuario = g.usuario 
	AND 
	g.fecha BETWEEN CURRENT_DATE - INTERVAL '3 months' AND CURRENT_DATE
) 
AND EXISTS (
	SELECT count(*)
	FROM asignacion asig 
	JOIN plan_promo pp ON pp.idArea = a.idAreaAND AND pp.cod_plan = a.cod_plan
	WHERE 
		a.nroUsuario = u.nroUsuario
    AND 
	    a.num_dispositivos > 3
    AND 
    (a.fecha_hasta IS NULL OR a.fecha_hasta >= CURRENT_DATE)
    AND 
    a.fecha_desde <= CURRENT_DATE
    GROUP BY a.fecha_desde
    HAVING COUNT(DISTINCT (a.idArea, a.cod_plan)) >= 2
	) 
)
```

```postgresql
-- Vista parcialmente actualizable
CREATE VIEW v3 AS (
SELECT u.*, 
	(
		SELECT count(*) 
		FROM plan_promo pp NATURAL JOIN asignacion asig 
		WHERE 
			asig.nroUsuario = u.nroUsuario 
		AND 
			(asig.fecha_hasta IS NULL OR asig.fecha_hasta >= current_date)
		AND 
			asig.fecha_desde <= current_date
	) as planes_asignados
FROM usuario u
WHERE NOT EXISTS (
	SELECT 1
	FROM gestion g
	WHERE 
		g.fecha BETWEEN CURRENT_DATE - INTERVAL '3 months' AND CURRENT_DATE
	AND g.usuario = u.nroUsuario
	) 
)
```
# Ej 3 

a) Una comparación de igualdad con NULL devuelve UNKNOWN. Si IN encuentra una coincidencia, devuelve TRUE; si no encuentra ninguna y la subconsulta contiene NULL, devuelve UNKNOWN. Como WHERE solo conserva las filas cuya condición es TRUE, esas filas se descartan.

b)
	b1_ Devuelve una sola fila con el valor del count en 10000
	b2_ Devuelve una tabla vacia
# Ej 4

a) 

```postgresql
ALTER TABLE asignacion
ADD CONSTRAINT limit_disp
CHECK (
NOT EXISTS (
	SELECT 1 
	FROM asignacion asig
	WHERE 
		asig.num_dispositivos <= 5 
	OR (
		asig.fecha_desde < asig.fecha_hasta 
	AND 
		asig.fecha_hasta IS NOT NULL
	)
	)
)
```

b)
```postgresql
CREATE ASSERTION reclamo_plan_asignado
CHECK (
    NOT EXISTS (
        SELECT 1
        FROM gestion g
        WHERE g.motivo = 'reclamo'
          AND g.idArea IS NOT NULL
          AND g.cod_plan IS NOT NULL
          AND NOT EXISTS (
              SELECT 1
              FROM asignacion a
              WHERE a.nroUsuario = g.usuario
                AND a.idArea = g.idArea
                AND a.cod_plan = g.cod_plan
          )
    )
);
```
# Ej 5

```postgresql
CREATE OR REPLACE FUNCTION carga_informe_ins() 
RETURNS TRIGGER AS $$
DECLARE 
var_cantAsigPlanPromo
var_cantAsigPlanTrad
var_cantGesPlanPromo
var_cantGesPlanTrad
var_descProm
BEGIN
	SELECT count(*) FROM plan_promo
	INSERT INTO InformePlan VALUES ()
	RETURN NEW;
END;
$$ language 'plpgsql'
```