---
tags:
  - tp1
---

# Tipos de datos SQL

| Tipo | Uso |
|---|---|
| `INTEGER` | Números enteros. |
| `NUMERIC(p,s)` / `DECIMAL(p,s)` | Números exactos: `p` indica la cantidad total de dígitos y `s` la cantidad de decimales. |
| `CHAR(n)` | Texto de longitud fija. |
| `VARCHAR(n)` | Texto de longitud variable, hasta `n` caracteres. |
| `DATE` | Fechas. |

**Ejemplos:**

- `DECIMAL(5,0)`: hasta cinco dígitos, sin decimales.
- `DECIMAL(7,2)`: hasta siete dígitos en total, de los cuales dos son decimales.
- `CHAR(2)`: código de idioma de longitud fija.
- `VARCHAR(120)`: título de hasta 120 caracteres.

## Leer las abreviaturas del enunciado

El TP1 usa E para entero, SLV para string de longitud variable, SLF para string de longitud fija, F para fecha y D para decimal. La elección del tipo debe seguir esos datos, además del tamaño indicado; no elegir VARCHAR para todo porque resulte más fácil de cargar.

En `NUMERIC(7,2)`, cinco dígitos quedan para la parte entera y dos para la fracción: `12345.67` entra, pero `123456.78` excede la precisión. En cambio, el número de un identificador no necesita decimales si el modelo lo define entero. Un teléfono puede contener signos, prefijos y ceros iniciales, por lo que suele representarse como texto.

```sql
codigo CHAR(2),
titulo VARCHAR(120),
importe NUMERIC(7,2),
fecha DATE
```

El tipo restringe la clase de valores; no obliga por sí solo a informar uno ni a que sea único. `VARCHAR(120)` admite NULL si no se declara NOT NULL. Para una FK, usar tipos compatibles con los de la clave referenciada y conservar también las longitudes previstas por el modelo.

## Fuente de estudio

Enunciado del TP1 en la carpeta `../../Practicos` y filminas de cátedra `BDI_01 (Intro - creacion esquemas).pdf` (carpeta de cursada 2023).
