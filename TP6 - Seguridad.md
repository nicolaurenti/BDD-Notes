---
tags:
  - tp6
---

# Práctico 6 — Seguridad y control de acceso

El objetivo es determinar quién puede realizar cada operación y quién puede conceder ese permiso. Las notas siguen el grafo de concesiones, las revocaciones y los roles usando los comandos y dependencias que presenta la cátedra.

## 1. [[Usuarios, roles, objetos y privilegios]]

## 2. [[GRANT y WITH GRANT OPTION]]

## 3. [[Grafo de permisos]]

## 4. [[REVOKE, RESTRICT y CASCADE]]

## 5. [[Permisos efectivos y PUBLIC]]

## 6. [[Creación y administración de roles]]

## 7. [[Acceso a esquemas y objetos auxiliares]]

## 8. [[Análisis de ejercicios de seguridad]]

## Errores frecuentes

- Confundir tener un permiso con poder delegarlo.
- Tratar ALL como superusuario o propietario.
- Suponer que SELECT habilita INSERT.
- Ignorar PUBLIC o membresías al calcular permisos efectivos.
- Revocar una columna y creer que eso anula una concesión de tabla.
- Quitar un rol y suponer que desaparecen sus usuarios.

Los comandos son ejemplos de estudio; no se modificaron roles ni permisos reales.

## Checklist del práctico

- [ ] Identificar objeto, privilegio y otorgante.
- [ ] Distinguir privilegio, grant option y admin option.
- [ ] Dibujar y actualizar el grafo de permisos.
- [ ] Seguir CASCADE y concesiones independientes.
- [ ] Diferenciar permisos de tabla y columna.
- [ ] Calcular acceso por roles y PUBLIC.
- [ ] Revisar esquema y objetos auxiliares.

## Conocimientos anteriores

- Relacionados: TP5 - Vistas, TP4 - Triggers.

## Criterio de estudio y fuentes

Las explicaciones se apoyan en este práctico y los anteriores. Se conserva la sintaxis de la cátedra; las diferencias de PostgreSQL se tratan por separado cuando las pide la consigna. Los ejemplos de teoría son didácticos y no sustituyen la resolución personal.

- Enunciado local: `../Practicos/Practico 6 - Seguridad.docx`.
- Filminas cotejadas: `BDI_06 (Seguridad).pdf`, en `../Contenido extra/Filminas y Practicos de cursada/2023/`.
