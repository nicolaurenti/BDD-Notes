---
tags:
  - tp7
---

# Práctico 7 — Optimización de consultas

Se estudia cómo obtener una estrategia de ejecución más eficiente sin cambiar el resultado. Las notas desarrollan las heurísticas de la cátedra, la lectura de EXPLAIN y la justificación de índices para las consultas del enunciado.

## 1. [[Equivalencia y rendimiento de consultas]]

## 2. [[EXPLAIN y EXPLAIN ANALYZE]]

## 3. [[Nodos de acceso y algoritmos de join]]

## 4. [[Índices y orden de columnas]]

## 5. [[Filtros e índices en fechas y strings]]

## 6. [[Reescritura de consultas con conteos y EXISTS]]

## 7. [[Análisis de planes y fechas del TP7]]

## 8. [[Método de comparación de rendimiento]]

## Checklist del práctico

- [ ] Interpretar costo, filas, ancho, tiempos y loops.
- [ ] Diferenciar EXPLAIN de EXPLAIN ANALYZE.
- [ ] Explicar accesos, algoritmos de join y agregados.
- [ ] Justificar selectividad y orden de índices compuestos.
- [ ] Distinguir bitmap de plan e índice bitmap persistente.
- [ ] Quitar joins solo con garantías de equivalencia.
- [ ] Usar EXISTS cuando interesa existencia y no multiplicación.
- [ ] Evitar comparaciones de rendimiento sin mediciones.

## Conocimientos anteriores

- Relacionados: TP2 - Consultas SQL, Subconsultas, DISTINCT, TP3 - Restricciones de integridad.

## Criterio de estudio y fuentes

Las explicaciones se apoyan en este práctico y los anteriores. Se conserva la sintaxis de la cátedra; las diferencias de PostgreSQL se tratan por separado cuando las pide la consigna. Los ejemplos de teoría son didácticos y no sustituyen la resolución personal.

- Enunciado local: `../Practicos/Practico 7 - Optimización de Consultas.docx`.
- Filminas cotejadas: `BDI_07 (Optimizacion).pdf`, en `../Contenido extra/Filminas y Practicos de cursada/2023/`.
