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

## Fuente de estudio

Enunciado del TP7 en la carpeta `../../Practicos` y filminas de cátedra `BDI_07 (Optimizacion).pdf` (carpeta de cursada 2023).
