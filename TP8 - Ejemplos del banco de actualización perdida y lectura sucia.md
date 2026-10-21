---
tags:
  - tp8
---

# TP8 - Ejemplos del banco de actualización perdida y lectura sucia

Intercalados del banco con transferencia, depósito y cálculo de totales, suponiendo ausencia del control de concurrencia indicado en la consigna.

## Ejemplo del banco: actualización perdida

Saldo inicial del cliente 10: 10000. T1 prepara una resta de 5000 y T3 un depósito de 15000.

```text
T1 lee 10000 y calcula 5000.
T3 lee 10000 y calcula 25000.
T3 escribe 25000 y confirma.
T1 escribe su cálculo viejo: 5000 y confirma.
```

Se pierde el depósito: el saldo correcto al aplicar ambas operaciones sería 20000. Es una simulación de un sistema sin control adecuado o de una aplicación que escribe resultados viejos; no afirmar que cualquier UPDATE aritmético en PostgreSQL se comporta así.

## Lectura sucia y transferencia parcial

Si T1 escribe el débito, T2 lo lee antes del COMMIT y después T1 aborta, T2 usó un valor que nunca quedó confirmado. Si T2 observa el débito sin el crédito correspondiente, puede obtener un total 5000 menor al correcto. Son problemas relacionados pero distintos.

## Fuente de estudio

Enunciado del TP8 en la carpeta `../Practicos` y filminas de cátedra `BDI_08 (Proc_Transacciones).pdf` (carpeta de cursada 2023).
