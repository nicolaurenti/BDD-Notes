---
tags:
  - tp6
---

# Grafo de permisos


Representar una arista por concesión con: otorgante, receptor, objeto, privilegio, columnas y si puede delegarse.

```text
Propietario -- SELECT + delegación --> adm -- SELECT --> doc
Propietario -- UPDATE + delegación --> adm -- UPDATE de columnas + delegación --> doc
Propietario -- DELETE -------------> adm
```

Si `adm` intenta `GRANT DELETE ... TO doc`, esa arista no queda autorizada por el DELETE recibido sin grant option. Antes de analizar una revocación, eliminar del grafo las concesiones que nunca fueron válidas.

Para cada sentencia preguntar: ¿el otorgante tiene autoridad para conceder ese privilegio y esas columnas? ¿La tiene por algún otro camino?

## Construirlo sentencia por sentencia

Dibujar un grafo separado por privilegio o rotular claramente cada arista. En cada paso, comprobar primero la autoridad del otorgante. Si la sentencia no está autorizada, no se agrega una arista y las sentencias siguientes no pueden apoyarse en ella.

Ejemplo con SELECT sobre T:

```text
A -- SELECT con WGO --> B -- SELECT --> D
A -- SELECT ---------> D
```

D tiene dos concesiones independientes: una de B y otra de A. Si A retira a B SELECT con CASCADE, desaparecen A→B y la concesión dependiente B→D, pero queda A→D. D sigue consultando.

La tabla auxiliar ayuda a no perder información:

| Otorgante | Receptor | Objeto | Privilegio | Columnas | WGO |
|---|---|---|---|---|---|
| A | B | T | SELECT | Todas | Sí |
| B | D | T | SELECT | Todas | No |
| A | D | T | SELECT | Todas | No |

No fusionar las dos entradas de D en una sola que diga «D tiene SELECT»: para calcular una revocación hace falta saber **de dónde lo obtuvo**. El grafo representa concesiones, no solo el resultado final de acceso.

## Fuente de estudio

Enunciado del TP6 en la carpeta `../Practicos` y filminas de cátedra `BDI_06 (Seguridad).pdf` (carpeta de cursada 2023).
