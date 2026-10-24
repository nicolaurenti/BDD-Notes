---
tags:
  - tp4
---

# TP4 - Ejemplo de carga y mantenimiento de TextosporAutor

Procedimiento de carga, grupos afectados por cada evento y caso de baja de la publicación más reciente.

## Carga inicial y mantenimiento de TextosporAutor

El procedimiento de carga inicial actúa sobre lo que ya existe; los triggers mantienen los cambios futuros. Por ejemplo, sobre una tabla resumen creada con los tipos apropiados de ARTICULO:

```sql
CREATE OR REPLACE PROCEDURE cargar_textos_por_autor()
LANGUAGE 'plpgsql' AS $$
BEGIN
    DELETE FROM TextosporAutor;
    INSERT INTO TextosporAutor (autor, cant_textos, fecha_ultima_public)
    SELECT autor, COUNT(*), MAX(fecha_pub)
    FROM articulo
    GROUP BY autor;
END;
$$;

-- Ejemplo de invocación para el esquema de práctica:
CALL cargar_textos_por_autor();
```

El DELETE pertenece al algoritmo de reconstrucción de la **tabla derivada**: permite repetir la carga sin duplicarla. Aquí solo se escribe como ejemplo; no se ejecutó. Se supone que autor identifica los grupos y que se definió qué hacer con autores nulos.

Para el mantenimiento, identificar qué grupos cambian:

| Evento en ARTICULO | Grupos que deben recalcularse |
|---|---|
| INSERT | Autor nuevo |
| DELETE | Autor anterior |
| UPDATE de autor | Autor anterior y autor nuevo |
| UPDATE de fecha | Autor de la fila |

Si Ana tiene fechas 2019 y 2022 y se borra su artículo de 2022, el resumen debe pasar a cantidad 1 y fecha 2019. Restar uno al conteo sin revisar MAX dejaría una fecha falsa. Una estrategia sencilla es recalcular COUNT y MAX de los autores afectados; al quedar sin artículos, decidir si se quita su fila del resumen o se conserva con cero y fecha NULL.

El trigger no puede reconstruir fechas de ingresos o cambios que nunca se guardaron. Al incorporar un historial, distinguir datos conocidos de supuestos iniciales; no presentar la fecha de instalación como si fuera la de ingreso real de todos los empleados.

## Fuente de estudio

Enunciado del TP4 en la carpeta `../../Practicos` y filminas de cátedra `BDI_04 (SQL Procedural).pdf` (carpeta de cursada 2023).
