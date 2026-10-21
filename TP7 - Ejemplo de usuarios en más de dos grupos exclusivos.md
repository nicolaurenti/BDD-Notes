---
tags:
  - tp7
---

# TP7 - Ejemplo de usuarios en más de dos grupos exclusivos

Se analizan la clave compuesta del usuario, el conteo de grupos y los supuestos que permiten simplificar la consulta.

## Ejercicio 4: más de dos grupos exclusivos

Antes de quitar GRUPO o el filtro `tipo_gr = 'E'`, verificar qué restricciones garantizan que toda fila de GR_EXCLUSIVO representa realmente un grupo exclusivo.

Si esa garantía existe y cada pertenencia es única, puede contarse en una subconsulta:

```sql
SELECT u.tipo_usuario, u.cod_usuario, u.apellido, u.nombre
FROM usuario u
WHERE EXISTS (
    SELECT 1
    FROM integra i
    JOIN gr_exclusivo ge ON ge.nro_grupo = i.nro_grupo
    WHERE i.tipo_usuario = u.tipo_usuario
      AND i.cod_usuario = u.cod_usuario
    GROUP BY i.tipo_usuario, i.cod_usuario
    HAVING COUNT(*) > 2
);
```

Sin unicidad de la pertenencia, evaluar `COUNT(DISTINCT i.nro_grupo)` para contar grupos. No confundir «más de 2 pertenencias» con «más de 2 grupos». Las columnas de la clave del usuario deben compararse juntas.

## Fuente de estudio

Enunciado del TP7 en la carpeta `../Practicos` y filminas de cátedra `BDI_07 (Optimizacion).pdf` (carpeta de cursada 2023).
