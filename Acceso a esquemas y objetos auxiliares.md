---
tags:
  - tp6
---

# Acceso a esquemas y objetos auxiliares


```sql
GRANT USAGE ON SCHEMA public TO ins_prov;
```

USAGE no da automáticamente SELECT/INSERT sobre todas las tablas. A la inversa, un permiso de tabla puede ser insuficiente sin acceso al esquema. Las secuencias usadas por valores por defecto también pueden requerir permisos propios, según cómo esté definida la columna.

Una vista puede exponer solo determinadas columnas o filas, pero hay que revisar permisos sobre las tablas base y el contexto de ejecución. Una función con privilegios de su propietario requiere especial cuidado con el esquema de resolución de nombres; no asumir que todo código se ejecuta con permisos del dueño.
