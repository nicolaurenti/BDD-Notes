---
tags:
  - tp4
---

# Práctico 4 — Acciones procedurales y triggers

Se parte de las restricciones del TP3 para elegir eventos críticos y escribir controles procedurales. También se estudian auditoría, historial, resúmenes y procedimientos almacenados invocados por el usuario. Los ejemplos de triggers siguen la cabecera EXECUTE PROCEDURE del enunciado y explican estado previo, ejecución y efecto.

## 1. [[Fundamentos y ejecución de triggers]]

## 2. [[Validación de restricciones con triggers]]

## 3. [[Auditoría, historial y resúmenes con triggers]]

## 4. Stored procedures: procedimientos almacenados

Un **stored procedure** es un bloque de código guardado en la base de datos que se ejecuta explícitamente mediante `CALL`. Puede recibir parámetros y realizar operaciones como cargar una tabla de informe. En estos ejercicios no se busca devolver una tabla como resultado de una consulta, sino dejar el resultado almacenado en una tabla existente.

### Cuándo elegirlo frente a un trigger

| Necesidad | Recurso apropiado |
|---|---|
| Generar un informe cuando el usuario lo solicita, usando los valores que indica | Procedimiento con parámetros |
| Reaccionar automáticamente ante un INSERT, UPDATE o DELETE | Trigger y su función asociada |
| Garantizar una regla de integridad ante cualquier modificación pertinente | Recurso declarativo del TP3; si no alcanza o no está soportado, triggers |

**Recibir parámetros no convierte una condición en una restricción de integridad.** En «generar el informe de pedidos con más de N productos desde F», N y F son criterios de selección: los pedidos que no cumplen se excluyen del informe, pero pueden existir en la base. En cambio, «no puede haber más de tres pedidos cancelados por cliente, dirección y año» es una regla que debe mantenerse siempre.

El procedimiento puede validar sus parámetros, pero su ejecución por sí sola no controla cambios realizados fuera de él. No necesita una grilla de eventos críticos: el usuario decide cuándo invocarlo.

### Sintaxis en PostgreSQL

```sql
CREATE OR REPLACE PROCEDURE nombre_procedimiento(p_fecha DATE)
LANGUAGE plpgsql
AS $$
BEGIN
    -- Sentencias que realizan la operación solicitada.
END;
$$;

CALL nombre_procedimiento(DATE '2026-10-01');
```

Los parámetros se declaran con nombre y tipo, separados por comas si hay varios. Se pueden usar en las consultas del cuerpo. El prefijo `p_` ayuda a distinguirlos de las columnas. `DECLARE` es opcional si no hacen falta variables locales. Esta forma no lleva `RETURNS trigger`, ni utiliza `NEW` u `OLD`.

**No confundir con `EXECUTE PROCEDURE` en la cabecera de un trigger:** en PostgreSQL esa cláusula histórica invoca una función que retorna `trigger`; no invoca un stored procedure creado con `CREATE PROCEDURE`.

### Patrón: renovar una tabla de informe sin bucles

Cuando la consigna dice que la tabla ya existe y debe contener **solo la información de los parámetros actuales**, el procedimiento retira el informe anterior con `DELETE` y carga el nuevo con `INSERT ... SELECT`.

Ejemplo adaptado de las filminas: cargar `artic_palabras` con los artículos publicados después de una fecha y su cantidad de palabras asociadas. Se supone que la tabla destino ya existe con las cuatro columnas indicadas.

```sql
CREATE OR REPLACE PROCEDURE completar_artic_pal(p_fecha DATE)
LANGUAGE plpgsql
AS $$
BEGIN
    DELETE FROM artic_palabras;

    INSERT INTO artic_palabras
        (id_articulo, titulo, fecha_pub, cant_palabras)
    SELECT a.id_articulo, a.titulo, a.fecha_pub,
           COUNT(c.id_articulo)
    FROM articulo AS a
    LEFT JOIN contiene AS c ON c.id_articulo = a.id_articulo
    WHERE a.fecha_pub > p_fecha
    GROUP BY a.id_articulo, a.titulo, a.fecha_pub;
END;
$$;

CALL completar_artic_pal(DATE '2023-04-01');
```

`INSERT ... SELECT` carga todas las filas producidas por la consulta en una sola sentencia: no requiere `FOR`, `LOOP` ni una variable por resultado. Si ningún artículo cumple, el informe queda vacío. Una nueva llamada reemplaza el contenido anterior.

**Ajuste respecto del ejemplo docente:** la filmina usa `COUNT(*)` con `LEFT JOIN`. Aquí se usa `COUNT(c.id_articulo)` para que un artículo sin palabras tenga cantidad **0**; `COUNT(*)` contaría la fila que conserva el join y daría **1**. Con dos asociaciones, el resultado es **2** en ambos casos.

Para el ejercicio de `informe_clientes`, el mismo patrón recibe `p_n INTEGER` y `p_f DATE`. Primero se identifican los pedidos que califican, con una fila por pedido, y luego se agrupan por cliente para calcular cantidad, promedio y fecha de entrega más reciente. Así el detalle no multiplica los pedidos al calcular el promedio.

## Resolución

[[Resolución del TP4 - Triggers]]

## Checklist del práctico

- [ ] Elegir momento, evento y granularidad.
- [ ] Usar OLD/NEW y el retorno apropiado.
- [ ] Modificar NEW sin actualizaciones recursivas innecesarias.
- [ ] Completar la grilla de eventos de todas las tablas afectadas.
- [ ] Diferenciar validación, auditoría e historial.
- [ ] Resolver carga inicial y mantenimiento posterior.
- [ ] Distinguir una regla permanente de una operación solicitada por el usuario.
- [ ] Crear e invocar un procedimiento con parámetros mediante CALL.
- [ ] Renovar un informe con DELETE e INSERT ... SELECT, sin bucles.
- [ ] Simular varias filas sin asumir un orden no garantizado.
- [ ] Justificar los eventos críticos y comprobar los conteos en INSERT y UPDATE.

## Conocimientos anteriores

- Relacionados: TP3 - Restricciones de integridad; consultas y nulos del TP2.

## Criterio de estudio y fuentes

Las explicaciones se apoyan en este práctico y los anteriores. Se conserva la sintaxis de la cátedra; las diferencias de PostgreSQL se tratan por separado cuando las pide la consigna. Los ejemplos de teoría son didácticos y no sustituyen la resolución personal.

- Enunciado local: `../../Practicos/Practico 4 - Acciones Procedurales.docx`.
- Filminas cotejadas: `BDI_04 (SQL Procedural).pdf`, en `../../Contenido extra/Filminas y Practicos de cursada/2023/`.
- Stored procedures: filminas de 2023, páginas PDF 26–30 (ejemplo de carga en la 30); material de 2021 `BDI_2021_05_SQ Procedural.pdf`, página PDF 19, en `../../Contenido extra/Filminas y Practicos de cursada/2021/filminas en pdf - bdd/`.
- Bibliografía del tema: Elmasri y Navathe, *Fundamentals of Database Systems*, 6.ª edición, sección 13.4.1, páginas impresas 473–475 (PDF 502–504), en `../../Contenido extra/Libro Teorico Practico.pdf`.
