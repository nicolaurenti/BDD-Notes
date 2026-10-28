---
tags:
  - tp7
---

# TP7 - Optimización de consultas

Resumen dividido en ocho temas para estudiar sin tanto desplazamiento. Cada nota conserva la estructura **idea central → cómo razonarlo → ejemplo → errores frecuentes → qué recordar**.

## Temas

1. [[TP7 - Planes de ejecución]]
2. [[TP7 - Reescrituras equivalentes y comparación de planes]]
3. [[TP7 - Joins, subconsultas y ensambles innecesarios]]
4. [[TP7 - DISTINCT y eliminación de duplicados]]
5. [[TP7 - Agrupamiento y conteos]]
6. [[TP7 - Índices simples y compuestos]]
7. [[TP7 - Índices según las condiciones de búsqueda]]
8. [[TP7 - Árboles B+ y Bitmap]]

## Resolución del práctico

[[Resolución del TP7 - Optimización de consultas]] reúne los cinco ejercicios, sus justificaciones y los planes comprobados.

## Checklist de repaso

- [ ] Puedo leer un árbol de plan y distinguir estimado/real.
- [ ] Puedo explicar las heurísticas y las estadísticas que usa el optimizador.
- [ ] Justifico la equivalencia con claves, multiplicidades y NULL.
- [ ] Distingo JOIN, EXISTS y ausencia.
- [ ] Sé cuándo DISTINCT y COUNT cambian el resultado.
- [ ] Elijo columnas y orden de índices con una justificación.
- [ ] Analizo funciones, fechas y patrones LIKE.
- [ ] Distingo B+, Bitmap persistente y bitmap scan.

## Material y alcance

Fuentes de cátedra, dentro de Bases de Datos y fuera del baúl:

- Practicos/Practico 7 - Optimización de Consultas.docx, incluidos sus diagramas de investigación y mensajería.
- Contenido extra/Filminas y Practicos de cursada/2023/BDI_07 (Optimizacion).pdf: procesamiento, heurísticas, estadísticas, equivalencias algebraicas y EXPLAIN.

Las referencias oficiales de PostgreSQL complementan los detalles del motor; las reglas de Oracle se identifican donde las usa el enunciado. Los ejemplos numéricos del resumen son ilustrativos. Los planes comprobados y sus condiciones se documentan en la resolución.
