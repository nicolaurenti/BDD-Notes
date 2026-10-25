---
tags:
  - tp7
---

# Equivalencia y rendimiento de consultas

Optimizar significa reducir trabajo conservando el resultado: mismas filas, mismas repeticiones requeridas y mismo tratamiento de nulos y entidades sin relacionados.

Pasar de JOIN a EXISTS puede evitar duplicados cuando solo interesa existencia, pero no conserva una consulta que necesita cada pareja. Agregar DISTINCT para ocultar un join incompleto no lo corrige.

Antes de proponer índices, identificar filtros, uniones, claves, cardinalidades y frecuencia de escritura. En Películas, departamento se identifica por `(id_distribuidor, id_departamento)`.

## Heurísticas de la cátedra

La optimización heurística transforma el árbol de una consulta en otro equivalente que intenta trabajar con menos datos. La estimación de costos compara estrategias usando estadísticas, como número de filas, bloques ocupados y cantidad de valores diferentes. El plan elegido tiene un costo estimado conveniente; no significa que se haya demostrado el óptimo absoluto entre todas las estrategias imaginables.

Las reglas centrales son aplicar las selecciones temprano, comenzar por las más restrictivas, reemplazar producto cartesiano más condición de reunión por un join y proyectar temprano las columnas necesarias. Al proyectar no se pueden quitar las claves que todavía hacen falta para unir o agrupar.

## Ejemplo de transformación

Para investigadores que trabajan en ROBOTICS, filtrar EMPLEADO por cargo y PROYECTO por nombre antes de reunirlos con TRABAJA_EN reduce las entradas del join. Si EMPLEADO tiene 1000 filas pero solo 20 investigadores, es más barato considerar esas 20 que arrastrar los 1000 empleados por todas las combinaciones. Después del ensamble se proyecta apellido si esa es la salida solicitada.

El beneficio depende de los datos y accesos disponibles; el número 20 es ilustrativo, no una medición. Mover condiciones es válido cuando conserva la semántica. En particular, llevar sin análisis un filtro de ON a WHERE en un LEFT JOIN puede eliminar entidades sin coincidencias, como se vio en TP2. Una consulta más rápida que cambió esas filas no es una optimización equivalente.

## Método de comparación de rendimiento

1. Confirmar igualdad de resultados y duplicados sobre casos relevantes.
2. Obtener plan estimado; medir SELECT con ANALYZE si se necesita.
3. Comparar estimaciones con filas reales y revisar los nodos con mayor trabajo.
4. Evaluar índices ya existentes antes de proponer otros.
5. Verificar estadísticas; datos sintéticos pequeños no representan siempre producción.
6. Repetir con condiciones comparables de caché y parámetros; no atribuir todo a la reescritura.

No se ejecutaron consultas, ANALYZE ni creaciones de índices para redactar este resumen. Los planes concretos deben obtenerse de la conexión y el esquema elegidos.

### Presentar evidencia suficiente en la respuesta

Una comparación útil incluye SQL original, SQL alternativo, razón de equivalencia, plan de cada uno y diferencia observada. Por ejemplo: «La alternativa consulta existencia en TRABAJA_EN; conserva una fila por empleado y elimina la necesidad lógica de DISTINCT. En el plan medido se compara el trabajo de eliminación de duplicados». Sin un plan medido, dejar la última parte como hipótesis a comprobar.

Si una versión tarda 0.2 ms y otra 0.3 ms en una tabla de seis filas, una ejecución aislada no permite una conclusión general. Revisar las estimaciones, las filas que pasan por cada nodo y los accesos disponibles puede ser más informativo para explicar el ejercicio que declarar una ganadora por esa diferencia pequeña.

Antes de proponer un índice adicional, comprobar si una PK o UNIQUE ya aporta uno con el prefijo necesario. Después justificar por qué las condiciones frecuentes lo usarían. El informe debe distinguir tres cosas: transformación equivalente, beneficio esperado y beneficio efectivamente observado. No se inventan salidas EXPLAIN para presentar una expectativa como medición.

## Fuente de estudio

Enunciado del TP7 en la carpeta `../../Practicos` y filminas de cátedra `BDI_07 (Optimizacion).pdf` (carpeta de cursada 2023).
