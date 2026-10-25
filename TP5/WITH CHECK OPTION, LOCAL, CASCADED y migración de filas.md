---
tags:
  - tp5
handwriting-page-id: bc3f529a-e4af-4732-8a54-6779436e3274
---

# WITH CHECK OPTION, LOCAL, CASCADED y migración de filas

```sql
CREATE VIEW envios500_controlados AS
SELECT id_proveedor, id_articulo, cantidad
FROM envio
WHERE cantidad >= 500
WITH CHECK OPTION;
```

Sin chequeo, insertar una cantidad 300 puede modificar la tabla base y, aun así, no aparecer en la vista. Actualizar 700 a 100 puede hacer que una fila desaparezca de la vista.

Con chequeo, INSERT y UPDATE sobre esa vista deben dejar las filas visibles según las condiciones exigidas. No vuelve actualizable una vista que no lo era; tampoco sustituye las restricciones de las tablas.

| Operación sobre ENVIOS500        | Sin WCO                          | Con WCO                         |
| -------------------------------- | -------------------------------- | ------------------------------- |
| Insertar 500                     | Pasa el filtro                   | Pasa el filtro                  |
| Insertar 300                     | Puede insertarse y quedar oculta | Se rechaza                      |
| Cambiar una fila visible a 100   | Puede migrar fuera               | Se rechaza                      |
| Cambiar una fila visible a 1000  | Pasa el filtro                   | Pasa el filtro                  |
| Referenciar artículo inexistente | La FK puede rechazar             | La FK puede rechazar igualmente |

Solo se actualizan filas actualmente visibles. Si un INSERT de 300 quedó oculto, un UPDATE posterior sobre la vista no lo encuentra, aunque filtre por su proveedor. WCO no impide escrituras directas sobre la tabla base que dejen filas fuera de la vista.

## Seguir una modificación

La vista determina primero qué filas se seleccionan. WCO comprueba después que el resultado propuesto siga satisfaciendo los filtros exigidos.

### Ejemplo de selección y migración de filas

Se comparan envíos de 700 y 300 unidades y un UPDATE que intenta cambiar su visibilidad.

#### Diferenciar selección y comprobación posterior

Supongamos ENVIO con `(P1, A1, 700)` y `(P2, A2, 300)`. ENVIOS500 ve solo la primera. Un UPDATE sobre ENVIOS500 que busque P2 no encuentra la segunda fila, aunque proponga llevarla a 1000: el WHERE de la vista decide qué filas se pueden seleccionar antes de aplicar SET.

Si el UPDATE selecciona P1 y propone 100, sin WCO modifica la base y la fila deja de aparecer. Con WCO, se comprueba el resultado propuesto y se rechaza la sentencia porque 100 no satisface `cantidad >= 500`.

La «migración» no es un traslado a otra tabla: es dejar de pertenecer al resultado de la vista. La fila puede seguir existiendo en ENVIO. Para explicar el ejercicio, mostrar por separado el estado base y las filas visibles.

## Qué no hace el chequeo

WCO no completa datos omitidos ni valida referencias en lugar de las FK. Insertar `(P1, A3, 700)` supera el filtro de cantidad pero puede fallar si A3 no existe. Tampoco convierte una vista con SUM en automáticamente actualizable. DELETE retira una fila; no produce una fila nueva que deba satisfacer el filtro. El caso central de WCO es el resultado de INSERT y UPDATE efectuados por la vista.

## CHECK OPTION LOCAL y CASCADED

- **LOCAL:** agrega el control de la vista actual; los chequeos que ya tengan las vistas inferiores siguen aplicándose.
- **CASCADED:** exige las condiciones de la vista y de todas las vistas subyacentes, tengan o no WCO propio.
- Sin especificar modalidad, `WITH CHECK OPTION` usa **CASCADED**.

Ejemplo del ejercicio 4:

```sql
CREATE VIEW tarea10000hs AS
SELECT * FROM tarea
WHERE max_horas > 10000
WITH LOCAL CHECK OPTION;

CREATE VIEW tarea10000rep AS
SELECT * FROM tarea10000hs
WHERE id_tarea LIKE '%REP%'
WITH LOCAL CHECK OPTION;
```

| Inserción | Resultado por WCO |
|---|---|
| MGR, máximo 20000, en tarea10000rep | Rechaza: el id no contiene REP |
| REPA, máximo 5500, en tarea10000hs | Rechaza: máximo insuficiente |
| CC_REP, máximo 9000, en tarea10000rep | Rechaza: incumple el WCO de la vista inferior |
| ROM, máximo 12000, en tarea10000hs | Pasa los filtros; confirmar las demás restricciones |

Si la vista inferior no tuviera WCO, LOCAL en la superior no agregaría por sí solo el control del filtro inferior. CASCADED sí lo haría.

### Comparar combinaciones de chequeos

La diferencia se observa cuando la vista inferior no tiene WCO propio. Si ya lo tiene, LOCAL en la superior no desactiva el control inferior.

#### Ejemplos de diferencias entre LOCAL y CASCADED

Tabla de alternativas con V1 definida por cantidad >= 500 y V2 por cantidad < 1000.

#### Un ejemplo donde LOCAL y CASCADED sí difieren

Sean V1, definida sobre ENVIO con `cantidad >= 500` y **sin WCO**, y V2, definida sobre V1 con `cantidad < 1000`. Si V2 tiene LOCAL y se inserta una cantidad 300 por ella, el control agregado por V2 solo exige `cantidad < 1000`: se puede insertar en la base aunque quede fuera de V1 y V2. Con CASCADED en V2 se exige también `cantidad >= 500` y se rechaza.

Si V1 ya tiene WCO, LOCAL en V2 no lo desactiva. Entonces 300 se rechaza también con LOCAL. Por eso no basta con mirar la última línea de la vista superior: hay que recorrer toda la cadena y anotar qué chequeos están activos.

| WCO de V1 (`>= 500`) | WCO de V2 (`< 1000`) | Insertar 300 por V2 |
|---|---|---|
| Ninguno | LOCAL | Supera el control local; queda oculta |
| Ninguno | CASCADED | Rechazo por el filtro de V1 |
| LOCAL | LOCAL | Rechazo por el WCO propio de V1 |

Los casos suponen válidas las demás restricciones de la tabla. En sintaxis SQL la palabra es **CASCADED**, aunque el enunciado use ocasionalmente «CASCADE» al describir las alternativas.

## Fuente de estudio

Enunciado del TP5 en la carpeta `../../Practicos` y filminas de cátedra `BDI_05 (Vistas).pdf` (carpeta de cursada 2023).
