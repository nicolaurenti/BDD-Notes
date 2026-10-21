---
tags:
  - tp8
---

# TP8 - Ejemplos de grafos con y sin ciclo

Se construyen las aristas a partir del orden de operaciones y se contrastan un grafo acíclico y uno cíclico.

## Un ejemplo con y sin ciclo

Considérese:

```text
r1(A), w1(A), r2(A), w2(B), r3(B), c1, c2, c3
```

w1(A) precede a r2(A), así que se agrega T1→T2. w2(B) precede a r3(B), así que se agrega T2→T3. El grafo es acíclico y el orden serial compatible es T1, T2, T3. Los COMMIT no añaden conflictos de lectura/escritura al grafo.

Si antes de c1 se añade `w3(A)`, aparece T2→T3 por la lectura r2(A) anterior a esa escritura y también T1→T3. Eso **todavía no genera un ciclo**: todas las aristas siguen hacia adelante. Para cerrar el ciclo, haría falta, por ejemplo, una `r1(B)` posterior a w2(B), que añade T2→T1 y contradice T1→T2.

Este contraste sirve para revisar el dibujo: más aristas no implica necesariamente un ciclo. Si hay T1→T2 y T1→T3 sin arista entre T2 y T3, son posibles los órdenes T1,T2,T3 y T1,T3,T2.

## Fuente de estudio

Enunciado del TP8 en la carpeta `../Practicos` y filminas de cátedra `BDI_08 (Proc_Transacciones).pdf` (carpeta de cursada 2023).
