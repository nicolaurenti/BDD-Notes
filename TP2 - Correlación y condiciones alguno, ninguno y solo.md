---
tags:
  - tp2
---

# TP2 - Correlación y condiciones alguno, ninguno y solo

Seguí una institución por vez y compará las preguntas «alguno», «ninguno» y «solo».

## Evaluar una correlación sin memorizar la plantilla

Supongamos instituciones 20, 50 y 80. Solo hay voluntarios en 20 y 50. Al evaluar el NOT EXISTS de la nota, para la institución 20 la subconsulta encuentra una fila y NOT EXISTS da FALSE. Para la 80 no encuentra ninguna y da TRUE. El `SELECT 1` no cuenta voluntarios ni devuelve el identificador de la institución: indica que el contenido de la fila encontrada no interesa.

Si se omite `v.id_institucion = i.id_institucion`, la subconsulta deja de preguntar por cada institución y pregunta si hay algún voluntario en toda la tabla. Como lo hay, NOT EXISTS sería falso para **todas** las instituciones, incluso para la 80.

## “Alguno”, “ninguno” y “solo”

«Tiene alguno que cumple P» se expresa con EXISTS sobre P. «No tiene ninguno que cumple P» se expresa con NOT EXISTS sobre P. «Solo tiene voluntarios que cumplen P» exige que no exista un voluntario que incumpla P; si además debe tener voluntarios, se agrega EXISTS. Esa última condición evita aceptar por ausencia de casos a una institución vacía.

Por ejemplo, si una institución tiene una persona con 500 horas y otra con 2000, encontrar a la primera no demuestra «todos aportan menos de 1000». Para demostrarlo hay que buscar a quien alcance o supere 1000 y exigir que no exista. Si hay NULL, decidir si falta de información cuenta como incumplimiento y agregar el caso al predicado.

## Fuente de estudio

Enunciado del TP2 en la carpeta `../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
