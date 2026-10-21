---
tags:
  - tp2
---

# Subconsultas y condiciones de ausencia

## Subconsultas

Una subconsulta es una consulta incluida dentro de otra.

### Pertenencia y existencia

IN compara pertenencia; EXISTS pregunta si hay alguna fila y NOT EXISTS si no hay ninguna. La correlación define de qué entidad se está comprobando esa existencia.

[[TP2 - Ejemplos básicos de IN, EXISTS y NOT EXISTS|Ejemplos básicos de IN, EXISTS y NOT EXISTS]]

#### Subconsulta correlacionada

Hace referencia a una fila de la consulta exterior.

En los ejemplos anteriores, la condición:

```sql
v.id_institucion = i.id_institucion
```

relaciona la subconsulta con la institución que se está evaluando.

### Nulidad en IN y NOT IN

Un NULL dentro de NOT IN puede impedir confirmar la ausencia de coincidencias. Filtrar nulos o usar NOT EXISTS requiere decidir también qué hacer con nulos exteriores.

[[TP2 - NOT IN y resultados nulos|NOT IN y resultados nulos]]

## Ningún relacionado incumple

Para exigir que no haya incumplimientos se buscan los casos inválidos y se exige su ausencia. Si también se necesita algún relacionado, agregar una prueba de existencia.

[[TP2 - Ausencia de incumplimientos con NOT EXISTS|Ausencia de incumplimientos con NOT EXISTS]]

## Razonar la correlación y la ausencia

La correlación vincula la subconsulta con la fila exterior. Sin ella, se evalúa una existencia global en vez de la de cada entidad.

[[TP2 - Correlación y condiciones alguno, ninguno y solo|Correlación y condiciones alguno, ninguno y solo]]

## Fuente de estudio

Enunciado del TP2 en la carpeta `../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
