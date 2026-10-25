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

## Qué información agrega la agregación

Supongamos una inscripción `(curso 10, alumno 25)` y otra `(curso 20, alumno 25)`. El alumno es el mismo, pero son dos inscripciones. Si se quiere señalar como referente una inscripción concreta, almacenar solo `25` pierde la información de a qué curso corresponde.

La referencia debe apuntar a la clave de INSCRIPTO. Una fila que indique `(10, 25)` exige que ese alumno esté inscripto en ese curso. Si el alumno existe pero nunca se inscribió en el curso 10, la FK debe rechazar la referencia.

En el modelo del práctico, tomar los nombres y el orden de columnas de la clave de INSCRIPTO tal como figuran en el diagrama. La idea de la agregación es la que justifica la FK compuesta: la otra relación participa con la **asociación completa**, no con uno de sus extremos aislado.

Si la referencia produce un ciclo entre tablas, se pueden crear primero las estructuras y agregar esa FK mediante ALTER TABLE. El ciclo de referencias no se resuelve eliminando una columna necesaria para identificar la inscripción.

## Fuente de estudio

Enunciado del TP1 en la carpeta `../../Practicos` y filminas de cátedra `BDI_01 (Intro - creacion esquemas).pdf` (carpeta de cursada 2023).
