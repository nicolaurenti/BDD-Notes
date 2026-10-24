---
tags:
  - tp2
---

# TP2 - Ejemplo de EXCEPT

Entradas del ejemplo: A = [1, 2, 2, 3] y B = [2, 3, 4]. Los resultados se ordenan para facilitar su lectura.

## EXCEPT — lo que está en la primera y no en la segunda

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

### Ejemplo del práctico con EXCEPT

Tareas desarrolladas históricamente que no se realizan actualmente:

```sql
SELECT id_tarea
FROM historico

EXCEPT

SELECT id_tarea
FROM voluntario;
```

La primera consulta aporta las tareas históricas; la segunda excluye las que algún voluntario realiza actualmente. El resultado contiene identificadores, sin repetirlos: no los datos completos de cada tarea.

## Fuente de estudio

Enunciado del TP2 en la carpeta `../../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
