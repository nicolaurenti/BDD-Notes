---
tags:
  - tp7
---

# 4. DISTINCT y eliminación de duplicados

## Idea central

DISTINCT conserva una sola copia de cada combinación completa de valores seleccionados. No elimina «empleados repetidos» en abstracto: compara exactamente las columnas del SELECT.

## Cómo razonarlo

La PK de una tabla evita duplicados en ella, pero un join uno a muchos puede repetir sus columnas. Antes de retirar DISTINCT, demostrar que la proyección no se repetirá.

El motor puede eliminar duplicados ordenando y aplicando Unique o usando agrupamiento hash. Ambas alternativas consumen recursos; si el conjunto es grande pueden necesitar más memoria o trabajo en disco. Un índice o un orden ya disponible puede ayudar, pero no garantiza que desaparezca el trabajo.

## Ejemplo

Hay dos empleados diferentes llamados Ana; uno participa en dos proyectos y el otro en uno:

| SELECT sobre el join | Resultado |
|---|---|
| e.id_empleado, e.nombre | (1,Ana), (1,Ana), (2,Ana). |
| DISTINCT e.id_empleado, e.nombre | (1,Ana), (2,Ana). |
| DISTINCT e.nombre | Una sola fila Ana. |

Con EXISTS y `SELECT e.*`, una fila por empleado y su PK en la salida hacen innecesario DISTINCT. Si se devuelve solo nombre, EXISTS podría producir dos Ana: no es equivalente a DISTINCT nombre.

## Errores frecuentes

Agregar DISTINCT para ocultar un join mal planteado puede encubrir el error y agregar costo. Tampoco puede retirarse porque «hay claves primarias»: importa la multiplicidad después de unir y proyectar. Para eliminar duplicados, dos NULL se consideran iguales a estos efectos; no se aplica la lógica de `NULL = NULL` de un WHERE.

## Qué recordar

Preguntar **qué entidad quiero contar o devolver**, qué puede repetirla y si solo necesito existencia. DISTINCT tiene un significado y un costo, no es un arreglo automático. [Referencia técnica](https://www.postgresql.org/docs/15/queries-select-lists.html).
