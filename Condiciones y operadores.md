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

#### Precedencia lógica

`NOT` se evalúa antes que `AND`, y `AND` antes que `OR`.

Conviene usar paréntesis para expresar la intención:

```sql
WHERE (nombre LIKE 'A%' OR nombre LIKE 'C%')
  AND horas_aportadas > 1000
```

## Manejo de strings

Los patrones se construyen con LIKE: % representa cualquier secuencia y _ un carácter. Concatenación, funciones de texto y nulidad determinan qué se muestra en cada columna.

[[TP2 - Patrones y operaciones con textos|Patrones y operaciones con textos]]

## Manejo de fechas

Comparar fechas, extraer sus componentes y darles formato son operaciones distintas. Para edades importan los años cumplidos; para cumpleaños se comparan día y mes.

[[TP2 - Fechas, edades y cumpleaños|Fechas, edades y cumpleaños]]

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

Enunciado del TP2 en la carpeta `../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
