---
tags:
  - tp7
---

# Nodos de acceso y algoritmos de join

| Nodo | Idea | Cuándo puede convenir |
|---|---|---|
| Seq Scan | Leer la tabla y filtrar | Tabla pequeña o gran proporción de filas |
| Index Scan | Buscar por índice y obtener filas | Condición selectiva |
| Index Only Scan | Obtener columnas del índice cuando sea posible | Índice cubriente y visibilidad favorable |
| Bitmap Index/Heap Scan | Reunir ubicaciones y leer páginas agrupadas | Cantidad intermedia de coincidencias o combinación de índices |
| Nested Loop | Buscar coincidencias para cada fila exterior | Exterior pequeño, interior indexado |
| Hash Join | Construir hash y buscar por igualdad | Uniones de igualdad con volúmenes adecuados |
| Merge Join | Combinar entradas ordenadas | Orden disponible o rentable |
| HashAggregate / GroupAggregate | Agrupar y agregar | Conteos y resúmenes |
| Sort / Unique | Ordenar o eliminar duplicados | ORDER BY, DISTINCT y otros requisitos |

Que aparezca Seq Scan no demuestra un problema. Un índice también consume E/S, memoria y mantenimiento. Los planes dependen de estadísticas, volumen, selectividad y configuración; no se puede deducir un único plan físico del texto SQL.

## Comparar estrategias con un caso concreto

La elección depende del volumen seleccionado, el orden y los índices disponibles. Un caso selectivo y otro que recorre casi toda la tabla permiten contrastar costos.

### Ejemplos para comparar accesos y algoritmos de join

Ejemplos de búsquedas por entrega, Nested Loop, Hash Join y Merge Join.

Si se buscan los renglones de una sola entrega y hay un índice que comienza por nro_entrega, el motor puede localizar ese pequeño conjunto. Si se necesitan casi todos los renglones, leer muchas entradas del índice y luego sus páginas puede costar más que recorrer la tabla: por eso Seq Scan no significa por sí mismo «sin optimizar».

En **Nested Loop**, para cada fila de la entrada exterior se examina la entrada interior. Con tres películas seleccionadas y un índice de renglones por película puede resultar apropiado. Con miles de filas exteriores y búsquedas interiores caras, la repetición puede dominar el costo.

En **Hash Join**, se prepara una estructura hash con una entrada y se buscan coincidencias de igualdad desde la otra. En **Merge Join**, se recorren entradas ordenadas por las columnas de unión; si hay que ordenarlas, ese trabajo también cuenta. No se elige un algoritmo únicamente por cómo se escribió JOIN en SQL.

## Ubicar el trabajo de agrupación y duplicados

Un HashAggregate puede representar GROUP BY o eliminación de repeticiones; Sort más Unique es otra forma de resolver DISTINCT. Al cambiar un join por EXISTS, observar si desaparece ese trabajo y cuántas filas llegan al nodo superior. La explicación del TP debe conectar cada nodo con una operación de la consulta: filtro por género, búsqueda de renglones, asociación con entregas, agrupamiento o eliminación de duplicados.

## Fuente de estudio

Enunciado del TP7 en la carpeta `../../Practicos` y filminas de cátedra `BDI_07 (Optimizacion).pdf` (carpeta de cursada 2023).
