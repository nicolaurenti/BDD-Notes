---
tags:
  - tp8
---

# Práctico 8 — Concurrencia y recuperación

El práctico reúne dos problemas: que las transacciones concurrentes no produzcan interferencias inválidas y que una falla no deje efectos incompatibles con sus confirmaciones. Los ejemplos siguen schedules, bloqueos y el modelo de log y checkpoint presentado en las filminas.

## 1. [[Transacciones y propiedades ACID]]

## 2. [[Schedules y anomalías de concurrencia]]

## 3. [[Serializabilidad y grafo de precedencia]]

## 4. [[Bloqueos compartidos y exclusivos]]

## 5. [[Protocolo de dos fases y deadlocks]]

## 6. [[Recuperabilidad y abortos en cascada]]

## 7. [[MVCC y niveles de aislamiento]]

## 8. [[Log, UNDO, REDO y WAL]]

## 9. [[Actualización diferida e inmediata y políticas de buffers]]

## 10. [[Checkpoints y transacciones activas]]

## 11. [[Análisis de logs y cortes ante fallas]]

## Checklist del práctico

- [ ] Construir schedules de actualización perdida y lectura sucia.
- [ ] Detectar conflictos y armar el grafo de precedencia completo.
- [ ] Obtener órdenes topológicos o justificar ciclos.
- [ ] Simular esperas y detectar adquisición posterior al primer unlock.
- [ ] Verificar las dos fases y distinguir espera, deadlock e inanición.
- [ ] Separar serializabilidad, recuperabilidad y deadlock.
- [ ] Distinguir niveles del estándar y comportamiento PostgreSQL según el material.
- [ ] Distinguir valor anterior/nuevo, COMMIT y páginas persistidas.
- [ ] Relacionar STEAL/FORCE con UNDO/REDO.
- [ ] Evaluar checkpoints y cortes usando solo el prefijo disponible.

## Conocimientos anteriores

- Relacionados: TP4 - Triggers, TP3 - Restricciones de integridad, TP7 - Optimización.

## Criterio de estudio y fuentes

Las explicaciones se apoyan en este práctico y los anteriores. Se conserva la sintaxis de la cátedra; las diferencias de PostgreSQL se tratan por separado cuando las pide la consigna. Los ejemplos de teoría son didácticos y no sustituyen la resolución personal.

- Enunciado local: `../Practicos/Practico 8 - Concurrencia y Recuperacion.docx`.
- Filminas cotejadas: `BDI_08 (Proc_Transacciones).pdf`, en `../Contenido extra/Filminas y Practicos de cursada/2023/`.
