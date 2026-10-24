---
tags:
  - tp4
---

# TP4 - Resolución del ejercicio 1

Se implementan con triggers los controles del TP3 que requieren consultar otras filas o conservar cambios anteriores. Se comprueba el estado **final de la sentencia** cuando la regla relaciona varias filas; así no se rechaza un cambio conjunto cuyo resultado final sea válido.

## Qué conserva implementación declarativa

| Regla del TP3 | Recurso |
|---|---|
| 3.a Nacionalidades permitidas | CHECK sobre nacionalidad |
| 3.b Publicación desde 2010 | CHECK sobre fecha_pub |
| 3.c Después de 2020, solo Argentina | CHECK que expresa la implicación |
| 5.a No más de 70 años cumplidos | CHECK al escribir fecha_nacimiento |

Para 5.a, la forma local es:

```sql
ALTER TABLE voluntario
ADD CONSTRAINT tp4_edad_maxima
CHECK (EXTRACT(YEAR FROM AGE(CURRENT_DATE, fecha_nacimiento)) <= 70);
```

No requiere consultar otras filas. Este control se evalúa ante escrituras; no se reevalúa solo al cumplir años. Un trigger tampoco se dispara por el paso del tiempo: la regla temporal permanente necesitaría una revisión adicional. No se presenta el CHECK como un control que actúe todos los días.

## Controles procedurales y eventos críticos

| Regla | Tabla | INSERT | UPDATE relevante | DELETE | Granularidad elegida |
|---|---|---|---|---|---|
| 3.d Máximo 15 palabras | contiene | Sí | id_articulo | No | AFTER STATEMENT |
| 3.e Si es argentino, máximo 10 | contiene | Sí | id_articulo | No | AFTER STATEMENT |
| 3.e Si es argentino, máximo 10 | articulo | No | nacionalidad | No | AFTER STATEMENT |
| 5.b Horas no superiores al coordinador | voluntario | Sí | horas_aportadas, id_coordinador, claves referenciadas | No, con las FK dadas | AFTER STATEMENT |
| 5.c Horas dentro de la tarea | voluntario | Sí | horas_aportadas, id_tarea | No | AFTER STATEMENT |
| 5.c Horas dentro de la tarea | tarea | No | min_horas, max_horas | No, con la FK dada | AFTER STATEMENT |
| 5.d Misma tarea que coordinador | voluntario | Sí | id_tarea, id_coordinador | No, con la FK dada | AFTER STATEMENT |
| 5.e Hasta 3 cambios por año | voluntario | No | id_institucion, si cambia realmente | No | AFTER ROW |

En ARTICULO, una nueva fila no tiene todavía asociaciones previas en CONTIENE si se respetan las FK. En TAREA, un alta tampoco tiene voluntarios referenciándola previamente. Los borrados indicados no crean la desigualdad; las referencias existentes quedan a cargo de las FK. Se consideran estables los identificadores de voluntarios y tareas en los cambios del práctico.

## Implementaciones

- [[TP4 - Resolución 1 - Máximos de palabras por artículo|Límites de palabras y cambio de nacionalidad]].
- [[TP4 - Resolución 1 - Horas de voluntarios y tareas|Horas del coordinado, del coordinador y límites de tarea]].
- [[TP4 - Resolución 1 - Misma tarea que el coordinador|Tarea del voluntario y de su coordinador]].
- [[TP4 - Resolución 1 - Cambios anuales de institución|Historial específico de cambios de institución]].

Fuente: `Practico 4 - Acciones Procedurales.docx`, TP3 y esquema correspondiente de la cátedra. Se usa `EXECUTE PROCEDURE`.
