---
tags:
  - tp8
---

# TP8 - Ejemplo de transferencia y propiedades ACID

Dos saldos iniciales y una transferencia de 5000 para distinguir atomicidad, consistencia, aislamiento y durabilidad.

## Ejemplo de transferencia

T1 debe restar 5000 al cliente 10 del banco 1 y sumarlos al cliente 13 del banco 2. Con saldos iniciales 10000 y 30000, el estado final correcto es 5000 y 35000: la suma sigue siendo 40000.

Atomicidad exige que no quede confirmado solo el débito. Consistencia exige respetar las reglas de la base antes y después de la unidad de trabajo. Aislamiento evita que una operación concurrente obtenga resultados incompatibles con el modelo de ejecución admitido. Durabilidad asegura la persistencia de la transferencia confirmada frente a una falla contemplada por el sistema.

```sql
BEGIN;
UPDATE cliente
SET saldo = saldo - 5000
WHERE id_cliente = 10 AND id_banco = 1;

UPDATE cliente
SET saldo = saldo + 5000
WHERE id_cliente = 13 AND id_banco = 2;
COMMIT;
```

Es una representación de la unidad lógica, no una ejecución sobre la base. En una implementación completa también deben comprobarse existencia de las cuentas y reglas de saldo que corresponda exigir. BEGIN y COMMIT no convierten una lógica incorrecta en correcta.

## Fuente de estudio

Enunciado del TP8 en la carpeta `../Practicos` y filminas de cátedra `BDI_08 (Proc_Transacciones).pdf` (carpeta de cursada 2023).
