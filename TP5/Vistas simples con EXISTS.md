---
tags:
  - tp5
---

# Vistas simples con EXISTS

Cuando solo se necesita filtrar entidades por relaciones, una subconsulta evita multiplicarlas y puede conservar un FROM simple:

```sql
CREATE VIEW distribuidores_con_entregas AS
SELECT d.*
FROM distribuidor d
WHERE EXISTS (
    SELECT 1
    FROM entrega e
    WHERE e.id_distribuidor = d.id_distribuidor
);
```

Esto es distinto de proyectar un join y eliminar duplicados con DISTINCT. Para «departamentos con más de 2 empleados», agrupar en la subconsulta por la clave completa de departamento.

## Relación entre el FROM y la actualizabilidad

Un distribuidor con diez entregas aparece diez veces en un join con ENTREGA. EXISTS lo conserva una vez porque solo exige encontrar alguna entrega; no incorpora sus filas al resultado exterior. Por eso no hace falta DISTINCT para recuperar una sola fila por distribuidor.

La vista del ejemplo mantiene a DISTRIBUIDOR como única fuente del FROM exterior y proyecta sus columnas directas. La subconsulta en WHERE filtra; no convierte cada columna en un cálculo. Esto permite justificar por qué puede conservar actualización automática en PostgreSQL, a diferencia del join exterior con DISTINCT.

Para «algún departamento con más de dos empleados», el conteo debe hacerse por departamento completo `(id_distribuidor, id_departamento)`. Contar todos los empleados del distribuidor puede aceptar uno con dos departamentos de dos empleados, aunque ninguno supere dos. Elegir una forma actualizable no autoriza a simplificar la condición hasta cambiar el enunciado.

## Fuente de estudio

Enunciado del TP5 en la carpeta `../../Practicos` y filminas de cátedra `BDI_05 (Vistas).pdf` (carpeta de cursada 2023).
