# Brief — Presentación del Algoritmo de Shor

## Contexto

Somos un grupo de 2 alumnos de la materia **Computación Cuántica** de la carrera de
Ingeniería Informática en el **ITBA**. Estamos terminando la cursada y nos pidieron
**dar una clase sobre el algoritmo de Shor**. Necesitamos que nos ayudes a construir
la presentación que vamos a dictar.

En la materia ya trabajamos con compuertas cuánticas de 1 y de varios qubits, con notación
de Dirac, y con circuitos cuánticos. Asumí ese nivel: no hace falta explicar qué es un qubit
o una superposición desde cero, pero sí explicar bien las piezas específicas del algoritmo.

---

## Restricciones (NO NEGOCIABLES — deben estar sí o sí)

Estas son condiciones duras. La presentación no se considera completa si falta alguna.

1. **Explicación concreta y detallada del algoritmo de Shor.** No una descripción
   conceptual de alto nivel: el procedimiento real, paso a paso, con la parte clásica y la
   parte cuántica claramente diferenciadas.

2. **Ejemplos numéricos completamente resueltos en las diapositivas**, con **N = 15**.
   Resueltos paso a paso, con todos los valores intermedios calculados y visibles (no "queda
   como ejercicio"). Tienen que poder leerse y entenderse sin que el orador improvise la
   cuenta. Queremos mostrar **dos casos** con el mismo N = 15:

   - **Primero, un "caso feo" (fallo) con k = 14.** Es coprimo con 15, su período es r = 2
     (par), pero `14^(r/2) mod 15 = 14 = N − 1`. Eso dispara la condición de fallo del paso
     final (`p + 1 = N`), así que el algoritmo **no da factores y hay que reintentar con
     otro k**. Sirve para mostrar que Shor es probabilístico y que existen dos modos de
     fallo: `r` impar, o `k^(r/2) ≡ −1 (mod N)`.
   - **Después, el "caso feliz" con k = 7.** Coprimo, período r = 4 (par),
     `7^(4/2) mod 15 = 4 ≠ 14`, y entonces `GCD(4+1,15)=5` y `GCD(4−1,15)=3`. Factores 3 y 5.

   **Aclaración importante para no inventar dificultad inexistente:** con N = 15 *todos* los
   k coprimos tienen período r ∈ {2, 4}, que son potencias de 2. Por eso el post-procesamiento
   por **fracciones continuas sale exacto** y NO hay un "caso feo" en el sentido de fracciones
   continuas sucias. El caso feo disponible con 15 es el del **fallo por `p = N − 1` (k=14)**,
   no un post-procesamiento complicado. Un ejemplo de fracciones continuas no triviales
   requeriría un N mayor (p. ej. N = 21); no lo fabriques para N = 15.

3. **Diagramas de circuitos cuánticos.** Se esperan diagramas de circuito explícitos para
   la parte cuántica (registros, compuertas, QFT / QFT inversa, mediciones). Indicar qué
   compuertas se usan y por qué.

4. **Prerrequisitos del input N.** Explicar las condiciones que debe cumplir N para que el
   algoritmo aplique:
   - N **no** puede ser primo.
   - N **no** puede ser par (si no, 2 ya es factor).
   - N **no** puede ser de la forma `n^x` (potencia perfecta de un entero).

5. **GCD — método y honestidad sobre su uso.** El método concreto para calcular `GCD(N, k)`
   no es lo central. Si se menciona alguno (p. ej. el **Algoritmo de Euclides**), explicarlo
   muy brevemente y aclarar honestamente: ¿se usa en la práctica?, ¿es solo didáctico o
   realmente se implementa así al programar Shor completo en algo como Qiskit? (Responder con
   precisión: Euclides sí se usa en la práctica para los GCD, es eficiente; lo que NO se hace
   clásicamente es el paso de hallar el período.)

6. **El período de la función `f(x) = k^x mod N` es el corazón del algoritmo.** Explicar
   que **esta es la parte que corre en la computadora cuántica** (los demás pasos son
   clásicos y eficientes). Dejar esto explícito.

7. **Por qué el período `r` permite factorizar — la cadena algebraica completa.** Esta es la
   parte conceptualmente más difícil y la que NO puede quedar como afirmación suelta. Hay que
   explicarla de forma encadenada, partiendo del hecho de que `r` es el período de
   `f(x) = k^x mod N`, lo que significa `k^r ≡ 1 (mod N)`, es decir `N | (k^r − 1)`.
   A partir de ahí, las tres explicaciones siguientes deben estar **todas** presentes y
   conectadas entre sí (no como hechos independientes):

   - **(a) Por qué `r` debe ser par.** Partiendo de `k^r − 1 ≡ 0 (mod N)`, mostrar que si `r`
     es par podemos escribir `k^r − 1 = (k^(r/2) − 1)(k^(r/2) + 1)`, factorización que es la
     que abre la puerta a hallar los divisores. Si `r` es impar esa factorización en enteros
     no aplica (no existe `r/2` entero), y por eso el algoritmo descarta ese intento y
     reintenta con otro `k`. Definir aquí `p = r/2` y la cantidad `k^p mod N`.

   - **(b) Por qué se exige `k^(r/2) + 1 ≢ 0 (mod N)` (en el brief lo expresan como
     `p + 1 ≠ N`, con `p = k^(r/2) mod N`).** Tenemos
     `N | (k^(r/2) − 1)(k^(r/2) + 1)`. Para que esto sea útil, N tiene que **repartir** sus
     factores entre los dos paréntesis (uno con un factor primo de N, el otro con el resto).
     El intento **falla** si N divide enteramente a uno solo de los factores:
     - si `N | (k^(r/2) − 1)`, entonces `k^(r/2) ≡ 1`, lo que contradice que `r` sea el
       período mínimo (sería un período más chico);
     - si `N | (k^(r/2) + 1)`, entonces `k^(r/2) ≡ −1 (mod N)`, es decir `k^(r/2) mod N = N−1`.
     Este segundo es justamente el caso que en las diapositivas se chequea como `p + 1 = N`
     (con `p` el residuo), y es el modo de fallo del **caso feo k=14**. En ambos fallos
     el GCD del paso siguiente daría trivial (1 o N), por eso se reintenta con otro `k`.

   - **(c) Por qué `GCD(k^(r/2) ± 1, N)` da, con alta probabilidad, factores no triviales de N.**
     Una vez garantizado que N **no** divide por completo a ninguno de los dos factores (caso
     (b) superado), N debe compartir parte de sus divisores con `k^(r/2) − 1` y la otra parte
     con `k^(r/2) + 1`. Por eso `GCD(k^(r/2) − 1, N)` y `GCD(k^(r/2) + 1, N)` extraen cada uno
     un divisor propio de N (en N=15: 3 y 5). Conviene explicitar que es el **GCD** —y no la
     resta directa— lo que aísla el factor, y que por eso este paso final del algoritmo es
     clásico y eficiente (Euclides).

   - **(d) "Alta probabilidad" — cota cuantitativa.** Cerrar explicando por qué, sobre un `k`
     elegido al azar (coprimo con N), las dos condiciones —`r` par **y** `k^(r/2) ≢ −1`— se
     cumplen con buena probabilidad, de modo que pocos reintentos bastan. Dar la cota y
     **citarla desde el paper** `resources/An_overview_of_Quantum_Cryptography_and.pdf`
     (la cota habitual para N con al menos dos factores primos impares distintos es una
     probabilidad de éxito ≥ 1/2 por intento, a veces expresada como `1 − 1/2^(m−1)` con `m`
     el número de factores primos distintos; verificar la formulación exacta contra el paper
     y citar de ahí, no de memoria).

8. **Quantum Fourier Transform (QFT).** Explicar que el período se halla mediante la QFT
   (estimación de fase). Dar una explicación de qué es la QFT: cómo funciona y **en qué
   difiere de la transformada de Fourier clásica (DFT/FFT)** — no hace falta una derivación
   completa, pero sí la idea clara de qué hace sobre el estado cuántico y por qué es eficiente.

---

## Deseos (preferencias — modificables según tu criterio)

Temas a cubrir (podés reordenar, fusionar o agregar si mejora la clase):

- Motivación
- Prerrequisitos (ver restricción 4)
- Repaso de **RSA** — clave para entender por qué Shor importa. Enfocar en *por qué*
  factorizar N rompe RSA. Que sea autocontenido pero breve.
- El problema en sí (factorización / orden-finding)
- El algoritmo (parte clásica + parte cuántica)
- Ejemplo concreto con N = 15 (ver restricción 2)
- Complejidad **temporal** (no espacial): Shor vs. mejor algoritmo clásico conocido
  (GNFS), y la comparación cualitativa de crecimiento.
- Implicancias en la criptografía actual
- Estado actual del hardware cuántico
- Criptografía post-cuántica: **NIST PQC**
- Conclusiones

Otros parámetros:

- **Idioma:** español **neutro** (no rioplatense).
- **Exponenciación modular controlada (`U: |y⟩ → |k·y mod N⟩`):** mostrarla como **bloque
  abstracto** en el circuito (no hace falta desarrollar su construcción interna a nivel
  compuertas). Una nota aclarando que es el bloque más costoso es suficiente.
- **Duración objetivo:** entre 20 y 60 minutos. Hacé tantas diapositivas como haga falta.
- **Notas de orador por diapositiva:** para cada slide, agregá notas con los puntos que el
  orador debe mencionar. Idealmente incluí también las preguntas típicas que puede hacer la
  audiencia y cómo responderlas.

---

## Fuentes

- **Fuente de verdad del algoritmo:** usá el paper en
  `resources/An_overview_of_Quantum_Cryptography_and.pdf`. De ahí debe salir el detalle
  técnico del algoritmo de Shor, las cotas de complejidad y la probabilidad de éxito.
- **Material de referencia de estilo / contenido previo:** en `resources/` hay una
  presentación de **Criptografía Clásica**. Tenela en cuenta para no repetir innecesariamente
  lo ya cubierto (especialmente si RSA ya aparece ahí) y para mantener coherencia de estilo.
- **Guía estructural (poco técnica, solo como esqueleto narrativo):**
  https://kaustubhrakhade.medium.com/shors-factoring-algorithm-94a0796a13b1
  Sirve para el orden didáctico de los pasos, **no** como fuente técnica de autoridad.
- Para las secciones que envejecen rápido — **estado del hardware** y **NIST PQC** —
  buscá información actualizada y citá fuente y fecha. En PQC usá los nombres de los
  estándares finales (FIPS 203/204/205 → ML-KEM, ML-DSA, SLH-DSA), no solo los nombres de
  la competencia (Kyber, Dilithium).

---

## Formato de salida

1. Empezá generando la presentación en **Markdown** (un archivo `.md`), que será nuestra
   fuente de verdad con todo el desarrollo matemático, los circuitos y las notas de orador.
2. **No generes todo de una.** Primero proponé el **esqueleto** (lista de diapositivas con
   título y 2–3 bullets cada una) para que lo revisemos. Una vez aprobado, desarrollá el
   contenido completo.
3. Después podemos pasar el resultado a Canva para la versión visual.

---

## Antes de empezar

Si algo del alcance, el nivel de detalle o el formato no te queda claro, **hacénos las
preguntas que necesites antes de generar el esqueleto.** Preferimos resolver dudas ahora a
rehacer todas las diapositivas después.