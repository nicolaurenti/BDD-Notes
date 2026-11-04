Resolución del final [agosto 2020](file:///Users/nicolaurenti/Desktop/Bases%20de%20Datos/Contenido%20extra/Finales/Final%202020%20Agosto.pdf)

# Ej 1

```mysql
SELECT a.cod_carrera, a.nombre_asig
FROM asignatura a
WHERE a.cant_hs_p > 30 and NOT EXISTS (
SELECT 1 
FROM profesor p
WHERE departamento IS DISTINCT "Computación"
)
```

# E j 2

```mysql
CREATE ASSERTION ass_profesores_20
CHECK (
	NOT EXISTS(
	SELECT 1 
	FROM prof_exclusivo pe JOIN profesor p ON p.legajo = pe.legajo
	WHERE p.titulo IS NOT NULL AND (
	(SELECT count( * ) FROM asignatura_profesor ap WHERE ap.legajo = pe.legajo) * 5 >=(SELECT count( * ) FROM asignatura a2)
	)
)
```

# Ej 3

Eventos criticos del ejercicio 2:

- INSERT
	- asignatura: al insertar una asignatura varia el porcentaje de materias a la que pertenece el profesor
	- asignatura_profesor: al agregar una relacion reduce la cantidad de asignaturas asociadas a un profesor
	- prof_exclusivo: al agregar un nuevo profesor exclusivo
- UPDATE 
	-  profesor: al setear el titulo de un profesor como NULL ya no permite que cumpla con los requisistos de la consigna
- DELETE
	- asignatura: impacta en los porcentajes de asignatura y asignaturas del profesor
	- asignatura_profesor: al eliminar la asociacion asignatura-profesor se afecta el porcentaje buscado

## Encabezados de triggers

CREATE TRIGGER 

```mysql
CREATE TRIGGER tr_ins_asig
AFTER INSERT ON asignatura -- se chequea despues de agregarlo si se cumple la condicion, se puede hacer AFTER y luego no realizar la accion
FOR EACH ROW -- se tiene que ejecutar por cada fila involucrada DUDA: las "ROW" son las afectadas en la tabla?
EXECUTE PROCEDURE controlar_participacion_profesor();
```

```mysql
CREATE TRIGGER tr_ins_asig_prof
AFTER INSERT ON asignatura_profesor 
FOR EACH ROW 
EXECUTE PROCEDURE controlar_participacion_profesor();
```

```mysql
CREATE TRIGGER tr_ins_prof_excl
AFTER INSERT ON prof_exclusivo 
FOR EACH ROW 
EXECUTE PROCEDURE controlar_participacion_profesor();
```

```mysql
CREATE TRIGGER tr_upt_prof
AFTER UPDATE ON profesor 
FOR EACH ROW 
EXECUTE PROCEDURE controlar_participacion_profesor();
```

```mysql
CREATE TRIGGER tr_del_asig
AFTER DELETE ON asignatura 
FOR EACH ROW 
EXECUTE PROCEDURE controlar_participacion_profesor();
```

```mysql
CREATE TRIGGER tr_del_asig_prof
AFTER DELETE ON asignatura_profesor 
FOR EACH ROW 
EXECUTE PROCEDURE controlar_participacion_profesor();
```

## Control de la condicion

```mysql
CREATE OR REPLACE FUNCTION controlar_participacion_profesor()
RETURNS trigger AS $$
DECLARE
    total_asignaturas BIGINT;
    cantidad_asignaturas BIGINT;
BEGIN
    -- Solo se controla si es exclusivo y tiene título.
    IF EXISTS (
        SELECT 1
        FROM profesor p
        JOIN prof_exclusivo pe ON pe.legajo = p.legajo
        WHERE p.legajo = NEW.legajo
          AND p.titulo IS NOT NULL
    ) THEN
        SELECT COUNT(*)
        INTO total_asignaturas
        FROM asignatura;

        SELECT COUNT(*)
        INTO cantidad_asignaturas
        FROM asignatura_profesor ap
        WHERE ap.legajo = NEW.legajo;

        IF cantidad_asignaturas * 5 >= total_asignaturas THEN
            RAISE EXCEPTION
                'El profesor % debe participar en menos del 20 %% de las asignaturas',
                NEW.legajo;
        END IF;
    END IF;

    RETURN NULL;
END;
$$ LANGUAGE 'plpgsql';
```

# Ej4

	-  Rechaza porque cuatrimestre = 2
	- Acepta porque cuatrimestre = 1
	- Acepta, como es la tabla de origen, no hay restricciones
	- Acepta porque AsigIntro no tiene CO, pero al momento de consultarla no aparecera "Base de datos"
	- Rechaza, al ser CASCADE, aunque cant_hs_p cumple la condicion de AsigP(el valor es 30 y la condicion pide que sea mayor a 10) no cumple la condicion de AsigIntro(que el nombre_asig comience con "Intro")
	- Permite por el mismo motivo que "Base de datos"



# Ej 5

Si la asignatura que cumple con cod_asignatura=234 y cod_carrera = 6 tiene referencias en las tablas profesor o asignatura_profesor, al no tener declarada ninguna restriccion(CASCADE, SET NULL y NO ACTION) por defecto la restriccion es NO ACTION y no se elimina esta fila

# Ej 6

Una restricción de integridad referencial establece que cada valor de una clave foránea debe corresponder a una clave existente en la tabla referenciada, salvo las excepciones por nulos según el tipo de MATCH.

Por ejemplo, si un profesor referencia la asignatura `(2, 3)`, la restricción se satisface si esa asignatura existe.

Ante una FK compuesta con nulos, MATCH SIMPLE considera satisfecha la restricción si al menos un componente es NULL. MATCH FULL permite que todos sean NULL, pero rechaza una combinación de nulos y valores. MATCH PARTIAL exige una coincidencia para los componentes no nulos; si todos son NULL, la restricción se satisface.

# Ej 7

La afirmación es incorrecta. En un protocolo basado en marcas de tiempo, el control de concurrencia se realiza comparando la marca de cada transacción con las marcas de lectura y escritura de los datos, para respetar el orden establecido. Por ejemplo, si una transacción con marca 20 ya leyó un dato, una transacción con marca 10 no puede escribirlo posteriormente: se rechaza la escritura y se aborta la transacción más antigua. No se analizan bloqueos, como ocurre en los protocolos basados en bloqueos.