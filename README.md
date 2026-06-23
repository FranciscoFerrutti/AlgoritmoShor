# El Algoritmo de Shor

Presentación para la materia **Computación Cuántica** — Ingeniería Informática, ITBA.

Tema: factorización de enteros en tiempo polinómico cuántico y sus implicancias
criptográficas.

---

## Contenido del repositorio

| Archivo / directorio | Descripción |
|---|---|
| `shor_presentacion.tex` | Fuente LaTeX (Beamer) — **la fuente de verdad** |
| `shor_presentacion.md` | Fuente Markdown con el desarrollo completo y notas de orador |
| `shor_presentacion_for_marp.md` | Versión para Marp (referencia visual) |
| `brief.md` | Restricciones no negociables y deseos de la presentación |
| `docs/TASK_BEAMER.md` | Especificación técnica de la generación Beamer |
| `docs/FRAME_CHECKLIST.md` | Checklist de trazabilidad slide-a-slide |
| `resources/` | Papers y material de la cátedra usados como fuentes |

---

## Compilar el PDF

Requiere TeX Live (o MiKTeX) con los paquetes `beamer`, `metropolis`,
`quantikz` y `physics`. En Ubuntu/Debian:

```bash
sudo apt install texlive-latex-extra texlive-science latexmk
```

Compilar:

```bash
latexmk -pdf shor_presentacion.tex
```

O manualmente (dos pasadas para que queden bien las referencias internas):

```bash
pdflatex shor_presentacion.tex
pdflatex shor_presentacion.tex
```

El PDF resultante (`shor_presentacion.pdf`) está en `.gitignore` — se genera
localmente desde el `.tex`.

> **Nota (usuarios de Claude Code):** este repo usa el skill
> [`latex-document`](https://github.com/ndpvt-web/latex-document-skill).
> Con el skill instalado, Claude puede compilar, hacer lint y previsualizar
> el `.tex` automáticamente dentro de una sesión.
>
> Instalación:
> ```bash
> git clone https://github.com/ndpvt-web/latex-document-skill
> cp -r latex-document-skill ~/.claude/skills/latex-document
> bash ~/.claude/skills/latex-document/setup.sh   # instala TeX Live y dependencias
> ```

---

## Estructura de la presentación (41 slides)

1. **Motivación y RSA** — por qué factorizar importa, repaso de RSA
2. **El problema** — order-finding, prerrequisitos de N, Euclides
3. **Por qué el período permite factorizar** — cadena algebraica completa (a–d)
4. **QFT y circuitos** — QFT vs. FFT, estimación de fase, circuito cuántico
5. **Ejemplo N = 15** — caso feo k = 14 (fallo) y caso feliz k = 7 (éxito)
6. **Complejidad e implicancias** — Shor vs. GNFS, hardware actual, NIST PQC
7. **Conclusiones y referencias**

---

## Fuentes principales

- M. Nielsen, I. Chuang, *Quantum Computation and Quantum Information* —
  Teorema A4.13 (cota de probabilidad), cap. 5 (order-finding, QFT).
- P. Shor, "Algorithms for Quantum Computation", FOCS 1994.
- C. Gidney, arXiv:2505.15917 (2025) — estimación de recursos para RSA-2048.
- NIST FIPS 203/204/205 (ML-KEM, ML-DSA, SLH-DSA), finalizados agosto 2024.
