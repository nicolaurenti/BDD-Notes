---
tags:
  - tp8
---

# Problemas de concurrencia entre transacciones

**El problema aparece cuando la intercalación cambia el significado del trabajo.** No alcanza con que cada sentencia individual sea correcta. En estos ejemplos teóricos suponemos que no hay un mecanismo que impida la anomalía.

## 1. Actualización perdida

Dos transacciones calculan cambios a partir del mismo valor viejo y una sobrescribe el resultado de la otra. Saldo inicial: 100.

| Paso | T1: sumar 20 | T2: restar 10 |
|---|---|---|
| 1 | Lee 100 | |
| 2 | | Lee 100 |
| 3 | Escribe 120 y confirma | |
| 4 | | Escribe 90 y confirma |

El saldo termina en **90**, aunque aplicando ambos cambios debería ser **110**. Se perdió el incremento de T1. Cada transacción conserva el valor que leyó en su memoria local; escribir no actualiza automáticamente esa copia en la otra transacción.

## 2. Lectura sucia

T1 cambia el saldo de 100 a 120 sin confirmar. T2 lee 120. Luego T1 aborta y el saldo vuelve a 100: T2 utilizó un valor que nunca quedó confirmado.

La lectura ya es sucia cuando utiliza un cambio **ajeno no confirmado**, aunque T1 finalmente confirme. El aborto posterior hace visible el riesgo. Si T2 depende de ese dato, puede ser necesario revertirla también: **rollback en cascada**. Si ya confirmó sobre esa dependencia, el plan puede ser irrecuperable.

## 3. Lectura no repetible y fantasma

- **No repetible:** T1 lee una cuenta con saldo 100; T2 la cambia a 130 y confirma; T1 vuelve a leer la misma cuenta y obtiene 130.
- **Fantasma:** T1 consulta las cuentas con saldo mayor que 1000; T2 inserta una cuenta que cumple la condición y confirma; T1 repite la consulta y aparece una fila adicional. También puede cambiar el conjunto por una baja o por una actualización.

La diferencia útil es **valor de un dato ya leído** frente a **conjunto de filas que cumple una condición**. Las filminas describen el fantasma como un cambio de resultado de consulta; esta distinción permite reconocerlo con precisión.

## 4. Análisis inconsistente

T1 transfiere 50 entre A=100 y B=200. T2 suma ambas cuentas mientras ocurre la transferencia:

1. T1 descuenta: A queda en 50.
2. T2 lee A=50 y B=200, antes de la acreditación.
3. T1 acredita: B queda en 250 y confirma.

T2 obtiene **250**, aunque el total antes y después es **300**. Mezcló etapas distintas de una operación lógica. No necesita leer dos veces un mismo dato para equivocarse.

## Cómo reconocerlos en una grilla

Anotá qué valor lee cada transacción, cuándo escribe y cuándo confirma o aborta. Después compará el resultado con los órdenes seriales posibles. Escribir un valor no significa confirmarlo, y recibir una lectura no garantiza que represente un estado consistente del conjunto.
