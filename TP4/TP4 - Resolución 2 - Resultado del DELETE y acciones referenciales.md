---
tags:
  - tp4
---

# TP4 - Resolución 2 - Resultado del DELETE y acciones referenciales

La operación pedida es:

```sql
DELETE FROM entrega WHERE id_video = 3582;
```

Primero se analiza la FK de RENGLON_ENTREGA hacia ENTREGA. La auditoría no elimina ni reemplaza esa restricción.

## Con el SQL local proporcionado

En `catedra/peliculas.sql`, `fk_re_entrega` no declara ON DELETE CASCADE: queda NO ACTION. Si alguna entrega seleccionada tiene renglones que la referencian, el DELETE se rechaza. ENTREGA y RENGLON_ENTREGA quedan sin cambios; tampoco quedan nuevas filas de auditoría de la sentencia fallida.

Si se seleccionan entregas sin renglones, sí pueden borrarse: ROW registra una fila por entrega y STATEMENT una entrada con su cantidad. Si no se selecciona ninguna, ROW registra cero entradas y STATEMENT una con cantidad cero.

## Si la instancia del ejercicio declara CASCADE

Si la FK permite propagar la baja, se borran las entregas seleccionadas y sus renglones. Con M entregas y R renglones, ROW registra M + R entradas, cada una con cantidad 1.

En la prueba controlada con dos entregas y tres renglones asociados, la alternativa STATEMENT de esta resolución produjo dos entradas: ENTREGA/DELETE/cantidad 2 y RENGLON_ENTREGA/DELETE/cantidad 3. Se verificó el resultado ejecutándolo; no se dedujo que bastara cambiar FOR EACH ROW por STATEMENT.

### Ejemplo reproducible: datos iniciales, tablas finales y auditoría

El enunciado no proporciona la instancia de datos de las entregas. Por eso estos identificadores son un **ejemplo controlado**, no una afirmación sobre tus datos actuales.

ENTREGA inicial:

| nro_entrega | fecha_entrega | id_video | id_distribuidor |
|---|---|---:|---:|
| 100 | 2026-01-01 | 3582 | 1 |
| 101 | 2026-01-02 | 3582 | 1 |
| 102 | 2026-01-03 | 999 | 1 |

RENGLON_ENTREGA inicial:

| nro_entrega | codigo_pelicula | cantidad |
|---:|---:|---:|
| 100 | 1 | 10 |
| 100 | 2 | 20 |
| 101 | 1 | 30 |
| 102 | 1 | 40 |

Se parte de HIS_ENTREGA vacía y se ejecuta el DELETE por id_video 3582.

#### Con NO ACTION

El DELETE se rechaza por los tres renglones que referencian las entregas 100 y 101. Las dos tablas conservan todas las filas de arriba. HIS_ENTREGA sigue vacía para ambas alternativas.

#### Con ON DELETE CASCADE

ENTREGA queda con una única fila:

| nro_entrega | fecha_entrega | id_video | id_distribuidor |
|---|---|---:|---:|
| 102 | 2026-01-03 | 999 | 1 |

RENGLON_ENTREGA queda con `(102, 1, 40)`.

HIS_ENTREGA, alternativa ROW:

| tabla_origen | operacion | cant_reg_afectados |
|---|---|---:|
| entrega | DELETE | 1 |
| entrega | DELETE | 1 |
| renglon_entrega | DELETE | 1 |
| renglon_entrega | DELETE | 1 |
| renglon_entrega | DELETE | 1 |

HIS_ENTREGA, alternativa STATEMENT probada:

| tabla_origen | operacion | cant_reg_afectados |
|---|---|---:|
| entrega | DELETE | 2 |
| renglon_entrega | DELETE | 3 |

Las tablas muestran el contenido, no un orden garantizado de disparo ni de consulta. Cada entrada tiene además su nro_registro, el timestamp de la prueba y CURRENT_USER. Esos valores dependen de la ejecución.

No se cambió la FK de tus tablas. El caso con cascada se probó en otro esquema temporal cuya definición ya la incluía.
