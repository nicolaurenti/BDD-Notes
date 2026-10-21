---
tags:
  - tp1
---

# Inserción de filas con INSERT


Se utiliza `INSERT INTO`.

```sql
INSERT INTO articulo (
    id_articulo,
    titulo,
    autor,
    fecha_pub
)
VALUES (
    1,
    'Bases de datos',
    'Ana',
    DATE '2026-10-18'
);
```

También pueden insertarse varias filas:

```sql
INSERT INTO palabra (cod_p, idioma, descripcion)
VALUES
    (1, 'ES', 'Datos'),
    (2, 'ES', 'Tablas'),
    (1, 'EN', 'Data');
```

Las filas con código `1` son válidas porque la PK incluye también el idioma.

## Cómo se corresponden columnas y valores

La primera expresión de VALUES se asigna a la primera columna listada, la segunda a la segunda, etc. Escribir la lista evita depender del orden físico de las columnas. Una columna omitida toma su valor por defecto o NULL; si es obligatoria y no tiene un valor apropiado, la inserción falla.

Para cargar una asociación en CONTIENE, primero deben existir el artículo y la palabra. Una fila `(1, 1, 'ES')` no crea automáticamente ninguno de esos padres: solo los referencia. Si la palabra cargada es `(1, 'EN')`, la asociación en español será inválida.

En el ejercicio de «por lo menos cinco tuplas», no basta con contar cinco VALUES: deben ser cinco filas válidas bajo **todas** las restricciones. Conviene elegir datos que muestren distintos casos del modelo, por ejemplo un equipo usado por varias grabaciones o un mismo código de palabra en idiomas diferentes.

## Fuente de estudio

Enunciado del TP1 en la carpeta `../Practicos` y filminas de cátedra `BDI_01 (Intro - creacion esquemas).pdf` (carpeta de cursada 2023).
