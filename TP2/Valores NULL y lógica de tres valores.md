---
tags:
  - tp2
---

# Valores NULL y lógica de tres valores

`NULL` representa un valor desconocido o ausente.

Las condiciones pueden resultar:

- Verdaderas.
- Falsas.
- Desconocidas: `UNKNOWN`.

`WHERE` y `HAVING` conservan únicamente los resultados **verdaderos**.

```sql
-- Correcto.
WHERE telefono IS NULL

-- Incorrecto para buscar nulos.
WHERE telefono = NULL
```

## Comparar y reemplazar valores ausentes

Una comparación con NULL suele dar UNKNOWN; negarla no la vuelve verdadera. IS NULL prueba la ausencia de valor y COALESCE permite elegir un reemplazo explícito.

[[TP2 - Comparaciones, negación y COALESCE con NULL|Comparaciones, negación y COALESCE con NULL]]

### Ejemplo integrado de consultas con NULL

El caso de cuatro equipos muestra cómo se aplican estas reglas a agregaciones, DISTINCT, concatenación, NOT IN y autojoins.

[[TP2 - Ejercicio de Equipo con todos los puntos nulos|Ejercicio de Equipo con todos los puntos nulos]]

### Puntos para repasar

- [ ] Distinguir valor nulo, cero y texto vacío.
- [ ] Evaluar comparaciones y negaciones con `UNKNOWN`.
- [ ] Reconocer que `WHERE`, `HAVING` y las coincidencias de `ON` requieren `TRUE`.
- [ ] Distinguir `COUNT(*)` de `COUNT(columna)` y explicar un promedio sin valores.
- [ ] Explicar por qué `DISTINCT` reúne nulos sin que `NULL = NULL` sea verdadero.
- [ ] Detectar el efecto de los nulos en `NOT IN` y en la concatenación.
- [ ] Diferenciar reemplazar el resultado de una agregación de reemplazar sus datos de entrada.
- [ ] Contar todas las parejas de un join y la fila sin coincidencia de un `LEFT JOIN`.

Estas reglas se aplican a las agregaciones, subconsultas, textos y joins del TP2.

## Cómo se combina UNKNOWN con AND y OR

UNKNOWN no se convierte en FALSE: AND y OR pueden decidirse por el otro operando. La tabla de verdad permite justificar cuándo una fila pasa el WHERE.

[[TP2 - AND y OR con UNKNOWN|AND y OR con UNKNOWN]]

## Fuente de estudio

Enunciado del TP2 en la carpeta `../../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
