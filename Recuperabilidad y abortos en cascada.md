---
tags:
  - tp8
---

# Recuperabilidad y abortos en cascada


Si T2 lee un valor escrito por T1:

- **Recuperable:** T2 no confirma antes de T1.
- **Sin abortos en cascada:** T2 espera el COMMIT de T1 para leer ese valor.
- **Estricto:** nadie lee ni sobrescribe un ítem escrito por T1 hasta que T1 termine.

Un grafo de precedencia sin ciclos no demuestra por sí solo que el orden de COMMIT sea recuperable. Distinguir grafo de conflictos y grafo de esperas: este último se usa para detectar deadlock.

## Los COMMIT determinan la recuperabilidad

En `w1(A), r2(A), c2, a1`, T2 leyó un valor de T1 y confirmó antes de conocer si T1 confirmaba. Cuando T1 aborta, no es posible simplemente cancelar una T2 que ya confirmó: el schedule es no recuperable.

En `w1(A), r2(A), c1, c2`, se respeta el orden de confirmación: es recuperable respecto de esa dependencia. Aun así hubo lectura de un dato no confirmado; si T1 hubiera abortado antes de c1, T2 habría necesitado abortar. Recuperable no significa libre de abortos en cascada.

En `w1(A), c1, r2(A), c2`, la lectura espera la confirmación del escritor: se evita esa cascada. La condición de estricto es más fuerte: también impide que otra transacción sobrescriba A mientras T1 no termine.

## Relación con el grafo de conflictos

El primer ejemplo tiene una dependencia T1→T2 por A y puede no tener ciclo, pero es no recuperable por el orden de confirmación. Por eso hay dos preguntas distintas: «¿equivale por conflictos a un orden serial?» y «¿se pueden respetar las dependencias al confirmar o abortar?». Responder la segunda exige mirar lecturas de escrituras ajenas y COMMIT, no solo el grafo de precedencia.

## Fuente de estudio

Enunciado del TP8 en la carpeta `../Practicos` y filminas de cátedra `BDI_08 (Proc_Transacciones).pdf` (carpeta de cursada 2023).
