---
tags:
  - tp4
---

# Validación de restricciones con triggers

El trabajo comienza por la regla y sus **eventos críticos**, no por escribir CREATE TRIGGER. Un evento es crítico si puede convertir un estado válido en inválido. La misma restricción puede necesitar controles en varias tablas.

## Elegir los eventos críticos

Para completar la grilla se enumeran todas las tablas y modificaciones capaces de romper la regla, incluidas las de los extremos de una relación.

### Ejemplo de grilla de eventos críticos

Regla del ejemplo: cada proveedor solo puede proveer a sucursales de su misma localidad.

#### Completar la grilla de eventos

Para «cada proveedor solo provee a sucursales de su localidad», la relación se registra en PROVEE, pero las localidades están en las tablas de sus extremos.

| Tabla | INSERT | UPDATE | DELETE |
|---|---|---|---|
| PROVEE | Revisar nueva asociación | Revisar si cambia proveedor o sucursal | Elimina una asociación: no crea esta infracción |
| PROVEEDOR | No crea una asociación por sí solo | Un cambio de localidad exige revisar sus asociaciones | Revisar la acción referencial; restringir o quitar asociaciones no crea una desigualdad |
| SUCURSAL | No crea una asociación por sí sola | Un cambio de localidad exige revisar sus asociaciones | Mismo razonamiento sobre sus referencias |

Si proveedor y sucursal son de Tandil, la asociación es válida. Si luego la sucursal pasa a Azul, puede quedar inválida **sin tocar PROVEE**. Un único trigger de INSERT sobre PROVEE no mantiene la regla completa.

Para «horas dentro de los límites de la tarea» hay que revisar inserción/cambio de horas o tarea del voluntario y cambios de mínimos/máximos en TAREA. Para «nadie supera a su coordinador», también es crítico reducir las horas de quien coordina a otros.

## Validar conteos por grupo

En un máximo por grupo, el conteo cambia según el momento del trigger: AFTER observa la fila incorporada; BEFORE todavía no. También se controla mover una asociación a otro grupo.

### Ejemplo de máximo de palabras por artículo

Implementación y pruebas del máximo de quince asociaciones por artículo. Se conserva la sintaxis del enunciado.

#### Ejemplo completo de un máximo por artículo

Se supone que cada fila de CONTIENE representa una asociación única de artículo y palabra. Un control AFTER ROW puede consultar el estado que ya incluye la inserción o el cambio de artículo:

```sql
CREATE OR REPLACE FUNCTION controlar_max_palabras()
RETURNS trigger AS $$
DECLARE
    cantidad INTEGER;
BEGIN
    SELECT COUNT(*) INTO cantidad
    FROM contiene
    WHERE id_articulo = NEW.id_articulo;

    IF cantidad > 15 THEN
        RAISE EXCEPTION 'El articulo supera las 15 palabras';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE 'plpgsql';

CREATE TRIGGER tr_max_palabras
AFTER INSERT OR UPDATE OF id_articulo ON contiene
FOR EACH ROW
EXECUTE PROCEDURE controlar_max_palabras();
```

Con 14 asociaciones, insertar una deja 15 y pasa. Con 15, insertar otra deja 16 y la excepción rechaza la sentencia; AFTER no significa que el cambio ya sea irreversible. Mover una asociación a un artículo con 15 también se rechaza. DELETE disminuye el conteo y no puede infringir este máximo.

El límite depende del momento: en BEFORE INSERT el conteo todavía no incluye la nueva fila, por lo que se rechazaría cuando el conteo previo ya fuera 15. Copiar la misma comparación `> 15` en un BEFORE admitiría indebidamente la número 16.

## Comparar y corregir la fila propuesta

OLD y NEW permiten reconocer cambios reales. BEFORE puede corregir NEW directamente; la prioridad entre reglas se expresa con IF y ELSIF para que una no rechace el efecto obligatorio de otra.

### Ejemplo de cambio de tarea y variación de horas

Desarrollo del ejercicio 5: reiniciar horas al cambiar tarea y comprobar una variación interpretada como margen de ±10 %. Se explicitan los supuestos.

#### Comparar valores anteriores y propuestos

Para un cambio real, se comparan OLD y NEW. `UPDATE OF id_tarea` indica que la columna figura en SET; no asegura que su valor sea distinto. `IS DISTINCT FROM` permite detectar diferencia incluso con NULL: ambos nulos no son distintos; solo uno nulo sí.

En el ejercicio 5, cambiar de tarea debe colocar horas en cero aunque la sentencia proponga otra cantidad. Se modifica NEW en BEFORE: así se guarda directamente la fila corregida, sin emitir otro UPDATE sobre la misma tabla.

```sql
CREATE OR REPLACE FUNCTION controlar_cambio_voluntario()
RETURNS trigger AS $$
BEGIN
    IF NEW.id_tarea IS DISTINCT FROM OLD.id_tarea THEN
        NEW.horas_aportadas := 0;
    ELSIF NEW.horas_aportadas IS DISTINCT FROM OLD.horas_aportadas THEN
        IF NEW.horas_aportadas IS NULL
           OR OLD.horas_aportadas IS NULL THEN
            RAISE EXCEPTION 'No se puede calcular la variacion con horas nulas';
        END IF;
        IF NEW.horas_aportadas < OLD.horas_aportadas * 0.90
           OR NEW.horas_aportadas > OLD.horas_aportadas * 1.10 THEN
            RAISE EXCEPTION 'La variacion supera el 10 por ciento';
        END IF;
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE 'plpgsql';

CREATE TRIGGER tr_cambio_voluntario
BEFORE UPDATE ON voluntario
FOR EACH ROW
EXECUTE PROCEDURE controlar_cambio_voluntario();
```

El segundo requisito tiene una redacción ambigua. Este ejemplo declara la interpretación «variación máxima de ±10 % respecto del valor anterior», con horas no negativas, y rechaza cambios de horas si no se puede calcular la variación por un nulo. No se atribuye esa política de nulos al enunciado.

Con 100 horas, 90 y 110 pasan; 89 y 111 no. Si cambia la tarea y la sentencia propone 105 horas, se guarda 0: el primer IF tiene prioridad y el ELSIF evita rechazar ese cero obligatorio por la regla porcentual. Con horas anteriores 0, un margen del 10 % solo permite 0.

## Comprobar el diseño

Para cada regla preparar un caso válido, uno inválido y uno en el límite. Además de insertar, modificar las columnas y tablas relacionadas: una prueba que nunca cambia TAREA no detecta la falta de ese control.

En la prueba, anotar estado previo, sentencia, resultado esperado y estado final. Un UPDATE con varias filas debe mantener la regla sobre todas; no probar solamente operaciones de una fila. Los fragmentos son material de estudio y no fueron instalados ni ejecutados en las bases del proyecto.

## Fuente de estudio

Enunciado del TP4 en la carpeta `../../Practicos` y filminas de cátedra `BDI_04 (SQL Procedural).pdf` (carpeta de cursada 2023).
