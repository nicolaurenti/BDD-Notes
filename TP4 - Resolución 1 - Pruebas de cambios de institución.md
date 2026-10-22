---
tags:
  - tp4
---

# TP4 - Resolución 1 - Pruebas de cambios de institución

Se parte de voluntario 11 en institución 1, sin cambios registrados en el año actual y con instituciones 2, 3 y 4 existentes. Los demás datos cumplen las reglas de horas y tarea.

En una misma transacción:

```sql
UPDATE voluntario SET id_institucion = 2 WHERE nro_voluntario = 11;
UPDATE voluntario SET id_institucion = 3 WHERE nro_voluntario = 11;
UPDATE voluntario SET id_institucion = 4 WHERE nro_voluntario = 11;
```

Las tres se aceptan; el historial contiene `(11, hoy, 1, 2)`, `(11, hoy, 2, 3)` y `(11, hoy, 3, 4)`.

```sql
-- Cuarto cambio en el año: rechaza RI_5E.
UPDATE voluntario SET id_institucion = 1 WHERE nro_voluntario = 11;
```

Si se ejecutan esas cuatro instrucciones dentro de BEGIN, el error deja la transacción abortada: ROLLBACK revierte también las tres anteriores. Para observar tres confirmadas, realizar COMMIT después de la tercera y probar la cuarta en otra transacción. En ambos casos el cuarto cambio nunca queda registrado como válido.

Otros casos independientes, partiendo de institución 1:

```sql
-- Mismo valor: se acepta y no registra un cambio.
UPDATE voluntario SET id_institucion = 1 WHERE nro_voluntario = 11;

-- Con la política declarada, estos dos pasos cuentan dos cambios.
UPDATE voluntario SET id_institucion = NULL WHERE nro_voluntario = 11;
UPDATE voluntario SET id_institucion = 2 WHERE nro_voluntario = 11;
```

Tres eventos del año anterior no consumen el límite del actual. Se verificó que el filtro de año permite tres cambios nuevos aunque existan esos tres registros previos.
