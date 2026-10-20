---
tags:
  - tp2
---

# Resolución del TP2 — Consultas SQL

Recopilación de las consultas y anotaciones compartidas durante la práctica. Se conservan los dos archivos pegados tal como fueron enviados; esta recopilación no implica que todas las respuestas estén corregidas ni completa ejercicios no incluidos.

## 1. Grabaciones

Conexión de referencia: base `postgres`, esquema `grabaciones`. Las consultas usan nombres de tabla sin esquema; verificar base y esquema antes de ejecutarlas. La consulta b toma el año de la fecha en que se ejecuta.

```sql
-- a)
SELECT DISTINCT g.casa_discografica
FROM grabacion g
JOIN grab_comercial gc
    -- Había usado LEFT JOIN, pero eso deja casas sin grabaciones comerciales.
    ON g.nro_grabacion = gc.nro_grabacion;

-- b)
SELECT DISTINCT g.fecha_grabacion
FROM grabacion g
WHERE EXTRACT(YEAR FROM g.fecha_grabacion)
    = EXTRACT(YEAR FROM CURRENT_DATE);
```

## 2. Voluntarios

Conexión de referencia: base `voluntarios`, esquema `public`. El archivo comienza comprobando base y esquema; `SET search_path` no cambia la base conectada.

Incluye ejercicios de la primera parte, consultas 1.1 a 1.8 de la segunda y las cinco respuestas de operadores de conjuntos identificadas como EJ3 en el texto enviado.

```sql
SET search_path TO public;

SELECT current_database(), current_schema();
-- EJ2
-- a)
SELECT v.apellido, v.nombre, v.e_mail
FROM voluntario v
WHERE v.horas_aportadas > 1000
ORDER BY v.apellido;

--b)
SELECT v.apellido, v.telefono
FROM voluntario v
WHERE v.id_institucion IN (20, 50)
ORDER BY (v.apellido, v.nombre);

--c)
SELECT v.apellido || ', ' || v.nombre AS "Apellido y Nombre",
       v.e_mail                       AS "Dirección de mail"
FROM voluntario v
WHERE v.telefono LIKE '+11%';

--EJ4
--a)
SELECT v.apellido || ',' || v.nombre             AS "apellido y nombre",
       to_char(v.fecha_nacimiento, 'YYYY-MM-DD') AS "fecha de nacimiento"
FROM voluntario v
ORDER BY EXTRACT(YEAR FROM v.fecha_nacimiento);

--b)
SELECT *
FROM voluntario v
WHERE EXTRACT(YEAR FROM v.fecha_nacimiento) >= 1998
  AND v.horas_aportadas > 5000
ORDER BY v.nro_voluntario;

--c)
SELECT v.nombre, v.apellido, EXTRACT(YEAR FROM AGE(CURRENT_DATE, v.fecha_nacimiento)) AS "Edad"
FROM voluntario v
WHERE EXTRACT(DAY FROM v.fecha_nacimiento) = EXTRACT(DAY FROM current_date)
  AND EXTRACT(MONTH FROM v.fecha_nacimiento) = EXTRACT(MONTH FROM current_date)
;

--EJ5
--a)
SELECT *
FROM direccion
ORDER BY id_direccion, id_pais
LIMIT 10;

--b)
SELECT *
FROM tarea t
WHERE t.nombre_tarea LIKE 'O%'
   OR t.nombre_tarea LIKE 'A%'
   OR t.nombre_tarea LIKE 'C%'
ORDER BY t.id_tarea
OFFSET 10 LIMIT 5;

--EJ6
--a)
SELECT MIN(v.horas_aportadas) AS "MIN", MAX(v.horas_aportadas) AS "MAX", AVG(v.horas_aportadas) AS "AVG"
FROM voluntario v
WHERE extract(YEAR FROM AGE(CURRENT_DATE, v.fecha_nacimiento)) > 25;

--b)
SELECT v.id_institucion, count(*) as "Cant. empleados"
FROM voluntario v
GROUP BY v.id_institucion;

--c)
SELECT min(v.fecha_nacimiento) AS "Mas viejo", max(v.fecha_nacimiento) as "Mas joven"
FROM voluntario v;

--d)
SELECT h.nro_voluntario AS "Voluntario", count(DISTINCT h.id_tarea) as "Cant. tareas"
FROM historico h
GROUP BY h.nro_voluntario;

--e)
SELECT id_coordinador, id_institucion
FROM voluntario
WHERE id_coordinador IS NOT NULL
  AND id_institucion IS NOT NULL
GROUP BY id_coordinador, id_institucion
HAVING count(*) < 3;

--TP2 Parte 2
--EJ1
--1.1)
SELECT h.nro_voluntario, v.apellido, v.nombre, count(h.id_tarea)
FROM public.voluntario v
         LEFT JOIN historico h ON v.nro_voluntario = h.nro_voluntario
GROUP BY h.nro_voluntario, v.apellido, v.nombre;

--1.2)
SELECT v.nombre, v.apellido, v.e_mail, v.telefono
FROM voluntario v
         JOIN tarea t ON t.id_tarea = v.id_tarea
         JOIN historico h ON h.nro_voluntario = v.nro_voluntario
WHERE (t.max_horas - t.min_horas <= 5000)
  AND (h.fecha_fin <= DATE '2000-01-01');

--1.3)
SELECT i.nombre_institucion, i.id_institucion, i.id_direccion
FROM institucion i
WHERE NOT EXISTS(SELECT 1
                 FROM voluntario v
                          JOIN tarea t ON t.id_tarea = v.id_tarea
                 WHERE v.id_institucion = i.id_institucion
                   AND v.horas_aportadas >= t.max_horas);

--1.4)
SELECT p.nombre_pais
FROM pais p
WHERE NOT EXISTS (SELECT 1
                  FROM historico h
                           JOIN institucion i ON i.id_institucion = h.id_institucion
                           JOIN direccion d ON d.id_direccion = i.id_direccion
                  WHERE p.id_pais = d.id_pais)
ORDER BY p.nombre_pais;

--1.5)
SELECT t.*
FROM tarea t
WHERE EXISTS (SELECT 1
              FROM historico h
              WHERE h.id_tarea = t.id_tarea)
  AND NOT EXISTS (SELECT 1
                  FROM voluntario v
                  WHERE v.id_tarea = t.id_tarea);

--1.6)
SELECT t.id_tarea, t.nombre_tarea, t.max_horas
FROM tarea t
WHERE NOT EXISTS(SELECT v.id_tarea
                 FROM voluntario v
                 WHERE v.id_tarea = t.id_tarea
                 GROUP BY v.id_tarea
                 HAVING count(v.id_tarea) > 1)
  AND EXISTS (SELECT *
              FROM voluntario v
                       JOIN institucion ON v.id_institucion = institucion.id_institucion
                       JOIN direccion d ON institucion.id_direccion = d.id_direccion
              WHERE d.ciudad = 'Munich'
                AND t.id_tarea = v.id_tarea)
ORDER BY t.id_tarea;

--1.7)
SELECT *
FROM institucion i
         JOIN historico h on i.id_institucion = h.id_institucion
         JOIN voluntario v on V.id_tarea = h.id_tarea
WHERE i.id_director IS NOT NULL
  AND v.id_institucion <> i.id_institucion;

--1.8)
SELECT i.*, v.nombre, v.apellido
FROM institucion i
         JOIN voluntario v ON i.id_institucion = v.id_institucion;

--EJ3
--1)
SELECT v.id_institucion
FROM voluntario v
GROUP BY v.id_institucion
HAVING count(*) = 1
EXCEPT
SELECT h.id_institucion
FROM historico h;
--2)
SELECT v.nro_voluntario, v.nombre, v.apellido
FROM voluntario v
         LEFT JOIN institucion i ON i.id_institucion = v.id_institucion
         LEFT JOIN direccion d ON i.id_direccion = d.id_direccion
         LEFT JOIN pais p ON d.id_pais = p.id_pais
         LEFT JOIN continente c ON p.id_continente = c.id_continente
WHERE c.nombre_continente = 'Europeo'
   OR EXISTS(SELECT 1
             FROM voluntario v2
             WHERE v2.id_coordinador = v.nro_voluntario)
INTERSECT
SELECT v3.nro_voluntario, v3.nombre, v3.apellido
FROM voluntario v3
         JOIN institucion i ON v3.nro_voluntario = i.id_director
ORDER BY apellido, nombre, nro_voluntario;
--3)
SELECT v.nro_voluntario
FROM voluntario v
EXCEPT
SELECT v2.nro_voluntario
FROM voluntario v2
         LEFT JOIN public.institucion i on i.id_institucion = v2.id_institucion
         LEFT JOIN direccion d ON i.id_direccion = d.id_direccion
         LEFT JOIN pais p ON d.id_pais = p.id_pais
         JOIN continente c ON p.id_continente = c.id_continente
WHERE c.nombre_continente = 'Europeo';
--4)
SELECT v.nro_voluntario, v.nombre, v.apellido
FROM voluntario v

EXCEPT

SELECT v.nro_voluntario, v.nombre, v.apellido
FROM voluntario v
         LEFT JOIN historico h ON h.nro_voluntario = v.nro_voluntario
GROUP BY v.nro_voluntario, v.nombre, v.apellido
HAVING COUNT(DISTINCT h.id_institucion) < (SELECT COUNT(*)
                                           FROM institucion)
ORDER BY nombre, apellido, nro_voluntario;
--5)

SELECT DISTINCT v.id_tarea
FROM voluntario v
EXCEPT
SELECT v2.id_tarea
FROM voluntario v2
GROUP BY v2.id_tarea
HAVING count(DISTINCT v2.id_institucion) < (SELECT count(*) FROM institucion)
```

## 3. Películas y anotaciones sobre valores nulos

Conexión de referencia para las consultas de Películas: base `peliculas`, esquema `public`. El bloque final contiene apuntes y respuestas sobre NULL, incluidos fragmentos ilustrativos dentro de un comentario SQL.

Incluye el ejercicio 3 de la primera parte, consultas 1.9 a 1.15 de la segunda y las anotaciones de los ejercicios 2.1, 2.2 y 2.3.

```sql
-- TP2

--EJ3
--a)
SELECT e.apellido, e.nombre, e.id_empleado
FROM empleado e
WHERE e.porc_comision IS NULL
   OR e.porc_comision = 0
ORDER BY e.apellido;

--b)
SELECT *
FROM distribuidor d
         JOIN internacional i ON d.id_distribuidor = i.id_distribuidor
WHERE d.telefono IS NULL;

--c)
SELECT d.id_distribuidor, d.id_departamento, d.nombre_departamento
FROM departamento d
WHERE d.jefe_departamento IS NULL;

--TP2 Parte 2

--1.9)
SELECT e.id_empleado,
       e.nombre,
       e.apellido,
       j.id_empleado AS "id Jefe",
       j.nombre      AS "nombre Jefe",
       j.apellido    AS "apellido Jefe"
FROM empleado e
         LEFT JOIN empleado j ON e.id_jefe = j.id_empleado;

--1.10)
SELECT DISTINCT e.id_empleado, e.nombre, e.apellido
FROM empleado e
JOIN empleado e2 ON e2.id_jefe = e.id_empleado
JOIN departamento d ON e.id_departamento = d.id_departamento AND e.id_distribuidor = d.id_distribuidor
JOIN ciudad c ON c.id_ciudad = d.id_ciudad
WHERE c.nombre_ciudad = 'Rawalpindi'
ORDER BY e.id_empleado;

--1.11)
SELECT DISTINCT n.id_distribuidor, n.nro_inscripcion
FROM nacional n
JOIN entrega e ON n.id_distribuidor = e.id_distribuidor
JOIN renglon_entrega r ON e.nro_entrega = r.nro_entrega
JOIN pelicula p ON p.codigo_pelicula = r.codigo_pelicula
WHERE p.idioma = 'Español' AND extract(YEAR from e.fecha_entrega) > 2010;

--1.12)
    SELECT *
    FROM pelicula p
    WHERE NOT EXISTS (
        SELECT 1
        FROM nacional n
        JOIN entrega e ON n.id_distribuidor = e.id_distribuidor
        JOIN renglon_entrega r ON e.nro_entrega = r.nro_entrega
        WHERE r.codigo_pelicula = p.codigo_pelicula
    );
--1.13)
SELECT  e.apellido, e.nombre
FROM empleado e
JOIN departamento d ON e.id_departamento = d.id_departamento
JOIN ciudad c ON d.id_ciudad = c.id_ciudad
JOIN pais p ON c.id_pais = p.id_pais
WHERE p.nombre_pais = 'Argentina' AND EXISTS (
    SELECT 1
    FROM empleado j
    WHERE j.porc_comision > 0.4 AND j.id_empleado = d.jefe_departamento
);
--1.14)
SELECT d.nombre_departamento, d.id_departamento
FROM departamento d
WHERE EXISTS(
  SELECT e.id_departamento
  FROM empleado e
  WHERE e.id_departamento = d.id_departamento
  GROUP BY (e.id_departamento)
  HAVING count(*)>3
);
--1.15)
SELECT d.*
FROM departamento d
WHERE (
          SELECT COUNT(*)
          FROM empleado e
          WHERE e.id_departamento = d.id_departamento
            AND e.id_distribuidor = d.id_distribuidor
      ) < (
          SELECT COUNT(*) * 0.10
          FROM empleado
      );
/*
* NULL representa un valor desconocido o ausente. No equivale a cero ni a una cadena vacía ('').
* Las comparaciones con NULL dan UNKNOWN. Tanto NULL = NULL como NULL <> NULL dan desconocido. Para detectar nulos usá IS NULL o IS NOT NULL.
* SQL maneja tres valores lógicos: TRUE, FALSE y UNKNOWN. Negar UNKNOWN con NOT sigue dando UNKNOWN.
* WHERE conserva únicamente condiciones verdaderas. Descarta tanto FALSE como UNKNOWN. Lo mismo sucede con las coincidencias de un INNER JOIN.
* Diferenciá los conteos: COUNT(*) cuenta filas; COUNT(columna) cuenta únicamente valores no nulos.
* Las agregaciones ignoran los nulos. AVG, SUM, MIN y MAX devuelven NULL si no hay ningún valor no nulo para calcular.
* DISTINCT reúne los nulos en un único resultado. Eso no significa que la comparación NULL = NULL sea verdadera.
* Cuidado con NOT IN si la subconsulta devuelve NULL. Puede producir UNKNOWN y descartar filas que esperabas obtener.
* La concatenación con || propaga los nulos: 'Texto' || NULL devuelve NULL. Podés reemplazarlos con COALESCE(descripcion, '').
* Un JOIN devuelve todas las parejas que coinciden. Con cuatro filas cuyos puntos son iguales y no nulos, el autojoin genera 16 filas; si todos son nulos, genera cero.
* Importa dónde ponés COALESCE:
    COALESCE(AVG(puntos), 0)  -- reemplaza el resultado nulo
    AVG(COALESCE(puntos, 0))  -- toma cada punto nulo como cero
    Con puntos 10, NULL, la primera devuelve 10 y la segunda 5.

En un LEFT JOIN, una fila sin coincidencias también se conserva. Por eso, para contar voluntarios por institución, COUNT(*) da 1 para una institución sin voluntarios; COUNT(v.nro_voluntario) da 0.

------------------------------------------------------ 2.1 -------------------------------------------------------------
a)
CREATE TABLE Equipo (
  Id 		int NOT NULL,
  puntos		int,
  descripcion 	varchar(20),
  CONSTRAINT pk_equipo PRIMARY KEY (Id)
);
con los siguientes datos:
INSERT INTO Equipo(id, puntos) VALUES (1, null), (2, null), (3, null), (4, null);

Qué retornan las siguientes consultas?
SELECT avg(puntos), count(puntos), count(*) FROM Equipo;
1. AVG(puntos), COUNT(puntos), COUNT(*)
Devuelve una fila:
avg	    count(puntos)	count(*)
NULL    0               4

SELECT id, ‘Su descripción es ’|| descripcion FROM equipo WHERE puntos NOT IN (select distinct puntos from equipo);

No devuelve filas. El filtro evalúa NULL NOT IN (NULL), que da UNKNOWN, y WHERE solo acepta TRUE.
Sin ese filtro, devolvería los cuatro identificadores con la segunda columna en NULL, porque 'Su descripción es ' || NULL devuelve NULL.

SELECT * FROM equipo WHERE puntos NOT IN (select distinct puntos from equipo);

No devuelve filas, por el mismo motivo: la subconsulta devuelve una fila con NULL y NULL NOT IN (NULL) da UNKNOWN.

SELECT * FROM equipo e1 JOIN equipo e2 ON (e1.puntos = e2.puntos);

No devuelve filas. Todas las parejas se comparan mediante NULL = NULL, que da UNKNOWN; ninguna satisface el ON.

Modifique la consulta 1 para que devuelva los valores nulos como ceros o blancos.
SELECT COALESCE(AVG(puntos), 0 <o ''>) AS promedio,
       COUNT(puntos),
       COUNT(*)
FROM equipo;

------------------------------------------------------ 2.2 -------------------------------------------------------------
Devuelve los voluntarios que tienen porcentajes que NO estan entre 0.15 y 0,30 y los que tengan su porcentaje en null

------------------------------------------------------ 2.3 -------------------------------------------------------------
La primera va a formar un grupo de cada id con la cantidad de voluntarios de ese id, por lo que se va a mostrar va a ser
el id y la cantidad de filas que tiene pero al usar LEFT JOIN si un id no tiene ningun voluntario asociado, se va a
agregar una fila con el id y null en la cantidad de voluntarios por lo que se va a mostrar <id>, 1
La segunda al agrupar por la institucion de los voluntarios termina mostrando las instituciones que poseen los
voluntarios y junto a ella la cantidad de voluntarios que tienen esa institucion, en el caso de que haya voluntarios
sin institucion(null) se contaran y se mostraran como el par NULL | <cantidad de voluntarios sin institucion>
*/
```
