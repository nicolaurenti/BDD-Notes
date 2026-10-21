---
tags:
  - tp2
---

# TP2 - Ejemplos de UNION ALL, INTERSECT ALL y EXCEPT ALL

Entradas del ejemplo: A = [1, 2, 2, 3] y B = [2, 3, 4]. Los resultados se ordenan para facilitar su lectura.

## ALL — trabajar con cantidades de apariciones

Sin `ALL`, estos operadores eliminan duplicados. Con `ALL`, importa cuántas veces aparece cada fila.

| Operador | Cantidad de apariciones de una fila |
|---|---|
| `UNION ALL` | Suma las apariciones de A y B. |
| `INTERSECT ALL` | Toma el mínimo de las apariciones en A y B. |
| `EXCEPT ALL` | Resta las de B a las de A, sin bajar de cero. |

### Ejemplos SQL

```sql
SELECT id_voluntario FROM grupo_a
UNION ALL
SELECT id_voluntario FROM grupo_b
ORDER BY id_voluntario;
-- Resultado: 1, 2, 2, 2, 3, 3, 4.
```

```sql
SELECT id_voluntario FROM grupo_a
INTERSECT ALL
SELECT id_voluntario FROM grupo_b
ORDER BY id_voluntario;
-- Resultado: 2, 3.
```

```sql
SELECT id_voluntario FROM grupo_a
EXCEPT ALL
SELECT id_voluntario FROM grupo_b
ORDER BY id_voluntario;
-- Resultado: 1, 2.
```

En `EXCEPT ALL`, B tiene una aparición del 2 y A tiene dos: se descuenta una y queda otra. Con `EXCEPT` sin ALL no quedaría ningún 2.

### Visualizar el conteo por valor

Cada celda indica **cuántas filas** con ese identificador hay, no el identificador en sí.

| Identificador | En A | En B | UNION ALL | INTERSECT ALL | A EXCEPT ALL B |
|---|---:|---:|---:|---:|---:|
| 1 | 1 | 0 | 1 | 0 | 1 |
| 2 | 2 | 1 | 3 | 1 | 1 |
| 3 | 1 | 1 | 2 | 1 | 0 |
| 4 | 0 | 1 | 1 | 0 | 0 |

En este ejemplo INTERSECT e INTERSECT ALL coinciden. Si el 2 apareciera dos veces en ambas entradas, INTERSECT devolvería un 2 e INTERSECT ALL devolvería dos.

## Fuente de estudio

Enunciado del TP2 en la carpeta `../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
