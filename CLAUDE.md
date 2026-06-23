# CLAUDE.md

Este archivo se lee automáticamente al iniciar Claude Code en este directorio.
No hace falta que el usuario repita instrucciones en cada sesión.

---

## Contexto del repo

Repo de la presentación del **Algoritmo de Shor** para la materia Computación
Cuántica (ITBA). El contenido ya está aprobado en Markdown
(`shor_presentacion.md`, 41 slides). La tarea pendiente es generar la versión
**Beamer (LaTeX)** de esa presentación.

---

## Qué leer antes de hacer cualquier cosa

Al iniciar una sesión en este repo, leé en este orden:

1. `brief.md` — las 8 restricciones no negociables. Son la autoridad final.
2. `docs/TASK_BEAMER.md` — especificación técnica completa de la tarea
   (tema de Beamer, paquetes, estructura, manejo de notas de orador, flujo de
   compilación incremental).
3. `shor_presentacion.md` — el contenido fuente, slide por slide, ya con
   notas de orador y preguntas típicas.
4. `.claude/skills/quantikz-beamer-patterns.md` — código ya resuelto para los
   diagramas de circuito y los errores típicos de compilación ya conocidos.
   Consultarlo ANTES de escribir un circuito a mano.
5. `docs/FRAME_CHECKLIST.md` — checklist de trazabilidad slide-a-slide y
   restricción-a-restricción. Usarlo para autoverificar el resultado final.

No se necesita releer `shor_presentacion_for_marp.md` ni el `.pdf` de Marp:
son solo referencia visual, no agregan contenido ni autoridad.

---

## Estado del skill externo

El skill `latex-document` ya está instalado en
`~/.claude/skills/latex-document/` (incluye `compile_latex.sh`, `latex_lint.sh`,
`latex_analyze.sh`, `latex_package_check.sh`, y el template
`assets/templates/presentation.tex`). No reinstalar ni correr `setup.sh` de
nuevo salvo que el usuario lo pida explícitamente.

---

## Si el usuario solo dice "generá la presentación" o similar

Interpretarlo como: ejecutar el flujo completo descripto en
`docs/TASK_BEAMER.md` — construir `shor_presentacion.tex` por secciones,
compilando con el skill después de cada sección, y cerrar verificando contra
`docs/FRAME_CHECKLIST.md`. No pedir que se repita el brief: ya está en los
archivos de este repo.

## Si el usuario pide cambios puntuales después de la primera versión

No reconstruir el documento de cero. Editar el frame correspondiente en
`shor_presentacion.tex`, recompilar solo (no relint completo salvo que el
cambio sea estructural), y mantener la numeración/título de los frames
alineada con `docs/FRAME_CHECKLIST.md` para no romper la trazabilidad.
