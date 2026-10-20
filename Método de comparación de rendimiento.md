---
tags:
  - tp7
---

# Método de comparación de rendimiento


1. Confirmar igualdad de resultados y duplicados sobre casos relevantes.
2. Obtener plan estimado; medir SELECT con ANALYZE si se necesita.
3. Comparar estimaciones con filas reales y revisar los nodos con mayor trabajo.
4. Evaluar índices ya existentes antes de proponer otros.
5. Verificar estadísticas; datos sintéticos pequeños no representan siempre producción.
6. Repetir con condiciones comparables de caché y parámetros; no atribuir todo a la reescritura.

No se ejecutaron consultas, ANALYZE ni creaciones de índices para redactar este resumen. Los planes concretos deben obtenerse de la conexión y el esquema elegidos.
