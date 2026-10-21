---
tags:
  - tp1
---

# Creación y modificación de tablas

## Ejemplo de definición del esquema

CREATE TABLE define columnas y restricciones. Se crean primero las tablas referenciadas y después las hijas; las FK pendientes se agregan con ALTER TABLE. Una FK puede declararse dentro de CREATE o agregarse después, pero no es necesario repetirla.

[[TP1 - Ejemplo de creación de Artículo, Palabra y Contiene|Ejemplo de creación de Artículo, Palabra y Contiene]]

## Diferenciar estructura y contenido

CREATE TABLE define una estructura nueva. ALTER TABLE cambia una estructura existente. UPDATE cambia los valores de sus filas: no agrega columnas ni declara claves. Por ejemplo:

```sql
ALTER TABLE articulo ADD COLUMN resumen VARCHAR(200);
```

Esto agrega una columna para todas las filas; no inserta un nuevo artículo. Si luego se incorpora una restricción, los datos existentes también deben cumplirla: agregar una FK no convierte referencias inválidas en válidas.

El TP pide incluir al inicio de un script de reconstrucción el borrado de tablas con IF EXISTS. La forma de estudio es `DROP TABLE IF EXISTS nombre_tabla;`: elimina la tabla, mientras DELETE elimina filas y conserva la definición. Para ordenar una reconstrucción, las tablas hijas se retiran antes que las referenciadas y se crean después de ellas. Aquí se describe la sintaxis del enunciado; no se ejecuta ninguna reconstrucción.

Un script completo debe poder leerse en ese orden: definición de padres, definición de hijos y relaciones pendientes con ALTER TABLE. Si el modelo contiene referencias circulares, diferir alguna FK hasta que existan ambas tablas conserva el diseño sin depender de una tabla todavía inexistente.

## Fuente de estudio

Enunciado del TP1 en la carpeta `../Practicos` y filminas de cátedra `BDI_01 (Intro - creacion esquemas).pdf` (carpeta de cursada 2023).
