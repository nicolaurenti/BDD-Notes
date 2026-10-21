---
tags:
  - tp2
---

# Operadores de conjuntos

Combinan los **resultados de dos consultas**. No unen columnas como un JOIN: comparan filas completas y devuelven filas con la misma estructura que las consultas de entrada.

| Operador | Resultado |
|---|---|
| `UNION` | Filas presentes en cualquiera de las consultas. |
| `INTERSECT` | Filas presentes en ambas consultas. |
| `EXCEPT` | Filas de la primera consulta que no aparecen en la segunda. |

Las consultas deben devolver:

- La misma cantidad de columnas.
- Tipos de datos compatibles en las posiciones correspondientes.

Los nombres de las columnas no necesitan ser iguales. Si se seleccionan varias columnas, se compara la **fila completa**, no cada columna de manera independiente.

## Ejemplos de pertenencia y duplicados

Las operaciones se entienden comparando pertenencia y cantidad de apariciones. Sin ALL se eliminan repetidos; con ALL se suman, toman mínimos o restan apariciones según el operador.

[[TP2 - Ejemplos de operadores de conjuntos|Ejemplos de operadores de conjuntos]]

## Nulos, orden y errores frecuentes

- Estos operadores comparan filas para pertenencia y duplicación; no usan la igualdad ordinaria con NULL como un ON. Por ejemplo, una fila `(NULL)` en ambas entradas produce una fila `(NULL)` con INTERSECT y ninguna con EXCEPT.
- Sin ORDER BY, ningún operador garantiza el orden del resultado; tampoco UNION ALL garantiza el orden de las entradas. Los ejemplos ordenan al final para facilitar la comparación.
- DISTINCT y ALL consideran la fila completa: `(1, 'Ana')` y `(1, 'Luis')` son filas diferentes.
- EXCEPT no funciona como NOT IN cuando intervienen nulos: no trasladar automáticamente las reglas de UNKNOWN de una comparación a los operadores de conjuntos.
- Al mezclar operadores, usar paréntesis para expresar la intención. INTERSECT tiene mayor precedencia que UNION y EXCEPT.

## Comprobación rápida

- [ ] ¿Las dos consultas devuelven la misma cantidad de columnas y tipos compatibles?
- [ ] ¿Quiero reunir, conservar coincidencias o excluir?
- [ ] ¿Importa cuántas veces aparece una fila? Si importa, evaluar ALL.
- [ ] ¿Está en el orden correcto la primera entrada de EXCEPT?
- [ ] ¿Estoy comparando las filas completas que pide el enunciado?

## Fuente de estudio

Enunciado del TP2 en la carpeta `../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
