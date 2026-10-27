---
tags:
  - tp8
---

# Log de transacciones y estrategias de recuperación

**Recuperar significa conservar las transacciones confirmadas y eliminar los efectos de las que quedaron incompletas.** Así se sostienen atomicidad y durabilidad ante una falla.

## Qué información conserva el log

El log o bitácora registra el avance de las transacciones. En el TP se usa:

```text
<T1, START>
<T1, A, 100, 120>
<T1, COMMIT>
```

El registro de escritura significa: T1 cambió A de **100 a 120**. No significa sumar 120. `COMMIT` marca confirmación; `ABORT` registra aborto. Algunas variantes distinguen además el fin del procesamiento (`END` o `FIN`).

La **escritura anticipada del log (WAL)** exige persistir la información necesaria antes de que la página modificada llegue al disco. Para confirmar de forma durable debe persistirse la información de log requerida, incluido el commit. Tener el log solo en memoria no protege contra su pérdida.

## UNDO y REDO

- **UNDO:** restaura el valor anterior. Recorre las escrituras que deben deshacerse **en orden inverso**.
- **REDO:** instala el valor nuevo. Recorre las escrituras que deben rehacerse **en orden original**.

Si T1 escribió A: 100→120 y después 120→140, deshacer exige 140→120→100. Rehacer exige 100→120→140.

Son asignaciones **idempotentes**: repetir “A=120” mantiene 120. Repetir “A=A+20” vuelve a incrementar y no sirve como reproducción directa del mismo registro.

## Estrategias del práctico

La clave es separar **confirmación** de **escritura de páginas en disco**. Los cambios pueden permanecer en buffers y perderse ante una falla.

| Estrategia | Qué puede llegar al disco | Sin commit al fallar | Con commit, si los cambios no están garantizados en disco |
|---|---|---|---|
| Diferida: NO UNDO/REDO | Solo cambios de transacciones confirmadas | No requiere UNDO | Requiere REDO |
| Inmediata: UNDO/NO REDO | Puede incluir cambios aún no confirmados; al confirmar se fuerzan los cambios | Requiere UNDO | No ocurre: la política garantiza su persistencia |
| Inmediata: UNDO/REDO | Puede incluir cambios no confirmados; confirmar no fuerza todas las páginas | Requiere UNDO | Requiere REDO |

**Diferida:** el trabajo no confirmado permanece en un espacio privado; una falla previa al commit no deja datos de esa transacción para deshacer. Después del commit puede faltar aplicar parte del trabajo: por eso existe REDO.

**Inmediata:** una página con cambios no confirmados puede haberse escrito. Se necesita UNDO. La diferencia entre sus dos variantes es si **todas** las páginas modificadas deben persistir antes de dar por confirmado el trabajo.

“Confirmó → REDO” es una regla de UNDO/REDO y NO UNDO/REDO bajo sus condiciones; no es una regla universal. Un checkpoint puede garantizar que ciertos cambios ya estén persistidos.

## Alcance del modelo

El TP trabaja con falla del sistema y almacenamiento persistente disponible: se pierde memoria, pero sobreviven datos y log durable. Una pérdida del disco necesita respaldos y, cuando existen, registros posteriores.

Las reglas anteriores son el modelo didáctico de la cátedra. No describen literalmente todos los pasos internos de recuperación de PostgreSQL. Tampoco sustituyen un control de concurrencia que impida dependencias incompatibles con la recuperación.
