---
tags:
  - tp2
---

# TP2 - Ejemplo de UNION

Entradas del ejemplo: A = [1, 2, 2, 3] y B = [2, 3, 4]. Los resultados se ordenan para facilitar su lectura.

## UNION — todo lo que aparece en alguno

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

## Fuente de estudio

Enunciado del TP2 en la carpeta `../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
