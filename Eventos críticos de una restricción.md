---
tags:
  - tp4
---

# Eventos críticos de una restricción


Ejemplo del TP: proveedor y sucursal deben ser de la misma localidad.

| Tabla | INSERT | UPDATE relevante | DELETE |
|---|---|---|---|
| PROVEE | Verificar la relación nueva | Si cambia proveedor o sucursal | Quita una posible infracción; considerar otras RI |
| PROVEEDOR | No crea relación por sí solo | Si cambia localidad, revisar sus relaciones | Depende de FK y cascadas |
| SUCURSAL | No crea relación por sí sola | Si cambia localidad, revisar sus relaciones | Depende de FK y cascadas |

Para cada restricción, listar **todas las operaciones capaces de romperla**. Ejemplos:

- Hasta 15 palabras: INSERT o cambio de artículo en `contiene`.
- Horas dentro de tarea: cambio de horas/tarea del voluntario **y** de los límites de la tarea.
- Horas no superiores al coordinador: cambio del coordinado **y** disminución de horas del coordinador.

Un trigger que cuenta filas puede fallar ante dos transacciones concurrentes que ven el mismo estado. Para garantizar límites globales se debe coordinar el acceso, por ejemplo bloqueando una fila padre común en todos los caminos relevantes, o usando aislamiento serializable con reintentos. Ver TP8 - Concurrencia y recuperación.
