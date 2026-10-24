---
tags:
  - tp2
---

# TP2 - Patrones y operaciones con textos

Ejemplos de prefijos, sufijos, concatenación y texto ausente.

## Manejo de strings

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

## Fuente de estudio

Enunciado del TP2 en la carpeta `../../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
