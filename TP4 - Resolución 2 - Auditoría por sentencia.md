---
tags:
  - tp4
---

# TP4 - Resolución 2 - Auditoría por sentencia

La entrada de auditoría se genera una vez por sentencia y contiene cuántas filas se modificaron. Para contar sin secuencias ni tablas de transición, se separa el trabajo en tres pasos:

1. BEFORE STATEMENT prepara el contador correspondiente a tabla y operación en cero.
2. Un auxiliar AFTER ROW suma uno por cada fila modificada.
3. AFTER STATEMENT toma el total y escribe la única entrada de auditoría de la sentencia.

El auxiliar ROW **cuenta**, pero no inserta en HIS_ENTREGA. La auditoría visible conserva granularidad STATEMENT.

## Código por partes, en orden de instalación

- [[TP4 - Resolución 2 - Contador y funciones auxiliares|1 — Tabla de contadores, reinicio y acumulación]].
- [[TP4 - Resolución 2 - Registro final de la sentencia|2 — Función que registra el total de la sentencia]].
- [[TP4 - Resolución 2 - Triggers de la alternativa STATEMENT|3 — Triggers de ambas tablas]].

Se necesita previamente HIS_ENTREGA. El script completo, que reúne estas tres partes, es `02_statement.sql`.

Los contadores distinguen tabla y operación para que una baja de entrega y sus bajas de renglones no se mezclen. Esta implementación se valida para los eventos del TP, con pruebas secuenciales y sin triggers que vuelvan a modificar recursivamente la misma tabla auditada.

Fuente: `Practico 4 - Acciones Procedurales.docx`, TP3 y esquema correspondiente de la cátedra. Se usa `EXECUTE PROCEDURE`.
