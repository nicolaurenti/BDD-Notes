---
tags:
  - tp3
---

# Resolución del TP3 — Restricciones de integridad

## Ejercicio 1 — Resolución corregida

Revisado contra el enunciado y el diagrama de `../../Practicos/Practico 3 - Restricciones de Integridad Declarativas.docx`.
### a) Incorporar las restricciones referenciales

| RIR | Hija → padre          | FK              | ON DELETE | ON UPDATE |
| --- | --------------------- | --------------- | --------- | --------- |
| R1  | Trabaja_En → Empleado | `(TipoE, NroE)` | CASCADE   | RESTRICT  |
| R2  | Trabaja_En → Proyecto | `(IdProy)`      | RESTRICT  | CASCADE   |
| R3  | Auspicio → Proyecto   | `(IdProy)`      | RESTRICT  | RESTRICT  |
| R4  | Auspicio → Empleado   | `(TipoE, NroE)` | SET NULL  | RESTRICT  |

```sql
-- R1
ALTER TABLE Trabaja_En
ADD CONSTRAINT fk_trabaja_en_empleado
FOREIGN KEY (TipoE, NroE)
REFERENCES Empleado (TipoE, NroE)
ON DELETE CASCADE
ON UPDATE RESTRICT;

-- R2
ALTER TABLE Trabaja_En
ADD CONSTRAINT fk_trabaja_en_proyecto
FOREIGN KEY (IdProy)
REFERENCES Proyecto (IdProy)
ON DELETE RESTRICT
ON UPDATE CASCADE;

-- R3
ALTER TABLE Auspicio
ADD CONSTRAINT fk_auspicio_proyecto
FOREIGN KEY (IdProy)
REFERENCES Proyecto (IdProy)
ON DELETE RESTRICT
ON UPDATE RESTRICT;

-- R4
ALTER TABLE Auspicio
ADD CONSTRAINT fk_auspicio_empleado
FOREIGN KEY (TipoE, NroE)
REFERENCES Empleado (TipoE, NroE)
ON DELETE SET NULL
ON UPDATE RESTRICT;
```

### b) Resultado de las operaciones

**Cada operación parte de los datos originales; no son acumulativas.**

| Tabla | Datos relevantes iniciales |
|---|---|
| Empleado | `('A', 1)`, `('B', 2)`, `('A', 2)` |
| Proyecto | `1`, `2`, `3` |
| Trabaja_En | `('A', 1, 1, …)`, `('A', 2, 2, …)` |
| Auspicio | `(2, …, 'A', 2)` |

#### b.1 — Borrar el proyecto 3

```sql
DELETE FROM Proyecto WHERE IdProy = 3;
```

**Aceptada.** No hay filas en Trabaja_En ni en Auspicio que referencien al proyecto 3. R2 y R3 tienen `ON DELETE RESTRICT`, pero no hay referencias que bloqueen la baja. Se elimina únicamente ese proyecto.

#### b.2 — Cambiar el proyecto 3 por 7

```sql
UPDATE Proyecto SET IdProy = 7 WHERE IdProy = 3;
```

**Aceptada.** Se cambia la clave del proyecto de 3 a 7. No hay referencias a 3 en las tablas hijas ni existe otro proyecto con clave 7. R2 no tiene filas a las que propagar el cambio y R3 no lo bloquea.

#### b.3 — Borrar el proyecto 1

```sql
DELETE FROM Proyecto WHERE IdProy = 1;
```

**Rechazada por R2.** La fila `('A', 1, 1, …)` de Trabaja_En referencia al proyecto 1 y R2 tiene `ON DELETE RESTRICT`. No se elimina ninguna fila.

#### b.4 — Borrar el empleado ('A', 2)

```sql
DELETE FROM Empleado WHERE TipoE = 'A' AND NroE = 2;
```

**Aceptada**, con estos efectos:

1. Se elimina el empleado `('A', 2)`.
2. **R1, ON DELETE CASCADE:** se elimina su asignación `('A', 2, 2, …)` de Trabaja_En.
3. **R4, ON DELETE SET NULL:** se conserva el auspicio del proyecto 2, pero su referencia al empleado pasa de `('A', 2)` a `(NULL, NULL)`.

El proyecto 2 sigue existiendo: borrar una fila hija de Trabaja_En no borra su padre Proyecto. El auspicio continúa referenciando a ese proyecto mediante R3.

#### b.5 — Cambiar de proyecto una asignación

```sql
UPDATE Trabaja_En SET IdProy = 3 WHERE IdProy = 1;
```

**Aceptada.** La asignación `('A', 1, 1, …)` pasa a `('A', 1, 3, …)`. El proyecto 3 existe, de modo que la nueva FK satisface R2. No se genera una PK duplicada con las filas dadas.

**Corrección:** R1 no bloquea este cambio. Se modifica `IdProy` en la hija Trabaja_En, mientras que R1 relaciona `(TipoE, NroE)` con Empleado. Su `ON UPDATE RESTRICT` se aplica al cambio de la clave referenciada del empleado cuando hay referencias; aquí esa clave no cambia.

Tampoco es la cascada de R2 la que permite esta operación: al cambiar directamente una FK hija, se comprueba que el nuevo padre exista.

#### b.6 — Cambiar el proyecto 2 por 5

```sql
UPDATE Proyecto SET IdProy = 5 WHERE IdProy = 2;
```

**Rechazada por R3.** El auspicio `(2, …, 'A', 2)` referencia al proyecto 2 y R3 tiene `ON UPDATE RESTRICT`.

R2 permitiría propagar el nuevo identificador a Trabaja_En mediante CASCADE, pero R3 impide completar la sentencia. No queda ningún cambio parcial: Proyecto, Trabaja_En y Auspicio conservan sus datos originales.

**Tu resultado era correcto, pero la causa era incorrecta:** el bloqueo corresponde a R3, no a R1. Cambiar `IdProy` no cambia la referencia al empleado `(TipoE, NroE)`.

### c) Comparación de SIMPLE, PARTIAL y FULL para R4

Se consideran los empleados originales: `('A', 1)`, `('B', 2)` y `('A', 2)`. R4 es **una sola FK compuesta** `(TipoE, NroE)`; son dos columnas de una misma referencia.

- **SIMPLE:** si alguna columna es NULL, la referencia pasa el control de R4. Sin NULL, exige la pareja completa en Empleado.
- **PARTIAL:** las columnas informadas deben coincidir con una misma fila de Empleado. Con ambas en NULL, acepta.
- **FULL:** acepta ambas columnas en NULL o ambas informadas y coincidentes con un empleado. Rechaza los nulos parciales.

Las siguientes sentencias explicitan las columnas y usan comillas SQL normales. Cada caso se evalúa sobre los datos originales.

```sql
-- c.1
INSERT INTO Auspicio (IdProy, NombreAuspiciante, TipoE, NroE)
VALUES (1, 'Dell', 'B', NULL);

-- c.2
INSERT INTO Auspicio (IdProy, NombreAuspiciante, TipoE, NroE)
VALUES (3, 'Oracle', NULL, NULL);

-- c.3
INSERT INTO Auspicio (IdProy, NombreAuspiciante, TipoE, NroE)
VALUES (5, 'Google', 'A', 3);

-- c.4
INSERT INTO Auspicio (IdProy, NombreAuspiciante, TipoE, NroE)
VALUES (1, 'HP', NULL, 3);
```

#### Resultado de R4 según el matching

| Caso | Referencia al empleado | SIMPLE | PARTIAL | FULL |
|---|---|---|---|---|
| c.1 | `('B', NULL)` | Acepta: hay un NULL. | Acepta: existe `('B', 2)`, que coincide en B. | Rechaza: solo una columna es NULL. |
| c.2 | `(NULL, NULL)` | Acepta. | Acepta. | Acepta: ambas son NULL. |
| c.3 | `('A', 3)` | Rechaza: no existe esa pareja. | Rechaza: sin NULL, exige la pareja completa. | Rechaza: no existe esa pareja. |
| c.4 | `(NULL, 3)` | Acepta: hay un NULL. | Rechaza: ningún empleado tiene NroE = 3. | Rechaza: solo una columna es NULL. |

#### Resultado de la inserción completa

| Caso | SIMPLE | PARTIAL | FULL | Otras restricciones |
|---|---|---|---|---|
| c.1 | Aceptada | Aceptada | Rechazada por R4 | El proyecto 1 existe. |
| c.2 | Aceptada | Aceptada | Aceptada | El proyecto 3 existe. |
| c.3 | Rechazada | Rechazada | Rechazada | Además de R4, incumple R3: el proyecto 5 no existe. |
| c.4 | Aceptada | Rechazada por R4 | Rechazada por R4 | El proyecto 1 existe. |

**Tus resultados de aceptación/rechazo en c eran correctos**, con estas precisiones en la explicación:

- En c.1, PARTIAL acepta porque **la parte informada B coincide** con un empleado. No omite la búsqueda por haber un NULL, como sí hace SIMPLE.
- En c.2, insertar `(NULL, NULL)` no crea una combinación nueva en Empleado: agrega un auspicio **sin empleado asociado**. No se necesita un empleado `(NULL, NULL)`.
- En c.3, además de no existir el empleado `('A', 3)`, tampoco existe el proyecto 5. El rechazo completo no depende solo del matching de R4.
- En c.4, el `3` es el número de empleado informado; no hay ningún empleado con ese número. FULL rechaza porque la referencia está parcialmente en NULL.

**Nota de implementación:** PARTIAL se analiza aquí como modalidad del estándar solicitada por el práctico. PostgreSQL no implementa `MATCH PARTIAL`; SIMPLE y FULL sí están soportados.

## Ejercicio 2 — Acciones referenciales y matching

### Relaciones del esquema

| RIR | Referencia | ON DELETE | ON UPDATE |
|---|---|---|---|
| R1 | Instalacion → Cliente | CASCADE | RESTRICT |
| R2 | Instalacion → Servicio | RESTRICT | RESTRICT |
| R3 | Referencia → Servicio | RESTRICT | CASCADE |
| R4 | Referencia → Cliente | RESTRICT | SET NULL |

### a) Operaciones acumulativas

Cada operación parte del resultado de la anterior, a diferencia del ejercicio 1.

#### a.1 — Borrar los clientes con NroC = 1

```sql
DELETE FROM Cliente WHERE NroC = 1;
```

**Aceptada.** Elimina dos clientes: `('A', 1)` y `('B', 1)`.

R1 propaga el borrado a sus instalaciones `('A', 1, 'S1', …)` y `('B', 1, 'S2', …)`. R4 no bloquea la operación porque las referencias existentes apuntan a `('C', 2)` y `('D', 3)`.

Los servicios permanecen: borrar instalaciones no borra sus servicios.

#### a.2 — Cambiar S2 por S5 en Instalacion

```sql
UPDATE Instalacion
SET IdServ = 'S5'
WHERE IdServ = 'S2';
```

**Se ejecuta sin error, pero modifica cero filas.** La instalación de S2 fue eliminada por la operación anterior, por lo que no se intenta guardar ninguna referencia a S5.

Sobre los datos originales, sería rechazada por R2 porque S5 no existe. Aquí los resultados son acumulativos.

#### a.3 — Cambiar la zona D por Z

```sql
UPDATE Cliente
SET Zona = 'Z'
WHERE Zona = 'D';
```

**Aceptada.** El cliente `('D', 3)` pasa a ser `('Z', 3)`.

No tiene instalaciones, por lo que R1 no impide modificar su clave. R4 tiene `ON UPDATE SET NULL`: las referencias a ese cliente quedan con **Zona y NroC en NULL**.

Al terminar quedan estas instalaciones, conservando los demás atributos:

| Zona | NroC | IdServ |
|---|---:|---|
| C | 2 | S1 |
| A | 2 | S3 |

Las referencias quedan:

| IdServ | Motivo | Zona | NroC |
|---|---|---|---:|
| S1 | Puntualidad | NULL | NULL |
| S2 | Calidad inst. | C | 2 |
| S3 | Costo | C | 2 |
| S1 | Atención | NULL | NULL |

### b) Ejemplos de matching

Se vuelve a los clientes originales: `(A,1)`, `(A,2)`, `(B,1)`, `(C,2)` y `(D,3)`.

Se puede usar esta forma de inserción:

```sql
INSERT INTO Referencia (IdServ, Motivo, Zona, NroC)
VALUES ('S1', 'Prueba', 'A', NULL);
```

Cada ejemplo se evalúa independientemente, sustituyendo las dos últimas columnas:

| Zona, NroC | SIMPLE | PARTIAL | FULL |
|---|---|---|---|
| `('A', 1)` | Acepta | Acepta | Acepta |
| `('A', 3)` | Rechaza | Rechaza | Rechaza |
| `('A', NULL)` | Acepta | Acepta: existen clientes A | Rechaza |
| `('Z', NULL)` | Acepta | Rechaza: no existe zona Z | Rechaza |
| `(NULL, 3)` | Acepta | Acepta: existe D,3 | Rechaza |
| `(NULL, 9)` | Acepta | Rechaza: no existe número 9 | Rechaza |
| `(NULL, NULL)` | Acepta | Acepta | Acepta |

S1 existe y el motivo propuesto no repite una clave existente. Se analiza cada modalidad por separado; no se declaran simultáneamente tres FK.

## Criterio para los ejercicios 3 a 5

Se distingue el SQL estándar solicitado por la cátedra de lo que admite PostgreSQL: este último no implementa `CREATE ASSERTION` ni subconsultas dentro de `CHECK`. Los CHECK locales de los ejercicios 3.a–c y 4.b–c sí pueden expresarse directamente en PostgreSQL.

Un CHECK rechaza FALSE, pero no UNKNOWN: si una información debe estar obligatoriamente presente, hay que exigirlo. En los casos donde el esquema admite NULL, se explicita el criterio elegido.

Referencias: [funciones del estándar no soportadas por PostgreSQL](https://www.postgresql.org/docs/current/unsupported-features-sql-standard.html) y [restricciones CHECK](https://www.postgresql.org/docs/current/ddl-constraints.html#DDL-CONSTRAINTS-CHECK-CONSTRAINTS).

Los fragmentos son de estudio: **no se ejecutó SQL**.

## Ejercicio 3 — Restricciones sobre artículos

### Clasificación

| Apartado | Tablas | Atributos relevantes | Tipo | Recurso |
|---|---|---|---|---|
| a | Articulo | nacionalidad | Atributo | CHECK |
| b | Articulo | fecha_pub | Atributo | CHECK |
| c | Articulo | fecha_pub, nacionalidad | Tupla | CHECK |
| d | Contiene | id_articulo, idioma, cod_palabra | Tabla | CHECK con subconsulta en el estándar |
| e | Articulo, Contiene | nacionalidad, id_articulo, idioma, cod_palabra | General | ASSERTION |

En Contiene, cada palabra se identifica por `(idioma, cod_palabra)`. Como su PK incluye también `id_articulo`, cada fila representa una asociación diferente: se puede contar con `COUNT(*)`.

### a) Nacionalidades permitidas

```sql
ALTER TABLE articulo
ADD CONSTRAINT ck_articulo_nacionalidad
CHECK (
    nacionalidad IN (
        'Argentina',
        'Español',
        'Inglesa',
        'Alemana',
        'Chilena'
    )
);
```

Sirve en PostgreSQL. Este CHECK permite NULL. Si además se exige conocer siempre la nacionalidad:

```sql
ALTER TABLE articulo
ALTER COLUMN nacionalidad SET NOT NULL;
```

### b) Publicaciones desde 2010

```sql
ALTER TABLE articulo
ADD CONSTRAINT ck_articulo_fecha
CHECK (fecha_pub >= DATE '2010-01-01');
```

Sirve en PostgreSQL. Incluye todo 2010 y los años posteriores. No prohíbe una fecha NULL por sí solo.

### c) Después de 2020, solamente argentinos

```sql
ALTER TABLE articulo
ADD CONSTRAINT ck_articulo_argentino_desde_2021
CHECK (
    fecha_pub < DATE '2021-01-01'
    OR (
        nacionalidad IS NOT NULL
        AND nacionalidad = 'Argentina'
    )
);
```

Sirve en PostgreSQL. Si es anterior a 2021, esta regla no exige nacionalidad argentina; si es desde 2021, debe tener nacionalidad argentina informada.

Se agrega `IS NOT NULL` para que una nacionalidad desconocida no deje pasar un artículo posterior a 2020. Una fecha NULL sigue sin quedar prohibida; exigirla sería otra condición.

### d) Máximo de 15 palabras clave por artículo

CHECK de tabla en SQL estándar:

```sql
ALTER TABLE contiene
ADD CONSTRAINT ck_max_15_palabras
CHECK (
    NOT EXISTS (
        SELECT id_articulo
        FROM contiene
        GROUP BY id_articulo
        HAVING COUNT(*) > 15
    )
);
```

Lectura: no puede existir un artículo con más de 15 asociaciones.

También puede expresarse con una aserción:

```sql
CREATE ASSERTION as_max_15_palabras
CHECK (
    NOT EXISTS (
        SELECT id_articulo
        FROM contiene
        GROUP BY id_articulo
        HAVING COUNT(*) > 15
    )
);
```

Son alternativas: no hace falta declarar ambas. Ninguna de estas dos formas está soportada por PostgreSQL.

### e) Artículos argentinos con hasta 10 palabras

**Ambigüedad del enunciado.** La lectura literal de «Sólo se pueden publicar artículos argentinos que contengan hasta 10 palabras claves» exige que todos los artículos publicados sean argentinos y tengan como máximo 10 palabras.

Como no hay un atributo que distinga borradores de publicaciones, se toma cada fila de Articulo como un artículo publicado:

```sql
CREATE ASSERTION as_publicaciones_argentinas_max_10
CHECK (
    NOT EXISTS (
        SELECT 1
        FROM articulo a
        WHERE a.nacionalidad IS NULL
           OR a.nacionalidad <> 'Argentina'
           OR (
               SELECT COUNT(*)
               FROM contiene c
               WHERE c.id_articulo = a.id_articulo
           ) > 10
    )
);
```

Con esta lectura, un artículo español con 5 palabras también se rechaza.

Si la intención es «los artículos argentinos pueden tener hasta 10 palabras», sin prohibir los de otras nacionalidades, la condición cambia:

```sql
CREATE ASSERTION as_argentinos_max_10
CHECK (
    NOT EXISTS (
        SELECT 1
        FROM articulo a
        WHERE a.nacionalidad = 'Argentina'
          AND (
              SELECT COUNT(*)
              FROM contiene c
              WHERE c.id_articulo = a.id_articulo
          ) > 10
    )
);
```

En la segunda interpretación, 3.d limita a todos a 15 y 3.e limita específicamente a los argentinos a 10. Son interpretaciones alternativas; dejar escrita la elegida.

## Ejercicio 4 — Restricciones sobre proveedores y productos

| Apartado | Tablas | Atributos relevantes | Tipo | Recurso |
|---|---|---|---|---|
| a | Provee | nro_prov, cod_producto | Tabla | CHECK con subconsulta |
| b | Sucursal | cod_suc | Atributo | CHECK |
| c | Producto | descripcion, presentacion | Tupla | CHECK |
| d | Provee, Proveedor, Sucursal | nro_prov, cod_suc, localidad | General | ASSERTION |

### a) Máximo de 20 productos por proveedor

```sql
ALTER TABLE provee
ADD CONSTRAINT ck_max_20_productos
CHECK (
    NOT EXISTS (
        SELECT nro_prov
        FROM provee
        GROUP BY nro_prov
        HAVING COUNT(DISTINCT cod_producto) > 20
    )
);
```

SQL estándar; no admitido por PostgreSQL. Se cuentan productos diferentes. En el esquema dado, la PK de Provee es `(cod_producto, nro_prov)`, por lo que `COUNT(*)` también produciría el mismo resultado por proveedor.

### b) Códigos que comiencen con S_

```sql
ALTER TABLE sucursal
ADD CONSTRAINT ck_prefijo_sucursal
CHECK (cod_suc LIKE 'S!_%' ESCAPE '!');
```

Sirve en PostgreSQL.

- `S_01`: aceptado.
- `S_CENTRO`: aceptado.
- `SA01`: rechazado.

El escape hace que `_` sea literal. Sin escape, `_` representa cualquier carácter. [Referencia de LIKE](https://www.postgresql.org/docs/current/functions-matching.html).

### c) Descripción y presentación no pueden ser ambas NULL

```sql
ALTER TABLE producto
ADD CONSTRAINT ck_datos_producto
CHECK (
    descripcion IS NOT NULL
    OR presentacion IS NOT NULL
);
```

Sirve en PostgreSQL.

| Descripción | Presentación | Resultado |
|---|---|---|
| Informada | NULL | Acepta |
| NULL | Informada | Acepta |
| Informada | Informada | Acepta |
| NULL | NULL | Rechaza |

### d) Proveedor y sucursal de la misma localidad

```sql
CREATE ASSERTION as_proveedor_sucursal_misma_localidad
CHECK (
    NOT EXISTS (
        SELECT 1
        FROM provee p
        JOIN proveedor pr
          ON pr.nro_prov = p.nro_prov
        JOIN sucursal s
          ON s.cod_suc = p.cod_suc
        WHERE pr.localidad IS NULL
           OR s.localidad IS NULL
           OR pr.localidad <> s.localidad
    )
);
```

Es general porque consulta tres tablas. Se elige exigir localidades conocidas para las asociaciones existentes: si alguna es NULL, no se puede garantizar que sean iguales.

El control también debe mantenerse si cambia la localidad del proveedor o de la sucursal, aunque no se modifique Provee. Se suponen las FK del esquema.

## Ejercicio 5 — Restricciones sobre voluntarios

| Apartado | Tablas | Atributos relevantes | Tipo | Recurso |
|---|---|---|---|---|
| a | Voluntario | fecha_nacimiento | Atributo, dependiente del tiempo | CHECK, con salvedad temporal |
| b | Voluntario | horas_aportadas, id_coordinador, nro_voluntario | Tabla | CHECK con subconsulta o ASSERTION |
| c | Voluntario, Tarea | id_tarea, horas_aportadas, min_horas, max_horas | General | ASSERTION |
| d | Voluntario | id_tarea, id_coordinador, nro_voluntario | Tabla | CHECK con subconsulta o ASSERTION |
| e | Historico, Voluntario | nro_voluntario, id_institucion, fechas | General, temporal | ASSERTION con una definición del cambio |

Los nombres se tomaron del esquema local de Voluntarios del TP2. Se suponen las FK existentes.

### a) No puede haber voluntarios de más de 70 años

Tomando la edad en años cumplidos, se admite a quien tiene 70 y se rechaza desde que cumple 71:

```sql
ALTER TABLE voluntario
ADD CONSTRAINT ck_edad_maxima
CHECK (
    fecha_nacimiento >
    CURRENT_DATE - INTERVAL '71' YEAR
);
```

La escritura habitual en PostgreSQL es:

```sql
ALTER TABLE voluntario
ADD CONSTRAINT ck_edad_maxima
CHECK (
    fecha_nacimiento >
    CURRENT_DATE - INTERVAL '71 years'
);
```

Son dos notaciones alternativas para el planteo, no dos restricciones a declarar juntas.

**Salvedad temporal:** PostgreSQL comprueba el CHECK al insertar o actualizar; no vuelve a revisar todas las filas cada día. Alguien podría cumplir 71 sin que se ejecute ninguna operación sobre su fila. Por eso este control al modificar datos no garantiza por sí solo una regla que deba cumplirse continuamente con el paso del tiempo.

Referencia: [comportamiento de los CHECK](https://www.postgresql.org/docs/current/ddl-constraints.html#DDL-CONSTRAINTS-CHECK-CONSTRAINTS).

### b) No aportar más horas que el coordinador

```sql
CREATE ASSERTION as_horas_no_superan_coordinador
CHECK (
    NOT EXISTS (
        SELECT 1
        FROM voluntario v
        JOIN voluntario c
          ON c.nro_voluntario = v.id_coordinador
        WHERE v.horas_aportadas IS NULL
           OR c.horas_aportadas IS NULL
           OR v.horas_aportadas > c.horas_aportadas
    )
);
```

Se elige exigir horas informadas cuando existe coordinador. Aunque hay dos alias, se lee dos veces una sola tabla: el alcance es de tabla.

- Voluntario: 100 horas; coordinador: 150 → acepta.
- Voluntario: 200; coordinador: 150 → rechaza.
- Sin coordinador → no hay comparación que hacer.

También hay que controlar si se reducen las horas del coordinador.

### c) Horas dentro de los límites de la tarea

```sql
CREATE ASSERTION as_horas_dentro_de_tarea
CHECK (
    NOT EXISTS (
        SELECT 1
        FROM voluntario v
        JOIN tarea t
          ON t.id_tarea = v.id_tarea
        WHERE v.horas_aportadas IS NULL
           OR t.min_horas IS NULL
           OR t.max_horas IS NULL
           OR v.horas_aportadas < t.min_horas
           OR v.horas_aportadas > t.max_horas
    )
);
```

Los límites están incluidos: si la tarea permite de 100 a 500, tanto 100 como 500 son válidos.

Se exigen horas y límites conocidos para cada asignación. Cambiar los límites de una tarea también puede generar una infracción.

### d) Misma tarea que el coordinador

```sql
CREATE ASSERTION as_misma_tarea_coordinador
CHECK (
    NOT EXISTS (
        SELECT 1
        FROM voluntario v
        JOIN voluntario c
          ON c.nro_voluntario = v.id_coordinador
        WHERE v.id_tarea <> c.id_tarea
    )
);
```

En el esquema, id_tarea es NOT NULL, por lo que no hace falta tratamiento de nulos para ese atributo.

- Ambos tienen `IT_PROG` → acepta.
- Uno tiene `IT_PROG` y el otro `SA_REP` → rechaza.
- Sin coordinador → la regla no exige coincidencia con nadie.

### e) Como máximo tres cambios de institución por año

Historico guarda períodos con tarea e institución. Dos períodos consecutivos pueden tener distinta tarea y la misma institución: **eso no es un cambio de institución**. No alcanza con contar filas del histórico.

Se adoptan estos supuestos:

- El histórico conserva todos los períodos terminados, ordenados por fecha_inicio, sin superposiciones.
- El período siguiente al último histórico es el actual, guardado en Voluntario.
- Se cuenta un cambio cuando ambas instituciones están informadas y son diferentes.
- Se considera año calendario y, para este planteo, fecha_fin registra la fecha efectiva del cambio.

```sql
CREATE ASSERTION as_max_3_cambios_institucion_anuales
CHECK (
    NOT EXISTS (
        SELECT cambios.nro_voluntario,
               EXTRACT(YEAR FROM cambios.fecha_cambio)
        FROM (
            -- Cambios entre períodos históricos consecutivos.
            SELECT h1.nro_voluntario,
                   h1.fecha_fin AS fecha_cambio
            FROM historico h1
            JOIN historico h2
              ON h2.nro_voluntario = h1.nro_voluntario
             AND h2.fecha_inicio > h1.fecha_inicio
            WHERE h1.id_institucion <> h2.id_institucion
              AND NOT EXISTS (
                  SELECT 1
                  FROM historico intermedio
                  WHERE intermedio.nro_voluntario =
                        h1.nro_voluntario
                    AND intermedio.fecha_inicio >
                        h1.fecha_inicio
                    AND intermedio.fecha_inicio <
                        h2.fecha_inicio
              )

            UNION ALL

            -- Cambio del último período histórico al actual.
            SELECT h.nro_voluntario,
                   h.fecha_fin AS fecha_cambio
            FROM historico h
            JOIN voluntario v
              ON v.nro_voluntario = h.nro_voluntario
            WHERE h.id_institucion <> v.id_institucion
              AND NOT EXISTS (
                  SELECT 1
                  FROM historico posterior
                  WHERE posterior.nro_voluntario =
                        h.nro_voluntario
                    AND posterior.fecha_inicio >
                        h.fecha_inicio
              )
        ) AS cambios
        GROUP BY cambios.nro_voluntario,
                 EXTRACT(YEAR FROM cambios.fecha_cambio)
        HAVING COUNT(*) > 3
    )
);
```

La consulta interior identifica transiciones reales de institución. La exterior prohíbe que un voluntario tenga más de tres en un año.

**La salvedad forma parte de la respuesta:** si fecha_fin indica el último día del período anterior y no la fecha del cambio, hay que ajustar la fecha usada para contar. La regla requiere que los cambios se registren efectivamente en el histórico: una aserción no genera ese historial automáticamente. Si hay datos históricos incompletos o instituciones desconocidas, no se puede garantizar el límite real con este planteo.

## Alcance de esta resolución

Quedan incorporados los ejercicios 1 al 5. Los ejercicios 2 al 5 reproducen la resolución desarrollada en el chat, con sus supuestos y las diferencias entre SQL estándar y PostgreSQL. No se ejecutaron las sentencias sobre las bases existentes.
