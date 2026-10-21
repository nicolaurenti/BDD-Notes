---
tags:
  - tp8
---

# Bloqueos compartidos y exclusivos


| Bloqueos sobre el mismo ítem | Compatible |
|---|---|
| S con S | Sí |
| S con X | No |
| X con X | No |

S permite leer; X permite escribir. En el práctico aparecen `r-lock` y `s-lock` como bloqueo compartido, y `w-lock` como exclusivo.

Una solicitud incompatible espera hasta que se libere el bloqueo. Al simular, una transacción bloqueada no puede seguir ejecutando instrucciones posteriores como si hubiera adquirido el recurso.

## Lectura, escritura y espera

Antes de leer un ítem se solicita el compartido; antes de escribir, el exclusivo. Si T1 tiene S(A), T2 puede obtener S(A), pero no X(A). Si T1 tiene X(A), ninguna de esas solicitudes de T2 se concede hasta liberar el bloqueo.

En el TP se usa tanto r-lock como s-lock para compartido: tomar el significado declarado por el enunciado, sin interpretar la letra s como «exclusivo». W-lock corresponde a escritura.

Supongamos T1 con X(B) y T2 que solicita S(B). Se anota «T2 espera por T1». Mientras espera, T2 no puede ejecutar su lectura de B, liberar B como si lo tuviera y calcular A+B. Primero debe obtenerlo. Esta regla cambia el resultado de una simulación si se trasladan los unlock al final.

## Diferenciar solicitud y concesión

En una tabla temporal conviene anotar por ítem quién tiene el bloqueo y quién espera. Un comando escrito en el schedule es una solicitud; solo cuando resulta compatible se continúa con la siguiente operación de esa transacción. El grafo de esperas tiene una arista del que espera al que retiene el recurso. Se usa para deadlock y no debe confundirse con el grafo de conflictos de serializabilidad.

## Fuente de estudio

Enunciado del TP8 en la carpeta `../Practicos` y filminas de cátedra `BDI_08 (Proc_Transacciones).pdf` (carpeta de cursada 2023).
