---
tags:
  - tp2
---

# TP2 - Ejemplos de operadores de conjuntos

Se usan las mismas entradas A y B en todos los casos: primero identificá las zonas de pertenencia y después compará las operaciones que siguen.

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

### Ejemplo de UNION

Entradas del ejemplo: A = [1, 2, 2, 3] y B = [2, 3, 4]. Los resultados se ordenan para facilitar su lectura.

```sql
SELECT id_voluntario FROM grupo_a
UNION
SELECT id_voluntario FROM grupo_b
ORDER BY id_voluntario;
```

**Resultado:**

| id_voluntario |
|---|
| 1 |
| 2 |
| 3 |
| 4 |

Reúne ambas entradas y elimina duplicados, incluidos los que ya estaban repetidos dentro de una misma entrada. El 2 aparece una sola vez.

```text
A: [1, 2, 2, 3]  ─┐
                   ├─ UNION ───► [1, 2, 3, 4]
B: [2, 3, 4]     ─┘
```

## INTERSECT — solo lo que aparece en ambos

INTERSECT conserva las filas presentes en ambas entradas.

### Ejemplo de INTERSECT

Entradas del ejemplo: A = [1, 2, 2, 3] y B = [2, 3, 4]. Los resultados se ordenan para facilitar su lectura.

```sql
SELECT id_voluntario FROM grupo_a
INTERSECT
SELECT id_voluntario FROM grupo_b
ORDER BY id_voluntario;
```

**Resultado:**

| id_voluntario |
|---|
| 2 |
| 3 |

El 1 queda afuera porque solo está en A; el 4, porque solo está en B. El 2 no se repite aunque aparezca dos veces en A.

```text
A: [1, 2, 2, 3]  ─┐
                   ├─ INTERSECT ► [2, 3]
B: [2, 3, 4]     ─┘
```

## EXCEPT — lo que está en la primera y no en la segunda

EXCEPT conserva lo exclusivo de la primera entrada: invertirlas cambia la pregunta.

### Ejemplo de EXCEPT

Entradas del ejemplo: A = [1, 2, 2, 3] y B = [2, 3, 4]. Los resultados se ordenan para facilitar su lectura.

```sql
SELECT id_voluntario FROM grupo_a
EXCEPT
SELECT id_voluntario FROM grupo_b
ORDER BY id_voluntario;
```

**Resultado:**

| id_voluntario |
|---|
| 1 |

Los identificadores 2 y 3 se excluyen porque aparecen en B. El 4 no se incorpora: no estaba en la primera entrada.

```text
A: [1, 2, 2, 3]  ─┐
                   ├─ A EXCEPT B ► [1]
B: [2, 3, 4]     ─┘
```

**En EXCEPT importa el orden:**

```sql
SELECT id_voluntario FROM grupo_b
EXCEPT
SELECT id_voluntario FROM grupo_a
ORDER BY id_voluntario;
```

Devuelve únicamente el **4**. En cambio, invertir las entradas de UNION o INTERSECT no cambia sus filas resultantes.

#### Ejemplo del práctico con EXCEPT

Tareas desarrolladas históricamente que no se realizan actualmente:

```sql
SELECT id_tarea
FROM historico

EXCEPT

SELECT id_tarea
FROM voluntario;
```

La primera consulta aporta las tareas históricas; la segunda excluye las que algún voluntario realiza actualmente. El resultado contiene identificadores, sin repetirlos: no los datos completos de cada tarea.

## ALL — trabajar con cantidades de apariciones

Con ALL se cuenta cuántas veces aparece cada fila: suma, mínimo o resta sin bajar de cero.

### Ejemplos de UNION ALL, INTERSECT ALL y EXCEPT ALL

Entradas del ejemplo: A = [1, 2, 2, 3] y B = [2, 3, 4]. Los resultados se ordenan para facilitar su lectura.

Sin `ALL`, estos operadores eliminan duplicados. Con `ALL`, importa cuántas veces aparece cada fila.

| Operador | Cantidad de apariciones de una fila |
|---|---|
| `UNION ALL` | Suma las apariciones de A y B. |
| `INTERSECT ALL` | Toma el mínimo de las apariciones en A y B. |
| `EXCEPT ALL` | Resta las de B a las de A, sin bajar de cero. |

#### Ejemplos SQL

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

#### Visualizar el conteo por valor

Cada celda indica **cuántas filas** con ese identificador hay, no el identificador en sí.

| Identificador | En A | En B | UNION ALL | INTERSECT ALL | A EXCEPT ALL B |
|---|---:|---:|---:|---:|---:|
| 1 | 1 | 0 | 1 | 0 | 1 |
| 2 | 2 | 1 | 3 | 1 | 1 |
| 3 | 1 | 1 | 2 | 1 | 0 |
| 4 | 0 | 1 | 1 | 0 | 0 |

En este ejemplo INTERSECT e INTERSECT ALL coinciden. Si el 2 apareciera dos veces en ambas entradas, INTERSECT devolvería un 2 e INTERSECT ALL devolvería dos.

## Fuente de estudio

Enunciado del TP2 en la carpeta `../../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
