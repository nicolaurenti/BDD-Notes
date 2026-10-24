---
tags:
  - tp4
---

# TP4 - Resolución 2 - Tabla de auditoría

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
