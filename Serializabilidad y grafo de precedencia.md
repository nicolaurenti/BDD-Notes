---
tags:
  - tp8
---

# Serializabilidad y grafo de precedencia


- **Serial:** termina una transacción antes de empezar otra.
- **Serializable:** el resultado observable puede explicarse como una ejecución serial bajo la equivalencia utilizada.
- En los ejercicios de grafos se comprueba **serializabilidad por conflictos**.

Dos operaciones entran en conflicto si pertenecen a transacciones distintas, acceden al mismo ítem y al menos una escribe: lectura–escritura, escritura–lectura o escritura–escritura. Dos lecturas no generan conflicto.

### Construcción del grafo

1. Un nodo por transacción.
2. Por cada conflicto en orden Ti antes que Tj, agregar Ti → Tj.
3. Si hay un ciclo, el schedule no es serializable por conflictos.
4. Si no hay ciclos, los órdenes topológicos son órdenes seriales equivalentes.

Ejemplo: `w1(A), r2(A), w2(B), r3(B)` genera T1 → T2 → T3. Si además una operación de T3 precede a una operación conflictiva de T1, aparece un ciclo.

No agregar aristas solo porque una transacción empieza antes. No concluir que un ciclo descarta cualquier otra noción de equivalencia: el criterio del grafo es por conflictos.
