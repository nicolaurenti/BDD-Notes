---
tags:
  - tp3
---

# TP3 - Ejemplos de restricciones de tabla

Se comparan la unicidad de un correo y el máximo de palabras por artículo.

## 3. Restricción de tabla

Para decidir si una fila es válida, se consideran **otras filas de la misma tabla**.

### Unicidad con UNIQUE — PostgreSQL

```sql
CREATE TABLE usuario_ejemplo (
    id_usuario INTEGER PRIMARY KEY,
    email VARCHAR(120) NOT NULL,
    CONSTRAINT uq_usuario_email UNIQUE (email)
);
```

- Primera fila con `ana@ejemplo.com`: aceptada.
- Otra fila con distinto id pero el mismo email: rechazada.

La regla no compara dos atributos de una fila: necesita garantizar que no haya **otra fila** con ese email. UNIQUE es el recurso declarativo específico para esta regla.

### Límite de filas por grupo — ejemplo del práctico

«Cada artículo puede tener como máximo 15 palabras clave». Suponiendo que cada fila de `contiene` representa una asociación única entre artículo y palabra:

```sql
-- SQL estándar: PostgreSQL NO implementa CREATE ASSERTION.
CREATE ASSERTION max_15_palabras_por_articulo
CHECK (
    NOT EXISTS (
        SELECT id_articulo
        FROM contiene
        GROUP BY id_articulo
        HAVING COUNT(*) > 15
    )
);
```

Lectura: **no debe existir un artículo con más de 15 filas en contiene**.

- Con 14 asociaciones, agregar una deja 15: permitido por esta regla.
- Con 15, agregar otra deja 16: rechazado.
- Mover una asociación a otro artículo también puede superar el límite: no controlar únicamente INSERT.

El alcance es de tabla porque el control consulta solo `contiene`, aunque use una aserción como recurso declarativo del estándar. No hace falta consultar `articulo` para verificar este máximo.

En PostgreSQL, este límite agregado no puede escribirse como CHECK con subconsulta. En el TP3 corresponde señalar que no puede implementarse con los recursos declarativos conocidos. En cambio, la unicidad del ejemplo anterior sí se implementa directamente con UNIQUE.

## Fuente de estudio

Enunciado del TP3 en la carpeta `../Practicos` y filminas de cátedra `BDI_03.P1 (Restricciones-1).pdf; BDI_03.P1 (Restricciones-2).pdf` (carpeta de cursada 2023).
