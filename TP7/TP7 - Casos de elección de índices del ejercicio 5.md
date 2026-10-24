---
tags:
  - tp7
---

# TP7 - Casos de elección de índices del ejercicio 5

Comparación de rangos, igualdad, prefijos y sufijos de texto; incluye la distinción entre un índice bitmap teórico y un nodo Bitmap del plan.

## Responder el ejercicio de índices sin cambiar el dialecto

El ejercicio 5 escribe MONTHS_BETWEEN y SYSDATE; conservar esa expresión al analizar la consulta tal como está en la consigna. Su propósito es filtrar antigüedad de entregas. Si además se quiere ejecutarla en PostgreSQL, presentar la traducción de fechas por separado y declarar cómo se interpreta el límite; no reemplazar silenciosamente el SQL original.

| Condición del ejercicio | Razonamiento sobre el acceso |
|---|---|
| id_distribuidor > 50 | Un B+ permite rangos; la conveniencia depende de cuántos distribuidores superan 50 |
| tipo = 'N' | Evaluar distribución y baja cardinalidad; no afirmar que un índice siempre ayuda |
| nombre = ... AND e_mail = ... | Un compuesto puede servir; un correo muy selectivo puede bastar como primer acceso |
| nombre LIKE 'An%' y correo igual | El prefijo del nombre puede acotar; comparar con el acceso por correo |
| nombre LIKE '%Maria' y correo igual | El sufijo no fija el comienzo del nombre; el correo sigue siendo una igualdad aprovechable |

## B+ y bitmap en el contexto de la consigna

El TP plantea árboles B+ como estructura habitual e invita a considerar bitmap. Un bitmap representa para cada valor qué filas lo poseen; puede ser útil en atributos de pocos valores distintos y consultas que combinan condiciones, especialmente cuando predominan lecturas. TIPO es el candidato conceptual de la lista por su baja cardinalidad. Esa elección se justifica con volumen, distribución y carga de modificaciones, no solo con que el atributo sea un carácter.

Distinguir ese **índice bitmap** de un Bitmap Index Scan en PostgreSQL: este último es una estrategia del plan que puede usar índices ordinarios para reunir ubicaciones de filas. Ver un nodo Bitmap no demuestra que se creó el tipo de índice teórico del enunciado.

## Fuente de estudio

Enunciado del TP7 en la carpeta `../../Practicos` y filminas de cátedra `BDI_07 (Optimizacion).pdf` (carpeta de cursada 2023).
