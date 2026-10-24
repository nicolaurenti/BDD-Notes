---
tags:
  - tp5
---

# TP5 - Ejemplos de diferencias entre LOCAL y CASCADED

Tabla de alternativas con V1 definida por cantidad >= 500 y V2 por cantidad < 1000.

## Un ejemplo donde LOCAL y CASCADED sí difieren

Sean V1, definida sobre ENVIO con `cantidad >= 500` y **sin WCO**, y V2, definida sobre V1 con `cantidad < 1000`. Si V2 tiene LOCAL y se inserta una cantidad 300 por ella, el control agregado por V2 solo exige `cantidad < 1000`: se puede insertar en la base aunque quede fuera de V1 y V2. Con CASCADED en V2 se exige también `cantidad >= 500` y se rechaza.

Si V1 ya tiene WCO, LOCAL en V2 no lo desactiva. Entonces 300 se rechaza también con LOCAL. Por eso no basta con mirar la última línea de la vista superior: hay que recorrer toda la cadena y anotar qué chequeos están activos.

| WCO de V1 (`>= 500`) | WCO de V2 (`< 1000`) | Insertar 300 por V2 |
|---|---|---|
| Ninguno | LOCAL | Supera el control local; queda oculta |
| Ninguno | CASCADED | Rechazo por el filtro de V1 |
| LOCAL | LOCAL | Rechazo por el WCO propio de V1 |

Los casos suponen válidas las demás restricciones de la tabla. En sintaxis SQL la palabra es **CASCADED**, aunque el enunciado use ocasionalmente «CASCADE» al describir las alternativas.

## Fuente de estudio

Enunciado del TP5 en la carpeta `../../Practicos` y filminas de cátedra `BDI_05 (Vistas).pdf` (carpeta de cursada 2023).
