---
tags:
  - tp4
---

# Práctico 4 — Acciones procedurales y triggers

Se parte de las restricciones del TP3 para elegir eventos críticos y escribir controles procedurales. También se estudian auditoría, historial y resúmenes. Los ejemplos siguen la cabecera EXECUTE PROCEDURE del enunciado y explican estado previo, ejecución y efecto.

## 1. [[Fundamentos y ejecución de triggers]]

## 2. [[Validación de restricciones con triggers]]

## 3. [[Auditoría, historial y resúmenes con triggers]]

## Resolución

[[Resolución del TP4 - Triggers]]

## Checklist del práctico

- [ ] Elegir momento, evento y granularidad.
- [ ] Usar OLD/NEW y el retorno apropiado.
- [ ] Modificar NEW sin actualizaciones recursivas innecesarias.
- [ ] Completar la grilla de eventos de todas las tablas afectadas.
- [ ] Diferenciar validación, auditoría e historial.
- [ ] Resolver carga inicial y mantenimiento posterior.
- [ ] Simular varias filas sin asumir un orden no garantizado.
- [ ] Justificar los eventos críticos y comprobar los conteos en INSERT y UPDATE.

## Conocimientos anteriores

- Relacionados: TP3 - Restricciones de integridad; consultas y nulos del TP2.

## Criterio de estudio y fuentes

Las explicaciones se apoyan en este práctico y los anteriores. Se conserva la sintaxis de la cátedra; las diferencias de PostgreSQL se tratan por separado cuando las pide la consigna. Los ejemplos de teoría son didácticos y no sustituyen la resolución personal.

- Enunciado local: `../Practicos/Practico 4 - Acciones Procedurales.docx`.
- Filminas cotejadas: `BDI_04 (SQL Procedural).pdf`, en `../Contenido extra/Filminas y Practicos de cursada/2023/`.
