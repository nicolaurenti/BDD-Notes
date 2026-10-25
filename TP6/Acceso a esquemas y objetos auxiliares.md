---
tags:
  - tp6
---

# Acceso a esquemas y objetos auxiliares

Los privilegios se conceden sobre objetos concretos. Un permiso de tabla no equivale a acceso general a todo su esquema, y un permiso sobre una vista no debe confundirse con una concesión directa sobre sus tablas base.

## Permisos y vistas según el material de cátedra

Las filminas explican que para definir una vista se requiere SELECT sobre las relaciones que usa su consulta. Para poder transmitir SELECT sobre la vista, su creador debe tener la capacidad correspondiente de delegación sobre sus relaciones base. Tener permiso para usar un dato no siempre permite ofrecer ese acceso a terceros.

Ejemplo: A concede a B SELECT con WGO sobre T1 y SELECT sin WGO sobre T2. B puede construir una consulta de ambas; no se deduce que pueda delegar libremente el SELECT de una vista que combine T1 y T2, porque le falta WGO sobre T2.

Para una vista actualizable, analizar también si el creador tiene INSERT, UPDATE o DELETE sobre la relación donde se propaga la operación. Que la vista sea estructuralmente actualizable, como se estudió en TP5, no concede esos privilegios.

Las filminas siguen las dependencias de los privilegios de una vista respecto de las relaciones base: al perder la autorización de la que dependen, se revisan la vista y sus concesiones. En ejercicios teóricos, justificar con ese modelo de dependencias y distinguirlo de una prueba concreta en el motor.

## Acceso al esquema en PostgreSQL

```sql
GRANT USAGE ON SCHEMA public TO ins_prov;
```

USAGE habilita el acceso al espacio de nombres; no da INSERT ni SELECT sobre todas sus tablas. A la inversa, un permiso de tabla puede necesitar además acceso al esquema. Si una inserción usa una secuencia para generar identificadores, esa secuencia es otro objeto y puede requerir su permiso propio.

Estos son controles adicionales para una comprobación en PostgreSQL. En los ejercicios de grafo, no inventar permisos auxiliares no mencionados para cambiar el resultado teórico solicitado sobre INSTALACION o USUARIO.

## Fuente de estudio

Enunciado del TP6 en la carpeta `../../Practicos` y filminas de cátedra `BDI_06 (Seguridad).pdf` (carpeta de cursada 2023).
