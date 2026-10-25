---
tags:
  - tp5
---

# Práctico 5 — Vistas

Una vista deriva información de tablas u otras vistas. El práctico pide definirla, justificar si admite actualizaciones y seguir la propagación hacia la base. Se separa el criterio SQL-1999 de las filminas del comportamiento PostgreSQL cuando la consigna pide ambos.

## 1. [[Definición y uso de vistas]]

### ¿Qué es una vista?

Una **vista es una consulta guardada con un nombre**. Después podemos usar ese nombre en el `FROM` de otra consulta, como si fuera una tabla.

Una vista común guarda la definición de la consulta, no una copia independiente de las filas. Sus datos se obtienen de las **tablas base**, es decir, las tablas de las que sale la información.

Por ejemplo, si tenemos una tabla `envio`:

| id_proveedor | id_articulo | cantidad |
| --- | --- | --- |
| 1 | 10 | 300 |
| 2 | 20 | 700 |
| 3 | 30 | 1200 |

Podemos definir una vista para consultar solamente los envíos de al menos 500 unidades:

```sql
CREATE VIEW envios500 AS
SELECT id_proveedor, id_articulo, cantidad
FROM envio
WHERE cantidad >= 500;
```

`CREATE VIEW` define la vista. Para consultar sus filas usamos:

```sql
SELECT *
FROM envios500;
```

El resultado contiene los envíos de 700 y 1200 unidades. El envío de 300 sigue existiendo en `envio`, pero no cumple la condición de la vista.

Si en la tabla base la cantidad 700 cambia a 800, la próxima consulta de la vista muestra 800. No tenemos que volver a crear la vista para reflejar ese cambio. Si cambia a 400, esa fila deja de aparecer en la vista porque ya no cumple su condición.

### ¿Para qué sirve?

- **Simplificar consultas:** guardar un SELECT y reutilizarlo por su nombre.
- **Presentar los datos necesarios:** elegir columnas y filas, o combinar datos de varias tablas.
- **Ofrecer una interfaz de acceso:** permitir consultar una vista con los permisos correspondientes, sin tener que dar acceso directo a todas las tablas base.

### Vistas simples, anidadas y con joins

Una vista simple puede seleccionar columnas y filtrar filas de una tabla, como `envios500`. Una vista anidada se define sobre otra vista:

```sql
CREATE VIEW envios500_m AS
SELECT *
FROM envios500
WHERE cantidad < 1000;
```

Una fila aparece en `envios500_m` si cumple **las dos condiciones**: `cantidad >= 500` y `cantidad < 1000`. Con los datos originales, aparece solamente el envío de 700 unidades.

Una vista también puede definirse con un `JOIN`, por ejemplo para mostrar cada envío junto con el nombre de su proveedor. La información sigue saliendo de las tablas base; la vista permite consultar esa combinación por un nombre.

**Idea clave:** la vista común ofrece una forma de ver los datos existentes. Crear la vista no duplica las filas ni elimina las que quedan fuera del filtro. Que podamos consultarla como una tabla no implica que podamos modificarla de cualquier manera; eso se analiza en el punto 2.

### Dudas de la conversación

**Si agregamos a `envio` un envío de 900 unidades, ¿hay que volver a crear la vista?**

No. Como el nuevo envío cumple `cantidad >= 500`, aparece la próxima vez que consultamos `envios500`, sin volver a crearla. La definición de la vista sigue siendo la misma; lo que cambió son los datos de la tabla base. Crear la vista no fija sus resultados en el momento de creación.

## 2. [[Actualización de vistas y restricciones base]]

### ¿Qué significa actualizar una vista?

En este tema, actualizar una vista significa realizar un `INSERT`, `UPDATE` o `DELETE` a través de ella. No significa refrescar su resultado ni cambiar su definición.

Como la vista común no almacena sus propias filas, una escritura aceptada debe traducirse en una modificación de sus tablas base. Por ejemplo, para la vista simple `envios500` del punto 1:

```sql
UPDATE envios500
SET cantidad = 800
WHERE id_proveedor = 2 AND id_articulo = 20;
```

Suponiendo que esos identificadores identifican un único envío, que inicialmente tiene cantidad 700 y que las restricciones permiten el cambio, se modifica esa fila en `envio`. La próxima consulta tanto de la tabla como de la vista muestra cantidad 800. No se modifica una copia independiente.

De la misma manera, un `INSERT` aceptado agrega una fila a la tabla base y un `DELETE` aceptado elimina la fila base correspondiente, con los efectos que determinen sus restricciones y triggers. Borrar una fila a través de una vista no consiste solamente en ocultarla de esa vista.

### Dos preguntas diferentes

1. **¿La vista admite esa modificación?** Hay que poder determinar sobre qué filas y columnas de las tablas base actuar y que el sistema admita esa traducción.
2. **¿La operación concreta es válida?** Aunque la vista sea actualizable, deben respetarse las restricciones de la tabla base: PK, UNIQUE, FK, NOT NULL y CHECK, entre otras.

Por ejemplo, supongamos una tabla `empleado` con `id_empleado` como PK, `nombre` e `id_departamento`. El departamento es obligatorio (`NOT NULL`) y no tiene `DEFAULT`. Definimos:

```sql
CREATE VIEW empleados_d20 AS
SELECT id_empleado, nombre
FROM empleado
WHERE id_departamento = 20;
```

Cambiar el nombre de un empleado visible puede ser válido. Sin embargo, este intento de inserción no aporta el departamento:

```sql
INSERT INTO empleados_d20 (id_empleado, nombre)
VALUES (101, 'Ana');
```

Suponiendo que no hay mecanismos adicionales que completen el dato, se rechaza por el `NOT NULL` de `id_departamento`. **El filtro `WHERE id_departamento = 20` no asigna automáticamente 20 a la columna omitida.** El filtro selecciona filas; no es un valor por defecto.

### Duda: si inserto una fila válida en la vista, ¿se agrega a la tabla base?

Sí. Si la vista admite el INSERT y la fila cumple las restricciones, insertar a través de `empleados_d20` incorpora la fila en `empleado`. No se crea una fila independiente guardada en la vista.

En la definición anterior, la vista omite `id_departamento`, por lo que el INSERT mostrado no era válido bajo los supuestos indicados. Si la vista incluyera esa columna, podríamos proporcionarla explícitamente:

```sql
CREATE VIEW empleados_d20_completa AS
SELECT id_empleado, nombre, id_departamento
FROM empleado
WHERE id_departamento = 20;

INSERT INTO empleados_d20_completa
    (id_empleado, nombre, id_departamento)
VALUES (101, 'Ana', 20);
```

Suponiendo que el identificador 101 está libre, el departamento 20 existe y se cumplen las demás restricciones, la fila se agrega a `empleado`. Como también cumple el filtro, se verá al consultar tanto `empleados_d20_completa` como `empleados_d20`. No hay que insertarla por separado en la tabla.

### Duda: si inserto una fila que no cumple el filtro de la vista, ¿se agrega igual a la tabla?

Sí, si la vista admite el INSERT, los datos cumplen las restricciones base y no hay un control adicional que lo impida. **El WHERE por sí solo no exige que la fila insertada quede visible en la vista.**

Usando `empleados_d20_completa` del ejemplo anterior, que incluye `id_departamento` y no tiene `WITH CHECK OPTION`:

```sql
INSERT INTO empleados_d20_completa
    (id_empleado, nombre, id_departamento)
VALUES (102, 'Luis', 30);
```

Si el identificador está libre, el departamento 30 existe y se cumplen las demás restricciones, Luis se agrega a `empleado`. Sin embargo, no aparece al consultar la vista, porque esta muestra solamente empleados del departamento 20. El INSERT se acepta aunque la fila resultante quede fuera del resultado de la vista.

Para exigir que las filas insertadas o modificadas a través de esta vista cumplan su condición, se agrega `WITH CHECK OPTION` a su definición. En ese caso, el INSERT con departamento 30 se rechaza y no incorpora esa fila a la tabla. Es el tema del punto 3.

### ¿Toda vista es actualizable?

No. Poder obtener un resultado no garantiza que exista una traducción admitida para modificar sus datos. Si una vista muestra `SUM(cantidad)`, cambiar ese total no indica cómo repartir el cambio entre los envíos que lo componen. En un join también hay que analizar qué tabla y qué filas recibirían la modificación.

En el práctico se distinguen los criterios **SQL-1999 presentados por la cátedra** y las reglas de **actualización automática de PostgreSQL**. Para vistas simples, la cátedra analiza conservación de la clave y ausencia de DISTINCT, agregados, información derivada y subconsultas en la lista SELECT. PostgreSQL puede admitir una vista simple aunque no proyecte la PK completa. Los joins requieren un análisis separado; no alcanza con aplicarles las reglas de una vista simple. El desarrollo comparativo está dentro de la nota de este tema.

**Idea clave:** que una vista sea actualizable no significa que cualquier escritura sea válida. Primero se analiza la vista y la operación que admite; después, los datos concretos y las restricciones base. Las condiciones adicionales de `WITH CHECK OPTION` se desarrollan en el punto 3.

## 3. [[WITH CHECK OPTION, LOCAL, CASCADED y migración de filas]]

### Qué controla WITH CHECK OPTION

En una vista simple sobre una tabla, `WITH CHECK OPTION` exige que las filas insertadas o modificadas a través de la vista cumplan su condición. Retomamos empleados del departamento 20:

```sql
CREATE VIEW empleados_d20_controlados AS
SELECT id_empleado, nombre, id_departamento
FROM empleado
WHERE id_departamento = 20
WITH CHECK OPTION;
```

Suponiendo válidas las restricciones de la tabla base:

| Operación a través de la vista | Sin CHECK OPTION | Con CHECK OPTION |
| --- | --- | --- |
| Insertar un empleado del departamento 20 | Se inserta y queda visible | Se inserta y queda visible |
| Insertar un empleado del departamento 30 | Se inserta en la base, pero queda fuera de la vista | Se rechaza |
| Cambiar un empleado visible del departamento 20 al 30 | Se modifica en la base y deja de verse | Se rechaza el cambio |
| Cambiar solo el nombre de un empleado visible | Se permite | Se permite: sigue cumpliendo el filtro |

### Migración de filas

Se llama migración de filas al cambio por el que una fila deja de pertenecer al resultado de la vista. No se traslada a otra tabla ni necesariamente se elimina: puede seguir existiendo en la misma tabla base.

Por ejemplo, Ana pertenece al departamento 20. Si a través de una vista sin chequeo cambiamos su departamento a 30, se modifica `empleado` y Ana deja de aparecer en la vista. Con el chequeo de la definición anterior, ese cambio se rechaza y Ana conserva el departamento 20.

### Primero se seleccionan las filas, después se comprueba el resultado

Un UPDATE sobre la vista solo puede alcanzar las filas actualmente visibles. Si Luis ya pertenece al departamento 30, no podemos seleccionarlo a través de esta vista para pasarlo al 20: no aparece entre las filas sobre las que actúa el UPDATE. Esa sentencia no modifica a Luis, aunque el valor propuesto sí cumpla el filtro.

En cambio, si la fila ya es visible, se propone el cambio y CHECK OPTION comprueba el estado resultante. No basta con que la fila cumpliera el filtro antes del UPDATE.

### Alcance del chequeo

- Controla INSERT y UPDATE realizados a través de la vista. No impide modificar directamente la tabla base para dejar filas fuera de su resultado.
- No impide un DELETE permitido: borrar la fila no genera una nueva versión que deba cumplir el filtro.
- No completa columnas omitidas, no reemplaza PK/FK/NOT NULL/CHECK y no vuelve actualizable una vista que no lo era.

### Cuando hay vistas anidadas: LOCAL y CASCADED

Con vistas sobre otras vistas, hay que precisar qué condiciones se comprueban:

- `WITH LOCAL CHECK OPTION` agrega el control de la condición de la vista actual; los controles propios de las vistas inferiores siguen vigentes.
- `WITH CASCADED CHECK OPTION` exige también las condiciones de todas las vistas inferiores, aunque estas no tengan CHECK OPTION propio.
- `WITH CHECK OPTION` sin modalidad equivale a CASCADED.

#### Un ejemplo con las mismas condiciones y distinto control

Definimos una vista inferior sin CHECK OPTION:

```sql
CREATE VIEW envios_desde500 AS
SELECT id_proveedor, id_articulo, cantidad
FROM envio
WHERE cantidad >= 500;
```

Sobre ella definimos dos alternativas de vista superior:

```sql
CREATE VIEW envios_rango_local AS
SELECT id_proveedor, id_articulo, cantidad
FROM envios_desde500
WHERE cantidad < 1000
WITH LOCAL CHECK OPTION;

CREATE VIEW envios_rango_cascaded AS
SELECT id_proveedor, id_articulo, cantidad
FROM envios_desde500
WHERE cantidad < 1000
WITH CASCADED CHECK OPTION;
```

Al consultar, **ambas muestran lo mismo**: cantidades mayores o iguales a 500 y menores que 1000. LOCAL y CASCADED no cambian los filtros del SELECT; cambian las condiciones comprobadas al escribir a través de la vista.

Para comparar los INSERT, suponemos casos independientes, identificadores libres y todas las restricciones base satisfechas:

| Cantidad insertada por la vista superior | Con LOCAL | Con CASCADED |
| --- | --- | --- |
| 300 | Se inserta en `envio`, pero no aparece en ninguna de estas vistas | Se rechaza por `cantidad >= 500` |
| 700 | Se inserta y queda visible | Se inserta y queda visible |
| 1200 | Se rechaza por `cantidad < 1000` | Se rechaza por `cantidad < 1000` |

Con LOCAL, 300 supera la condición propia de la vista superior (`300 < 1000`). La inferior no tiene chequeo, por lo que no impone `cantidad >= 500` como control de escritura. Con CASCADED, también se comprueba la condición inferior y se rechaza 300.

#### LOCAL no desactiva los chequeos inferiores

Si `envios_desde500` tuviera su propio `WITH CHECK OPTION`, insertar 300 por `envios_rango_local` también se rechazaría: el chequeo inferior sigue vigente. LOCAL no significa ignorar todo lo que está debajo, sino agregar el control propio sin exigir por sí mismo los filtros inferiores que no tengan chequeo.

El mismo razonamiento se aplica a UPDATE. Una fila visible de cantidad 700 podría cambiar a 300 mediante la alternativa LOCAL del ejemplo original, quedando fuera del resultado. La alternativa CASCADED lo impediría; también lo impediría un chequeo propio en la vista inferior.

**Método para resolver ejercicios:** distinguir primero los filtros que determinan qué filas se ven; después recorrer la cadena de vistas y anotar qué condiciones se controlan al escribir. No deducir los controles solo a partir de las filas visibles.

#### Duda: ¿CASCADED revisa todas las vistas y LOCAL solo la que se consulta?

La distinción corresponde a INSERT y UPDATE a través de una vista, no al SELECT. Al consultar, se aplican los filtros de toda la cadena en ambos casos.

CASCADED comprueba la condición de la vista por la que escribimos y las condiciones de todas sus vistas subyacentes. No comprueba otras vistas ajenas a esa cadena. LOCAL agrega el control de la condición propia, pero conserva los CHECK OPTION que ya existan debajo. Por eso, decir «LOCAL solo controla esta vista» es correcto únicamente si no hay chequeos inferiores que también deban aplicarse.

## 4. [[Preservación de clave en vistas de ensamble]]

Una tabla **preserva su clave en un join** cuando cada fila de esa tabla puede aparecer como máximo una vez en el resultado. Su clave sigue identificando una única fila del ensamble. No alcanza con incluir sus columnas de clave en el SELECT: hay que analizar si el join puede repetirlas.

### Ejemplo: empleados y departamentos

#### Ejemplo completo: tablas, vista y resultado

Tabla `empleado`: `legajo` es la PK e `id_departamento` es una FK hacia `departamento`.

| legajo | nombre | id_departamento |
| --- | --- | --- |
| 1 | Ana | 20 |
| 2 | Luis | 20 |

Tabla `departamento`: `id_departamento` es la PK.

| id_departamento | nombre |
| --- | --- |
| 20 | Ventas |

Creamos la vista:

```sql
CREATE VIEW empleados_con_departamento AS
SELECT e.legajo,
       e.nombre AS empleado,
       d.id_departamento,
       d.nombre AS departamento
FROM empleado e
JOIN departamento d
  ON e.id_departamento = d.id_departamento;
```

Para consultar el resultado ordenado:

```sql
SELECT *
FROM empleados_con_departamento
ORDER BY legajo;
```

La vista devuelve:

| Legajo | Empleado | Código de departamento | Departamento |
| --- | --- | --- | --- |
| 1 | Ana | 20 | Ventas |
| 2 | Luis | 20 | Ventas |

Buscar el legajo 1 identifica una sola fila: la de Ana. Buscar el departamento 20 devuelve dos filas: la de Ana y la de Luis. Por eso, en este join la clave del empleado sigue sirviendo para distinguir cada fila del resultado, pero la clave del departamento ya no alcanza. Eso significa que se preserva la clave de empleado y no la de departamento. El departamento 20 sigue siendo único en su tabla original; se repite al mostrarlo junto a cada empleado.

Supongamos `empleado` con PK `id_empleado` y FK `id_departamento`, que referencia la PK de `departamento`. Un departamento puede tener muchos empleados:

```sql
CREATE VIEW empleados_departamentos AS
SELECT e.id_empleado, e.nombre AS nombre_empleado,
       d.id_departamento, d.nombre AS nombre_departamento
FROM empleado e
JOIN departamento d
  ON e.id_departamento = d.id_departamento;
```

Un posible resultado es:

| id_empleado | nombre_empleado | id_departamento | nombre_departamento |
| --- | --- | --- | --- |
| 1 | Ana | 20 | Ventas |
| 2 | Luis | 20 | Ventas |
| 3 | Eva | 30 | Compras |

- **Empleado preserva su clave:** cada empleado se une con un único departamento como máximo, porque el departamento se busca por su PK. `id_empleado` no se repite.
- **Departamento no preserva su clave:** un departamento puede unirse con varios empleados. El 20 aparece en dos filas; `id_departamento` no identifica una única fila del resultado.

### Para qué sirve al actualizar

Cambiar el nombre de Ana se puede asociar a una fila concreta de `empleado`. En cambio, cambiar el nombre de Ventas desde la primera fila afectaría al mismo departamento que también aparece en la segunda: el cambio se reflejaría en ambas filas del resultado.

La preservación ayuda a razonar la propagación de escrituras según los criterios de cátedra. No significa que cualquier escritura sobre el join esté permitida. En PostgreSQL, una vista con este join no admite actualización automática; se necesita un mecanismo explícito, como los triggers INSTEAD OF del punto siguiente.

**Cómo decidirlo:** preguntar, para cada tabla, si una fila suya puede combinarse con más de una fila de la otra. Si puede, su clave puede repetirse y no queda preservada. La conclusión debe apoyarse en las restricciones del modelo, no solo en los datos actuales: aunque hoy hubiera un empleado por departamento, el modelo seguiría permitiendo varios.

## 5. [[Triggers INSTEAD OF sobre vistas]]

`INSTEAD OF` significa **«en lugar de»**. Un trigger de este tipo reemplaza la operación solicitada sobre la vista por las instrucciones de su función. Así definimos explícitamente cómo escribir en las tablas base.

### Continuamos con empleados y departamentos

Usamos la vista `empleados_con_departamento` del ejemplo anterior. En PostgreSQL, su join impide la actualización automática. Queremos permitir una sola operación: cambiar el nombre del empleado. No permitiremos cambiar legajo, código de departamento ni nombre de departamento mediante este UPDATE.

Estado inicial de la vista:

| legajo | empleado | id_departamento | departamento |
| --- | --- | --- | --- |
| 1 | Ana | 20 | Ventas |
| 2 | Luis | 20 | Ventas |

Creamos la función que implementa esa política:

```sql
CREATE OR REPLACE FUNCTION cambiar_nombre_empleado_vista()
RETURNS trigger
LANGUAGE plpgsql
AS $$
BEGIN
    IF NEW.legajo IS DISTINCT FROM OLD.legajo
       OR NEW.id_departamento IS DISTINCT FROM OLD.id_departamento
       OR NEW.departamento IS DISTINCT FROM OLD.departamento THEN
        RAISE EXCEPTION 'Solo se permite cambiar el nombre del empleado';
    END IF;

    UPDATE empleado
    SET nombre = NEW.empleado
    WHERE legajo = OLD.legajo;

    RETURN NEW;
END;
$$;

CREATE TRIGGER tr_cambiar_nombre_empleado_vista
INSTEAD OF UPDATE ON empleados_con_departamento
FOR EACH ROW
EXECUTE PROCEDURE cambiar_nombre_empleado_vista();
```

Ahora solicitamos:

```sql
UPDATE empleados_con_departamento
SET empleado = 'Ana María'
WHERE legajo = 1;
```

Para esa fila, `OLD.empleado` vale `'Ana'` y `NEW.empleado` vale `'Ana María'`. El trigger usa `OLD.legajo = 1` para localizar el empleado original y ejecuta el cambio en `empleado.nombre`. El IF admite el cambio porque las otras columnas mantienen sus valores. `IS DISTINCT FROM` permite comparar también cuando hay NULL.

Resultado de consultar la vista después, ordenada por legajo:

| legajo | empleado | id_departamento | departamento |
| --- | --- | --- | --- |
| 1 | Ana María | 20 | Ventas |
| 2 | Luis | 20 | Ventas |

En la tabla `empleado`, el nombre del legajo 1 también es Ana María. La tabla `departamento` conserva su fila `(20, Ventas)`.

**Qué aporta cada parte:** INSTEAD OF reemplaza el UPDATE de la vista; el UPDATE dentro de la función modifica la tabla base; el IF define qué cambios admitimos. `RETURN NEW` indica que se atendió la fila. No se vuelve a ejecutar automáticamente el UPDATE original sobre la vista después del trigger.

Este trigger atiende solo UPDATE. INSERT y DELETE necesitan su propia implementación. Las restricciones base siguen aplicándose, y cualquier filtro o regla adicional de la política debe controlarse explícitamente: agregar el trigger no decide por sí solo cómo debe propagarse cada cambio.

### Síntesis de la conversación

Un trigger INSTEAD OF sobre una vista permite definir qué hacer en lugar del INSERT, UPDATE o DELETE solicitado sobre ella. En el uso estudiado, su función traduce esa operación en modificaciones explícitas de una o varias tablas base. La modificación no ocurre solo por declarar INSTEAD OF: hay que escribir las instrucciones correspondientes dentro de la función. También podemos validar y rechazar operaciones que la política no admite.

## 6. [[Vistas con agregados]]

Una vista con agregados presenta resultados calculados mediante funciones como COUNT, SUM o AVG. Por ejemplo, podemos mostrar la cantidad de empleados de cada departamento en lugar de una fila por empleado.

### Tablas del ejemplo

Este ejemplo es independiente del anterior. Tabla `departamento`, con PK `id_departamento`:

| id_departamento | nombre |
| --- | --- |
| 20 | Ventas |
| 30 | Compras |
| 40 | Sistemas |

Tabla `empleado`, con PK `legajo` y FK `id_departamento`:

| legajo | nombre | id_departamento |
| --- | --- | --- |
| 1 | Ana | 20 |
| 2 | Luis | 20 |
| 3 | Eva | 30 |

### Definición y resultado de la vista

```sql
CREATE VIEW cantidad_empleados_departamento AS
SELECT d.id_departamento,
       d.nombre AS departamento,
       COUNT(e.legajo) AS cantidad_empleados
FROM departamento d
LEFT JOIN empleado e
  ON e.id_departamento = d.id_departamento
GROUP BY d.id_departamento, d.nombre;
```

Al ejecutar `SELECT * FROM cantidad_empleados_departamento ORDER BY id_departamento;` obtenemos:

| id_departamento | departamento | cantidad_empleados |
| --- | --- | --- |
| 20 | Ventas | 2 |
| 30 | Compras | 1 |
| 40 | Sistemas | 0 |

GROUP BY reúne las filas de cada departamento y COUNT cuenta sus empleados. LEFT JOIN conserva también Sistemas, aunque no tenga empleados. Contamos `e.legajo` porque es una PK no nula: la fila sin empleado tiene NULL en esa columna y COUNT no la cuenta. Con COUNT(*) contaríamos la fila del join y mostraríamos incorrectamente 1 para Sistemas. Con INNER JOIN, Sistemas no aparecería.

### Por qué el conteo no se modifica automáticamente

Si intentamos cambiar la cantidad de Ventas de 2 a 3 en la vista, no estamos indicando qué empleado agregar ni cuáles son su legajo, nombre y demás datos necesarios. El motor no tiene una traducción automática de ese cambio hacia las tablas base. Esta vista agregada no es automáticamente actualizable en PostgreSQL.

Lo habitual es agregar el empleado a `empleado`, con departamento 20. Al volver a consultar la vista, el conteo será 3 sin modificar su definición. Lo mismo sucede con SUM y AVG: cambiar el resultado agregado no determina por sí solo qué filas base modificar.

Se puede definir una política explícita con INSTEAD OF, por ejemplo permitir cambiar el nombre del departamento y rechazar cambios del conteo. El trigger no hace que COUNT sea un dato independiente almacenado en la vista.

## 7. [[Vistas simples con EXISTS]]

## 8. [[Operaciones acumulativas sobre vistas]]

## Checklist del práctico

- [ ] Definir vistas simples, anidadas, de join y agregadas.
- [ ] Distinguir actualizabilidad de validez de una escritura concreta.
- [ ] Explicar migración de filas y WCO.
- [ ] Resolver LOCAL/CASCADED sin ignorar WCO inferiores.
- [ ] Determinar claves preservadas y ambigüedad de propagación.
- [ ] Implementar una política explícita con INSTEAD OF.
- [ ] Conservar entidades sin relacionados en vistas de conteos.
- [ ] Seguir operaciones acumulativas y restricciones base.

## Conocimientos anteriores

- Relacionados: TP4 - Triggers; joins, conteos y subconsultas del TP2.

## Criterio de estudio y fuentes

Las explicaciones se apoyan en este práctico y los anteriores. Se conserva la sintaxis de la cátedra; las diferencias de PostgreSQL se tratan por separado cuando las pide la consigna. Los ejemplos de teoría son didácticos y no sustituyen la resolución personal.

- Enunciado local: `../../Practicos/Practico 5 - Vistas.docx`.
- Filminas cotejadas: `BDI_05 (Vistas).pdf`, en `../../Contenido extra/Filminas y Practicos de cursada/2023/`.
