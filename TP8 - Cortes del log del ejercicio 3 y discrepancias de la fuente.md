---
tags:
  - tp8
---

# TP8 - Cortes del log del ejercicio 3 y discrepancias de la fuente

Se conservan los registros impresos y se explican los valores anteriores que no se pueden reconstruir de forma consistente.

## Cómo resolver los cortes del log del ejercicio 3

Para cada corte, usar únicamente el prefijo que llegó al almacenamiento estable:

| Último registro conservado | Confirmadas en ese prefijo | Sin COMMIT |
|---|---|---|
| `<T1, D5, 50, 60>` | T2 | T1 |
| `<T1, D3, 50, 35>` | Ninguna | T1 y T2 |
| `<T3, D4, 90, 70>` | T1 y T2 | T3 |

En UNDO/REDO, rehacer las confirmadas y deshacer las no confirmadas que hayan podido persistir. T3 no interviene en los primeros dos cortes: ni siquiera había empezado.

Para obtener valores finales, seguir el algoritmo pedido y anotar cada restauración. El log impreso contiene valores anteriores que no se explican por las escrituras precedentes mostradas, por ejemplo D3 pasa de un nuevo valor 35 en T1 a un valor anterior 0 en T3. Antes de presentar un estado final como único, declarar si se aplica mecánicamente el log o si faltan operaciones/datos; no corregir esos valores en silencio.

## Cómo justificar una inconsistencia de la fuente

En el log del ejercicio 3, si una escritura deja D3=35 y una posterior informa D3 anterior=0 sin una escritura intermedia mostrada, no se puede reconstruir una historia única consistente con ambos datos como si nada faltara. Conservar los registros tal como están y señalar la discrepancia.

Se pueden listar las acciones mecánicas del algoritmo y sus valores anteriores/nuevos, pero hay que distinguir esa aplicación de la afirmación de un estado final consistente y único. Para un examen, explicar la hipótesis adoptada si se solicita continuar; no cambiar 0 por 35 silenciosamente. El error del enunciado no modifica qué transacciones tienen COMMIT dentro de cada corte.

## Fuente de estudio

Enunciado del TP8 en la carpeta `../Practicos` y filminas de cátedra `BDI_08 (Proc_Transacciones).pdf` (carpeta de cursada 2023).
