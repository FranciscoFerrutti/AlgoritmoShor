# TASK_BEAMER.md

Instrucciones para Claude Code: generar la presentación del Algoritmo de Shor
como un documento **Beamer (LaTeX)** usando el skill `latex-document` ya
instalado en `~/.claude/skills/latex-document/`.

---

## Jerarquía de fuentes (en caso de conflicto)

1. **`brief.md`** — las 8 restricciones NO NEGOCIABLES son la autoridad final
   sobre qué debe estar y cómo. Releerlo antes de escribir cualquier frame.
2. **`shor_presentacion.md`** — fuente de contenido y notas de orador. Ya está
   aprobado y completo (41 slides). Es la base para los `\frame{}`.
3. **`shor_presentacion_for_marp.md` / `.pdf`** — solo referencia de estilo
   visual (cómo se ve en Marp), **no** agrega contenido nuevo ni autoridad.
4. **`docs/FRAME_CHECKLIST.md`** (en este mismo paquete) — checklist de
   trazabilidad slide-a-slide. Usarlo para verificar que no falte nada al final.
5. **`.claude/skills/quantikz-beamer-patterns.md`** (en este mismo paquete) —
   patrones de código ya resueltos para los diagramas de circuito. Consultarlo
   ANTES de escribir los circuitos a mano, para no perder tiempo redescubriendo
   errores de compilación ya conocidos.

---

## Deliverable

- `shor_presentacion.tex` — Beamer, basado en
  `~/.claude/skills/latex-document/assets/templates/presentation.tex`
- `shor_presentacion.pdf` — compilado con el skill (`compile_latex.sh`)
- (Opcional) `shor_presentacion_notes.pdf` — versión con notas de orador
  visibles (ver sección "Notas de orador" abajo)

---

## Especificación técnica

### Documentclass y tema

```latex
\documentclass[aspectratio=169]{beamer}
\usetheme{metropolis}   % instalado localmente; alternativa: Madrid
```

`metropolis` está confirmado instalado en el sistema
(`beamerthememetropolis.sty` en texlive). Si por algún motivo no compila,
usar `\usetheme{Madrid}` como fallback — ambos están disponibles.

### Paquetes

```latex
\usepackage[utf8]{inputenc}
\usepackage[T1]{fontenc}
\usepackage{amsmath, amssymb}
\usepackage{physics}        % \ket, \bra, \braket
\usepackage{tikz}
\usetikzlibrary{quantikz2}  % diagramas de circuito
\usepackage{booktabs}       % tablas (NIST PQC, fracciones continuas)
\usepackage{xcolor}
\usepackage{hyperref}
```

### Idioma

Español **neutro** (no rioplatense) — coherente con `brief.md`. Revisar que
ningún frame use modismos regionales.

### Estructura en `\section{}`

Usar `\section{}` para que metropolis genere los separadores de sección.
Secciones sugeridas (agrupando los 41 slides del `.md`):

1. Motivación y RSA (slides 1–5)
2. El problema y prerrequisitos (slides 6–11)
3. Por qué el período permite factorizar (slides 12–16)
4. La parte cuántica: QFT y circuitos (slides 17–23)
5. Ejemplo resuelto: N = 15 (slides 24–31)
6. Complejidad e implicancias (slides 32–39)
7. Conclusiones y referencias (slides 40–41)

### Diagramas de circuito (quantikz)

Hay **dos** diagramas de circuito explícitos a typesettear con `quantikz2`:
- Slide 21 — circuito genérico de order-finding (t qubits de conteo + registro
  de trabajo + bloque U + QFT⁻¹ + medición).
- Slide 28 — el mismo circuito instanciado con k=7, N=15, t=8, n=4.

**No** reproducir el ASCII art tal cual: redibujarlo como diagrama `quantikz2`
real. Ver `.claude/skills/quantikz-beamer-patterns.md` para los patrones ya
resueltos (incluye cómo lograr el spanning de la caja QFT⁻¹ sobre varias filas
con una fila de puntos `\vdots` en el medio).

### Diagramas de flujo (clásico vs. cuántico)

Los diagramas de caja ASCII de los slides 9, 31 y 33 (pipeline clásico/cuántico,
flujo completo, comparación de crecimiento) **no** son circuitos cuánticos:
son diagramas de flujo. Redibujarlos con `tikz` puro (nodos rectangulares +
flechas), no con quantikz. Usar dos colores para distinguir bloques
**CLÁSICO** vs. **CUÁNTICO** (ver paleta sugerida en el patterns file).

### Tablas

Las tablas (QFT vs. DFT/FFT, fracciones continuas del ejemplo, NIST PQC) van
con `tabular` + `booktabs` (`\toprule`, `\midrule`, `\bottomrule`), igual que
en el resto del material de la materia.

### Notas de orador

Cada frame del `.md` tiene una sección **"Notas de orador"** y, en muchos
casos, **"Preguntas típicas y respuestas"**. Pasar ambas al campo `\note{}`
de beamer:

```latex
\begin{frame}{Título del slide}
  ... contenido ...
  \note{
    Puntos a mencionar: ...
    \\[4pt]
    \textbf{P:} pregunta típica \\
    \textbf{R:} respuesta
  }
\end{frame}
```

Por defecto las notas no se ven en el PDF normal. Para generar la versión con
notas visibles (para que el orador la lea), agregar — en un documento
**separado** que incluya el mismo `.tex` — el modo de notas de `pgfpages`:

```latex
\usepackage{pgfpages}
\setbeameroption{show notes on second screen=right}
```

No mezclar esto en el `.tex` principal: el PDF para la audiencia debe quedar
limpio, sin notas visibles.

### Bloque de exponenciación modular controlada (U)

Por restricción del brief: mostrar `U` como **bloque abstracto** en el
circuito (un solo `\gate{U^{2^j}}`), sin desarrollar su construcción interna.
Una nota al pie aclarando que es el bloque más costoso es suficiente.

---

## Flujo de trabajo recomendado (compilación incremental)

Con 41 frames, no escribir todo el `.tex` de una sola pasada. El propio skill
recomienda esta escala (ver `SKILL.md`): documentos de 11–20 "páginas" se
dividen, 21+ usan pipeline por lotes. Tratamiento sugerido aquí, por sección:

```bash
# 1. Armar el preámbulo + sección 1 (Motivación y RSA) primero
# 2. Compilar y corregir errores ANTES de seguir
bash ~/.claude/skills/latex-document/scripts/compile_latex.sh \
  shor_presentacion.tex --use-latexmk --preview --preview-dir ./outputs

# 3. Agregar la siguiente sección, recompilar, corregir
# ... repetir por las 7 secciones ...

# 4. Al final, lint + análisis completo
bash ~/.claude/skills/latex-document/scripts/latex_lint.sh shor_presentacion.tex
bash ~/.claude/skills/latex-document/scripts/latex_analyze.sh shor_presentacion.tex
bash ~/.claude/skills/latex-document/scripts/latex_package_check.sh shor_presentacion.tex
```

Este enfoque evita acumular 41 frames con un error de sintaxis temprano
escondido en el medio (los diagramas `quantikz2` son los puntos más frágiles).

---

## Verificación final (antes de declarar terminado)

1. Recorrer `docs/FRAME_CHECKLIST.md` y tildar cada slide.
2. Confirmar que las **8 restricciones no negociables** de `brief.md` están
   cubiertas (la checklist las referencia por número).
3. Confirmar que el PDF compiló sin errores fatales y sin warnings de
   `Overfull \hbox` graves (indican texto que se corta en el slide).
4. Confirmar que ningún frame quedó con texto en rioplatense.
5. Citar la cota de probabilidad (restricción 7d) **desde el paper**
   `resources/An_overview_of_Quantum_Cryptography_and.pdf` — releer la slide 16
   del `.md`, que ya documenta la cita exacta a usar (Nielsen & Chuang, no el
   paper de la cátedra).
