---
tags:
  - tp4
---

# TP4 - Resolución del ejercicio 2

Se necesita registrar quién y cuándo realizó INSERT, UPDATE o DELETE sobre ENTREGA o RENGLON_ENTREGA. La tabla histórica debe conservar los registros aunque las filas originales desaparezcan.

## Implementación

1. [[TP4 - Resolución 2 - Tabla de auditoría|Crear HIS_ENTREGA: campos, identificación y tabla de origen]].
2. Elegir una alternativa:
   - [[TP4 - Resolución 2 - Auditoría por fila|FOR EACH ROW: una entrada de auditoría por fila]].
   - [[TP4 - Resolución 2 - Auditoría por sentencia|FOR EACH STATEMENT: una entrada con el conteo global]].
3. [[TP4 - Resolución 2 - Resultado del DELETE y acciones referenciales|Resolver DELETE FROM ENTREGA WHERE id_video = 3582]].

Las alternativas se comparan en esquemas de práctica separados. Instalarlas juntas registraría dos veces las operaciones y no representaría ninguna de las dos soluciones pedidas.

## Pruebas realizadas

En ambas tablas se probaron INSERT, UPDATE y DELETE de varias filas. También se probó una sentencia que afecta cero filas: ROW no registra nada; STATEMENT registra una entrada con cantidad cero. Las funciones comparten cuerpo por tabla usando TG_TABLE_NAME, pero cada tabla tiene sus propios triggers.

La cantidad para una sentencia se obtuvo con un contador auxiliar; ni COUNT(*) de la tabla final ni OLD/NEW de un trigger STATEMENT proporcionan esa cantidad automáticamente.

Fuente: `Practico 4 - Acciones Procedurales.docx`, TP3 y esquema correspondiente de la cátedra. Se usa `EXECUTE PROCEDURE`.
