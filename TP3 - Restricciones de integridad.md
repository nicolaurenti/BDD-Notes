---
tags:
  - tp3
---

# Práctico 3 — Restricciones de integridad declarativas

Guía de los siete temas del TP3: qué significa cada regla, cómo se declara y qué detalles cambian el resultado. Los ejemplos son didácticos y no se ejecutaron sobre las bases existentes. Los fragmentos que se presentan como alternativas se estudian por separado.

## 1. Integridad referencial

Una clave foránea (FK) exige que una referencia de la tabla **hija** apunte a una clave existente de la tabla **padre**, salvo los nulos admitidos por el matching. Insertar una hija no crea al padre.

### FK simple y compuesta; CREATE TABLE y ALTER TABLE

```sql
CREATE TABLE libro (
    isbn VARCHAR(20) PRIMARY KEY
);

-- FK simple al crear la tabla hija.
CREATE TABLE ejemplar (
    isbn VARCHAR(20),
    nro_ejemplar INTEGER,
    PRIMARY KEY (isbn, nro_ejemplar),
    CONSTRAINT fk_ejemplar_libro
        FOREIGN KEY (isbn) REFERENCES libro (isbn)
        ON DELETE RESTRICT
        ON UPDATE CASCADE
);

-- Suponiendo que prestamo ya tiene estas columnas:
ALTER TABLE prestamo
ADD CONSTRAINT fk_prestamo_ejemplar
FOREIGN KEY (isbn, nro_ejemplar)
REFERENCES ejemplar (isbn, nro_ejemplar)
ON DELETE RESTRICT
ON UPDATE RESTRICT;
```

En una FK compuesta se busca **la combinación completa en una misma fila**. Si existen ejemplares `('L1', 1)` y `('L2', 2)`, una referencia a `('L1', 2)` es inválida. Dos FK independientes no reemplazan una compuesta.

**Puntual:** la FK se declara en la hija; las columnas se corresponden por posición y sus tipos deben ser compatibles. La clave referenciada debe tener la garantía de unicidad adecuada, habitualmente PK o UNIQUE. Los datos existentes deben cumplir la restricción cuando se agrega.

### Acciones ante bajas y modificaciones

Las acciones `ON DELETE` y `ON UPDATE` responden al borrado o cambio de la **clave del padre**.

| Acción | ON DELETE | ON UPDATE |
|---|---|---|
| RESTRICT | Impide borrar al padre si tiene referencias. | Impide cambiar su clave si tiene referencias afectadas. |
| CASCADE | Borra las filas hijas que lo referencian. | Copia la nueva clave a las referencias hijas. |
| SET NULL | Conserva las hijas y pone la FK en NULL. | Conserva las hijas y pone la FK en NULL. |

En la notación `[baja, modificación a derecha]`, `[C, R]` se traduce así:

```sql
-- Fragmento dentro de la declaración de una FK:
ON DELETE CASCADE
ON UPDATE RESTRICT
```

**Puntual:** SET NULL requiere que las columnas afectadas admitan NULL; no puede dejar una PK en NULL. CASCADE al actualizar no permite asignar directamente a la hija una referencia a un padre inexistente.

### Propagación y operaciones independientes o acumulativas

La propagación va del padre a sus hijas. Borrar un ejemplar no intenta borrar su libro. Si una fila recibe una cascada y otra restricción bloquea el resultado, falla la sentencia completa, sin cambios parciales de esa sentencia.

```sql
-- Ejemplo en papel: L1 tiene un ejemplar y no tiene préstamos.
DELETE FROM ejemplar WHERE isbn = 'L1';
DELETE FROM libro WHERE isbn = 'L1';
```

- **Acumulativas:** la primera sentencia elimina el ejemplar; la segunda puede borrar el libro, que ya no tiene esa referencia.
- **Independientes:** cada sentencia parte del estado original; la segunda se rechaza por RESTRICT porque el ejemplar sigue existiendo en su caso de partida.

Una sentencia que selecciona cero filas no modifica nada ni crea una referencia inválida. El TP3 usa operaciones independientes en el ejercicio 1 y acumulativas en el 2.

Ampliación: [[MATCH SIMPLE, FULL y PARTIAL|Integridad referencial, declaraciones y dirección de las acciones]].

## 2. MATCH SIMPLE, PARTIAL y FULL

El matching decide qué hacer cuando una FK compuesta contiene NULL. Se selecciona **una modalidad** para esa FK:

```sql
ALTER TABLE prestamo
ADD CONSTRAINT fk_prestamo_ejemplar_full
FOREIGN KEY (isbn, nro_ejemplar)
REFERENCES ejemplar (isbn, nro_ejemplar)
MATCH FULL
ON DELETE RESTRICT
ON UPDATE RESTRICT;
```

Este código es una alternativa a la FK de prestamo del punto anterior; no se agregan ambas para comparar modalidades. Cambiar `MATCH FULL` por `MATCH SIMPLE` o, en SQL estándar, `MATCH PARTIAL`.

| Modalidad | Cómo decide |
|---|---|
| SIMPLE | Si hay algún NULL, no exige coincidencia con el padre. Sin NULL, exige la combinación completa. |
| PARTIAL | Las partes informadas deben coincidir en una misma fila padre. Si todas son NULL, acepta. |
| FULL | Acepta todas NULL o todas informadas y coincidentes. Rechaza los nulos parciales. |

Si los únicos ejemplares son `('L1', 1)` y `('L2', 2)`:

| FK propuesta | SIMPLE | PARTIAL | FULL |
|---|---|---|---|
| `('L1', 1)` | Acepta | Acepta | Acepta |
| `('L1', 2)` | Rechaza | Rechaza | Rechaza |
| `('L1', NULL)` | Acepta | Acepta: existe L1 | Rechaza |
| `('L9', NULL)` | Acepta | Rechaza: no existe L9 | Rechaza |
| `(NULL, 2)` | Acepta | Acepta: existe un ejemplar número 2 | Rechaza |
| `(NULL, NULL)` | Acepta | Acepta | Acepta |

**Puntual:** aceptar `(NULL, NULL)` significa admitir una hija sin padre asociado; no crea una fila padre con nulos. El resultado de la FK no garantiza por sí solo que se acepte toda la inserción: también deben cumplirse PK, NOT NULL, otras FK y CHECK. PostgreSQL usa SIMPLE por defecto y admite FULL; no implementa PARTIAL.

## 3. Clasificación y recursos declarativos

La clasificación depende de **qué datos necesita comparar la regla**, no de dónde se escribe la cláusula.

| Alcance | Datos consultados | Ejemplo | Recurso apropiado |
|---|---|---|---|
| Atributo o dominio | Un valor | Precio positivo | CHECK, DOMAIN o NOT NULL según la regla |
| Tupla | Columnas de una fila | Inicio anterior o igual al fin | CHECK |
| Tabla | Varias filas de una tabla | Email no repetido; máximo por grupo | UNIQUE; CHECK con subconsulta en SQL-1999 para agregados |
| General | Varias tablas | Horas dentro de los límites de su tarea | ASSERTION en SQL-1999 |

```sql
-- Atributo: dominio reutilizable; VALUE es el valor a comprobar.
CREATE DOMAIN horas_no_negativas AS INTEGER
CHECK (VALUE >= 0);

-- Tupla: relaciona dos columnas de la misma fila.
ALTER TABLE periodo
ADD CONSTRAINT ck_orden_fechas
CHECK (fecha_inicio <= fecha_fin);

-- Tabla: impide repetir un valor en distintas filas.
ALTER TABLE usuario
ADD CONSTRAINT uq_usuario_email UNIQUE (email);

-- Obligatoriedad: se declara aparte de la unicidad.
ALTER TABLE usuario
ALTER COLUMN email SET NOT NULL;
```

**Puntual:** PK identifica una fila, exige unicidad y ausencia de NULL. UNIQUE por sí solo no exige un valor informado; PostgreSQL permite por defecto varios NULL. Comparar voluntario y coordinador usa dos filas de la **misma tabla**, por lo que el alcance es de tabla. Una ASSERTION también puede expresar una regla de tabla: su uso no cambia el alcance.

Ampliación: [[Alcance y clasificación de restricciones|Fundamentos, clasificación y recursos específicos]].

## 4. CHECK: valores, fechas, patrones, implicaciones y NULL

CHECK expresa una condición que debe cumplir el dato. En PostgreSQL controla la fila insertada o actualizada; no sirve para consultar otras filas.

### Valores permitidos, fechas y patrones

```mysql
ALTER TABLE articulo
ADD CONSTRAINT ck_nacionalidad
CHECK (nacionalidad IN (
    'Argentina', 'Español', 'Inglesa', 'Alemana', 'Chilena'
));

ALTER TABLE articulo
ADD CONSTRAINT ck_fecha_publicacion
CHECK (fecha_pub >= DATE '2010-01-01');

ALTER TABLE sucursal
ADD CONSTRAINT ck_prefijo_sucursal
CHECK (cod_suc LIKE 'S!_%' ESCAPE '!');
```

**Puntual:** desde 2010 incluye el 1 de enero de 2010; después de 2020 comienza el 1 de enero de 2021. En LIKE, `%` es cualquier secuencia y `_` es un carácter cualquiera: el escape convierte `_` en literal. `S_01` pasa el ejemplo; `SA01` no.

### Condiciones entre columnas e implicaciones

```sql
-- Al menos una de las dos columnas debe estar informada.
ALTER TABLE producto
ADD CONSTRAINT ck_datos_producto
CHECK (descripcion IS NOT NULL OR presentacion IS NOT NULL);

-- Si se publicó después de 2020, debe ser argentino.
ALTER TABLE articulo
ADD CONSTRAINT ck_argentino_desde_2021
CHECK (
    fecha_pub < DATE '2021-01-01'
    OR (nacionalidad IS NOT NULL AND nacionalidad = 'Argentina')
);
```

Una implicación «si A, entonces B» se construye como `NOT A OR B`. No se escribe `A AND B`: eso exigiría que todas las filas cumplieran también A. Un artículo de 2019 puede ser español; uno de 2022 debe ser argentino.

**Puntual:** `OR` permite que descripción y presentación estén ambas informadas; solo rechaza ambas NULL. Una cadena vacía no es NULL. El control de nacionalidad posterior a 2020 exige explícitamente ese dato; no obliga por sí solo a informar fecha_pub.

### Lógica de tres valores

| Resultado del CHECK | ¿Se acepta por ese control? |
|---|---|
| TRUE | Sí |
| FALSE | No |
| UNKNOWN, por NULL | Sí |

Por ejemplo, `CHECK (precio > 0)` no rechaza precio NULL. Para exigir un precio conocido, agregar `NOT NULL`. En cambio, WHERE conserva únicamente filas cuyo predicado da TRUE: una consulta que busca infractores con `precio <= 0` tampoco encuentra los NULL; deben buscarse con `IS NULL` si están prohibidos.

Ampliación: [[CHECK y lógica de tres valores|CHECK, implicaciones y construcción de condiciones]].

## 5. Subconsultas, conteos y ASSERTION

Cuando la regla requiere otras filas, se puede construir una consulta que encuentre infractores y exigir que **no exista ninguno**: `CHECK (NOT EXISTS (...))`.

### Máximos por grupo: CHECK de tabla en SQL-1999

```sql
-- SQL-1999 según el planteo de cátedra; no admitido en PostgreSQL.
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

Con 15 asociaciones se acepta; con 16 se rechaza. Se cuenta cada asociación artículo–palabra, que es única por la PK del esquema. Para contar productos diferentes, usar `COUNT(DISTINCT cod_producto)` si una misma asociación puede aparecer varias veces en el modelo considerado.

**Puntual:** agrupar por la entidad que tiene el límite, no contar todas las filas de la base. La infracción de «como máximo 15» es `> 15`, no `>= 15`. Un conteo correlacionado o agrupar directamente la tabla de asociaciones evita que un LEFT JOIN cuente una fila artificial para una entidad sin asociaciones.

### Comparar filas de una misma tabla

```sql
-- SQL estándar: el coordinador es otra fila de voluntario.
CREATE ASSERTION as_horas_coordinador
CHECK (
    NOT EXISTS (
        SELECT 1
        FROM voluntario v
        JOIN voluntario c
          ON c.nro_voluntario = v.id_coordinador
        WHERE v.horas_aportadas > c.horas_aportadas
    )
);
```

Esta versión supone horas conocidas para hacer la comparación; si deben prohibirse las desconocidas, agregar `IS NULL` en la búsqueda de infractores. Un voluntario sin coordinador queda fuera de la comparación. Se supone la FK existente que garantiza el coordinador cuando está informado.

### Comparar varias tablas

```sql
-- SQL estándar: regla general entre voluntario y tarea.
CREATE ASSERTION as_horas_en_rango
CHECK (
    NOT EXISTS (
        SELECT 1
        FROM voluntario v
        JOIN tarea t ON t.id_tarea = v.id_tarea
        WHERE v.horas_aportadas IS NULL
           OR t.min_horas IS NULL
           OR t.max_horas IS NULL
           OR v.horas_aportadas < t.min_horas
           OR v.horas_aportadas > t.max_horas
    )
);
```

Esta versión exige horas y límites informados para cada tarea asignada; los extremos del rango son válidos. Se suponen las FK del modelo.

**Puntual:** comprobar también modificaciones del coordinador o de los límites de la tarea: pueden invalidar otras filas aunque no se toque al voluntario. Para reglas como «solo puede proveer a sucursales de su localidad», hay que consultar las tablas que contienen ambas localidades y la asociación que las conecta. Una subconsulta sin el enlace correcto puede comparar filas que no tienen relación.

## 6. Restricciones temporales: edad e historial

Una regla temporal depende de una fecha o de acontecimientos pasados. Antes del SQL, definir el instante de evaluación, qué se cuenta y qué intervalo significa «un año».

### Edad

```sql
-- PostgreSQL: edad máxima de 70 años cumplidos al validar la fila.
ALTER TABLE voluntario
ADD CONSTRAINT ck_edad_maxima
CHECK (fecha_nacimiento > CURRENT_DATE - INTERVAL '71 years');
```

Se admite hasta el día anterior al cumpleaños 71, tomando la edad en años cumplidos. Si se usa otro criterio de edad, hay que explicitarlo. PostgreSQL no revalida diariamente un CHECK: una fila puede envejecer sin escrituras, por lo que esta declaración no mantiene por sí sola una condición permanente sobre la edad.

### Cambios de institución por año

```sql
-- Modelo didáctico: cambio_institucion no es una tabla del TP2.
-- Se supone una fila por cambio real, con fecha y voluntario conocidos.
-- SQL estándar: rechaza un cuarto cambio dentro del mismo año calendario.
CREATE ASSERTION as_max_3_cambios
CHECK (
    NOT EXISTS (
        SELECT nro_voluntario, EXTRACT(YEAR FROM fecha_cambio)
        FROM cambio_institucion
        GROUP BY nro_voluntario, EXTRACT(YEAR FROM fecha_cambio)
        HAVING COUNT(*) > 3
    )
);
```

El código muestra el conteo una vez identificados los cambios. En el TP2 existe **Historico**, que también registra tareas: primero hay que comparar instituciones de períodos consecutivos y considerar el paso del último período al estado actual, según cómo se registre la historia. No sustituir automáticamente cambio_institucion por Historico.

**Puntual:** cuatro cambios pueden alternar entre solo dos instituciones; `COUNT(DISTINCT id_institucion)` no cuenta cambios. Un cambio de tarea dentro de la misma institución tampoco cuenta. Año calendario y últimos 12 meses son reglas diferentes. Una restricción no genera por sí sola el historial que necesita consultar.

## 7. SQL-1999 frente a PostgreSQL

Responder por separado **cómo se expresa la regla en el estándar de la cátedra** y **si PostgreSQL admite esa declaración**.

| Recurso | SQL-1999 estudiado | PostgreSQL |
|---|---|---|
| PK, UNIQUE, NOT NULL, DOMAIN | Disponibles | Disponibles |
| FK simples y compuestas | Disponibles | Disponibles |
| RESTRICT, CASCADE, SET NULL | Disponibles | Disponibles |
| MATCH SIMPLE / FULL | Disponibles | Disponibles |
| MATCH PARTIAL | Disponible | No implementado |
| CHECK local sobre una fila | Disponible | Disponible |
| CHECK de tabla con subconsulta | Planteo declarativo del estándar | No admitido |
| CREATE ASSERTION | Disponible | No implementado |

**Puntual:** escribir un CHECK al final del CREATE TABLE no habilita a consultar otras filas en PostgreSQL. Que una regla no pueda expresarse con estas declaraciones en PostgreSQL no significa que sea inválida en la resolución SQL-1999. No presentar una ASSERTION como SQL ejecutable en PostgreSQL.

Referencia de implementación: [funciones del estándar no soportadas](https://www.postgresql.org/docs/current/unsupported-features-sql-standard.html), [restricciones y acciones referenciales](https://www.postgresql.org/docs/current/ddl-constraints.html) y [patrones LIKE](https://www.postgresql.org/docs/current/functions-matching.html). Las fuentes de cátedra figuran al final de esta nota.

## Método para resolver y justificar

1. Identificar las filas afectadas y las columnas modificadas.
2. Distinguir si se cambia al padre o directamente una referencia hija.
3. Seguir todas las acciones y comprobar las demás restricciones.
4. Conservar el resultado anterior solo si las operaciones son acumulativas.
5. Para reglas nuevas, identificar alcance, escribir la condición o buscar infractores y elegir el recurso SQL correspondiente.

Ampliación: [[Análisis de operaciones y restricciones|Análisis de operaciones, casos acumulativos y controles temporales]].

## Resolución del práctico

[[Resolución del TP3 - Restricciones de integridad|Respuestas y consultas del TP3]]

Resolución de los ejercicios 1 al 5, con justificaciones y supuestos.

## Checklist del práctico

- [ ] Identificar alcance y recurso declarativo adecuado.
- [ ] Escribir FK compuestas y distinguir padre de hijo.
- [ ] Simular cascadas completas sin efectos parciales.
- [ ] Resolver SIMPLE/FULL/PARTIAL con nulos.
- [ ] Diferenciar operaciones independientes y acumulativas.
- [ ] Traducir implicaciones y negación de infractores.
- [ ] Explicar por qué CHECK acepta UNKNOWN.
- [ ] Separar SQL estándar de lo implementado por PostgreSQL.

## Conocimientos anteriores

- Relacionados: TP1 - Derivación y definición de tablas y contenidos de NULL y consultas del TP2.

## Criterio de estudio y fuentes

Las explicaciones se apoyan en este práctico y los anteriores. Se conserva la sintaxis de la cátedra; las diferencias de PostgreSQL se tratan por separado cuando las pide la consigna. Los ejemplos de teoría son didácticos y no sustituyen la resolución personal.

- Enunciado local: `../Practicos/Practico 3 - Restricciones de Integridad Declarativas.docx`.
- Filminas cotejadas: `BDI_03.P1 (Restricciones-1).pdf; BDI_03.P1 (Restricciones-2).pdf`, en `../Contenido extra/Filminas y Practicos de cursada/2023/`.
