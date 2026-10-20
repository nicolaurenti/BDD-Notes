---
tags:
  - tp8
---

# Recuperabilidad y abortos en cascada


Si T2 lee un valor escrito por T1:

- **Recuperable:** T2 no confirma antes de T1.
- **Sin abortos en cascada:** T2 espera el COMMIT de T1 para leer ese valor.
- **Estricto:** nadie lee ni sobrescribe un ítem escrito por T1 hasta que T1 termine.

Un grafo de precedencia sin ciclos no demuestra por sí solo que el orden de COMMIT sea recuperable. Distinguir grafo de conflictos y grafo de esperas: este último se usa para detectar deadlock.
