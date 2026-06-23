# FRAME_CHECKLIST.md

Checklist de trazabilidad: cada slide de `shor_presentacion.md` → frame del
`.tex` → restricción no negociable de `brief.md` que cubre (si aplica).
Tildar cada fila al terminar esa parte del `.tex`. Sirve para no descubrir al
final que falta algo — chequear por sección, no recién al cierre.

Restricciones no negociables de `brief.md` (referenciadas por número):
1. Explicación concreta paso a paso del algoritmo (clásico + cuántico)
2. Ejemplo numérico completo N=15 (caso feo k=14, caso feliz k=7)
3. Diagramas de circuito cuántico explícitos
4. Prerrequisitos del input N
5. GCD/Euclides — honestidad sobre su uso real
6. El período de f(x) = k^x mod N como el corazón cuántico
7. Cadena algebraica completa (a)(b)(c)(d) de por qué r permite factorizar
8. QFT — qué es y en qué difiere de la DFT/FFT clásica

---

## Sección 1 — Motivación y RSA

- [ ] Slide 1 — Portada
- [ ] Slide 2 — Hoja de ruta
- [ ] Slide 3 — Motivación: por qué nos importa
- [ ] Slide 4 — Repaso express de RSA
- [ ] Slide 5 — Por qué factorizar N rompe RSA

## Sección 2 — El problema y prerrequisitos

- [ ] Slide 6 — El problema real: factorización ≡ order-finding *(restricción 6)*
- [ ] Slide 7 — Prerrequisitos sobre el input N *(restricción 4)*
- [ ] Slide 8 — El corazón: el período de f(x) = k^x mod N *(restricción 6)*
- [ ] Slide 9 — Esqueleto del algoritmo: clásico vs. cuántico *(restricción 1 — diagrama de flujo TikZ, no quantikz)*
- [ ] Slide 10 — La parte clásica, en detalle *(restricción 1)*
- [ ] Slide 11 — GCD y Euclides: honestidad sobre su uso *(restricción 5)*

## Sección 3 — Por qué el período permite factorizar (cadena algebraica)

- [ ] Slide 12 — Punto de partida: N | (k^r − 1) *(restricción 7, intro)*
- [ ] Slide 13 — (a) Por qué r debe ser par *(restricción 7a)*
- [ ] Slide 14 — (b) Por qué se exige k^(r/2)+1 ≢ 0 *(restricción 7b)*
- [ ] Slide 15 — (c) Por qué gcd(k^(r/2)±1, N) da factores *(restricción 7c)*
- [ ] Slide 16 — (d) "Alta probabilidad": la cota cuantitativa *(restricción 7d —
      **citar desde el paper**, ver nota abajo)*

  > **Atención especial slide 16:** el `.md` fuente ya aclara que el paper de
  > la cátedra NO trae la cota; se cita Nielsen & Chuang. Verificar que el
  > frame final mantenga esa honestidad (no atribuir la cota al paper de la
  > cátedra por error).

## Sección 4 — La parte cuántica: QFT y circuitos

- [ ] Slide 17 — La parte cuántica: hallar el período *(restricción 6)*
- [ ] Slide 18 — La QFT: qué es *(restricción 8)*
- [ ] Slide 19 — QFT vs. DFT/FFT clásica *(restricción 8 — tabla comparativa)*
- [ ] Slide 20 — Estimación de fase: el motor del order-finding
- [ ] Slide 21 — El circuito cuántico completo (diagrama) *(restricción 3 —
      quantikz, ver patrón §2 de `quantikz-beamer-patterns.md`)*
- [ ] Slide 22 — El bloque costoso: exponenciación modular controlada *(U como
      bloque abstracto, según deseo explícito del brief)*
- [ ] Slide 23 — De la medición al período: fracciones continuas

## Sección 5 — Ejemplo resuelto: N = 15

- [ ] Slide 24 — Ejemplo N=15: preparación *(restricción 2, restricción 4 aplicada)*
- [ ] Slide 25 — Caso feo (fallo): k = 14 *(restricción 2 — caso feo completo)*
- [ ] Slide 26 — Caso feliz: k = 7 — parte clásica *(restricción 2 — caso feliz)*
- [ ] Slide 27 — Caso k=7 — parte cuántica: la periodicidad
- [ ] Slide 28 — Caso k=7 — el circuito concreto *(restricción 3 — quantikz,
      ver patrón §3)*
- [ ] Slide 29 — Caso k=7 — mediciones y fracciones continuas (tabla)
- [ ] Slide 30 — Caso k=7 — cierre del ejemplo
- [ ] Slide 31 — El flujo completo, de un vistazo *(diagrama de flujo TikZ)*

## Sección 6 — Complejidad e implicancias

- [ ] Slide 32 — Complejidad: Shor vs. GNFS
- [ ] Slide 33 — Crecimiento comparativo (diagrama TikZ, no a escala)
- [ ] Slide 34 — Implicancias en la criptografía actual
- [ ] Slide 35 — Estado actual del hardware cuántico (2025–2026)
- [ ] Slide 36 — El "gap" honesto: lo más grande factorizado de verdad
- [ ] Slide 37 — Estimaciones de recursos para romper RSA-2048
- [ ] Slide 38 — Criptografía post-cuántica: NIST PQC (tabla con nombres FIPS)
- [ ] Slide 39 — Qué hacer hoy: "harvest now, decrypt later"

## Sección 7 — Cierre

- [ ] Slide 40 — Conclusiones
- [ ] Slide 41 — Referencias

---

## Verificación cruzada final (las 8 restricciones, todas tildadas)

- [ ] 1. Explicación paso a paso clásico+cuántico — cubierta en slides 9, 10, 17
- [ ] 2. Ejemplo N=15 completo (k=14 feo + k=7 feliz) — slides 24–30
- [ ] 3. Diagramas de circuito explícitos — slides 21, 28
- [ ] 4. Prerrequisitos de N — slide 7
- [ ] 5. GCD/Euclides, honestidad de uso — slide 11
- [ ] 6. Período como corazón cuántico — slides 6, 8, 17
- [ ] 7. Cadena algebraica (a)(b)(c)(d) completa y conectada — slides 12–16
- [ ] 8. QFT explicada + diferencia con DFT/FFT — slides 18–19

## Otros chequeos de forma (no son slides, son transversales)

- [ ] Notas de orador presentes en TODOS los frames que las tenían en el `.md`
      (campo `\note{}` de beamer)
- [ ] Preguntas típicas + respuestas incluidas dentro de esas mismas notas
- [ ] Español neutro en todo el documento (sin modismos rioplatenses)
- [ ] Notación `p = k^(r/2) mod N` (residuo) usada consistentemente, sin
      confundirla con los primos `p, q` de RSA — el `.md` fuente insiste mucho
      en esto (slides 10, 24)
- [ ] Nombres finales NIST (ML-KEM, ML-DSA, SLH-DSA) usados junto a los de
      competencia (Kyber, Dilithium, SPHINCS+), no solo estos últimos
