---
tags:
  - tp7
---

# Equivalencia y rendimiento de consultas


Optimizar significa reducir trabajo conservando el resultado: mismas filas, mismas repeticiones requeridas y mismo tratamiento de nulos y entidades sin relacionados.

Pasar de JOIN a EXISTS puede evitar duplicados cuando solo interesa existencia, pero no conserva una consulta que necesita cada pareja. Agregar DISTINCT para ocultar un join incompleto no lo corrige.

Antes de proponer índices, identificar filtros, uniones, claves, cardinalidades y frecuencia de escritura. En Películas, departamento se identifica por `(id_distribuidor, id_departamento)`.
