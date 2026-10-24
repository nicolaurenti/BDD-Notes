---
tags:
  - tp6
---

# TP6 - Ejemplo comparado de RESTRICT, CASCADE y GRANT OPTION FOR

Cadena A→B→C para comparar los tres resultados sobre la misma situación inicial.

## Comparar tres revocaciones sobre la misma cadena

Partiendo de `A → B con SELECT y WGO → C con SELECT`, sin otros caminos:

| Revocación de A a B | B después | C después |
|---|---|---|
| SELECT con RESTRICT | Se rechaza por la concesión dependiente; conserva lo anterior | Conserva lo anterior |
| SELECT con CASCADE | Pierde SELECT y su delegación | Pierde el SELECT dependiente |
| GRANT OPTION FOR SELECT con CASCADE | Conserva SELECT, pierde la delegación | Pierde el SELECT dependiente |

En el primer caso no se retira primero el permiso y se deja a C «colgado»: se rechaza la operación que produciría esa dependencia inválida. En CASCADE se siguen solo los caminos afectados; una concesión independiente puede mantener el acceso.

## Fuente de estudio

Enunciado del TP6 en la carpeta `../../Practicos` y filminas de cátedra `BDI_06 (Seguridad).pdf` (carpeta de cursada 2023).
