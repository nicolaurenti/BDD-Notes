---
tags:
  - tp6
---

# Análisis de ejercicios de seguridad


| Ejercicio | Qué registrar |
|---|---|
| 1 | Grafo inicial, intento inválido de delegar DELETE y efecto de revocaciones por tabla/columna |
| 2 | Orden de operaciones, usuarios/roles ya existentes, privilegios de PUBLIC y capacidad de delegar |
| 3 | Permisos por usuario antes y después del CASCADE; no mezclar INSERT y SELECT |

Ejemplo del ejercicio 3: si A concede INSERT y SELECT a B con delegación, y SELECT a C sin ella, B puede delegar ambos; C solo puede consultar, salvo que obtenga otra concesión. Si B delega INSERT a D y A retira ese INSERT a B con CASCADE, D pierde el camino dependiente; los SELECT no se ven afectados por esa revocación de INSERT.
