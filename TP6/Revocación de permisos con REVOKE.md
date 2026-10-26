---
tags:
  - tp6
---

# Revocación de permisos con REVOKE

## Revocar permisos con REVOKE

REVOKE retira una concesión de un privilegio. La estructura es REVOKE privilegio ON tabla FROM usuario, seguida del comportamiento ante permisos dependientes: CASCADE o RESTRICT.

Supongamos esta secuencia:

```sql
-- Ejecutado por A, propietario:
GRANT SELECT ON voluntario TO B WITH GRANT OPTION;

-- Ejecutado por B:
GRANT SELECT ON voluntario TO C;
```

El grafo es:

```text
A ── SELECT* ──→ B ── SELECT ──→ C
```

Si A ejecuta:

```sql
REVOKE SELECT ON voluntario FROM B CASCADE;
```

Se retira el permiso concedido a B y también la concesión dependiente de B a C. Suponiendo que no tienen otros permisos, ninguno puede seguir consultando la tabla. CASCADE propaga la revocación por las concesiones que quedan sin sustento; no elimina usuarios, tablas ni datos.

### Otro camino válido

Si A también había concedido SELECT directamente a C, esa concesión no depende de B:

```text
A ── SELECT* ──→ B ── SELECT ──→ C
A ── SELECT ──────────────────→ C
```

Al revocar a B con CASCADE, C conserva SELECT por su concesión directa de A. Para resolver un ejercicio hay que examinar todos los caminos válidos, no solo la cadena afectada.

### RESTRICT

Con el grafo inicial, A podría intentar:

```sql
REVOKE SELECT ON voluntario FROM B RESTRICT;
```

Como existe una concesión dependiente a C que tendría que retirarse, la sentencia falla y no realiza la revocación. RESTRICT exige que no haya una revocación dependiente que efectuar; CASCADE permite propagarla. Es conveniente escribir explícitamente la modalidad pedida por el ejercicio.

### Retirar solamente la capacidad de delegar

También es posible retirar la opción de delegación conservando el privilegio de uso:

```sql
REVOKE GRANT OPTION FOR SELECT ON voluntario FROM B CASCADE;
```

B conserva SELECT, pero pierde la capacidad de otorgarlo por esa concesión. Los permisos que había delegado y que dependían exclusivamente de esa autorización se revocan en cascada. Como siempre, otros caminos independientes pueden sostener permisos y opciones de delegación.

## Material

Enunciado: Practicos/Practico 6 - Seguridad.docx, dentro de la carpeta Bases de Datos, fuera del baúl.

Referencia para los detalles indicados de PostgreSQL: [GRANT](https://www.postgresql.org/docs/current/sql-grant.html) y [privilegios](https://www.postgresql.org/docs/17/ddl-priv.html).
