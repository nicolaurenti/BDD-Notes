---
tags:
  - tp3
---

# Integridad referencial y matching

## Acciones referenciales

Se declaran en la tabla que contiene la FK, pero se activan al modificar o borrar la fila **referenciada**.

| Acción | Al borrar la fila referenciada | Al modificar su clave |
|---|---|---|
| `RESTRICT` | Rechaza si hay referencias | Rechaza si hay referencias afectadas |
| `CASCADE` | Borra las filas que la referencian | Propaga la nueva clave a las FK |
| `SET NULL` | Pone la FK afectada en nulo | Pone la FK afectada en nulo |
| `SET DEFAULT` | Asigna el valor por defecto | Asigna el valor por defecto |
| `NO ACTION` | Exige que la FK quede satisfecha al comprobarla | Lo mismo |

`NO ACTION` puede permitir comprobación diferida si la restricción se declaró diferible. `RESTRICT` no difiere esa acción. No asumir que son siempre intercambiables.

- `SET NULL` puede fallar si una columna es `NOT NULL` o parte de una PK.
- `SET DEFAULT` también debe dejar una referencia válida.
- Una cascada puede alcanzar otras tablas; si otra restricción impide el resultado, falla toda la sentencia.
- Cambiar directamente una FK en la tabla hija exige que el nuevo padre exista; `ON UPDATE CASCADE` no crea padres.
- Borrar una fila hija no borra automáticamente su padre.

## Declarar una referencia

La FK se incorpora en la hija y referencia una clave completa del padre. Las acciones ON DELETE y ON UPDATE se eligen según la regla del modelo.

[[TP3 - Declaración de FK con ALTER TABLE|Declaración de FK con ALTER TABLE]]

## MATCH SIMPLE, FULL y PARTIAL

Controlan los nulos de una **FK compuesta**.

| Modalidad   | Regla                                                                                  |
| ----------- | -------------------------------------------------------------------------------------- |
| **SIMPLE**  | Si hay algún NULL, no busca coincidencia. Sin nulos, exige la pareja exacta.           |
| **FULL**    | Todos NULL o pareja exacta sin nulos. Rechaza nulos parciales.                         |
| **PARTIAL** | Los valores informados deben coincidir en una misma fila padre. Todos NULL se admiten. |

### Declaración y casos de matching

La declaración y la tabla de casos muestran qué se acepta cuando falta uno o ambos componentes de una FK compuesta.

[[TP3 - Casos de MATCH SIMPLE, FULL y PARTIAL|Casos de MATCH SIMPLE, FULL y PARTIAL]]

## Seguir una cadena de acciones

Las acciones se propagan desde una modificación del padre hacia las referencias hijas. Una modificación directa de la hija no crea padres ni dispara una cascada hacia arriba.

[[TP3 - Ejemplo de cascadas y dirección de las acciones|Ejemplo de cascadas y dirección de las acciones]]

## Justificar el matching como búsqueda de una fila

Para justificar un caso se busca una misma fila padre que cumpla los componentes exigidos por la modalidad. Después se comprueban las demás restricciones de la sentencia.

[[TP3 - Casos de MATCH SIMPLE, FULL y PARTIAL|Casos de MATCH SIMPLE, FULL y PARTIAL]]

## Fuente de estudio

Enunciado del TP3 en la carpeta `../Practicos` y filminas de cátedra `BDI_03.P1 (Restricciones-1).pdf; BDI_03.P1 (Restricciones-2).pdf` (carpeta de cursada 2023).
