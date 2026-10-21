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

## Un ejemplo con y sin ciclo

Considérese:

```text
r1(A), w1(A), r2(A), w2(B), r3(B), c1, c2, c3
```

w1(A) precede a r2(A), así que se agrega T1→T2. w2(B) precede a r3(B), así que se agrega T2→T3. El grafo es acíclico y el orden serial compatible es T1, T2, T3. Los COMMIT no añaden conflictos de lectura/escritura al grafo.

Si antes de c1 se añade `w3(A)`, aparece T2→T3 por la lectura r2(A) anterior a esa escritura y también T1→T3. Eso **todavía no genera un ciclo**: todas las aristas siguen hacia adelante. Para cerrar el ciclo, haría falta, por ejemplo, una `r1(B)` posterior a w2(B), que añade T2→T1 y contradice T1→T2.

Este contraste sirve para revisar el dibujo: más aristas no implica necesariamente un ciclo. Si hay T1→T2 y T1→T3 sin arista entre T2 y T3, son posibles los órdenes T1,T2,T3 y T1,T3,T2.

## Revisar todos los ítems

Para el ejercicio 2, hacer una lista por A, B, C, D y E y extraer los conflictos entre transacciones diferentes. Dos lecturas de A no generan arista, aunque ocurran muy separadas. Dos operaciones de T1 no generan una arista de T1 a sí misma. Después unir las aristas obtenidas: un ciclo puede formarse con conflictos sobre ítems distintos.

## Fuente de estudio

Enunciado del TP8 en la carpeta `../Practicos` y filminas de cátedra `BDI_08 (Proc_Transacciones).pdf` (carpeta de cursada 2023).
