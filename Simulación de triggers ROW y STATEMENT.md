---
tags:
  - tp4
---

# Simulación de triggers ROW y STATEMENT


El código del ejercicio resta un 5 % del sueldo mínimo; el nombre «autoincremento» no cambia lo que hace. Antes de interpretarlo, corregir terminadores y retorno según la granularidad.

Suponiendo el orden de inserción 700, 300, 700 y sueldos NUMERIC, el empleado inicial cobra 500:

| BEFORE ROW | Mínimo antes de cada fila | Descuento a filas ya presentes |
|---|---|---|
| Antes de insertar 700 | 500 | 25 |
| Antes de insertar 300 | 475 | 23.75 |
| Antes de insertar el último 700 | 300 | 15 |

Resultado por id: inicial `436.25`; id 2 `661.25`; id 3 `285`; id 4 `700`. En `BEFORE STATEMENT`, se descuenta una vez al inicial: `475`; los tres nuevos conservan `700, 300, 700`.

No asumir orden de procesamiento de un `INSERT ... SELECT` sin orden explícito. Si cambia el orden o el tipo numérico, revisar el resultado. Para simular: ejecutar mentalmente el trigger antes de cada inserción, no después de todas.
