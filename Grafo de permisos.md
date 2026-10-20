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
