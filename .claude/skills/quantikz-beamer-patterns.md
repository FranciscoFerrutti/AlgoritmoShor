# quantikz-beamer-patterns.md

Patrones de código ya resueltos para los diagramas de circuito de esta
presentación. Pensado para que Claude Code no tenga que redescubrir, a fuerza
de errores de compilación, problemas ya conocidos al combinar `beamer` +
`quantikz2` + `tikz`.

---

## 1. Frame con quantikz — no hace falta `[fragile]`

A diferencia de `\begin{verbatim}`, los entornos `tikzpicture` y `quantikz`
**no** requieren la opción `[fragile]` en el frame:

```latex
\begin{frame}{Título}
  \[
    \begin{quantikz}
      \lstick{$\ket{0}$} & \gate{H} & \qw
    \end{quantikz}
  \]
\end{frame}
```

---

## 2. Circuito genérico de order-finding (slide 21)

**Objetivo:** t qubits de conteo (mostrar 3 filas explícitas + una fila de
puntos en el medio) + 1 registro de trabajo agrupado (bundled) + caja QFT⁻¹
que abarca visualmente las filas de conteo.

**Truco clave:** `\gate[n]{...}` cuenta FILAS, no contenido. Se puede hacer
que una fila de `\vdots` quede "adentro" del span de la caja QFT⁻¹ sin que
eso rompa nada — la fila de puntos simplemente queda visualmente absorbida
por la caja grande.

```latex
\[
\begin{quantikz}[row sep=0.3cm]
\lstick{$\ket{0}$} & \gate{H} & \ctrl{4} & \qw      & \qw      & \gate[4]{\text{QFT}^{-1}} & \meter{} \\
\lstick{$\ket{0}$} & \gate{H} & \qw      & \ctrl{3} & \qw      &                            & \meter{} \\
\lstick{\vdots}    &          &          &          & \iddots  &                            & \vdots   \\
\lstick{$\ket{0}$} & \gate{H} & \qw      & \qw      & \ctrl{1} &                            & \meter{} \\
\lstick{$\ket{1}^{\otimes n}$} \qwbundle{n} & \qw & \gate{U^{2^0}} & \gate{U^{2^1}} & \gate{U^{2^{t-1}}} & \qw & \qw
\end{quantikz}
\]
```

Notas:
- `\qwbundle{n}` agrupa el registro de trabajo en una sola línea gruesa
  (representa los `n` qubits sin dibujarlos uno por uno).
- `\iddots` (de `mathdots`, ya cargado por `amsmath`/`physics` en cascada) o
  alternativamente `\ddots` funciona para la diagonal de puntos. Si
  `\iddots` no está disponible, usar `\reflectbox{$\ddots$}` o simplemente
  texto vacío en esa celda — no es crítico para la claridad del diagrama.
- Cada columna de control (`\ctrl{N}`) tiene un único control real; el resto
  de las filas en esa columna van vacías o con `\qw` — **no** repetir el
  control en varias filas de la misma columna.
- La fila vacía bajo el "QFT⁻¹" (filas 2 y 4 en esa columna) se deja
  completamente en blanco — sin `\qw` — porque ya están cubiertas por el
  `\gate[4]{}` de la fila 1.

---

## 3. Circuito concreto (slide 28, k=7, N=15)

**Diferencia clave respecto del genérico:** en el ASCII original el bloque de
exponenciación modular se muestra como **una sola caja colapsada** (no
expandida por cada potencia de U), y el registro de conteo también está
bundleado (8 qubits → una sola línea gruesa con `H^{\otimes 8}`). Mantener
ese mismo nivel de abstracción en el `quantikz2`:

```latex
\[
\begin{quantikz}[row sep=0.4cm]
\lstick{$\ket{0}^{\otimes 8}$} \qwbundle{8} & \gate{H^{\otimes 8}} & \ctrl{1} & \gate{\text{QFT}^{-1}} & \meter{} \\
\lstick{$\ket{1}=\ket{0001}$} \qwbundle{4} & \qw & \gate{U:\, \ket{y}\to\ket{7y \bmod 15}} & \qw & \qw
\end{quantikz}
\]
```

Esto reproduce fielmente el nivel de detalle del ASCII original: bloque U
abstracto, sin expandir las potencias `2^j` una por una.

---

## 4. Diagramas de flujo clásico/cuántico (NO son circuitos — usar TikZ puro)

Los slides 9, 31 y 33 son diagramas de flujo de algoritmo (pasos numerados,
ramas de "reintentar"), no circuitos cuánticos. **No** usar `quantikz` para
estos. Patrón con `tikz` puro y dos colores:

```latex
\usetikzlibrary{shapes.geometry, arrows.meta, positioning}

\tikzset{
  classical/.style={rectangle, draw, fill=blue!10, rounded corners,
                     text width=6.5cm, align=left, font=\small},
  quantum/.style={rectangle, draw, fill=orange!15, rounded corners,
                  text width=6.5cm, align=left, font=\small},
  retry/.style={-{Latex[length=2mm]}, dashed, red!70}
}

\begin{tikzpicture}[node distance=0.5cm]
  \node[classical] (filtro) {0. Filtrar N (no primo, no par, no potencia perfecta)};
  \node[classical, below=of filtro] (gcd) {1--2. Elegir $k$, calcular $g=\gcd(k,N)$};
  \node[quantum,   below=of gcd]    (periodo) {3. Hallar período $r$ de $f(x)=k^x \bmod N$ \\ (superposición + QFT)};
  \node[classical, below=of periodo] (check) {4--7. Paridad, $p=k^{r/2}\bmod N$, chequeo de fallo, GCD final};

  \draw[-{Latex[length=2mm]}] (filtro) -- (gcd);
  \draw[-{Latex[length=2mm]}] (gcd) -- (periodo);
  \draw[-{Latex[length=2mm]}] (periodo) -- (check);
  \draw[retry] (check.east) to[bend right=60] node[right, font=\tiny] {reintentar} (gcd.east);
\end{tikzpicture}
```

Esto es más robusto que reproducir el ASCII art con cajas Unicode (`┌─┐│└┘`),
que casi siempre rompe la compilación o se ve mal en LaTeX.

---

## 5. Errores ya conocidos — evitarlos directamente

| Error | Causa | Solución |
|---|---|---|
| `Missing $ inserted` dentro de `quantikz` | `\lstick{...}` o `\meter{...}` con `\ket{}` sin envolver en `$...$` | `\lstick{$\ket{0}$}`, nunca `\lstick{\ket{0}}` a secas |
| Compilación rota en argumento opcional | `[Título con $matemática$]` en un `tcolorbox`/`\begin{frame}[...]` | Evitar `$...$` dentro de argumentos opcionales de corchetes; usar texto plano |
| `quantikz` dentro de celda de `tabular` | Intentar meter un diagrama chico como contenido de tabla | No anidar `quantikz` en `tabular`; ponerlo en su propio `\[...\]` fuera de la tabla |
| Overfull hbox en frame con circuito ancho | Circuito con muchas columnas no entra en el ancho del frame | Envolver en `\resizebox{\textwidth}{!}{...}` alrededor del `quantikz` |
| `\gate[n]` no alinea bien con fila de `\vdots` en medio | Confundir alineación de columnas entre filas | Verificar que TODAS las filas tengan el mismo número de `&` (columnas), aunque algunas queden vacías |

---

## 6. Paleta de colores sugerida (coherente con el resto de la materia)

```latex
\definecolor{clasico}{RGB}{0,70,140}   % azul — pasos clásicos
\definecolor{cuantico}{RGB}{200,100,0} % naranja — el único paso cuántico
```

Usar `clasico` para todo bloque que sea CPU clásica, `cuantico` para el único
bloque que corre en la computadora cuántica (hallar el período). Esto refuerza
visualmente, en cada diagrama de flujo, el mensaje central del brief: "lo
único cuántico es encontrar el período".
