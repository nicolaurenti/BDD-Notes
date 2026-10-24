---
tags:
  - tp4
---

# TP4 - Ejemplo de grilla de eventos críticos

Regla del ejemplo: cada proveedor solo puede proveer a sucursales de su misma localidad.

## Completar la grilla de eventos

Para «cada proveedor solo provee a sucursales de su localidad», la relación se registra en PROVEE, pero las localidades están en las tablas de sus extremos.

| Tabla | INSERT | UPDATE | DELETE |
|---|---|---|---|
| PROVEE | Revisar nueva asociación | Revisar si cambia proveedor o sucursal | Elimina una asociación: no crea esta infracción |
| PROVEEDOR | No crea una asociación por sí solo | Un cambio de localidad exige revisar sus asociaciones | Revisar la acción referencial; restringir o quitar asociaciones no crea una desigualdad |
| SUCURSAL | No crea una asociación por sí sola | Un cambio de localidad exige revisar sus asociaciones | Mismo razonamiento sobre sus referencias |

Si proveedor y sucursal son de Tandil, la asociación es válida. Si luego la sucursal pasa a Azul, puede quedar inválida **sin tocar PROVEE**. Un único trigger de INSERT sobre PROVEE no mantiene la regla completa.

Para «horas dentro de los límites de la tarea» hay que revisar inserción/cambio de horas o tarea del voluntario y cambios de mínimos/máximos en TAREA. Para «nadie supera a su coordinador», también es crítico reducir las horas de quien coordina a otros.

## Fuente de estudio

Enunciado del TP4 en la carpeta `../../Practicos` y filminas de cátedra `BDI_04 (SQL Procedural).pdf` (carpeta de cursada 2023).
