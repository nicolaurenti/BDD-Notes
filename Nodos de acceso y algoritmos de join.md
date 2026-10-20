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
