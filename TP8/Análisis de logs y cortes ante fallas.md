---
tags:
  - tp8
---

# Análisis de logs y cortes ante fallas

Para cada corte se analiza únicamente el prefijo de log disponible: se identifican START y COMMIT, se forman las listas de recuperación según la estrategia y se recorren sus escrituras. La posición de una escritura aislada no basta para clasificar toda la transacción.

## Aplicación al log del ejercicio 2

La clasificación de T1–T4 cambia según se use diferida, inmediata con FORCE o UNDO/REDO. El desarrollo compara esas alternativas.

### Recuperación del log del ejercicio 2

Se aplica el modelo de checkpoint de la cátedra al log que contiene T1, T2, T3 y T4.

Orden relevante: T1 y T2 empiezan antes del checkpoint; T2 confirma antes; T1 confirma después; T3 queda sin COMMIT; T4 confirma antes de la falla.

Bajo la hipótesis del checkpoint didáctico anterior:

| Transacción | Diferida NO UNDO/REDO | Inmediata UNDO/NO REDO | Inmediata UNDO/REDO |
|---|---|---|---|
| T1 | REDO | Ninguna: FORCE al COMMIT | REDO |
| T2 | Ninguna: checkpoint | Ninguna | Ninguna: checkpoint |
| T3 | Descartar, sin UNDO de datos persistidos | UNDO | UNDO |
| T4 | REDO | Ninguna: FORCE al COMMIT | REDO |

Aunque algunas escrituras de T1 sean anteriores al checkpoint, no se excluye T1 solo por esa ubicación. En un algoritmo que registre páginas ya aseguradas, el trabajo concreto puede reducirse.

## Cortes e inconsistencias del log

Cada punto de corte tiene sus propios COMMIT disponibles. Se clasifican las transacciones sin incorporar información que aparece después de ese punto.

### Cortes del log del ejercicio 3 y discrepancias de la fuente

Se conservan los registros impresos y se explican los valores anteriores que no se pueden reconstruir de forma consistente.

#### Cómo resolver los cortes del log del ejercicio 3

Para cada corte, usar únicamente el prefijo que llegó al almacenamiento estable:

| Último registro conservado | Confirmadas en ese prefijo | Sin COMMIT |
|---|---|---|
| `<T1, D5, 50, 60>` | T2 | T1 |
| `<T1, D3, 50, 35>` | Ninguna | T1 y T2 |
| `<T3, D4, 90, 70>` | T1 y T2 | T3 |

En UNDO/REDO, rehacer las confirmadas y deshacer las no confirmadas que hayan podido persistir. T3 no interviene en los primeros dos cortes: ni siquiera había empezado.

Para obtener valores finales, seguir el algoritmo pedido y anotar cada restauración. El log impreso contiene valores anteriores que no se explican por las escrituras precedentes mostradas, por ejemplo D3 pasa de un nuevo valor 35 en T1 a un valor anterior 0 en T3. Antes de presentar un estado final como único, declarar si se aplica mecánicamente el log o si faltan operaciones/datos; no corregir esos valores en silencio.

#### Cómo justificar una inconsistencia de la fuente

En el log del ejercicio 3, si una escritura deja D3=35 y una posterior informa D3 anterior=0 sin una escritura intermedia mostrada, no se puede reconstruir una historia única consistente con ambos datos como si nada faltara. Conservar los registros tal como están y señalar la discrepancia.

Se pueden listar las acciones mecánicas del algoritmo y sus valores anteriores/nuevos, pero hay que distinguir esa aplicación de la afirmación de un estado final consistente y único. Para un examen, explicar la hipótesis adoptada si se solicita continuar; no cambiar 0 por 35 silenciosamente. El error del enunciado no modifica qué transacciones tienen COMMIT dentro de cada corte.

## Caso breve para practicar el método

El ejemplo independiente permite comprobar cómo cambia la recuperación al conservar o perder un COMMIT, sin depender de los datos inconsistentes del log impreso.

### Ejemplo breve de recuperación antes y después de COMMIT

Log pequeño de T1 sobre A y T2 sobre B, con dos cortes distintos.

#### Resolver cada corte como un log distinto

No mirar el log completo y clasificar para siempre a T1 como confirmada. En el corte que termina en `<T1, D3, 50, 35>`, su COMMIT todavía no existe; en el corte final sí. Marcar START y COMMIT solo dentro del prefijo correspondiente y después construir las listas UNDO y REDO.

Ejemplo breve independiente del log impreso:

```text
<T1, START>
<T1, A, 10, 15>
<T2, START>
<T2, B, 20, 30>
<T1, COMMIT>
FALLA
```

Con UNDO/REDO: T1 está en REDO y T2 en UNDO. Deshacer T2 restaura B=20; rehacer T1 asegura A=15. Si se corta antes de `<T1, COMMIT>`, ambas están sin confirmar: se restaura A=10 y B=20. El dato decisivo fue el COMMIT disponible, no que una transacción pareciera «terminada» en un relato externo.

## Tratar inconsistencias de la fuente

Si el valor anterior no se explica por las escrituras previas mostradas, se conserva la fuente y se declara la hipótesis, en lugar de corregirlo silenciosamente.

## Fuente de estudio

Enunciado del TP8 en la carpeta `../../Practicos` y filminas de cátedra `BDI_08 (Proc_Transacciones).pdf` (carpeta de cursada 2023).
