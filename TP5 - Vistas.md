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

Todavía no registramos dudas específicas. Las incorporaremos aquí junto con sus respuestas a medida que conversemos este tema.

## 2. [[Actualización de vistas y restricciones base]]

## 3. [[WITH CHECK OPTION y migración de filas]]

## 4. [[CHECK OPTION LOCAL y CASCADED]]

## 5. [[Preservación de clave en vistas de ensamble]]

## 6. [[Triggers INSTEAD OF sobre vistas]]

## 7. [[Vistas con agregados]]

## 8. [[Vistas simples con EXISTS]]

## 9. [[Operaciones acumulativas sobre vistas]]

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

- Enunciado local: `../Practicos/Practico 5 - Vistas.docx`.
- Filminas cotejadas: `BDI_05 (Vistas).pdf`, en `../Contenido extra/Filminas y Practicos de cursada/2023/`.
