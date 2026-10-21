---
tags:
  - tp5
---

# Definición y uso de vistas


Una vista común guarda una consulta con nombre: al consultarla, se obtiene información de sus tablas base. No guarda por separado una copia de sus filas. Una vista materializada sí almacena un resultado y requiere actualizarlo con el mecanismo correspondiente.

Sirve para simplificar consultas, presentar un subconjunto de datos y ofrecer una interfaz de acceso. En este práctico interesa qué información presenta y cómo se propagan las modificaciones hacia sus tablas base.

```sql
CREATE VIEW envios500 AS
SELECT id_proveedor, id_articulo, cantidad
FROM envio
WHERE cantidad >= 500;

CREATE VIEW envios500_m AS
SELECT *
FROM envios500
WHERE cantidad < 1000;
```

La segunda vista devuelve cantidades entre 500 y 999 si son enteras. Se usa `_` en el nombre para evitar un identificador con guion que requiera comillas.

## Qué se guarda y qué se vuelve a obtener

Con envíos de cantidades 300, 700 y 1200, ENVIOS500 muestra 700 y 1200. ENVIOS500_M se basa en ese resultado y conserva solo 700. Si se modifica la tabla ENVIO y el valor 700 pasa a 800, la próxima consulta de ambas vistas muestra 800: no hay que copiar nuevamente las filas.

La vista se usa en FROM como una tabla:

```sql
SELECT id_proveedor, cantidad
FROM envios500_m
ORDER BY id_proveedor;
```

Su definición no impone un orden general de presentación: la consulta que necesita un orden debe pedirlo. Tampoco crea nuevas PK o FK independientes; las restricciones que rigen las escrituras son las de las tablas base.

## Nombrar las columnas

La forma `CREATE VIEW nombre (columna1, columna2) AS SELECT ...` permite dar nombres a la salida. Si se omite esa lista, se usan los nombres y alias del SELECT. En una vista de join conviene distinguir nombres coincidentes, por ejemplo nombre de empleado y nombre de departamento, para que quede claro qué dato representa cada columna.

Antes de crear una vista anidada, definir la inferior. Para analizarla, sustituir mentalmente el nombre inferior por su consulta: así se ven todas las condiciones que debe cumplir una fila para ser visible.

## Fuente de estudio

Enunciado del TP5 en la carpeta `../Practicos` y filminas de cátedra `BDI_05 (Vistas).pdf` (carpeta de cursada 2023).
