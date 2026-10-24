---
tags:
  - tp4
---

# Validación de restricciones con triggers

El trabajo comienza por la regla y sus **eventos críticos**, no por escribir CREATE TRIGGER. Un evento es crítico si puede convertir un estado válido en inválido. La misma restricción puede necesitar controles en varias tablas.

## Elegir los eventos críticos

Para completar la grilla se enumeran todas las tablas y modificaciones capaces de romper la regla, incluidas las de los extremos de una relación.

[[TP4 - Ejemplo de grilla de eventos críticos|Ejemplo de grilla de eventos críticos]]

## Validar conteos por grupo

En un máximo por grupo, el conteo cambia según el momento del trigger: AFTER observa la fila incorporada; BEFORE todavía no. También se controla mover una asociación a otro grupo.

[[TP4 - Ejemplo de máximo de palabras por artículo|Ejemplo de máximo de palabras por artículo]]

## Comparar y corregir la fila propuesta

OLD y NEW permiten reconocer cambios reales. BEFORE puede corregir NEW directamente; la prioridad entre reglas se expresa con IF y ELSIF para que una no rechace el efecto obligatorio de otra.

[[TP4 - Ejemplo de cambio de tarea y variación de horas|Ejemplo de cambio de tarea y variación de horas]]

## Comprobar el diseño

Para cada regla preparar un caso válido, uno inválido y uno en el límite. Además de insertar, modificar las columnas y tablas relacionadas: una prueba que nunca cambia TAREA no detecta la falta de ese control.

En la prueba, anotar estado previo, sentencia, resultado esperado y estado final. Un UPDATE con varias filas debe mantener la regla sobre todas; no probar solamente operaciones de una fila. Los fragmentos son material de estudio y no fueron instalados ni ejecutados en las bases del proyecto.

## Fuente de estudio

Enunciado del TP4 en la carpeta `../../Practicos` y filminas de cátedra `BDI_04 (SQL Procedural).pdf` (carpeta de cursada 2023).
