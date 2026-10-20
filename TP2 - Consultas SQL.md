---
tags:
  - tp2
---

# Práctico 2 — Consultas SQL

Los contenidos están agrupados en ocho bloques para estudiar temas relacionados juntos. Cada bloque corresponde a una única nota del mapa.

## 1. [[Estructura general de una consulta|Estructura, ejecución y presentación de resultados]]

SELECT, orden lógico, DISTINCT, ORDER BY, primeros N y paginación.

## 2. [[Condiciones y operadores|Condiciones, textos y fechas]]

Comparaciones, AND/OR/NOT, patrones de texto, funciones de strings, fechas y edades.

## 3. [[Valores NULL y lógica de tres valores|Valores NULL y lógica de tres valores]]

UNKNOWN, IS NULL, COALESCE y resultados del ejercicio 2.1.

## 4. [[Funciones de agregación|Agregaciones, agrupamiento y porcentajes]]

COUNT/SUM/AVG, GROUP BY/HAVING, promedios condicionales y comparación con totales.

## 5. [[Diferencias entre joins|Joins y conteos de filas relacionadas]]

INNER/OUTER JOIN, autojoins, filtros en ON/WHERE y conteos sin duplicaciones.

## 6. [[Subconsultas|Subconsultas y condiciones de ausencia]]

IN, EXISTS, NOT EXISTS y exclusión de entidades cuando existe un incumplimiento.

## 7. [[Operadores de conjuntos|Operadores de conjuntos]]

UNION, INTERSECT, EXCEPT, ALL y visualización de entradas y resultados.

## 8. [[Consultas con “todos”|Consultas con “todos”]]

División relacional y patrón de doble NOT EXISTS.

## Resolución del práctico

[[Resolución del TP2 - Consultas SQL|Consultas y respuestas de Grabaciones, Voluntarios y Películas]]

Recopilación de las resoluciones compartidas, conservando las consultas y anotaciones originales.

## Checklist del práctico

- [ ] Manejar strings y patrones con `LIKE`.
- [ ] Manejar fechas, edades e intervalos.
- [ ] Entender `NULL` y la lógica de tres valores.
- [ ] Distinguir `COUNT(*)`, `COUNT(columna)` y `COUNT(DISTINCT columna)`.
- [ ] Calcular promedios condicionales y porcentajes.
- [ ] Diferenciar `WHERE` y `HAVING`.
- [ ] Agrupar por una o varias columnas.
- [ ] Utilizar `DISTINCT` correctamente.
- [ ] Diferenciar los tipos de joins.
- [ ] Comprender el efecto de filtrar en `ON` o en `WHERE`.
- [ ] Resolver autojoins.
- [ ] Contar entidades sin multiplicarlas por los joins.
- [ ] Incluir entidades sin relacionados.
- [ ] Resolver subconsultas con `IN`, `EXISTS` y `NOT EXISTS`.
- [ ] Reconocer el problema de `NOT IN` con nulos.
- [ ] Resolver condiciones de “ninguno”, “solo” y “todos”.
- [ ] Utilizar operadores de conjuntos y entender `ALL`.
- [ ] Comparar cantidades de grupos con totales generales.
- [ ] Ordenar, limitar y paginar resultados.
- [ ] Explicar el orden lógico de una consulta.
## Repaso del ejercicio 2.1 — Valores nulos

Resultados y errores frecuentes en [[Valores NULL y lógica de tres valores#Ejercicio 2.1: cuatro equipos con puntos y descripción nulos]]. Para el apartado b, ver [[Funciones de agregación#COALESCE: reemplazar el resultado o los datos de entrada|Agregaciones — COALESCE]].
