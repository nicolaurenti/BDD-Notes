---
tags:
  - tp4
---

# Historial, resúmenes y carga inicial


Para permanencias por departamento, guardar ingreso y cada intervalo de asignación; identificar el departamento con su clave completa `(id_distribuidor, id_departamento)`. Al cambiar de departamento, cerrar el intervalo anterior y abrir el siguiente. Decidir si un intervalo abierto participa del promedio y cómo se calcula su duración.

Para `TextosporAutor`:

1. Carga inicial con `INSERT ... SELECT autor, COUNT(*), MAX(fecha_pub) FROM articulo GROUP BY autor` sobre una tabla resumen inicialmente vacía.
2. Mantenimiento ante INSERT, UPDATE y DELETE.
3. En un cambio de autor, actualizar el autor anterior y el nuevo.
4. Si se borra el artículo más reciente, recalcular el máximo; no basta con restar uno al conteo.
5. Definir qué hacer con autores sin artículos y cómo evitar duplicados al repetir una carga.

Un trigger instalado hoy no conoce fechas históricas que nunca se registraron. Un total de permanencia que crece con el tiempo no se mantiene exacto únicamente con triggers de escritura; puede calcularse al consultar los intervalos.
