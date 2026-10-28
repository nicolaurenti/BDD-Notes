---
tags:
  - tp7
---

# 5. Agrupamiento y conteos

## Idea central

GROUP BY reúne filas que comparten claves; los agregados resumen cada grupo. WHERE filtra filas antes del agrupamiento; HAVING filtra grupos según lo calculado.

## Cómo razonarlo

Primero identificar la unidad: ¿personas, participaciones o grupos distintos? Después comprobar si los joins multiplican esa unidad.

| Expresión | Qué cuenta |
|---|---|
| COUNT(*) | Todas las filas del grupo. |
| COUNT(columna) | Filas donde esa columna no es NULL. |
| COUNT(DISTINCT columna) | Valores distintos no NULL de esa columna. |

Una restricción como «más de dos grupos exclusivos» significa tres o más; filtrar por tipo debe ocurrir antes del conteo. Las claves compuestas deben conservarse completas para no mezclar usuarios diferentes.

## Ejemplo

```sql
SELECT i.tipo_usuario, i.cod_usuario
FROM integra i
JOIN gr_exclusivo ge ON ge.nro_grupo = i.nro_grupo
GROUP BY i.tipo_usuario, i.cod_usuario
HAVING COUNT(*) > 2;
```

Si la PK de integra es (tipo_usuario,cod_usuario,nro_grupo) y gr_exclusivo tiene una fila por grupo, cada fila del join representa una pertenencia a un grupo exclusivo distinto. Un usuario con tres aparece; uno con dos no.

Puede agruparse esta relación estrecha primero y recién después unir con usuario para obtener apellido y nombre. Así se evita transportar nombres por todo el agrupamiento. Es una propuesta de reducción de trabajo, no una garantía de un plan más rápido.

Con LEFT JOIN, un padre sin hijos conserva una fila extendida con NULL: COUNT(*) da uno y COUNT(clave_hijo) da cero. Esa diferencia es importante para consultas que incluyen entidades sin relaciones.

## Errores frecuentes

Contar filas multiplicadas por otro join sobrecuenta entidades. COUNT(DISTINCT ...) puede expresar el conteo correcto, pero no sustituye analizar la causa. Mover condiciones de HAVING a WHERE solo es válido si conserva la semántica; una condición sobre el agregado no puede evaluarse sobre cada fila individual.

## Qué recordar

**Filtrar lo pertinente → contar la unidad correcta → aplicar el umbral → recuperar datos descriptivos.** [Referencia técnica](https://www.postgresql.org/docs/15/queries-table-expressions.html).
