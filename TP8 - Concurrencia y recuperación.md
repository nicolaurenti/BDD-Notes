---
tags:
  - tp8
---

# Práctico 8 — Concurrencia y recuperación

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
- [ ] Diferenciar 2PL básico, estricto y riguroso.
- [ ] Separar serializabilidad, recuperabilidad y deadlock.
- [ ] Explicar MVCC y niveles reales de PostgreSQL.
- [ ] Distinguir valor anterior/nuevo, COMMIT y páginas persistidas.
- [ ] Relacionar STEAL/FORCE con UNDO/REDO.
- [ ] Evaluar checkpoints y cortes usando solo el prefijo disponible.

## Material

- Enunciado principal: `../Practicos/Practico 8 - Concurrencia y Recuperacion.docx`.
- Relacionados: TP4 - Triggers, TP3 - Restricciones de integridad, TP7 - Optimización.
