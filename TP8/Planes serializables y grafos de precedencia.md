---
tags:
  - tp8
---

# Planes serializables y grafos de precedencia

Un **plan de ejecución** o *schedule* ordena las operaciones de varias transacciones, respetando el orden interno de cada una.

- **Serial:** termina una transacción antes de comenzar la siguiente.
- **Concurrente o no serial:** intercala sus operaciones.
- **Serializable:** equivale a algún orden serial. Puede tener operaciones intercaladas.

El grafo del TP comprueba **serializabilidad por conflictos**. Es un criterio estructural: no depende de que ciertos valores iniciales produzcan casualmente el mismo resultado. La equivalencia por vistas es más general y no es lo que prueba este grafo.

## Qué operaciones generan un conflicto

Deben pertenecer a **transacciones distintas**, actuar sobre el **mismo ítem** y tener **al menos una escritura**.

| Primero en el plan | Después | Arista |
|---|---|---|
| `r1(X)` | `w2(X)` | T1 → T2 |
| `w1(X)` | `r2(X)` | T1 → T2 |
| `w1(X)` | `w2(X)` | T1 → T2 |
| `r1(X)` | `r2(X)` | Ninguna |
| `w1(X)` | `r2(Y)` | Ninguna si X e Y son ítems distintos |

`r1(X)` significa que T1 lee X. La flecha expresa: **para conservar este conflicto, T1 debe preceder a T2 en el orden serial equivalente**.

## Cómo construir y leer el grafo

1. Creá un nodo por transacción, incluso si no tiene conflictos.
2. Compará las operaciones sobre cada ítem en orden temporal. No mires solo las consecutivas: un conflicto puede estar separado por otras operaciones.
3. Agregá cada precedencia entre transacciones. Una arista repetida no se dibuja de nuevo; podés rotularla con los ítems que la justifican.
4. Buscá ciclos. **Grafo acíclico ⇔ plan serializable por conflictos.**
5. Si es acíclico, obtené órdenes topológicos: elegí un nodo sin predecesores pendientes, retiralo y repetí. Cada secuencia completa es un orden serial equivalente.

## Dos ejemplos breves

**Sin ciclo:** `w1(A), r2(A), w2(B), r3(B)` genera T1 → T2 por A y T2 → T3 por B. El orden equivalente es **T1, T2, T3**. Si T3 fuera independiente, podría ubicarse en cualquier lugar que no altere T1 antes de T2.

**Con ciclo:** `r1(A), r2(B), w1(B), w2(A)` genera T2 → T1 por B y T1 → T2 por A. Exige que ambas vayan primero: no existe un orden serial equivalente por conflictos.

## Qué recordar

No se requiere que el plan coincida con **todos** los órdenes seriales; alcanza con alguno. Un grafo sin ciclos no prueba por sí solo que se eviten lecturas sucias o que el plan sea recuperable: serializabilidad y recuperación evalúan propiedades distintas.
