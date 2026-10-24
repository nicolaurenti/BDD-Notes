---
tags:
  - tp2
---

# TP2 - Ejemplos de operadores de conjuntos

Se usan las mismas entradas A y B en todos los casos: primero identificá las zonas de pertenencia y después abrí la operación que necesites.

## Tablas de ejemplo

Supongamos que estas dos consultas devuelven los identificadores de dos grupos de voluntarios:

**Tabla `grupo_a`:**

| id_voluntario |
|---|
| 1 |
| 2 |
| 2 |
| 3 |

**Tabla `grupo_b`:**

| id_voluntario |
|---|
| 2 |
| 3 |
| 4 |

El 2 está repetido en A para observar qué sucede con los duplicados.

### Mapa de pertenencia

| Solo en A | En A y en B | Solo en B |
|:---:|:---:|:---:|
| 1 | 2, 3 | 4 |

Pensá en tres zonas: izquierda exclusiva, zona compartida y derecha exclusiva. Sin ALL, cada identificador aparece una sola vez en el resultado.

| Operación | Solo en A: 1 | Compartidos: 2 y 3 | Solo en B: 4 |
|---|:---:|:---:|:---:|
| A UNION B | Incluye | Incluye | Incluye |
| A INTERSECT B | Excluye | Incluye | Excluye |
| A EXCEPT B | Incluye | Excluye | Excluye |
| B EXCEPT A | Excluye | Excluye | Incluye |

## UNION — todo lo que aparece en alguno

UNION reúne las dos entradas y elimina duplicados; UNION ALL conserva sus apariciones.

[[TP2 - Ejemplo de UNION|Ejemplo de UNION]]

## INTERSECT — solo lo que aparece en ambos

INTERSECT conserva las filas presentes en ambas entradas.

[[TP2 - Ejemplo de INTERSECT|Ejemplo de INTERSECT]]

## EXCEPT — lo que está en la primera y no en la segunda

EXCEPT conserva lo exclusivo de la primera entrada: invertirlas cambia la pregunta.

[[TP2 - Ejemplo de EXCEPT|Ejemplo de EXCEPT]]

## ALL — trabajar con cantidades de apariciones

Con ALL se cuenta cuántas veces aparece cada fila: suma, mínimo o resta sin bajar de cero.

[[TP2 - Ejemplos de UNION ALL, INTERSECT ALL y EXCEPT ALL|Ejemplos de UNION ALL, INTERSECT ALL y EXCEPT ALL]]

## Fuente de estudio

Enunciado del TP2 en la carpeta `../../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
