---
tags:
  - tp6
---

# Análisis de ejercicios de seguridad


| Ejercicio | Qué registrar |
|---|---|
| 1 | Grafo inicial, intento inválido de delegar DELETE y efecto de revocaciones por tabla/columna |
| 2 | Orden de operaciones, usuarios/roles ya existentes, privilegios de PUBLIC y capacidad de delegar |
| 3 | Permisos por usuario antes y después del CASCADE; no mezclar INSERT y SELECT |

Ejemplo del ejercicio 3: si A concede INSERT y SELECT a B con delegación, y SELECT a C sin ella, B puede delegar ambos; C solo puede consultar, salvo que obtenga otra concesión. Si B delega INSERT a D y A retira ese INSERT a B con CASCADE, D pierde el camino dependiente; los SELECT no se ven afectados por esa revocación de INSERT.

## Plantilla de estado después de cada comando

Mantener una tabla por usuario con privilegios utilizables, privilegios delegables y fuente de cada uno. Para una sentencia GRANT: revisar la autoridad del otorgante, el objeto y las columnas; después incorporar solo lo concedido válidamente. Para REVOKE: localizar la concesión retirada y seguir sus dependencias.

No agrupar INSERT y SELECT en una misma marca de «acceso»: una revocación de INSERT no cambia SELECT. Tampoco atribuir WGO al destinatario si la sentencia que lo beneficia no lo incluye.

## Un razonamiento del ejercicio 3

A es propietario; B recibe INSERT y SELECT con WGO; C recibe solo SELECT. Entonces A y B pueden conceder SELECT a D; C no puede apoyándose solo en lo dado. Los tres pueden consultar, pero solo A y B pueden insertar en ese punto. Si B concede INSERT a D, D puede usarlo sin delegarlo, porque esa concesión no incluyó WGO.

Cuando A retira INSERT a B con CASCADE, se pierden ese INSERT y la concesión B→D que dependía de él. B conserva SELECT con WGO; C conserva SELECT; D no gana ni pierde SELECT si nunca hubo una concesión sobre ese privilegio. La respuesta debe reflejar las sentencias realmente ejecutadas, no todos los permisos que alguien podría haber concedido.

## Fuente de estudio

Enunciado del TP6 en la carpeta `../Practicos` y filminas de cátedra `BDI_06 (Seguridad).pdf` (carpeta de cursada 2023).
