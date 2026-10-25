---
tags:
  - tp2
---

# Condiciones, textos y fechas

## Condiciones y operadores

| Operador | Uso |
|---|---|
| `=`, `<>`, `<`, `>`, `<=`, `>=` | Comparaciones. |
| `AND` | Deben cumplirse ambas condiciones. |
| `OR` | Debe cumplirse al menos una condición. |
| `NOT` | Niega una condición. |
| `IN (...)` | Pertenencia a un conjunto. |
| `BETWEEN a AND b` | Intervalo inclusivo: incluye ambos extremos. |
| `LIKE` | Comparación de texto mediante patrones. |
| `IS NULL` / `IS NOT NULL` | Comprobación de nulidad. |

```sql
SELECT *
FROM voluntario
WHERE id_institucion IN (20, 50)
  AND horas_aportadas BETWEEN 1000 AND 5000;
```

### Precedencia lógica

`NOT` se evalúa antes que `AND`, y `AND` antes que `OR`.

Conviene usar paréntesis para expresar la intención:

```sql
WHERE (nombre LIKE 'A%' OR nombre LIKE 'C%')
  AND horas_aportadas > 1000
```

## Manejo de strings

Los patrones se construyen con LIKE: % representa cualquier secuencia y _ un carácter. Concatenación, funciones de texto y nulidad determinan qué se muestra en cada columna.

### Patrones y operaciones con textos

Ejemplos de prefijos, sufijos, concatenación y texto ausente.

| Recurso                                     | Función                                             |
| ------------------------------------------- | --------------------------------------------------- |
| `texto1 &#124;&#124; texto2`                          | Concatenar texto                                    |
| `CONCAT(...)`                               | Concatenar argumentos; ignora los argumentos nulos. |
| `LOWER(texto)`                              | Convertir a minúsculas.                             |
| `UPPER(texto)`                              | Convertir a mayúsculas.                             |
| `TRIM(texto)`                               | Quitar espacios de los extremos.                    |
| `LENGTH(texto)`                             | Obtener la cantidad de caracteres.                  |
| `SUBSTRING(texto FROM inicio FOR cantidad)` | Extraer una parte del texto.                        |
| `REPLACE(texto, buscar, reemplazo)`         | Reemplazar apariciones de un texto.                 |

```sql
SELECT
    apellido || ', ' || nombre AS apellido_nombre,
    LOWER(e_mail) AS correo
FROM voluntario;
```

#### Patrones con LIKE

- `%`: representa cero o más caracteres.
- `_`: representa exactamente un carácter.
- `ILIKE`: comparación sin distinguir mayúsculas y minúsculas en PostgreSQL.

```sql
-- Comienza con +11.
WHERE telefono LIKE '+11%'

-- Termina en @gmail.com.
WHERE e_mail LIKE '%@gmail.com'

-- Contiene el texto ana, sin distinguir mayúsculas.
WHERE nombre ILIKE '%ana%'

-- Tiene A como segundo carácter.
WHERE nombre LIKE '_A%'
```

> [!warning]
> Con `||`, concatenar un texto con `NULL` produce `NULL`. Puede evitarse con `COALESCE` o utilizando `CONCAT`.

```sql
SELECT nombre || ' ' || COALESCE(telefono, 'Sin teléfono')
FROM voluntario;
```

#### Descripción ausente

```sql
SELECT id, 'Su descripción es ' || COALESCE(descripcion, '') AS detalle
FROM equipo;
```

Si `descripcion` es `NULL`, devuelve el prefijo `Su descripción es `, conservando el espacio final. Sin `COALESCE`, la expresión con `||` devuelve `NULL`.

El bloque de NULL del TP2 desarrolla la lógica de tres valores.

## Manejo de fechas

Comparar fechas, extraer sus componentes y darles formato son operaciones distintas. Para edades importan los años cumplidos; para cumpleaños se comparan día y mes.

### Fechas, edades y cumpleaños

Desarrolla formato de fecha, edad, cumpleaños de hoy y límites de un año.

| Recurso | Función |
|---|---|
| `CURRENT_DATE` | Fecha actual. |
| `EXTRACT(YEAR FROM fecha)` | Extraer el año. |
| `EXTRACT(MONTH FROM fecha)` | Extraer el mes. |
| `EXTRACT(DAY FROM fecha)` | Extraer el día. |
| `AGE(fecha)` | Obtener el intervalo entre la fecha actual y una fecha. |
| `TO_CHAR(fecha, formato)` | Convertir una fecha a texto con formato. |
| `TO_DATE(texto, formato)` | Convertir un texto a fecha. |
| `INTERVAL` | Representar una duración. |

#### Mostrar una fecha con formato

```sql
SELECT TO_CHAR(fecha_nacimiento, 'YYYY-MM-DD')
FROM voluntario;
```

> [!note]
> `TO_CHAR` devuelve texto. Para comparar u ordenar cronológicamente, normalmente conviene trabajar con la fecha original.

#### Calcular edad en años cumplidos

```sql
SELECT
    nombre,
    apellido,
    EXTRACT(YEAR FROM AGE(fecha_nacimiento)) AS edad
FROM voluntario;
```

#### Buscar cumpleaños de hoy

```sql
SELECT nombre, apellido
FROM voluntario
WHERE EXTRACT(MONTH FROM fecha_nacimiento)
        = EXTRACT(MONTH FROM CURRENT_DATE)
  AND EXTRACT(DAY FROM fecha_nacimiento)
        = EXTRACT(DAY FROM CURRENT_DATE);
```

#### Buscar fechas dentro de un año

```sql
WHERE fecha_nacimiento >= DATE '1998-01-01'
  AND fecha_nacimiento < DATE '1999-01-01'
```

#### Más joven y más viejo

```sql
SELECT
    MAX(fecha_nacimiento) AS nacimiento_mas_joven,
    MIN(fecha_nacimiento) AS nacimiento_mas_viejo
FROM voluntario;
```

La fecha de nacimiento más reciente corresponde al más joven.

## Por qué importan los paréntesis y los límites

Si se busca «nombres que comienzan con O, A o C y más de 1000 horas», la condición es:

```sql
WHERE (nombre LIKE 'O%' OR nombre LIKE 'A%' OR nombre LIKE 'C%')
  AND horas_aportadas > 1000
```

Sin los paréntesis, AND se une solo con la última alternativa: una persona cuyo nombre empieza con O podría pasar aunque tenga 200 horas. Para detectar este error, probar una fila que cumpla la primera alternativa pero incumpla el requisito común.

Con fechas, «a partir de 1998» incluye todos los años siguientes; «durante 1998» limita el intervalo a ese año. En el primer caso basta `fecha_nacimiento >= TO_DATE('1998-01-01', 'YYYY-MM-DD')`. En el segundo se agrega el límite anterior a 1999. No reemplazar ambas expresiones por una comparación con el año 1998 sin leer la consigna.

Para edad, restar los años de nacimiento y actual puede anticipar el cumpleaños: una persona nacida en diciembre aún no cumplió años en octubre. `EXTRACT(YEAR FROM AGE(fecha_nacimiento))` trabaja con años cumplidos. Para cumpleaños, comparar día y mes, porque igualar la fecha completa exigiría también el año actual.

## Fuente de estudio

Enunciado del TP2 en la carpeta `../../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
