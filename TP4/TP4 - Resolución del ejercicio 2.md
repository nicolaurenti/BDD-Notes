---
tags:
  - tp4
---

# TP4 - Resolución del ejercicio 2

Se necesita registrar quién y cuándo realizó INSERT, UPDATE o DELETE sobre ENTREGA o RENGLON_ENTREGA. La tabla histórica debe conservar los registros aunque las filas originales desaparezcan.

## Implementación

### Crear HIS_ENTREGA: campos, identificación y tabla de origen

Se mantienen las cinco columnas mínimas y se agrega `tabla_origen` para distinguir ENTREGA y RENGLON_ENTREGA. TIMESTAMP conserva fecha y hora. Se amplía la precisión numérica para no limitar la auditoría a 9999 registros.

```sql
-- Tabla común a las dos alternativas de auditoría. No usar ambas a la vez.
-- No se usan SEQUENCE, SERIAL ni IDENTITY.
CREATE TABLE his_entrega (
    nro_registro NUMERIC(10,0),
    fecha TIMESTAMP NOT NULL,
    operacion VARCHAR(10) NOT NULL,
    cant_reg_afectados NUMERIC(10,0) NOT NULL,
    usuario VARCHAR(120) NOT NULL,
    tabla_origen VARCHAR(40) NOT NULL,
    CONSTRAINT pk_his_entrega PRIMARY KEY (nro_registro)
);
```

No se declaran FK hacia las filas auditadas: un registro de DELETE debe poder persistir después de su baja. nro_registro identifica el **registro de auditoría**, no una entrega ni un renglón.

El valor se calcula en la función con `COALESCE(MAX(nro_registro), 0) + 1`, sin SEQUENCE, SERIAL ni IDENTITY. Ese método se usa con las pruebas secuenciales del práctico; no se presenta como numeración segura para escrituras simultáneas.

Fuente: `Practico 4 - Acciones Procedurales.docx`, TP3 y esquema correspondiente de la cátedra. Se usa `EXECUTE PROCEDURE`.

### Elegir una alternativa

- [[TP4 - Resolución 2 - Auditoría por fila|FOR EACH ROW: una entrada de auditoría por fila]].
- [[TP4 - Resolución 2 - Auditoría por sentencia|FOR EACH STATEMENT: una entrada con el conteo global]].

### Aplicar el DELETE del ejercicio

[[TP4 - Resolución 2 - Resultado del DELETE y acciones referenciales|Resolver DELETE FROM ENTREGA WHERE id_video = 3582]].

Las alternativas se comparan en esquemas de práctica separados. Instalarlas juntas registraría dos veces las operaciones y no representaría ninguna de las dos soluciones pedidas.

## Pruebas realizadas

En ambas tablas se probaron INSERT, UPDATE y DELETE de varias filas. También se probó una sentencia que afecta cero filas: ROW no registra nada; STATEMENT registra una entrada con cantidad cero. Las funciones comparten cuerpo por tabla usando TG_TABLE_NAME, pero cada tabla tiene sus propios triggers.

La cantidad para una sentencia se obtuvo con un contador auxiliar; ni COUNT(*) de la tabla final ni OLD/NEW de un trigger STATEMENT proporcionan esa cantidad automáticamente.

Fuente: `Practico 4 - Acciones Procedurales.docx`, TP3 y esquema correspondiente de la cátedra. Se usa `EXECUTE PROCEDURE`.
