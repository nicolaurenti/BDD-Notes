---
tags:
  - tp2
---

# Consultas con “todos”

## Patrón de doble NOT EXISTS

**No existe un elemento requerido para el cual no exista una relación.**

Voluntarios que trabajaron históricamente en todas las instituciones:

```sql
SELECT v.nro_voluntario, v.nombre, v.apellido
FROM voluntario v
WHERE NOT EXISTS (
    SELECT 1
    FROM institucion i
    WHERE NOT EXISTS (
        SELECT 1
        FROM historico h
        WHERE h.nro_voluntario = v.nro_voluntario
          AND h.id_institucion = i.id_institucion
    )
);
```

Lectura:

> Seleccionar voluntarios para los que no exista una institución en la que no hayan trabajado.

> [!note]
> Si el conjunto de instituciones está vacío, la condición se cumple para todos los voluntarios. Si el enunciado exige otra interpretación, hay que agregar una comprobación de existencia.

## Desarmar las dos negaciones con un ejemplo

Sean las instituciones requeridas 20 y 50. El historial contiene a Ana en ambas, a Luis solo en 20 y a Eva en ninguna.

Para Luis, la subconsulta más interna busca una relación con la institución evaluada. Para 20 la encuentra; para 50 no. Por eso el NOT EXISTS interno identifica a 50 como **institución faltante**. La subconsulta intermedia devuelve esa falta y el NOT EXISTS exterior excluye a Luis. Para Ana no hay institución faltante, así que se la incluye. Eva tiene faltantes y queda fuera.

Las dos correlaciones son necesarias: una fija a la persona exterior y otra a la institución requerida. Si falta cualquiera, se puede terminar probando relaciones de otras personas o con otras instituciones.

## Conjunto requerido y repeticiones

«Todas las instituciones» recorre INSTITUCION. «Todas las instituciones de un país» requiere filtrar ese conjunto en el nivel intermedio. Recorrer solo las instituciones que aparecen en HISTORICO cambiaría la pregunta: dejaría de exigir las que nunca recibieron voluntarios.

Varios períodos de Ana en la institución 20 no compensan su ausencia en 50. EXISTS solo pregunta si hay una coincidencia, así que los duplicados históricos no alteran esta prueba. Una solución por conteo necesita contar instituciones distintas y compararlas con el conjunto requerido; igualar cantidad de registros con cantidad de instituciones puede dar un falso positivo.

Con conjunto requerido vacío no puede encontrarse un faltante: por eso la condición resulta verdadera. Si la consigna exige al menos una institución, se expresa por separado, sin cambiar el significado de las dos negaciones.

## Fuente de estudio

Enunciado del TP2 en la carpeta `../../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
