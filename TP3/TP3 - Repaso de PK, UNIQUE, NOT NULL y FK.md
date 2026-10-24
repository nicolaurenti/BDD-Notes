---
tags:
  - tp3
---

# TP3 - Repaso de PK, UNIQUE, NOT NULL y FK

Recordatorio de las restricciones básicas del TP1 y su uso al incorporar una referencia.

## PK, UNIQUE, NOT NULL y FK

- `PRIMARY KEY`: identifica la fila; incluye unicidad y ausencia de nulos.
- `UNIQUE`: impide combinaciones repetidas; en PostgreSQL admite por defecto varios nulos.
- `NOT NULL`: obliga a informar el atributo.
- `FOREIGN KEY`: exige una referencia válida, salvo los casos de nulidad permitidos por el matching.

Una FK compuesta comprueba la **combinación completa**. Dos FK separadas no expresan lo mismo.

```sql
ALTER TABLE trabaja_en
ADD CONSTRAINT fk_trabaja_empleado
FOREIGN KEY (tipo_e, nro_e)
REFERENCES empleado (tipo_e, nro_e)
ON DELETE CASCADE
ON UPDATE RESTRICT;
```

Los nombres de columnas del ejemplo son ilustrativos; en los ejercicios 1 y 2 se deben tomar los nombres y flechas del diagrama del enunciado.

## Fuente de estudio

Enunciado del TP3 en la carpeta `../../Practicos` y filminas de cátedra `BDI_03.P1 (Restricciones-1).pdf; BDI_03.P1 (Restricciones-2).pdf` (carpeta de cursada 2023).
