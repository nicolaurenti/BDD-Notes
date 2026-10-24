---
tags:
  - tp2
---

# TP2 - Ejemplo de INTERSECT

Entradas del ejemplo: A = [1, 2, 2, 3] y B = [2, 3, 4]. Los resultados se ordenan para facilitar su lectura.

## INTERSECT — solo lo que aparece en ambos

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

## Fuente de estudio

Enunciado del TP2 en la carpeta `../../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
