---
tags:
  - tp5
---

# Práctico 5 — Vistas

Una vista deriva información de tablas u otras vistas. El práctico pide definirla, justificar si admite actualizaciones y seguir la propagación hacia la base. Se separa el criterio SQL-1999 de las filminas del comportamiento PostgreSQL cuando la consigna pide ambos.

## 1. [[Definición y uso de vistas]]

## 2. [[Actualización de vistas y restricciones base]]

## 3. [[WITH CHECK OPTION y migración de filas]]

## 4. [[CHECK OPTION LOCAL y CASCADED]]

## 5. [[Preservación de clave en vistas de ensamble]]

## 6. [[Triggers INSTEAD OF sobre vistas]]

## 7. [[Vistas con agregados]]

## 8. [[Vistas simples con EXISTS]]

## 9. [[Operaciones acumulativas sobre vistas]]

## Checklist del práctico

- [ ] Definir vistas simples, anidadas, de join y agregadas.
- [ ] Distinguir actualizabilidad de validez de una escritura concreta.
- [ ] Explicar migración de filas y WCO.
- [ ] Resolver LOCAL/CASCADED sin ignorar WCO inferiores.
- [ ] Determinar claves preservadas y ambigüedad de propagación.
- [ ] Implementar una política explícita con INSTEAD OF.
- [ ] Conservar entidades sin relacionados en vistas de conteos.
- [ ] Seguir operaciones acumulativas y restricciones base.

## Conocimientos anteriores

- Relacionados: TP4 - Triggers; joins, conteos y subconsultas del TP2.

## Criterio de estudio y fuentes

Las explicaciones se apoyan en este práctico y los anteriores. Se conserva la sintaxis de la cátedra; las diferencias de PostgreSQL se tratan por separado cuando las pide la consigna. Los ejemplos de teoría son didácticos y no sustituyen la resolución personal.

- Enunciado local: `../Practicos/Practico 5 - Vistas.docx`.
- Filminas cotejadas: `BDI_05 (Vistas).pdf`, en `../Contenido extra/Filminas y Practicos de cursada/2023/`.
