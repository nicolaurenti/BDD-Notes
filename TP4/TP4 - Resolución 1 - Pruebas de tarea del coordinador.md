---
tags:
  - tp4
---

# TP4 - Resolución 1 - Pruebas de tarea del coordinador

Estado inicial: coordinador 10 con tarea T1; coordinados 11 y 12 también con T1; coordinador 20 con T2. T1 admite 100…1000 horas y T2 admite 100…2000. Aportes: 900, 400, 500 y 800 respectivamente.

Cada caso parte de esa instancia, con los demás atributos y referencias válidos, y se revierte después de probarlo.

```sql
-- Rechaza: cambia solo la tarea de un coordinado.
UPDATE voluntario SET id_tarea = 'T2' WHERE nro_voluntario = 11;

-- Rechaza: cambia el coordinador y deja a sus coordinados con otra tarea.
UPDATE voluntario SET id_tarea = 'T2' WHERE nro_voluntario = 10;

-- Rechaza: reasigna a alguien cuyo coordinador nuevo hace otra tarea.
UPDATE voluntario SET id_coordinador = 20 WHERE nro_voluntario = 11;

-- Acepta: una sentencia cambia el grupo completo y deja consistencia final.
UPDATE voluntario SET id_tarea = 'T2'
WHERE nro_voluntario IN (10, 11, 12);
```

El rechazo debe identificar RI_5D. Los aportes elegidos no infringen los límites de horas, de modo que el caso aísla la regla de tarea.
