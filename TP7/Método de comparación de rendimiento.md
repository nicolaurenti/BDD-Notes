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

## Presentar evidencia suficiente en la respuesta

Una comparación útil incluye SQL original, SQL alternativo, razón de equivalencia, plan de cada uno y diferencia observada. Por ejemplo: «La alternativa consulta existencia en TRABAJA_EN; conserva una fila por empleado y elimina la necesidad lógica de DISTINCT. En el plan medido se compara el trabajo de eliminación de duplicados». Sin un plan medido, dejar la última parte como hipótesis a comprobar.

Si una versión tarda 0.2 ms y otra 0.3 ms en una tabla de seis filas, una ejecución aislada no permite una conclusión general. Revisar las estimaciones, las filas que pasan por cada nodo y los accesos disponibles puede ser más informativo para explicar el ejercicio que declarar una ganadora por esa diferencia pequeña.

Antes de proponer un índice adicional, comprobar si una PK o UNIQUE ya aporta uno con el prefijo necesario. Después justificar por qué las condiciones frecuentes lo usarían. El informe debe distinguir tres cosas: transformación equivalente, beneficio esperado y beneficio efectivamente observado. No se inventan salidas EXPLAIN para presentar una expectativa como medición.

## Fuente de estudio

Enunciado del TP7 en la carpeta `../../Practicos` y filminas de cátedra `BDI_07 (Optimizacion).pdf` (carpeta de cursada 2023).
