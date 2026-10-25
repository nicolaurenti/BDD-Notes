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

## 4. [[Preservación de clave en vistas de ensamble]]

## 5. [[Triggers INSTEAD OF sobre vistas]]

## 6. [[Vistas con agregados]]

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
