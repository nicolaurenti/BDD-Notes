---
tags:
  - tp3
---

# TP3 - Elección de acciones y errores al declarar una FK

Equivalencia entre la notación R/C/N del práctico y las cláusulas SQL, con errores frecuentes.

## Elegir las acciones según la regla

Las acciones anteriores son decisiones de los ejemplos. En otro modelo podrían corresponder otras acciones:

```sql
-- Fragmentos alternativos para la declaración de una FK:
ON DELETE CASCADE
ON UPDATE RESTRICT
```

`CASCADE` al borrar elimina las filas hijas que referencian al padre. Con `SET NULL`, en cambio, se conservan esas filas y se ponen en NULL las columnas de la FK; esas columnas deben admitir nulos.

Cuando una consigna usa un par **[baja, modificación a derecha]**, el primer componente corresponde a `ON DELETE` y el segundo a `ON UPDATE`. La modificación a derecha se refiere al cambio de la clave referenciada en la tabla padre.

| Símbolo | Acción SQL |
|---|---|
| `R` | `RESTRICT` |
| `C` | `CASCADE` |
| `N` | `SET NULL` |

Las acciones se evalúan junto con las demás restricciones: una cascada puede quedar impedida por otra regla. Si se omite `MATCH`, PostgreSQL usa `MATCH SIMPLE`.

## Errores frecuentes

- Alterar la tabla padre en lugar de la hija.
- Invertir las acciones de baja y modificación.
- Usar solo una parte de una clave compuesta o cambiar el orden de correspondencia.
- Confundir la PK completa de la hija con las columnas que necesita una FK concreta.
- Creer que `ON UPDATE CASCADE` permite asignar directamente a una hija una FK cuyo padre no existe.

Los ejemplos usan la forma ALTER TABLE del práctico; no se ejecutaron sobre las bases existentes.

## Fuente de estudio

Enunciado del TP3 en la carpeta `../Practicos` y filminas de cátedra `BDI_03.P1 (Restricciones-1).pdf; BDI_03.P1 (Restricciones-2).pdf` (carpeta de cursada 2023).
