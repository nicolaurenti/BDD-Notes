---
tags:
  - tp6
---

# Práctico 6 — Seguridad y control de acceso

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

## Material

- Enunciado principal: `../Practicos/Practico 6 - Seguridad.docx`.
- Relacionados: TP5 - Vistas, TP4 - Triggers.
