---
tags:
  - tp8
---

# Análisis de logs y cortes ante fallas

## Aplicación al log del ejercicio 2

Orden relevante: T1 y T2 empiezan antes del checkpoint; T2 confirma antes; T1 confirma después; T3 queda sin COMMIT; T4 confirma antes de la falla.

Bajo la hipótesis del checkpoint didáctico anterior:

| Transacción | Diferida NO UNDO/REDO | Inmediata UNDO/NO REDO | Inmediata UNDO/REDO |
|---|---|---|---|
| T1 | REDO | Ninguna: FORCE al COMMIT | REDO |
| T2 | Ninguna: checkpoint | Ninguna | Ninguna: checkpoint |
| T3 | Descartar, sin UNDO de datos persistidos | UNDO | UNDO |
| T4 | REDO | Ninguna: FORCE al COMMIT | REDO |

Aunque algunas escrituras de T1 sean anteriores al checkpoint, no se excluye T1 solo por esa ubicación. En un algoritmo que registre páginas ya aseguradas, el trabajo concreto puede reducirse.

## Cómo resolver los cortes del log del ejercicio 3

Para cada corte, usar únicamente el prefijo que llegó al almacenamiento estable:

| Último registro conservado | Confirmadas en ese prefijo | Sin COMMIT |
|---|---|---|
| `<T1, D5, 50, 60>` | T2 | T1 |
| `<T1, D3, 50, 35>` | Ninguna | T1 y T2 |
| `<T3, D4, 90, 70>` | T1 y T2 | T3 |

En UNDO/REDO, rehacer las confirmadas y deshacer las no confirmadas que hayan podido persistir. T3 no interviene en los primeros dos cortes: ni siquiera había empezado.

Para obtener valores finales, seguir el algoritmo pedido y anotar cada restauración. El log impreso contiene valores anteriores que no se explican por las escrituras precedentes mostradas, por ejemplo D3 pasa de un nuevo valor 35 en T1 a un valor anterior 0 en T3. Antes de presentar un estado final como único, declarar si se aplica mecánicamente el log o si faltan operaciones/datos; no corregir esos valores en silencio.
