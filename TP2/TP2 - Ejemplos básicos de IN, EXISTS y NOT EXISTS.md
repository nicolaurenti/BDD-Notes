---
tags:
  - tp2
---

# TP2 - Ejemplos básicos de IN, EXISTS y NOT EXISTS

Consultas breves para distinguir las tres preguntas.

#### IN

Comprueba si un valor pertenece al resultado:

```sql
SELECT *
FROM voluntario
WHERE nro_voluntario IN (
    SELECT nro_voluntario
    FROM historico
);
```

#### EXISTS

Comprueba si la subconsulta devuelve al menos una fila:

```sql
SELECT *
FROM institucion i
WHERE EXISTS (
    SELECT 1
    FROM voluntario v
    WHERE v.id_institucion = i.id_institucion
);
```

#### NOT EXISTS

Comprueba que no exista ninguna fila:

```sql
SELECT *
FROM institucion i
WHERE NOT EXISTS (
    SELECT 1
    FROM voluntario v
    WHERE v.id_institucion = i.id_institucion
);
```

## Fuente de estudio

Enunciado del TP2 en la carpeta `../../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
