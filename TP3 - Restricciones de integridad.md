---
tags:
  - tp3
---

# Práctico 3 — Restricciones de integridad declarativas

Se estudian restricciones declarativas: clasificar qué datos compara cada regla, expresarla según SQL-1999 y distinguir cuáles declaraciones admite PostgreSQL. Las limitaciones del motor se indican sin adelantar una implementación procedural.

Cuatro bloques: clasificar la regla, controlar referencias, expresar chequeos y analizar las operaciones del práctico.

## 1. [[Alcance y clasificación de restricciones|Fundamentos y clasificación de restricciones]]

Alcance de atributo, tupla, tabla y general; DOMAIN, PK, UNIQUE, NOT NULL y FK.

## 2. [[MATCH SIMPLE, FULL y PARTIAL|Integridad referencial y matching]]

Acciones ON DELETE/ON UPDATE y tratamiento de nulos con MATCH SIMPLE, FULL y PARTIAL.

[[MATCH SIMPLE, FULL y PARTIAL#Agregar restricciones de integridad referencial con ALTER TABLE|Ejemplos de ALTER TABLE para agregar FK simples y compuestas]], con identificación de tabla hija y padre y elección de acciones referenciales.

## 3. [[CHECK y lógica de tres valores|CHECK y ASSERTION]]

Chequeos de fila, lógica de tres valores, implicaciones, conteos y aserciones del estándar.

## 4. [[Análisis de operaciones y restricciones|Análisis de ejercicios y operaciones]]

Método para simular operaciones, distinguir casos acumulativos y reconocer qué datos necesita cada control.

## Resolución del práctico

[[Resolución del TP3 - Restricciones de integridad|Respuestas y consultas del TP3]]

Archivo para ir completando la resolución del práctico.

## Checklist del práctico

- [ ] Identificar alcance y recurso declarativo adecuado.
- [ ] Escribir FK compuestas y distinguir padre de hijo.
- [ ] Simular cascadas completas sin efectos parciales.
- [ ] Resolver SIMPLE/FULL/PARTIAL con nulos.
- [ ] Diferenciar operaciones independientes y acumulativas.
- [ ] Traducir implicaciones y negación de infractores.
- [ ] Explicar por qué CHECK acepta UNKNOWN.
- [ ] Separar SQL estándar de lo implementado por PostgreSQL.

## Conocimientos anteriores

- Relacionados: TP1 - Derivación y definición de tablas y contenidos de NULL y consultas del TP2.

## Criterio de estudio y fuentes

Las explicaciones se apoyan en este práctico y los anteriores. Se conserva la sintaxis de la cátedra; las diferencias de PostgreSQL se tratan por separado cuando las pide la consigna. Los ejemplos de teoría son didácticos y no sustituyen la resolución personal.

- Enunciado local: `../Practicos/Practico 3 - Restricciones de Integridad Declarativas.docx`.
- Filminas cotejadas: `BDI_03.P1 (Restricciones-1).pdf; BDI_03.P1 (Restricciones-2).pdf`, en `../Contenido extra/Filminas y Practicos de cursada/2023/`.
