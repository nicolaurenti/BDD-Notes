---
tags:
  - tp2
---

# TP2 - Fechas, edades y cumpleaños

Desarrolla formato de fecha, edad, cumpleaños de hoy y límites de un año.

## Manejo de fechas

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

## Fuente de estudio

Enunciado del TP2 en la carpeta `../../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
