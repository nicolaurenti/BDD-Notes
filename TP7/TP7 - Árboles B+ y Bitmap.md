---
tags:
  - tp7
---

# 8. Árboles B+ y Bitmap

## Idea central

Las estructuras de índices resuelven necesidades diferentes. El práctico compara el árbol B+ con Bitmap: hay que elegir según las búsquedas y el modo de uso de los datos.

## Cómo razonarlo

El **árbol B+** es balanceado y tiene varios hijos por nodo. Los nodos internos guían la búsqueda; las hojas contienen entradas ordenadas y referencias a los datos, con enlaces que facilitan recorridos de rangos. No es un árbol binario. Sirve para igualdad, intervalos y acceso ordenado.

Un **índice Bitmap** asocia valores con mapas de bits que representan ubicaciones de filas. Permite combinar condiciones mediante operaciones de bits. Es un candidato típico para columnas con pocos valores distintos y cargas principalmente de consulta, con pocas modificaciones concurrentes.

| Criterio | Árbol B+ | Bitmap persistente |
|---|---|---|
| Igualdad selectiva | Habitualmente apropiado. | Puede servir; evaluar cardinalidad y carga. |
| Rangos y orden | Ventaja natural por orden de claves. | No es la primera elección del práctico para estos casos. |
| Pocos valores y combinación de filtros | Puede servir según la distribución. | Candidato típico. |
| Muchas modificaciones concurrentes | Opción general habitual. | Su mantenimiento y bloqueo pueden hacerlo inconveniente. |

## Ejemplo

Supongamos cuatro distribuidores con tipos N,I,N,I. Un bitmap conceptual para N es `1010`: señala el primero y el tercero. Otro filtro puede producir `1100`; su AND da `1000`, dejando solo el primero. Es una ilustración; las implementaciones comprimen y organizan físicamente estos mapas.

En el ejercicio 5, `tipo='N'` es el candidato natural a evaluar como Bitmap si el motor lo soporta y la carga lo favorece. Fecha e identificadores por rango se razonan inicialmente con B+.

**Diferencia de PostgreSQL:** usa la denominación B-tree para su índice ordenado habitual y no ofrece un método estándar `USING bitmap` como índice persistente. Bitmap Index Scan en un plan puede construir un mapa en ejecución a partir de un B-tree: no demuestra que se haya creado un índice Bitmap.

Hash es otra estructura: organiza búsquedas por igualdad y no proporciona orden ni rangos. No sustituye automáticamente a B+ y no es necesario añadirlo para toda igualdad.

## Errores frecuentes

«Pocos valores distintos» no basta para ordenar crear un índice: importan tamaño, combinación de filtros, proporción recuperada y mantenimiento. No confundir estructura Bitmap con estrategia de bitmap scan. No afirmar que B+ sea apropiado solo para rangos: también resuelve igualdad.

## Qué recordar

**Estructura + operador + distribución + carga de trabajo** justifican la elección. [PostgreSQL: tipos](https://www.postgresql.org/docs/15/indexes-types.html) y [bitmap scans](https://www.postgresql.org/docs/15/indexes-bitmap-scans.html); [Oracle: diseño de Bitmap](https://docs.oracle.com/en/database/oracle/oracle-database/21/tgdba/designing-and-developing-for-performance.html).
