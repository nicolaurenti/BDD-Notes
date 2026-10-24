---
tags:
  - tp7
---

# TP7 - Ejemplos para comparar accesos y algoritmos de join

Ejemplos de búsquedas por entrega, Nested Loop, Hash Join y Merge Join.

## Comparar estrategias con un caso concreto

Si se buscan los renglones de una sola entrega y hay un índice que comienza por nro_entrega, el motor puede localizar ese pequeño conjunto. Si se necesitan casi todos los renglones, leer muchas entradas del índice y luego sus páginas puede costar más que recorrer la tabla: por eso Seq Scan no significa por sí mismo «sin optimizar».

En **Nested Loop**, para cada fila de la entrada exterior se examina la entrada interior. Con tres películas seleccionadas y un índice de renglones por película puede resultar apropiado. Con miles de filas exteriores y búsquedas interiores caras, la repetición puede dominar el costo.

En **Hash Join**, se prepara una estructura hash con una entrada y se buscan coincidencias de igualdad desde la otra. En **Merge Join**, se recorren entradas ordenadas por las columnas de unión; si hay que ordenarlas, ese trabajo también cuenta. No se elige un algoritmo únicamente por cómo se escribió JOIN en SQL.

## Fuente de estudio

Enunciado del TP7 en la carpeta `../../Practicos` y filminas de cátedra `BDI_07 (Optimizacion).pdf` (carpeta de cursada 2023).
