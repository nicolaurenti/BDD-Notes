---
tags:
  - tp3
---

# Resolución del TP3 — Restricciones de integridad

## Ejercicio 1 — Resolución corregida

Revisado contra el enunciado y el diagrama de `../Practicos/Practico 3 - Restricciones de Integridad Declarativas.docx`.

Se usan los nombres del diagrama: `Trabaja_En`, `TipoE`, `NroE` e `IdProy`. El texto de b.5 escribe `TrabajaEn`; aquí se unifica como `Trabaja_En`. Si las tablas reales tienen nombres con guiones bajos como `tipo_e`, adaptar los identificadores. Los ejemplos son de estudio: no se ejecutó SQL.

### a) Incorporar las restricciones referenciales

Las tablas y sus claves primarias ya existen. Cada FK se declara en la tabla hija, que contiene las columnas de referencia.

| RIR | Hija → padre | FK | ON DELETE | ON UPDATE |
|---|---|---|---|---|
| R1 | Trabaja_En → Empleado | `(TipoE, NroE)` | CASCADE | RESTRICT |
| R2 | Trabaja_En → Proyecto | `(IdProy)` | RESTRICT | CASCADE |
| R3 | Auspicio → Proyecto | `(IdProy)` | RESTRICT | RESTRICT |
| R4 | Auspicio → Empleado | `(TipoE, NroE)` | SET NULL | RESTRICT |

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

**Corrección de tu respuesta:** las relaciones y acciones de R1, R2 y R3 estaban bien. En R4 habías escrito `ALTER TABLE TRABAJA_EN`: corresponde **Auspicio**. Sus columnas `TipoE` y `NroE` admiten NULL según el diagrama.

Se omite `MATCH` en estas sentencias; en PostgreSQL el valor por defecto es SIMPLE. El apartado c compara por separado las tres modalidades para R4.

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

Tu respuesta era correcta.

#### b.2 — Cambiar el proyecto 3 por 7

```sql
UPDATE Proyecto SET IdProy = 7 WHERE IdProy = 3;
```

**Aceptada.** Se cambia la clave del proyecto de 3 a 7. No hay referencias a 3 en las tablas hijas ni existe otro proyecto con clave 7. R2 no tiene filas a las que propagar el cambio y R3 no lo bloquea.

Tu respuesta era correcta. Se parte otra vez de la instancia original: el proyecto 3 existe al comenzar este caso.

#### b.3 — Borrar el proyecto 1

```sql
DELETE FROM Proyecto WHERE IdProy = 1;
```

**Rechazada por R2.** La fila `('A', 1, 1, …)` de Trabaja_En referencia al proyecto 1 y R2 tiene `ON DELETE RESTRICT`. No se elimina ninguna fila.

Tu respuesta era correcta.

#### b.4 — Borrar el empleado ('A', 2)

```sql
DELETE FROM Empleado WHERE TipoE = 'A' AND NroE = 2;
```

**Aceptada**, con estos efectos:

1. Se elimina el empleado `('A', 2)`.
2. **R1, ON DELETE CASCADE:** se elimina su asignación `('A', 2, 2, …)` de Trabaja_En.
3. **R4, ON DELETE SET NULL:** se conserva el auspicio del proyecto 2, pero su referencia al empleado pasa de `('A', 2)` a `(NULL, NULL)`.

El proyecto 2 sigue existiendo: borrar una fila hija de Trabaja_En no borra su padre Proyecto. El auspicio continúa referenciando a ese proyecto mediante R3.

**Tu respuesta era incompleta:** habías identificado el SET NULL de Auspicio, pero faltaban la baja del empleado y la cascada en Trabaja_En. Se ponen en NULL **ambas columnas de R4**, no el `IdProy` de Auspicio.

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

## Pendientes

Los demás ejercicios del TP3 todavía no están incorporados a esta resolución.
