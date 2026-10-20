---
tags:
  - tp1
---

# Agregación en el modelo de datos


Una agregación permite tratar una relación como una unidad que participa en otra relación.

En el ejercicio de cursos, `ES_REFERENTE` se relaciona con una **inscripción**, formada por un curso y un alumno.

El referente se representa mediante una FK compuesta desde `CURSO` hacia `INSCRIPTO`:

```sql
FOREIGN KEY (cod_inscripto, lu_inscripto)
    REFERENCES inscripto (cod_inscripto, lu_inscripto)
```

Esto garantiza que exista la inscripción indicada por esa pareja.

Una FK únicamente hacia `ALUMNO` comprobaría que el alumno existe, pero no que existe esa inscripción.
