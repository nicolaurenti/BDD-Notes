---
tags:
  - tp2
---

# TP2 - Conteos de entidades relacionadas y ausentes

Consultas para incluir entidades sin relacionados y evitar multiplicaciones.

## Conteos de joins

### Conservar entidades sin relacionados

Cantidad de voluntarios de cada institución, incluyendo las que tienen cero:

```sql
SELECT
    i.id_institucion,
    COUNT(v.nro_voluntario) AS cantidad
FROM institucion i
LEFT JOIN voluntario v
    ON v.id_institucion = i.id_institucion
GROUP BY i.id_institucion;
```

Se cuenta la PK del voluntario:

- Sin voluntarios, `COUNT(v.nro_voluntario)` devuelve `0`.
- Con `COUNT(*)`, se contaría la fila extendida con nulos y devolvería `1`.

### Evitar multiplicaciones

Unir un voluntario con varias filas históricas genera varias filas para ese voluntario.

Si se quieren contar personas diferentes:

```sql
COUNT(DISTINCT v.nro_voluntario)
```

> [!important]
> Antes de agregar, identificar qué representa cada fila del resultado del join: una persona, una tarea, una entrega o una combinación.

## Fuente de estudio

Enunciado del TP2 en la carpeta `../../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
