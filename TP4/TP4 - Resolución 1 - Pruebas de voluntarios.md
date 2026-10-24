---
tags:
  - tp4
---

# TP4 - Resolución 1 - Pruebas de voluntarios

Estado inicial de cada caso: T1 permite 100…1000 horas y T2 permite 100…2000. El coordinador 10 hace T1 y aporta 900 horas; sus coordinados 11 y 12 hacen T1 y aportan 400 y 500. Otro coordinador, 20, hace T2 y aporta 800. Los demás atributos obligatorios y las referencias son válidos.

Cada caso se ejecuta independientemente y se revierte. Los errores indican la regla que rechaza; no se conserva un estado parcialmente modificado.

```sql
-- 5.b: el coordinado supera al coordinador. Rechaza RI_5B.
UPDATE voluntario SET horas_aportadas = 950 WHERE nro_voluntario = 11;

-- 5.b: disminuir al coordinador invalida a sus coordinados. Rechaza RI_5B.
UPDATE voluntario SET horas_aportadas = 300 WHERE nro_voluntario = 10;

-- 5.b: nuevo valor permitido. Se acepta.
UPDATE voluntario SET horas_aportadas = 450 WHERE nro_voluntario = 11;

-- 5.c: bajo el mínimo. Rechaza RI_5C.
UPDATE voluntario SET horas_aportadas = 50 WHERE nro_voluntario = 11;

-- 5.c: sobre el máximo, sin superar a ningún coordinador. Rechaza RI_5C.
UPDATE voluntario SET horas_aportadas = 1100 WHERE nro_voluntario = 10;

-- TAREA: máximo insuficiente. Rechaza RI_5C.
UPDATE tarea SET max_horas = 800 WHERE id_tarea = 'T1';

-- TAREA: mínimo que excluye al voluntario 11. Rechaza RI_5C.
UPDATE tarea SET min_horas = 450 WHERE id_tarea = 'T1';

-- TAREA: ampliación de rango. Se acepta.
UPDATE tarea SET min_horas = 90, max_horas = 1200 WHERE id_tarea = 'T1';
```

También se probaron INSERT válidos e inválidos para las tres reglas. Los controles actúan al insertar y al actualizar, y no dependen de que el cliente haga una consulta previa.
