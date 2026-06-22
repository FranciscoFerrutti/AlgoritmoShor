# El algoritmo de Shor
### Factorización de enteros en tiempo polinómico cuántico

**Computación Cuántica — Ingeniería Informática — ITBA**

> Documento fuente en Markdown. Contiene el desarrollo matemático completo, los
> diagramas de circuito y las notas de orador por diapositiva. La versión visual
> (Canva) se deriva de este archivo.
>
> Idioma: español neutro. Nivel asumido: compuertas de 1 y varios qubits, notación
> de Dirac, circuitos cuánticos. No se reexplica qué es un qubit o una superposición.

---

## Slide 1 — Portada

# El algoritmo de Shor
### Cómo una computadora cuántica factoriza enteros y por qué eso rompe RSA

Materia: Computación Cuántica · ITBA
Expositores: [Nombre 1] · [Nombre 2]

**Notas de orador:**
- Presentarse y enmarcar: "Hoy vamos a ver el algoritmo que convirtió a la computación cuántica en un problema de seguridad nacional."
- Adelantar la promesa de la charla: vamos a entender el procedimiento real paso a paso, resolver a mano un ejemplo con N = 15, y ver qué tan lejos (o cerca) está la amenaza real.
- Aclarar el contrato con la audiencia: no vamos a repetir qué es un qubit; sí vamos a desarmar cada pieza específica de Shor.

**Preguntas típicas y respuestas:**
- *"¿Esto ya rompe RSA hoy?"* → No. Hoy el hardware no alcanza; al final mostramos el estado real y las estimaciones. Pero la amenaza es lo bastante creíble como para que ya se estén estandarizando reemplazos.

---

## Slide 2 — Hoja de ruta

**De qué vamos a hablar**

1. Motivación y por qué factorizar rompe RSA
2. El problema: factorización como *order-finding* (hallar el período)
3. Prerrequisitos sobre el número N
4. El algoritmo: parte clásica + parte cuántica
5. Por qué el período permite factorizar (la cadena algebraica completa)
6. La QFT y el circuito cuántico
7. Ejemplo resuelto con N = 15: un fallo (k = 14) y un éxito (k = 7)
8. Complejidad: Shor vs. el mejor clásico (GNFS)
9. Implicancias, estado del hardware y criptografía post-cuántica

**Notas de orador:**
- Señalar las dos columnas mentales que conviene mantener toda la charla: **qué corre en una computadora clásica** y **qué corre en la cuántica**. Es el hilo conductor.
- Avisar que el corazón conceptual está en los puntos 5 y 6, y el corazón práctico en el 7.

---

## Slide 3 — Motivación: por qué nos importa

- Gran parte de la seguridad de Internet (TLS, firmas, certificados) se apoya en que **factorizar un entero grande es difícil** para una computadora clásica.
- "Difícil" significa: el mejor algoritmo clásico conocido (GNFS) tarda **tiempo sub-exponencial** — para una clave RSA de 2048 bits, fuera del alcance práctico.
- En 1994 Peter Shor mostró un algoritmo **cuántico** que factoriza en **tiempo polinómico**. Eso convierte un problema "imposible" en uno "tratable".
- Consecuencia directa: si existiera una computadora cuántica suficientemente grande y confiable, **RSA y los esquemas basados en logaritmo discreto (DH, DSA, ECDSA) dejarían de ser seguros.**

**Notas de orador:**
- Enfatizar el salto cualitativo: no es "más rápido", es **otra clase de complejidad** (sub-exponencial → polinómica).
- Conectar con la materia previa de Criptografía: la seguridad de RSA es una *suposición computacional*, no una garantía matemática absoluta. Shor ataca exactamente esa suposición.
- Mencionar que el paper de referencia (Ugwuishiwu et al., 2020) ubica históricamente a Shor (Bell Labs, 1994) y a Feynman (1981) como punto de partida de la idea de computación cuántica.

**Preguntas típicas y respuestas:**
- *"¿No alcanza con usar claves más largas?"* → No salva a RSA: como Shor es polinómico, duplicar el tamaño de la clave solo aumenta el costo polinómicamente para el atacante cuántico. Es una carrera que el defensor clásico pierde.

---

## Slide 4 — Repaso express de RSA

> RSA ya se vio en la cursada de Criptografía (Clase 04). Acá solo lo necesario para
> entender por qué **factorizar N** lo rompe.

**Generación de claves (textbook RSA):**
- Se eligen dos primos `p, q` y se define el módulo `N = p · q`.
- Se calcula `φ(N) = (p − 1)(q − 1)`.
- Se elige `e` con `gcd(e, φ(N)) = 1`, y se calcula `d` tal que `e · d ≡ 1 (mod φ(N))`.
- Clave pública: `(N, e)` · Clave privada: `(N, d)`.

**Operaciones:**
- Cifrado: `c ≡ m^e (mod N)`
- Descifrado: `m ≡ c^d (mod N)`

Lo público es `(N, e)`. El secreto efectivo es `d`, y `d` se deriva de `φ(N)`.

**Notas de orador:**
- Recordar que esto es exactamente lo de la Clase 04; no detenerse en seguridad CPA/CCA ni padding (PKCS#1). Solo importa la estructura aritmética.
- Marcar la pieza clave para la próxima slide: **`d` se obtiene a partir de `φ(N)`, y `φ(N)` se obtiene de `p` y `q`.**

**Preguntas típicas y respuestas:**
- *"¿Y los esquemas con curvas elípticas?"* → ECDSA/ECDH se basan en logaritmo discreto, no en factorización, pero Shor también resuelve el logaritmo discreto (con una variante). O sea, también caen.

---

## Slide 5 — Por qué factorizar N rompe RSA

La cadena es directa:

```
conocer p, q   ⟶   φ(N) = (p−1)(q−1)   ⟶   d ≡ e⁻¹ (mod φ(N))   ⟶   clave privada
```

- El atacante conoce `N` y `e` (son públicos).
- Si logra **factorizar `N = p · q`**, calcula `φ(N)` trivialmente.
- Con `φ(N)` y `e`, obtiene `d` con el **algoritmo de Euclides extendido** (eficiente).
- Con `d` descifra cualquier mensaje y firma como si fuera el dueño de la clave.

**La seguridad de RSA se apoya por completo en que nadie pueda factorizar `N`.**
Shor ataca ese único punto.

**Notas de orador:**
- Recalcar: no hace falta "romper la matemática de RSA". Basta con factorizar. Toda la fortaleza está concentrada en ese cuello de botella.
- Conectar: el resto de la charla es, en el fondo, "cómo Shor factoriza N".

**Preguntas típicas y respuestas:**
- *"¿No se puede obtener φ(N) sin factorizar?"* → Calcular φ(N) para N = p·q es computacionalmente equivalente a factorizar; no se conoce atajo. Por eso factorizar es *el* problema.

---

## Slide 6 — El problema real: factorización ≡ order-finding

Shor **no** factoriza por fuerza bruta. Reduce la factorización a otro problema:
**hallar el período (orden) de una función.**

Dado `N` a factorizar y un `k` coprimo con `N`, se define:

```
f(x) = k^x mod N
```

Esta función es **periódica**: existe un mínimo `r > 0` tal que

```
f(x + r) = f(x)   para todo x      ⟺      k^r ≡ 1 (mod N)
```

A ese `r` se lo llama el **orden de k módulo N** (o período de `f`).

> **Idea central:** conocer `r` permite, con álgebra elemental + GCD, recuperar
> factores de `N`. Y hallar `r` es lo único que necesita la computadora cuántica.

**Notas de orador:**
- Insistir: el "milagro" cuántico no es factorizar directamente, es **encontrar el período**. Todo lo demás es clásico y barato.
- Dar intuición de por qué `f` es periódica: las potencias de `k` módulo N viven en un conjunto finito (el grupo multiplicativo Z*_N), así que tarde o temprano se repiten, y al repetirse lo hacen cíclicamente.

**Preguntas típicas y respuestas:**
- *"¿Por qué k debe ser coprimo con N?"* → Si `gcd(k, N) ≠ 1`, ese GCD ya **es** un factor de N (terminamos sin necesidad de la parte cuántica). El caso interesante es `gcd(k, N) = 1`, donde `f` es periódica en el grupo multiplicativo.

---

## Slide 7 — Prerrequisitos sobre el input N

Antes de correr Shor, se descartan clásicamente los casos triviales. **N debe cumplir:**

1. **N no es primo.** Si fuera primo, no hay nada que factorizar (test de primalidad clásico y eficiente, p. ej. Miller-Rabin).
2. **N no es par.** Si lo fuera, `2` ya es un factor. Se chequea mirando el último bit.
3. **N no es una potencia perfecta `n^x`** (con `n, x` enteros, `x ≥ 2`). Si lo fuera, la raíz `n` es un factor, y se halla con métodos clásicos eficientes (probar raíces x-ésimas para `x` hasta `log₂ N`).

Si N pasa estos filtros, es un **compuesto impar con al menos dos factores primos distintos** — el caso para el que Shor está diseñado.

**Notas de orador:**
- Explicar el porqué de cada descarte: garantizan que existe una factorización no trivial y que el método del orden va a funcionar (necesitamos que N tenga ≥ 2 primos distintos; eso reaparece en la cota de probabilidad).
- Subrayar que estos chequeos son **clásicos y rápidos**: no "gastamos" la computadora cuántica en ellos.

**Preguntas típicas y respuestas:**
- *"¿Por qué excluir potencias perfectas si igual son compuestas?"* → Porque si `N = pᵏ` tiene un solo primo distinto, el argumento de probabilidad de Shor se debilita (la cota `1 − 1/2^{m−1}` necesita m ≥ 2 primos distintos), y además la raíz se halla trivialmente en clásico. Es más barato sacarlas antes.

---

## Slide 8 — El corazón: el período de f(x) = k^x mod N

```
f(x) = k^x mod N
```

- Es **periódica** con período `r` = orden de `k` módulo `N`.
- Hallar `r` es **el único paso que requiere la computadora cuántica.**
- ¿Por qué no clásico? Clásicamente habría que evaluar `f` en muchísimos puntos para detectar la repetición: para N de miles de bits, el período puede ser astronómicamente grande. **No se conoce un método clásico eficiente para hallar `r`.**
- La computadora cuántica evalúa `f` en **superposición sobre todos los x a la vez** y luego usa interferencia (la QFT) para extraer la periodicidad.

> Todos los demás pasos del algoritmo (GCD, chequeos, derivar factores) son
> **clásicos y polinómicos**. La ventaja cuántica vive enteramente acá.

**Notas de orador:**
- Este es el mensaje que debe quedar grabado: **"lo único cuántico es hallar el período".**
- Adelantar el mecanismo sin entrar todavía: superposición para evaluar todo a la vez + QFT para que la periodicidad se vuelva medible. El detalle viene en las slides de QFT.

**Preguntas típicas y respuestas:**
- *"Si la cuántica evalúa todo en paralelo, ¿por qué no leo directamente la respuesta?"* → Porque al medir colapsa a un solo valor al azar. El truco no es el paralelismo bruto, sino usar interferencia para que lo que se mide codifique el período. Lo vemos en la parte de QFT.

---

## Slide 9 — Esqueleto del algoritmo: clásico vs. cuántico

```
┌─────────────────────────── CLÁSICO ───────────────────────────┐
│ 0. Filtrar N (no primo, no par, no potencia perfecta)          │
│ 1. Elegir k al azar, 1 < k < N                                 │
│ 2. g = gcd(k, N)   (Euclides)                                  │
│      └─ si g ≠ 1  → ¡suerte! g es factor. FIN.                 │
│      └─ si g = 1  → seguir                                     │
└────────────────────────────────────────────────────────────────┘
                              │
                ┌─────────── CUÁNTICO ───────────┐
                │ 3. Hallar el período r de       │
                │    f(x) = k^x mod N             │
                │    (superposición + QFT)        │
                └─────────────────────────────────┘
                              │
┌─────────────────────────── CLÁSICO ───────────────────────────┐
│ 4. Si r es IMPAR  → descartar, volver a 1 (otro k)            │
│ 5. Calcular p = k^(r/2) mod N                                  │
│ 6. Si p + 1 ≡ 0 (mod N), es decir p = N−1                     │
│       → descartar, volver a 1 (otro k)                         │
│ 7. Factores: gcd(p − 1, N) y gcd(p + 1, N)   (Euclides)       │
└────────────────────────────────────────────────────────────────┘
```

**Notas de orador:**
- Caminar el diagrama de arriba hacia abajo. Señalar visualmente que **un solo bloque es cuántico** (paso 3).
- Marcar los dos puntos de reintento (pasos 4 y 6): de ahí sale que Shor es **probabilístico**.
- Aclarar que el paso 2 es un "regalo": a veces el k random ya comparte factor con N y terminamos sin cuántica (raro, pero gratis chequearlo).

**Preguntas típicas y respuestas:**
- *"¿Cuántas veces hay que reintentar?"* → Pocas en promedio: la probabilidad de éxito por intento es ≥ 1/2 (lo justificamos en la slide de la cota). Unos pocos k bastan.

---

## Slide 10 — La parte clásica, en detalle

1. **Elegir** `k` uniformemente al azar en `1 < k < N`.
2. **GCD inicial:** `g = gcd(k, N)`.
   - Si `g ≠ 1`: `g` es un factor no trivial → listo (caso afortunado, sin cuántica).
   - Si `g = 1`: `k` es coprimo con N → tiene sentido buscar su orden.
3. *(cuántico: obtener `r`)*
4. **Paridad:** si `r` es impar → no sirve, elegir otro `k`.
5. **Calcular** `p = k^(r/2) mod N`.
6. **Chequeo de fallo:** si `p = N − 1` (equivale a `k^(r/2) ≡ −1`) → no sirve, otro `k`.
7. **Extraer factores:** `gcd(p − 1, N)` y `gcd(p + 1, N)` dan factores no triviales.

Todos estos pasos son **clásicos y eficientes** (exponenciación modular rápida + Euclides).

**Notas de orador:**
- Remarcar que la exponenciación modular `k^(r/2) mod N` del paso 5 se hace por exponenciación binaria (square-and-multiply): rapidísima en clásico.
- Aclarar la convención de notación que usamos: en el brief, `p` denota el **residuo** `k^(r/2) mod N` (no un primo). Cuidado para no confundir con los primos `p, q` de RSA. Lo vamos a decir explícito en la slide del ejemplo.

**Preguntas típicas y respuestas:**
- *"¿El GCD final puede dar 1 o N (trivial)?"* → Sí, y por eso existen los chequeos de los pasos 4 y 6: están justamente para descartar de antemano los k que llevarían a GCD trivial. Lo justificamos algebraicamente enseguida.

---

## Slide 11 — GCD y Euclides: honestidad sobre su uso

**¿Cómo se calcula `gcd(a, N)`?** Con el **algoritmo de Euclides**:

```
gcd(a, b):
    while b ≠ 0:
        a, b = b, a mod b
    return a
```

Idea: `gcd(a, b) = gcd(b, a mod b)`, reduciendo el problema hasta que un término sea 0.

**¿Se usa de verdad o es solo didáctico?**
- **Sí se usa en la práctica.** Euclides es eficiente (tiempo polinómico, `O((log N)²)` o mejor) y es exactamente lo que se ejecuta para los GCD al implementar Shor en, por ejemplo, Qiskit.
- Lo que **no** se hace clásicamente es el **paso de hallar el período `r`**: ese es el único que no tiene método clásico eficiente conocido y por eso va a la computadora cuántica.

> Moraleja: los GCD y el resto del post-procesamiento son baratos. La dificultad
> está concentrada, otra vez, en hallar el período.

**Notas de orador:**
- Responder de frente a la duda del brief: Euclides no es "de juguete", es producción. La frontera clásico/cuántico **no** pasa por el GCD, pasa por el período.
- Si hay tiempo, mencionar que Euclides extendido también es lo que usa RSA para calcular `d` — o sea, la misma herramienta aparece a ambos lados.

**Preguntas típicas y respuestas:**
- *"¿Cuán rápido es Euclides?"* → Polinómico en la cantidad de dígitos; el número de pasos está acotado por algo proporcional a log del menor de los dos números (relacionado con Fibonacci). Despreciable frente a todo lo demás.

---

## Slide 12 — Por qué el período permite factorizar (1/5): el punto de partida

Partimos del hecho que define a `r`:

```
r es el período de f(x) = k^x mod N   ⟺   k^r ≡ 1 (mod N)
```

Reescribiendo:

```
k^r ≡ 1 (mod N)   ⟺   k^r − 1 ≡ 0 (mod N)   ⟺   N | (k^r − 1)
```

Es decir: **N divide a `k^r − 1`.**

A partir de acá vamos a mostrar, encadenadas, cuatro cosas:
- **(a)** por qué `r` debe ser **par**,
- **(b)** por qué exigimos `k^(r/2) + 1 ≢ 0 (mod N)`,
- **(c)** por qué `gcd(k^(r/2) ± 1, N)` da factores,
- **(d)** por qué todo esto funciona con **alta probabilidad**.

**Notas de orador:**
- Dejar claro que esto **no** es una colección de hechos sueltos: es una sola deducción que arranca en `N | (k^r − 1)`.
- Pedir a la audiencia que retenga esa única ecuación; todo lo que sigue se desprende de ella.

**Preguntas típicas y respuestas:**
- *"¿Por qué `r` es el período *mínimo* y no cualquier exponente con k^x ≡ 1?"* → Por definición de orden: `r` es el menor positivo con `k^r ≡ 1`. Esa minimalidad la vamos a usar en (b).

---

## Slide 13 — (a) Por qué r debe ser par

Tenemos `N | (k^r − 1)`. Si `r` es **par**, podemos definir `p = r/2` entero y factorizar como diferencia de cuadrados:

```
k^r − 1 = (k^(r/2))² − 1 = (k^(r/2) − 1)(k^(r/2) + 1)
```

Esta factorización es **la que abre la puerta**: N divide al producto de dos números más chicos y manejables, `k^(r/2) − 1` y `k^(r/2) + 1`.

Si `r` es **impar**:
- `r/2` no es entero, **no existe** esa factorización en enteros,
- no hay forma de partir `k^r − 1` en dos factores tratables,
- **se descarta este `k` y se reintenta** con otro.

> Conclusión: necesitamos `r` par para tener los dos "ladrillos" `k^(r/2) ∓ 1`.

**Notas de orador:**
- Hacer visible el truco: es la identidad de secundaria `a² − b² = (a−b)(a+b)` aplicada con `a = k^(r/2)`, `b = 1`.
- Definir acá la notación que se usará en el ejemplo: `p = k^(r/2) mod N` (el residuo). Avisar de nuevo que este `p` no es un primo.

**Preguntas típicas y respuestas:**
- *"¿Qué tan seguido `r` sale impar?"* → Depende de N, pero combinado con la otra condición la probabilidad de éxito por intento es ≥ 1/2. Para N = 15 todos los k coprimos tienen r ∈ {2, 4}, así que ningún k falla por imparidad (lo vemos en el ejemplo).

---

## Slide 14 — (b) Por qué se exige k^(r/2) + 1 ≢ 0 (mod N)

De (a): `N | (k^(r/2) − 1)(k^(r/2) + 1)`.

Para que esto sea **útil**, N tiene que **repartir** sus factores primos entre los dos paréntesis (un primo de N en uno, el resto en el otro). El intento **falla** si N divide **enteramente a uno solo** de los factores:

- **Si `N | (k^(r/2) − 1)`** → `k^(r/2) ≡ 1 (mod N)`. Pero eso significaría que `r/2` ya es un período, **contradiciendo que `r` es el período mínimo**. Así que este caso no puede ocurrir si `r` es el orden real.

- **Si `N | (k^(r/2) + 1)`** → `k^(r/2) ≡ −1 (mod N)`, es decir `k^(r/2) mod N = N − 1`.
  Este sí puede ocurrir, y es el que se chequea en las diapositivas como **`p + 1 = N`** (con `p` el residuo). **Es el modo de fallo del "caso feo" k = 14.**

En ambos fallos, el GCD del paso siguiente daría algo **trivial** (1 o N). Por eso, si `p = N − 1`, se descarta `k` y se reintenta.

**Notas de orador:**
- Explicar la intuición de "repartir": queremos que cada paréntesis se lleve *parte* de los primos de N. Si uno se lleva *todos*, el otro no se lleva ninguno y el GCD no aísla nada nuevo.
- El primer subcaso (`≡ 1`) se elimina solo por minimalidad de `r`; el segundo (`≡ −1`) es el genuino modo de fallo que hay que chequear explícitamente.
- Anclar con el ejemplo que viene: k = 14 da r = 2, `14^1 mod 15 = 14 = N − 1` → falla.

**Preguntas típicas y respuestas:**
- *"¿Por qué `≡ −1` rompe el método?"* → Porque entonces `k^(r/2) + 1 ≡ 0`, N divide por completo a ese factor, y `gcd(k^(r/2) − 1, N)` se vuelve trivial. No se extrae ningún primo nuevo.

---

## Slide 15 — (c) Por qué gcd(k^(r/2) ± 1, N) da factores no triviales

Superado el chequeo (b), sabemos que **N no divide por completo a ninguno** de los dos factores `k^(r/2) − 1` ni `k^(r/2) + 1`. Pero sí divide al **producto**.

Como `N = p₁·p₂·…` (sus primos), y ningún paréntesis se lleva todos los primos:
- algún primo de N divide a `k^(r/2) − 1`,
- y otro primo de N divide a `k^(r/2) + 1`.

Por lo tanto:

```
gcd(k^(r/2) − 1, N)   y   gcd(k^(r/2) + 1, N)
```

**comparten cada uno parte de los primos de N**, y devuelven **divisores propios** (ni 1 ni N).

Para N = 15 esto dará exactamente **3 y 5**.

> Es el **GCD** —no la resta directa— lo que aísla el factor. Y como Euclides es
> clásico y eficiente, **este último paso es clásico y barato.**

**Notas de orador:**
- Subrayar por qué GCD y no resta: `k^(r/2) − 1` puede ser un número enorme y no necesariamente un factor de N; el GCD con N extrae la *parte común*, que sí es un factor.
- Cerrar la cadena: (a) nos dio la factorización, (b) garantizó que es útil, (c) extrae los factores con GCD.

**Preguntas típicas y respuestas:**
- *"¿Siempre los dos GCD dan factores, o a veces uno solo?"* → Tras pasar (b), típicamente ambos dan factores complementarios. Basta con uno para terminar; si N = p·q, un GCD da p y N/p da q.

---

## Slide 16 — (d) "Alta probabilidad": la cota cuantitativa

¿Con qué probabilidad un `k` random (coprimo con N) cumple **ambas** condiciones —`r` par **y** `k^(r/2) ≢ −1`?

**Teorema (Nielsen & Chuang, *Quantum Computation and Quantum Information*, Teorema A4.13 / §5.3.1):**

> Sea `N` un compuesto impar con `m` factores primos distintos. Si se elige `k`
> uniformemente al azar entre los coprimos con N, entonces
>
> ```
> Pr[ r par  y  k^(r/2) ≢ −1 (mod N) ]  ≥  1 − 1 / 2^(m−1)
> ```

- Como N (tras los filtros) tiene `m ≥ 2` primos distintos: **probabilidad de éxito ≥ 1 − 1/2 = 1/2** por intento.
- Si N tiene más primos distintos, la cota mejora (más cerca de 1).
- Con éxito ≥ 1/2 por intento, **el número esperado de reintentos es ≤ 2.** Pocos `k` bastan.

> Nota de fuentes: el paper de la cátedra (Ugwuishiwu et al., 2020) describe el
> procedimiento pero **no da esta cota cuantitativa**; se cita por eso la fuente
> estándar (Nielsen & Chuang). Shor (1994) prueba un resultado equivalente.

**Notas de orador:**
- Explicar de dónde sale el `2^(m−1)`: el "−1" de los modos de fallo se reparte por el teorema chino del resto entre los m primos; la fracción de k "malos" está acotada por `1/2^{m−1}`.
- Ser honestos con la audiencia y con la cátedra: el paper asignado es divulgativo y no incluye la cota; por eso citamos Nielsen & Chuang. (El paper incluso tiene una errata: escribe el producto como `(m^{P/2}−1)(m^{P/2}−1)` en lugar de `…+1`.)

**Preguntas típicas y respuestas:**
- *"¿Y si tengo mala suerte muchas veces seguidas?"* → La probabilidad de fallar t veces seguidas es ≤ 1/2^t; cae exponencialmente. En la práctica, 1 o 2 intentos.

---

## Slide 17 — La parte cuántica: hallar el período

Objetivo del subprograma cuántico: dado `k` y `N`, hallar `r` tal que `k^r ≡ 1 (mod N)`.

Se usan **dos registros**:
- **Registro de conteo** (counting / "input"): `t` qubits, inicializado en `|0⟩`.
  Va a contener la información de fase de la que se extrae `r`.
  Se elige `t` con `2^t ≥ N²` (margen para que las fracciones continuas separen bien).
- **Registro de trabajo** (work): `n = ⌈log₂ N⌉` qubits, inicializado en `|1⟩`.
  Va a contener los valores `k^x mod N`.

**Pasos (esquema, detalle en las próximas slides):**
1. Superposición uniforme en el registro de conteo (Hadamards).
2. Exponenciación modular controlada: `|x⟩|1⟩ → |x⟩|k^x mod N⟩`.
3. **QFT inversa** sobre el registro de conteo.
4. **Medir** el registro de conteo → estimación de `s/r`.
5. **Fracciones continuas** (clásico) → recuperar `r`.

**Notas de orador:**
- Aclarar la elección de `t`: con `2^t ≥ N²` se garantiza que el resultado de la medición permite recuperar `r` por fracciones continuas de manera unívoca.
- El paso 2 entrelaza los dos registros: la periodicidad de `f` queda "grabada" en el estado conjunto.

**Preguntas típicas y respuestas:**
- *"¿Por qué `2^t ≥ N²` y no `≥ N`?"* → Para que la mejor aproximación racional `s/r` con denominador `< N` sea única; con menos resolución dos fracciones distintas podrían confundirse.

---

## Slide 18 — La QFT: qué es

La **Transformada de Fourier Cuántica** sobre `n` qubits (`M = 2^n` estados base) actúa así:

```
QFT |j⟩ = (1/√M) Σ_{k=0}^{M−1} e^{2πi · j·k / M} |k⟩
```

- Mapea un estado base `|j⟩` a una **superposición uniforme con fases que giran** a una "frecuencia" proporcional a `j`.
- Sobre una superposición periódica (como la que produce la exponenciación modular), las fases **interfieren constructivamente** en los múltiplos de `M/r` y **destructivamente** en el resto.
- Resultado: al medir, los valores probables están concentrados cerca de `c ≈ s · M / r` → de ahí sale `r`.

**Implementación eficiente (forma producto):** se construye con **Hadamards** y **rotaciones de fase controladas** `R_d` (fase `e^{2πi/2^d}`):

```
R_d = [ 1      0       ]
      [ 0   e^{2πi/2^d} ]
```

Usa `O(n²)` compuertas.

**Notas de orador:**
- La metáfora útil: la QFT "escucha" el estado y revela en qué frecuencia se repite. La periodicidad de `f` se traduce en picos en el dominio transformado.
- No hace falta derivar la forma producto compuerta por compuerta; basta mostrar que son H + rotaciones controladas y que son `O(n²)`.

**Preguntas típicas y respuestas:**
- *"¿QFT o QFT inversa en Shor?"* → En el circuito de order-finding se aplica la **QFT inversa** sobre el registro de conteo (es estimación de fase). La diferencia es solo el signo del exponente; el costo es el mismo.

---

## Slide 19 — QFT vs. DFT/FFT clásica

| | **DFT / FFT (clásica)** | **QFT (cuántica)** |
|---|---|---|
| Sobre qué actúa | Vector de `M` amplitudes **explícitas** en memoria | Amplitudes de una **superposición** de `n = log₂ M` qubits |
| Costo | FFT: `O(M log M) = O(2ⁿ · n)` | `O(n²) = O((log M)²)` compuertas |
| Salida | Todas las `M` componentes, **legibles** | Estado transformado; al medir, **una sola muestra** (probabilística) |
| Acceso a resultados | Lectura completa del espectro | No se puede leer todo: la medición colapsa |

**La diferencia clave:**
- La QFT es **exponencialmente más rápida** en cantidad de operaciones…
- …pero **no entrega el espectro completo**: solo se puede *muestrear* una salida.
- El arte de Shor es diseñar el circuito para que **lo que se muestrea codifique justo el período `r`.**

**Notas de orador:**
- Desactivar el malentendido común: "la QFT calcula la FFT más rápido y la leo entera". Falso. No hay lectura completa.
- La ganancia exponencial en operaciones se "paga" con acceso restringido a la salida; por eso hace falta el post-procesamiento (fracciones continuas) y la repetición.

**Preguntas típicas y respuestas:**
- *"Entonces, ¿la QFT no sirve para acelerar FFTs comunes?"* → No directamente, justamente porque no se pueden leer todas las salidas. Sirve dentro de algoritmos (como Shor) donde alcanza con muestrear cierta información estructural.

---

## Slide 20 — Estimación de fase: el motor del order-finding

El subprograma de período es una **estimación de fase** del operador `U`:

```
U |y⟩ = |k · y mod N⟩
```

- Los autovectores de `U` tienen autovalores `e^{2πi s/r}` (con `s = 0,…,r−1`).
- El estado inicial `|1⟩` del registro de trabajo es una **superposición de esos autovectores**.
- Aplicando `U` controlada `2^j` veces desde cada qubit de conteo, se "imprime" la fase `s/r` en el registro de conteo.
- La **QFT inversa** convierte esa fase en un número medible `c ≈ s·2^t/r`.

```
k^x mod N  ─┐
            ├─ periodicidad r  ─→  fases  s/r  ─→  QFT⁻¹  ─→  medir c ≈ s·2^t / r
estados base┘
```

**Notas de orador:**
- Para audiencia que vio estimación de fase: decir explícitamente "order-finding = estimación de fase de la compuerta de multiplicación por k".
- Para quien no la recuerde: la idea operativa es que las potencias controladas de U graban la fase `s/r` en el registro de conteo, y la QFT⁻¹ la vuelve legible como entero.

**Preguntas típicas y respuestas:**
- *"¿Por qué arrancar el work register en `|1⟩` y no en `|0⟩`?"* → Porque `|1⟩` es combinación de todos los autovectores relevantes de U (la suma de autovectores da el estado `|1⟩`), lo que permite que cualquier `s` aparezca. `|0⟩` sería absorbido (0·k = 0) y no daría información.

---

## Slide 21 — El circuito cuántico completo (diagrama)

```
                t qubits de conteo (init |0⟩)
  |0⟩ ─[H]─────────●───────────────────────────[       ]──[M]
  |0⟩ ─[H]─────────┼────────●──────────────────[ QFT⁻¹ ]──[M]
  |0⟩ ─[H]─────────┼────────┼────────●─────────[       ]──[M]
   ⋮     ⋮         │        │        │             ⋮        ⋮
  |0⟩ ─[H]─────────┼────────┼────────┼──●───────[       ]──[M]
                   │        │        │  │
              ┌────┴────────┴────────┴──┴────┐
  |1⟩ ═══════│   k^(2^0)  k^(2^1) ...  mod N  │═══════════════
   n qubits  │   (exponenciación modular      │   (registro de
   de trabajo│    controlada — bloque U)      │    trabajo, no se mide)
  |0…0⟩ ═════└──────────────────────────────────┘═══════════════
```

**Lectura del circuito:**
1. `H` en los `t` qubits de conteo → superposición uniforme de todos los `x`.
2. Bloque de **exponenciación modular controlada**: cada qubit de conteo `j` controla `U^(2^j)`, dejando `|x⟩|k^x mod N⟩`.
3. **QFT⁻¹** sobre el registro de conteo.
4. **Medición** del registro de conteo → `c`. (El registro de trabajo no se mide.)

**Notas de orador:**
- Recorrer las cuatro capas. Enfatizar que las mediciones son solo del registro de conteo.
- Mencionar que el registro de trabajo "se sacrifica": su rol es entrelazarse para grabar la fase; no se lee.

**Preguntas típicas y respuestas:**
- *"¿Cuántos qubits en total?"* → `t + n`, con `t ≈ 2·log₂N` y `n ≈ log₂N`. Para N = 15: t = 8, n = 4 → 12 qubits en esta versión "de libro" (las implementaciones optimizadas usan muchos menos).

---

## Slide 22 — El bloque costoso: exponenciación modular controlada

```
U : |y⟩ → |k · y mod N⟩       y, encadenando:    |x⟩|1⟩ → |x⟩|k^x mod N⟩
```

- Se trata como un **bloque abstracto** en el circuito (no desarrollamos su construcción interna a nivel compuertas).
- Internamente: sumadores modulares, multiplicadores modulares y exponenciación por cuadrados controlada.
- **Es el bloque más costoso del algoritmo**: domina el conteo de compuertas y de qubits auxiliares (ancillas). La complejidad total de Shor está gobernada por este paso.

> En implementaciones reales (Qiskit y similares) esta es la parte que más
> recursos consume y la que más se optimiza. La QFT, en comparación, es barata.

**Notas de orador:**
- Justificar la decisión de mostrarlo como caja: su construcción interna es un tema en sí mismo y no aporta a la intuición del algoritmo. Lo importante es saber que **acá está el cuello de botella de recursos**.
- Conectar con la slide de complejidad: el `O((log N)³)` viene esencialmente de este bloque.

**Preguntas típicas y respuestas:**
- *"¿Por qué es reversible si exponenciar 'pierde' información?"* → Se implementa de forma reversible guardando el input: `|x⟩|0⟩ → |x⟩|k^x mod N⟩`. Toda compuerta cuántica es unitaria, así que se diseña el circuito para no descartar información (usando ancillas y des-computación).

---

## Slide 23 — De la medición al período: fracciones continuas

La medición da un entero `c` con `c / 2^t ≈ s / r` (para algún `s` desconocido).
Para recuperar `r` se usa el **desarrollo en fracciones continuas** de `c / 2^t`:
sus **convergentes** dan la mejor aproximación racional con denominador acotado, y ese denominador es (un divisor de) `r`.

**Método (clásico, eficiente):**
```
c/2^t = a₀ + 1/(a₁ + 1/(a₂ + ...))   →   convergentes p_i/q_i   →   r = q_i  (el adecuado)
```

**Importante para N = 15:** todos los `k` coprimos tienen `r ∈ {2, 4}`, que son
**potencias de 2**. Como el registro tiene `2^t` estados, las fracciones `s/r`
salen **exactas** (`s/2`, `s/4`) y las fracciones continuas son triviales. **No hay
"caso feo" de fracciones continuas sucias con N = 15.**

> Un ejemplo donde las fracciones continuas hacen trabajo no trivial requeriría un
> N mayor (p. ej. **N = 21**, con períodos como `r = 6`). No lo fabricamos acá:
> con N = 15 el post-procesamiento es exacto por construcción.

**Notas de orador:**
- Dejar clarísima la aclaración del brief: el único "caso feo" disponible con 15 es el **fallo por `p = N−1`** (k = 14), **no** un post-procesamiento complicado.
- Si preguntan por el caso no trivial, mencionar N = 21 como ejemplo donde `r = 6` obliga a usar convergentes en serio, pero no desarrollarlo (no está en el alcance).

**Preguntas típicas y respuestas:**
- *"¿Por qué fracciones continuas y no redondear `c·r/2^t`?"* → Porque `r` es justamente lo desconocido. Fracciones continuas recuperan el denominador `r` a partir solo de `c/2^t`, sin conocerlo de antemano.

---

## Slide 24 — Ejemplo N = 15: preparación

**Objetivo:** factorizar `N = 15`.

**Filtros (prerrequisitos):**
- ¿Primo? No (15 = 3·5).
- ¿Par? No (impar).
- ¿Potencia perfecta `n^x`? No.
- → 15 es compuesto impar con 2 primos distintos. **Apto para Shor.**

**Tamaño de registros (versión de libro):**
- Trabajo: `n = ⌈log₂ 15⌉ = 4` qubits.
- Conteo: `2^t ≥ 15² = 225` → `t = 8` qubits (`2^8 = 256`).

**Notación (¡atención!):** en lo que sigue, `p = k^(r/2) mod 15` es un **residuo**,
no un número primo. Los factores buscados son 3 y 5.

Vamos a ver **dos casos** con el mismo N = 15:
- **Caso feo (fallo):** `k = 14`.
- **Caso feliz (éxito):** `k = 7`.

**Notas de orador:**
- Mostrar que los filtros son instantáneos.
- Reforzar la advertencia de notación una vez más: `p` = residuo. Es la fuente número uno de confusión en esta charla.

**Preguntas típicas y respuestas:**
- *"¿No es trampa que ya sepamos que 15 = 3·5?"* → Sí, 15 es didáctico. El punto no es descubrir 3 y 5 (obvios), sino **ver el mecanismo completo funcionando** con números chequeables a mano.

---

## Slide 25 — Caso feo (fallo): k = 14

**1. Coprimalidad:** `gcd(14, 15) = 1` ✓ (coprimo, seguimos).

**2. Período (parte cuántica):** `f(x) = 14^x mod 15`:
```
x:     0   1   2   3   4 ...
14^x:  1  14   1  14   1 ...     →   r = 2   (par ✓)
```

**3. `r` es par** → calculamos `p = 14^(r/2) mod 15 = 14^1 mod 15 = 14`.

**4. Chequeo de fallo `p + 1 = N`:**
```
p + 1 = 14 + 1 = 15 = N        →    ¡FALLA!
```
Esto es `14^(r/2) ≡ −1 (mod 15)`: N divide por completo a `(p + 1)`.
Si igual calculáramos los GCD, saldrían **triviales**:
```
gcd(p − 1, N) = gcd(13, 15) = 1        gcd(p + 1, N) = gcd(15, 15) = 15
```
→ ni 1 ni 15 sirven como factor.

**Conclusión:** `k = 14` **no factoriza**. Se descarta y se reintenta con otro `k`.

**Notas de orador:**
- Este caso ilustra el modo de fallo (b)-`≡ −1` con números a la vista. Hacer notar que `r` era par (no falló por imparidad) y aún así no sirvió.
- Mensaje: Shor es **probabilístico**; hay que estar dispuesto a reintentar. Existen dos modos de fallo: `r` impar, o `k^(r/2) ≡ −1`. Acá vimos el segundo.

**Preguntas típicas y respuestas:**
- *"¿k = 14 no es simplemente −1 mod 15?"* → Exacto: `14 ≡ −1 (mod 15)`, así que `14² ≡ 1` y su orden es 2, con `14^1 ≡ −1`. Es el ejemplo más limpio del modo de fallo `≡ −1`.

---

## Slide 26 — Caso feliz: k = 7 — parte clásica

**1. Coprimalidad:** `gcd(7, 15) = 1` ✓.

**2. Período (parte cuántica, lo resolvemos en detalle en las próximas slides):**
```
f(x) = 7^x mod 15      →      r = 4
```

**3. `r = 4` es par** ✓ → `p = 7^(r/2) mod 15 = 7^2 mod 15 = 49 mod 15 = 4`.

**4. Chequeo `p + 1 = N`?:**
```
p + 1 = 4 + 1 = 5 ≠ 15        →    PASA ✓   (7^2 ≢ −1 mod 15)
```

**5. Extraer factores con GCD:**
```
gcd(p − 1, N) = gcd(4 − 1, 15) = gcd(3, 15) = 3
gcd(p + 1, N) = gcd(4 + 1, 15) = gcd(5, 15) = 5
```

### → Factores de 15: **3 y 5**. ✓

**Notas de orador:**
- Mostrar la cadena completa de (a)→(b)→(c) instanciada: `r` par (a), `p ≠ N−1` (b), GCDs dan 3 y 5 (c).
- Verificar en voz alta: 3 × 5 = 15. Cierra.

**Preguntas típicas y respuestas:**
- *"¿Y si me hubiera tocado k = 4 (r = 2)?"* → `4^1 mod 15 = 4`, `p+1=5≠15`, `gcd(3,15)=3`, `gcd(5,15)=5`. También funciona. Varios k buenos coexisten con los malos (2, 7, 8, 13 dan r = 4; 4, 11 dan r = 2 y andan; 14 falla).

---

## Slide 27 — Caso k = 7 — parte cuántica: la periodicidad

`f(x) = 7^x mod 15`, evaluada en superposición por el circuito:

```
x:      0   1   2   3   4   5   6   7   ...
7^x:    1   7   4  13   1   7   4  13   ...
        └──────────────┘
          período r = 4
```

El registro de trabajo, tras la exponenciación modular, queda **entrelazado** con el de conteo: para cada `x`, guarda `7^x mod 15 ∈ {1, 7, 4, 13}`.

La secuencia `1, 7, 4, 13, 1, 7, 4, 13, …` **se repite cada 4 pasos** → esa es la
periodicidad que la QFT⁻¹ va a convertir en algo medible.

**Notas de orador:**
- Mostrar el ciclo de 4 valores explícito. Es la "señal periódica" que vamos a transformar.
- Recordar: clásicamente, detectar este período en N grande sería inviable; acá es chico para poder verlo a mano.

**Preguntas típicas y respuestas:**
- *"¿El registro de trabajo toma los 4 valores a la vez?"* → Sí, en superposición entrelazada con `x`. Pero no lo medimos: medir el registro de conteo (tras QFT⁻¹) es lo que revela el período.

---

## Slide 28 — Caso k = 7 — el circuito concreto

```
        registro de conteo: t = 8 qubits (init |0⟩)
  |0⟩⊗8 ─[H⊗8]──●──────────────────[ QFT⁻¹ (8 qubits) ]──[ medir → c ]
                │
           ┌────┴───────────────────────┐
  |0001⟩ ══│  U^(2^j): |y⟩→|7·y mod 15⟩  │═══════ (registro de trabajo: 4 qubits,
   (=|1⟩)  │  exponenciación modular     │         init |1⟩, NO se mide)
           └─────────────────────────────┘
```

- Conteo: 8 qubits (`2^8 = 256 ≥ 15² = 225`).
- Trabajo: 4 qubits, inicializado en `|1⟩` = `|0001⟩`.
- Bloque `U`: multiplicación por 7 módulo 15, aplicada `2^j` veces controlada por el qubit `j` de conteo.
- QFT⁻¹ sobre los 8 qubits de conteo, luego medición.

**Notas de orador:**
- Es el mismo circuito genérico de la slide 21, ahora con números: k = 7, N = 15, t = 8, n = 4.
- Aclarar que esta es la versión "de libro"; las demostraciones experimentales históricas con 15 usaron circuitos comprimidos (con conocimiento previo de la respuesta), no esta versión genérica.

**Preguntas típicas y respuestas:**
- *"¿8 qubits de conteo no es mucho para N = 15?"* → Para garantizar fracciones continuas unívocas, sí se usa `t ≈ 2 log₂N`. En N = 15, como `r` es potencia de 2, alcanzaría con menos, pero mantenemos la regla general para no hacer trampa.

---

## Slide 29 — Caso k = 7 — mediciones y fracciones continuas

Con `r = 4` y `2^t = 256`, la medición del registro de conteo arroja, con alta
probabilidad, uno de los valores `c ≈ s · 256 / 4 = s · 64` (`s = 0,1,2,3`):

```
c ∈ {0, 64, 128, 192}      (picos de la QFT⁻¹)
```

Para cada `c` se calcula `c / 256` y se aplican **fracciones continuas**:

| `c`  | `c / 256` | fracción reducida | denominador | ¿da r = 4? |
|------|-----------|-------------------|-------------|------------|
| 0    | 0/256     | 0                 | —           | inútil (s=0) → repetir |
| 64   | 64/256    | **1/4**           | **4**       | ✓ `r = 4` |
| 128  | 128/256   | 1/2               | 2           | candidato r=2; `7²=49≡4≠1` → falla, repetir |
| 192  | 192/256   | **3/4**           | **4**       | ✓ `r = 4` |

- `c = 64` y `c = 192` dan directamente **`r = 4`**.
- `c = 128` reduce a `1/2` (porque `s = 2` y `r = 4` comparten el factor 2): da el candidato `r = 2`, que se **verifica** (`7² mod 15 = 4 ≠ 1`) y se **descarta** → se repite la medición.
- `c = 0` no aporta (`s = 0`) → se repite.

> Con repetir la medición unas pocas veces, se obtiene `r = 4` con altísima probabilidad.

**Notas de orador:**
- Mostrar que la medición es probabilística: no todos los outcomes sirven, pero la mayoría sí, y los inútiles se detectan al verificar `k^candidato mod N = 1`.
- El caso `c = 128 → 1/2` es pedagógicamente valioso: ilustra por qué a veces el denominador es solo un **divisor** de `r` y hay que verificar/repetir.

**Preguntas típicas y respuestas:**
- *"¿Cómo sé que `r = 2` es falso sin conocer r?"* → Se verifica directamente: `7² mod 15 = 4 ≠ 1`, así que 2 no es período. La verificación `k^r mod N = 1` es clásica y barata.
- *"¿Las probabilidades de cada pico son iguales?"* → Para `r | 2^t` (como acá), los cuatro picos son exactos y equiprobables (≈ 1/4 cada uno). En el caso general no son exactos y aparece dispersión alrededor de `s·2^t/r`.

---

## Slide 30 — Caso k = 7 — cierre del ejemplo

**Recapitulación del flujo completo con k = 7:**

```
N = 15  →  filtros OK  →  k = 7  →  gcd(7,15)=1
   └─[CUÁNTICO]→ período r = 4  (medición 64/256 → 1/4)
   →  r par ✓
   →  p = 7^2 mod 15 = 4
   →  p+1 = 5 ≠ 15  ✓ (no es el modo de fallo)
   →  gcd(4−1,15) = 3      gcd(4+1,15) = 5
   →  15 = 3 × 5   ✓✓
```

**Lo que mostró el ejemplo:**
- El **único** paso cuántico fue hallar `r = 4`.
- Todo lo demás (filtros, GCD, chequeos, álgebra) fue **clásico y trivial**.
- Comparado con `k = 14`, se ve que Shor es **probabilístico**: el k importa.

**Notas de orador:**
- Cerrar el arco narrativo: un fallo (14) y un éxito (7), mismo N, todo a mano.
- Reconectar con la teoría: este ejemplo instanció las cuatro piezas (a, b, c, d) de la cadena algebraica.

**Preguntas típicas y respuestas:**
- *"En N grande, ¿este mismo procedimiento escala?"* → Sí, idéntico. Lo único que crece es el costo de la parte cuántica (más qubits, bloque de exponenciación modular más grande), pero el esquema es el mismo.

---

## Slide 31 — El flujo completo, de un vistazo

```
        ┌─────────────────────────────────────────────────────┐
        │  N (compuesto impar, no potencia perfecta)           │
        └───────────────────────┬─────────────────────────────┘
                                 ▼
                    elegir k al azar, 1<k<N
                                 ▼
                    g = gcd(k,N)  ──(g≠1)──►  factor g. FIN
                                 │ (g=1)
                                 ▼
                ╔═══════ CUÁNTICO ═══════╗
                ║ hallar período r de     ║
                ║ f(x)=k^x mod N (QFT)    ║
                ╚════════════╤════════════╝
                             ▼
              r impar? ──sí──►  descartar, otro k
                  │ no
                  ▼
              p = k^(r/2) mod N
                  ▼
              p+1 = N? ──sí──►  descartar, otro k
                  │ no
                  ▼
        factores = gcd(p−1,N), gcd(p+1,N).  FIN
```

**Notas de orador:**
- Usar esta slide como resumen/repaso antes de pasar a complejidad e implicancias.
- Volver a marcar el único bloque cuántico y los dos puntos de reintento.

---

## Slide 32 — Complejidad: Shor vs. el mejor clásico (GNFS)

Comparamos **tiempo** (no espacio) para factorizar un entero de `n = log₂ N` bits.

**Mejor clásico conocido — General Number Field Sieve (GNFS):** sub-exponencial
```
exp( ( (64/9)^(1/3) + o(1) ) · (ln N)^(1/3) · (ln ln N)^(2/3) )
```
Crece más lento que exponencial puro, pero **mucho más rápido que cualquier polinomio**.

**Shor (cuántico):** polinómico
```
Õ( (log N)² )  en compuertas cuánticas (con multiplicación rápida)
≈ O( (log N)³ )  en la versión estándar
```
más post-procesamiento clásico polinómico (Euclides, fracciones continuas).

> Salto cualitativo: **sub-exponencial → polinómico.** No es "una constante mejor":
> es otra clase de complejidad.

**Notas de orador:**
- Aclarar que las cotas finas de Shor (`(log N)²·log log N·log log log N` con FFT-multiplicación) vienen de Nielsen & Chuang / Shor 1994, **no** del paper de la cátedra, que solo dice vagamente "raíz cuadrada del tiempo" (afirmación imprecisa que conviene corregir).
- El costo real está dominado por la exponenciación modular (bloque U).

**Preguntas típicas y respuestas:**
- *"¿'Polinómico' significa rápido en la práctica hoy?"* → Polinómico en *operaciones lógicas ideales*. En hardware real, la corrección de errores multiplica enormemente los recursos físicos. Lo vemos en la slide de estimaciones.

---

## Slide 33 — Crecimiento comparativo (intuición)

```
tiempo
  │                                          GNFS (clásico, sub-exp)
  │                                       ╱
  │                                   ╱
  │                              ╱
  │                        ╱
  │                  ╱
  │            ╱
  │       ╱                         _______________ Shor (polinómico)
  │    ╱            ______________/
  │ ╱ ____________/
  └────────────────────────────────────────────────► tamaño de N (bits)
```

- Para números chicos, la diferencia no se nota.
- Para **claves RSA reales (2048+ bits)**, GNFS se va a tiempos astronómicos mientras Shor se mantiene tratable (en una máquina cuántica ideal suficientemente grande).
- Esa brecha es exactamente la amenaza criptográfica.

**Notas de orador:**
- Gráfico cualitativo, no a escala. El mensaje es la **forma** de las curvas: una explota, la otra crece suave.
- Recordar el "pero": "máquina cuántica ideal suficientemente grande" es el gran condicional, que aterrizamos en las próximas slides.

---

## Slide 34 — Implicancias en la criptografía actual

**Qué rompe Shor (si hay hardware suficiente):**
- **RSA** (factorización) — cifrado y firma.
- **Diffie-Hellman, ElGamal, DSA** (logaritmo discreto) — variante de Shor.
- **ECDH, ECDSA** (logaritmo discreto en curvas elípticas) — también caen, e incluso con *menos* qubits que RSA del mismo nivel de seguridad.

**Qué NO rompe (o solo debilita levemente):**
- **Criptografía simétrica (AES):** Shor no aplica. El ataque cuántico relevante es **Grover**, que solo da una aceleración cuadrática → se mitiga **duplicando el tamaño de clave** (AES-128 → AES-256).
- **Funciones de hash (SHA-2/3):** igual, Grover cuadrático → usar hashes más largos.

> Resumen: **la criptografía de clave pública clásica es la víctima principal**; la
> simétrica sobrevive con claves más largas.

**Notas de orador:**
- Distinguir clarísimo: Shor mata la **clave pública** basada en factorización/log discreto; **no** mata la simétrica.
- Grover ≠ Shor: Grover es búsqueda, aceleración cuadrática, fácil de compensar. Es un error común confundirlos.

**Preguntas típicas y respuestas:**
- *"Entonces, ¿AES-256 está bien?"* → Frente a lo conocido hoy, sí: Grover lo lleva a ~128 bits de seguridad efectiva, todavía robusto. El problema urgente es la clave pública.

---

## Slide 35 — Estado actual del hardware cuántico (2025–2026)

- **Google "Willow" (fines 2024):** 105 qubits superconductores; demostró
  **corrección de errores "por debajo del umbral"** (el error lógico baja al
  agregar qubits físicos). Hito cualitativo, no una máquina factorizadora.
- **IBM — hoja de ruta a tolerancia a fallos:** sistema **"Starling" para 2029**
  con **~200 qubits lógicos** y 100 millones de operaciones corregidas; hitos
  intermedios *Loon* (2025), *Kookaburra* (2026), *Cockatoo* (2027). Meta de
  **~1000 qubits lógicos** a inicios de la década de 2030, usando códigos **qLDPC**.
- **Realidad para Shor:** todavía **no existe** una máquina capaz de correr Shor
  sobre números criptográficamente relevantes. Falta tolerancia a fallos a escala.

> Distinción clave: **qubits físicos** (ruidosos, abundantes) vs. **qubits lógicos**
> (corregidos, escasos y caros). Shor a escala RSA necesita miles de lógicos.

**Notas de orador:**
- Citar fechas y fuentes (ver slide de referencias). Estos datos envejecen rápido: revisar antes de cada exposición.
- El punto honesto: hay progreso real y sostenido, pero romper RSA sigue estando a años de distancia según las hojas de ruta públicas.

**Preguntas típicas y respuestas:**
- *"¿Cuándo se rompe RSA-2048?"* → Nadie lo sabe con certeza. Las hojas de ruta apuntan a la década de 2030 para máquinas con miles de qubits lógicos. Por eso la migración a PQC empieza **ahora** (riesgo "harvest now, decrypt later").

---

## Slide 36 — El "gap" honesto: lo más grande factorizado de verdad

- **En hardware cuántico real, ejecutando Shor:** los récords son **15** (2001, IBM,
  ~8 qubits) y **21** (2012, ~10 qubits). Eso es todo lo confiable.
- Muchos "récords" mediáticos de factorización de números grandes usan **circuitos
  pre-compilados con conocimiento previo de la respuesta**, o métodos **adiabáticos
  / de optimización** (no Shor) que **no escalan**. No cuentan como Shor genuino.
- **En simulación clásica** de Shor (no hardware cuántico) se han factorizado
  semiprimos de ~12 dígitos (p. ej. 549.755.813.701 = 712.321 × 771.781), pero eso
  mide la simulación, no una ventaja cuántica real.

> La distancia entre "15 en hardware real" y "2048 bits en RSA" es **enorme**. Hay
> que ser honestos: la amenaza es **futura y creíble**, no presente.

**Notas de orador:**
- Este slide es el antídoto contra el hype. Si la audiencia se queda con una sola idea escéptica, que sea esta.
- Explicar por qué los "récords grandes" son engañosos: si ya se conocen los factores, se puede comprimir el circuito a algo trivial; no demuestra capacidad de factorizar lo desconocido.

**Preguntas típicas y respuestas:**
- *"¿Y los titulares que dicen que ya factorizaron números enormes?"* → Casi siempre son recocido cuántico (D-Wave) o trucos de compilación, no Shor escalable. La comunidad los considera no representativos.

---

## Slide 37 — Estimaciones de recursos para romper RSA-2048

¿Cuántos qubits harían falta para RSA-2048 con Shor?

- **Gidney & Ekerå (2019):** ~**20 millones** de qubits físicos ruidosos, ~8 horas.
- **Gidney (mayo 2025, arXiv:2505.15917):** **menos de 1 millón** de qubits físicos
  ruidosos, **< 1 semana** — unos **1399 qubits lógicos** codificados. Una reducción
  de ~20× respecto de 2019, vía mejor aritmética modular y codificación más eficiente.

**Lectura:**
- La barrera **sigue siendo enorme** frente a las máquinas actuales (cientos de qubits físicos, ~decenas de lógicos en el mejor caso).
- Pero las estimaciones **bajan año a año**: el blanco se acerca por mejoras de algoritmo y de corrección de errores, no solo por más hardware.

**Notas de orador:**
- Contar la historia 2019 → 2025 como evidencia de que el "cuánto falta" se mueve: cada avance algorítmico recorta requisitos.
- Reforzar: no hace falta esperar a tener la máquina para preocuparse; basta con que la trayectoria sea creíble.

**Preguntas típicas y respuestas:**
- *"Si baja tan rápido, ¿no será inminente?"* → Las reducciones son en el *modelo de recursos*; construir ~1 millón de qubits físicos con corrección de errores sigue siendo un desafío de ingeniería de años. Pero justifica migrar ya.

---

## Slide 38 — Criptografía post-cuántica: NIST PQC

Algoritmos diseñados para resistir a Shor (y a Grover), corribles en hardware clásico actual. **NIST** finalizó los primeros estándares:

| Estándar | Algoritmo (competencia) | Nombre final | Tipo | Fecha |
|----------|------------------------|--------------|------|-------|
| **FIPS 203** | CRYSTALS-Kyber | **ML-KEM** | Encapsulado de clave (KEM) | 13-ago-2024 |
| **FIPS 204** | CRYSTALS-Dilithium | **ML-DSA** | Firma digital | 13-ago-2024 |
| **FIPS 205** | SPHINCS+ | **SLH-DSA** | Firma (basada en hash) | 13-ago-2024 |
| **FIPS 206** (borrador) | FALCON | **FN-DSA** | Firma | en proceso (2026) |
| (5.º algoritmo) | **HQC** | — | KEM (respaldo, código) | selecc. 11-mar-2025; borrador ~2026, final ~2027 |

- ML-KEM, ML-DSA y FN-DSA se basan en **retículos** (lattices); SLH-DSA en **hashes**; HQC en **códigos** (familia distinta, como respaldo ante un eventual ataque a retículos).

**Notas de orador:**
- Usar los **nombres finales** (ML-KEM, ML-DSA, SLH-DSA), no solo los de competencia (Kyber, Dilithium, SPHINCS+). El brief lo pide explícitamente.
- Explicar por qué HQC: NIST quiere un KEM de **familia matemática diferente** (códigos) por diversificación de riesgo frente a los basados en retículos.

**Preguntas típicas y respuestas:**
- *"¿Por qué dos KEM (ML-KEM y HQC)?"* → Diversificación: si apareciera un ataque a retículos, HQC (códigos) sería un respaldo ya estandarizado.
- *"¿Estos sí son seguros contra cuántica?"* → Se basan en problemas sin algoritmo cuántico eficiente conocido. No hay prueba absoluta (como tampoco la hay para RSA), pero resistieron años de criptoanálisis público.

---

## Slide 39 — Qué hacer hoy: "harvest now, decrypt later"

- **Amenaza presente aunque la máquina sea futura:** un adversario puede **capturar
  hoy** tráfico cifrado y **descifrarlo mañana**, cuando exista la computadora
  cuántica. Datos con vida útil larga (secretos de estado, salud, propiedad
  intelectual) ya están en riesgo.
- **Por eso la migración empezó:** estándares finalizados (2024–2025), navegadores y
  bibliotecas (TLS) ya incorporan **modos híbridos** (clásico + ML-KEM).
- **Recomendación práctica:** inventariar dónde se usa clave pública, priorizar lo de
  larga vida, y adoptar esquemas **híbridos** durante la transición.

**Notas de orador:**
- "Harvest now, decrypt later" es el argumento que convierte una amenaza futura en una acción presente. Es la razón de que NIST se apurara.
- Híbrido = se combinan un esquema clásico y uno PQC; el atacante tiene que romper **ambos**. Da seguridad durante la incertidumbre de transición.

**Preguntas típicas y respuestas:**
- *"¿Conviene tirar RSA ya?"* → No de golpe; se migra a híbridos para no perder interoperabilidad ni asumir riesgos de implementaciones PQC inmaduras. Pero empezar ya, sí.

---

## Slide 40 — Conclusiones

- Shor **reduce factorizar a hallar el período** de `f(x) = k^x mod N`; ese período
  es **lo único que corre en la computadora cuántica**.
- La cadena algebraica es cerrada: `r` par → `k^r−1=(k^{r/2}−1)(k^{r/2}+1)` →
  evitar `k^{r/2}≡−1` → `gcd(k^{r/2}±1, N)` da factores, con **éxito ≥ 1/2 por intento**.
- La **QFT** (estimación de fase) extrae el período; es exponencialmente más barata
  que la FFT en operaciones, pero solo permite **muestrear** la salida.
- Complejidad: **polinómica** (Shor) vs. **sub-exponencial** (GNFS) — un salto de clase.
- **Hoy** no hay hardware para romper RSA real (récords reales: 15 y 21), pero las
  hojas de ruta y las estimaciones decrecientes hacen la amenaza **creíble**.
- La respuesta ya existe: **NIST PQC** (ML-KEM, ML-DSA, SLH-DSA, + HQC) y migración
  **híbrida**, motivada por **"harvest now, decrypt later"**.

**Notas de orador:**
- Cerrar volviendo a las dos columnas: clásico vs. cuántico. Lo único cuántico fue el período; todo lo demás, clásico.
- Frase de cierre sugerida: "Shor no hace que la computadora cuántica adivine los factores: hace que escuche la frecuencia con la que una función se repite, y de esa frecuencia caen los factores."

**Preguntas típicas y respuestas:**
- *"¿Qué deberíamos estudiar para profundizar?"* → Nielsen & Chuang cap. 5 (estimación de fase y order-finding); el paper de Shor 1994; documentación de Qiskit sobre la implementación de order-finding.

---

## Slide 41 — Referencias

**Fuente técnica del algoritmo (estándar):**
- M. Nielsen, I. Chuang, *Quantum Computation and Quantum Information*, Cambridge — cap. 5 (order-finding, estimación de fase) y Apéndice A4 (cota de probabilidad, Teorema A4.13).
- P. Shor, "Algorithms for Quantum Computation: Discrete Logarithms and Factoring", FOCS 1994.

**Material de la cátedra:**
- C. H. Ugwuishiwu et al., "An Overview of Quantum Cryptography and Shor's Algorithm", *IJATCSE* 9(5), 2020 — usado para narrativa/motivación. (No contiene cotas de complejidad ni de probabilidad; tiene una errata en la factorización `(m^{P/2}−1)(m^{P/2}−1)`.)
- Clase 04 — Criptografía: Cifrado asimétrico y firma digital, ITBA (RSA, DH, ElGamal, firmas).

**Estado del hardware y estimaciones (consultado junio 2026):**
- Google Quantum AI, chip "Willow" (corrección por debajo del umbral), 2024–2025.
- IBM, hoja de ruta a tolerancia a fallos ("Starling" 2029), comunicado del 10-jun-2025.
- C. Gidney, "How to factor 2048-bit RSA integers with less than a million noisy qubits", arXiv:2505.15917, 2025.
- C. Gidney, M. Ekerå, "How to factor 2048 bit RSA integers in 8 hours using 20 million noisy qubits", 2019.

**Criptografía post-cuántica (consultado junio 2026):**
- NIST, FIPS 203 (ML-KEM), 204 (ML-DSA), 205 (SLH-DSA), finalizados 13-ago-2024.
- NIST, selección de HQC como 5.º algoritmo (KEM), 11-mar-2025; FIPS 206 (FN-DSA/Falcon) en borrador.

**Guía narrativa (no técnica):**
- K. Rakhade, "Shor's Factoring Algorithm", Medium (solo orden didáctico).

**Notas de orador:**
- Tener a mano estas referencias para responder preguntas de profundidad.
- Recordar a la audiencia que las cifras de hardware/PQC se verificaron en junio de 2026 y conviene refrescarlas.
