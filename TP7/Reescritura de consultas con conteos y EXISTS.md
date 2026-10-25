---
tags:
  - tp7
---

# Reescritura de consultas con conteos y EXISTS

## Reescrituras de cantidad y existencia

Quitar una unión o cambiarla por EXISTS exige justificar qué filas y repeticiones se conservan. La existencia no reemplaza una condición de cantidad mínima.

### Ejemplos de reescritura con conteos y existencia

Casos de tareas con más de cinco voluntarios e investigadores que participan en proyectos.

#### Ejercicio 2: tareas con más de 5 voluntarios

Se puede quitar la tarea repetida dentro de la subconsulta:

```sql
SELECT t.*
FROM tarea t
WHERE t.id_tarea IN (
    SELECT v.id_tarea
    FROM voluntario v
    GROUP BY v.id_tarea
    HAVING COUNT(*) > 5
);
```

El conteo se realiza directamente sobre voluntario. Una subconsulta que devuelve un grupo con id NULL no lo hace coincidir con una PK de tarea no nula. Otra opción es un conteo correlacionado, pero no asegurar que sea más rápido: comparar planes.

#### Ejercicio 3: joins innecesarios y DISTINCT

Para investigadores que trabajan en algún proyecto:

```sql
SELECT e.*
FROM empleado e
WHERE e.cargo = 'investigador'
  AND EXISTS (
      SELECT 1
      FROM trabaja_en te
      WHERE te.id_empleado = e.id_empleado
  );
```

- EMPLEADO es necesario para obtener sus datos y filtrar cargo.
- PROYECTO puede ser redundante si una FK garantiza la existencia del proyecto para cada asociación válida y no se filtra por atributos de proyecto. Si esa garantía no existe o la FK puede ser nula y representa una asociación incompleta, revisar la equivalencia.
- DISTINCT en el join original elimina repeticiones producidas por varios proyectos, pero tiene un costo posible de ordenamiento o hash. EXISTS expresa directamente una condición de existencia.

Para proyectos llamados IA con empleados, aplicar el mismo patrón desde PROYECTO y filtrar su nombre, correlacionando con TRABAJA_EN. El índice cuyo prefijo sea `cod_proyecto` puede favorecer esa búsqueda.

## Reescritura sobre un subtipo

Antes de eliminar GRUPO o su filtro, comprobar qué restricciones garantizan la pertenencia al subtipo exclusivo y la unicidad de cada asociación.

### Ejemplo de usuarios en más de dos grupos exclusivos

Se analizan la clave compuesta del usuario, el conteo de grupos y los supuestos que permiten simplificar la consulta.

#### Ejercicio 4: más de dos grupos exclusivos

Antes de quitar GRUPO o el filtro `tipo_gr = 'E'`, verificar qué restricciones garantizan que toda fila de GR_EXCLUSIVO representa realmente un grupo exclusivo.

Si esa garantía existe y cada pertenencia es única, puede contarse en una subconsulta:

```sql
SELECT u.tipo_usuario, u.cod_usuario, u.apellido, u.nombre
FROM usuario u
WHERE EXISTS (
    SELECT 1
    FROM integra i
    JOIN gr_exclusivo ge ON ge.nro_grupo = i.nro_grupo
    WHERE i.tipo_usuario = u.tipo_usuario
      AND i.cod_usuario = u.cod_usuario
    GROUP BY i.tipo_usuario, i.cod_usuario
    HAVING COUNT(*) > 2
);
```

Sin unicidad de la pertenencia, evaluar `COUNT(DISTINCT i.nro_grupo)` para contar grupos. No confundir «más de 2 pertenencias» con «más de 2 grupos». Las columnas de la clave del usuario deben compararse juntas.

## Comprobar la equivalencia con duplicados

Si un investigador participa en tres proyectos, el join original produce tres filas de ese empleado. DISTINCT deja una. EXISTS también produce una porque la consulta exterior recorre EMPLEADO y la subconsulta solo verifica presencia de una asociación. Quitar DISTINCT y conservar el join produciría tres: no sería equivalente a la salida pedida.

Quitar PROYECTO solo es correcto si su existencia está garantizada por la FK apropiada y no se necesitan filtros ni datos suyos. Si se pide «participa en IA», el nombre pertenece a PROYECTO y no se puede eliminar ese control.

Para tareas con más de cinco voluntarios, agrupar directamente VOLUNTARIO evita reunir dentro de la subconsulta la tabla TAREA que se vuelve a consultar afuera. Pero no se puede reemplazar el conteo por un simple EXISTS: tener uno no equivale a tener más de cinco.

En grupos exclusivos, probar un usuario con tres pertenencias repetidas al mismo grupo y otro con tres grupos diferentes. Si el modelo permite la primera situación, COUNT(*) confunde pertenencias con grupos. COUNT(DISTINCT nro_grupo) o la garantía de unicidad de la asociación son lo que permite afirmar que se cuentan grupos distintos.

## Fuente de estudio

Enunciado del TP7 en la carpeta `../../Practicos` y filminas de cátedra `BDI_07 (Optimizacion).pdf` (carpeta de cursada 2023).
