---
tags:
  - tp3
---

# Práctico 3 — Restricciones de integridad declarativas

Cuatro bloques: clasificar la regla, controlar referencias, expresar chequeos y analizar las operaciones del práctico.

## 1. [[Alcance y clasificación de restricciones|Fundamentos y clasificación de restricciones]]

Alcance de atributo, tupla, tabla y general; DOMAIN, PK, UNIQUE, NOT NULL y FK.

## 2. [[MATCH SIMPLE, FULL y PARTIAL|Integridad referencial y matching]]

Acciones ON DELETE/ON UPDATE y tratamiento de nulos con MATCH SIMPLE, FULL y PARTIAL.

## 3. [[CHECK y lógica de tres valores|CHECK y ASSERTION]]

Chequeos de fila, lógica de tres valores, implicaciones, conteos y aserciones del estándar.

## 4. [[Análisis de operaciones y restricciones|Análisis de ejercicios y operaciones]]

Método para simular operaciones, distinguir casos acumulativos y reconocer controles que requieren otros eventos.

## Checklist del práctico

- [ ] Identificar alcance y recurso declarativo adecuado.
- [ ] Escribir FK compuestas y distinguir padre de hijo.
- [ ] Simular cascadas completas sin efectos parciales.
- [ ] Resolver SIMPLE/FULL/PARTIAL con nulos.
- [ ] Diferenciar operaciones independientes y acumulativas.
- [ ] Traducir implicaciones y negación de infractores.
- [ ] Explicar por qué CHECK acepta UNKNOWN.
- [ ] Separar SQL estándar de lo implementado por PostgreSQL.

## Material

- Enunciado principal: `../Practicos/Practico 3 - Restricciones de Integridad Declarativas.docx`.
- Filminas de cátedra: `BDI_2021_04_Restricciones integridad-P1.pptx` y `P2.pptx`.
- Relacionados: TP1 - Derivación y definición de tablas, Valores NULL y lógica de tres valores, TP4 - Triggers.
