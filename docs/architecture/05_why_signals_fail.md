# Burgundy — Módulo Educativo: "Por qué fallan las señales" (Why Signals Fail)

**Proyecto:** Burgundy — Simulador educativo de trading
**Autor / Firma:** tsuloid
**Documento:** 05_why_signals_fail.md
**Idioma:** Español (LATAM)
**Estado:** Especificación de diseño — sin código, sin pantallas, sin desarrollo todavía.

> Paleta Burgundy de referencia: fondo profundo `#1A1617`, acento principal `#571324`, superficies y divisores `#2E2E2E`, resaltado crítico `#C9A050`, velas alcistas `#4A6D56`, velas bajistas `#802F3E`.

---

## 1. Visión general del módulo (Module Overview)

### 1.1 Qué es este módulo

"Por qué fallan las señales" es un módulo educativo de Burgundy enfocado **exclusivamente** en el comportamiento de copiar señales de trading: las señales que circulan en Telegram, TikTok, WhatsApp, Discord y grupos de "traders expertos" que abundan en LATAM.

El módulo **no** dice que las señales son siempre malas. Tampoco promueve servicios de señales. Su objetivo es más profundo y más útil:

> **Enseñar que una señal no es un resultado. Una señal es solo una idea de otra persona, y el resultado depende del contexto, el timing, la gestión de riesgo, la disciplina de ejecución y la responsabilidad personal de quien la opera.**

### 1.2 Por qué existe este módulo

El usuario objetivo de Burgundy es un principiante LATAM vulnerable. Comportamientos típicos que este módulo ataca directamente:

| Comportamiento real del principiante | Lo que el módulo le enseña |
|---|---|
| Copia señales de Telegram sin entender el contexto | Una señal sin contexto es una apuesta, no una operación |
| Entra tarde porque vio la señal 20 minutos después | El entry delay (retraso de entrada) destruye el risk/reward |
| Usa apalancamiento 20x porque "el del grupo lo usa" | El leverage del proveedor no es el leverage del seguidor |
| Opera sin stop loss porque la señal "no lo decía" | Sin invalidación clara, no hay operación: hay esperanza |
| Sale por pánico antes del objetivo | La salida emocional convierte señales correctas en pérdidas |
| Confunde una señal ganadora con habilidad propia | Resultado ≠ proceso. Se evalúa el proceso, no solo el P/L |

### 1.3 La idea central que el usuario debe internalizar

**La misma señal, ejecutada por dos personas distintas, puede ganar para una y perder para la otra.** La diferencia no está en la señal: está en el spread del broker, el slippage, el momento de entrada, el tamaño de posición, el apalancamiento, la colocación del stop loss y el comportamiento emocional durante la operación.

### 1.4 Filosofía de simulación aplicada al módulo

Siguiendo la filosofía central de Burgundy:

- **El simulador fuerza un contexto de aprendizaje, no un resultado.** El camino del mercado (market path) se genera **antes** de que el usuario actúe, usando semillas deterministas (deterministic seeds).
- **La app nunca manipula el precio después de la entrada del usuario.** Si el usuario pierde, pierde porque sus decisiones interactuaron mal con un camino de mercado ya fijado, no porque el sistema lo castigó.
- **Mismo template de escenario + parámetros + seed + versión del generador = mismo camino de mercado, siempre.** Esto permite el momento educativo más poderoso del módulo: el replay, donde el usuario repite la **misma** señal sobre el **mismo** mercado y comprueba que un cambio en su ejecución cambia el resultado.
- Las decisiones del usuario afectan: órdenes, posiciones, cuenta, riesgo, P/L, diario (journal), errores detectados, feedback y puntaje. **No** afectan el camino de precios generado.

### 1.5 Modos donde vive el módulo

| Modo | Uso en este módulo |
|---|---|
| Tutorial guiado | Lecciones con seeds fijos, paso a paso, con explicaciones |
| Challenge (desafío) | Escenarios de señales con seeds fijos, puntuados, con ranking local |
| Sandbox | Señales generadas con seeds aleatorios para práctica libre |
| Replay | Repetir cualquier escenario con la misma seed y comparar ejecuciones |
| Modo histórico (futuro) | Escenarios inspirados en eventos reales (post-MVP, ver sección 13) |

---

## 2. Objetivos de aprendizaje (Learning Objectives)

Al completar el módulo, el usuario debe ser capaz de:

1. **Explicar** por qué una señal idéntica produce resultados distintos en cuentas distintas (spread, slippage, timing, tamaño, leverage).
2. **Calcular** si una señal sigue siendo válida cuando la ve con retraso, evaluando el risk/reward restante.
3. **Identificar** cuándo una señal carece de invalidación clara (sin stop loss definido) y rechazarla.
4. **Ajustar** el tamaño de posición y el apalancamiento a su propia cuenta, no a la del proveedor de la señal.
5. **Reconocer** el slippage y el spread como costos reales que cambian el precio de entrada y de salida.
6. **Distinguir** entre resultado y proceso: una buena operación puede perder y una mala operación puede ganar.
7. **Detectar** sus propias salidas emocionales (pánico, codicia, mover el stop) en el diario de errores.
8. **Decidir no entrar** como una decisión válida y, muchas veces, la mejor decisión.
9. **Asumir responsabilidad personal**: quien ejecuta la operación es responsable del resultado, no el proveedor de la señal.
10. **Entender el horizonte temporal**: el proveedor puede operar a días mientras el seguidor mira velas de 5 minutos.

### Términos técnicos que el módulo enseña (regla de terminología real)

| Término (en inglés, estándar) | Explicación en español claro |
|---|---|
| Spread | La diferencia entre el precio de compra (ask) y el precio de venta (bid). Es el primer costo invisible de cada operación. |
| Slippage | Recibir un precio de ejecución peor del esperado, sobre todo cuando el mercado se mueve rápido. |
| Drawdown | Cuánto cae tu cuenta desde su punto más alto. Mide el dolor acumulado, no solo una pérdida puntual. |
| Leverage (apalancamiento) | Controlar una posición más grande con menos capital. Multiplica tanto la ganancia potencial como el riesgo. |
| Risk/Reward (riesgo/beneficio) | Cuánto arriesgas comparado con cuánto buscas ganar. Una señal con R/R 1:3 vista tarde puede quedar en 1:0.5. |
| Liquidación (liquidation) | Cuando el apalancamiento es tan alto que una caída pequeña consume todo el margen y el broker cierra tu posición a la fuerza. |
| Invalidación | El nivel de precio donde la idea de la operación deja de ser válida. Si no existe, no hay plan: hay esperanza. |
| Entry delay | El retraso entre el momento en que se emitió la señal y el momento en que tú la ves y la ejecutas. |
| Time horizon (horizonte temporal) | El plazo en el que el proveedor espera que la idea funcione: minutos, horas, días o semanas. |

---

## 3. Secuencia de lecciones (Lesson Sequence)

El módulo tiene **8 lecciones** en orden pedagógico fijo. Cada lección combina explicación breve + simulación interactiva con seed fija + revisión de errores + feedback.

| # | Lección | Concepto central | Escenarios asociados (sección 6/7) |
|---|---|---|---|
| L1 | "La misma señal, dos destinos" | Una señal no garantiza nada: dos traders, mismo mensaje, resultados opuestos | E1 |
| L2 | "Tu broker no es su broker" | Precios distintos entre brokers, spread, fills diferentes | E4 |
| L3 | "Llegaste tarde" | Entry delay y destrucción del risk/reward; saltarse una señal es válido | E3, E8 |
| L4 | "El apalancamiento no se copia" | Leverage mismatch, tamaño de cuenta, liquidación | E2 |
| L5 | "Sin invalidación no hay plan" | Operar sin stop loss, mover el stop, no tener salida definida | E5 |
| L6 | "El enemigo eres tú" | Salidas emocionales, parciales mal tomadas, pánico y codicia | E6 |
| L7 | "Volatilidad: cuando el precio te esquiva" | Slippage en picos de volatilidad, noticias, spread ampliado | E7 |
| L8 | "Proceso vs. resultado" | Una buena operación puede perder; una mala puede ganar; el puntaje de proceso | E9, E10 |

**Reglas de progresión:**
- Las lecciones se desbloquean en orden (modo de progresión desbloqueable).
- Cada lección otorga XP por completarla y XP extra por decisiones de proceso correctas (no por ganar dinero simulado).
- Al terminar L8 se desbloquea el **Challenge "Cazador de señales"** (sección 10).
- Todas las lecciones usan **seeds fijas de tutorial** (ver sección 5).

**Estructura interna de cada lección (patrón uniforme):**
1. **Contexto** (1–2 pantallas conceptuales de texto, sin diseño aún): qué vas a ver y por qué importa.
2. **La señal**: el usuario recibe una señal simulada con formato realista de grupo de Telegram (activo, dirección, entrada, SL/TP si aplica).
3. **Decisión**: el usuario decide si entra, cuándo, con qué tamaño, qué leverage, dónde pone el stop.
4. **Ejecución sobre el camino pre-generado**: el mercado se reproduce; el usuario gestiona la posición.
5. **Revisión de errores**: el detector de errores (sección 7) lista lo que hizo bien y mal.
6. **Feedback y puntaje de proceso** (secciones 8 y 9).
7. **Replay opcional**: repetir con la misma seed para probar otra ejecución.

---

## 4. Contratos de Contexto de Aprendizaje (Learning Context Contracts)

Un **Learning Context Contract (LCC)** es la declaración formal de qué garantiza el escenario y qué queda en manos del usuario. Es el mecanismo que asegura que Burgundy **fuerza contexto, no resultado**.

### 4.1 Estructura estándar del LCC

Cada lección y escenario declara:

| Campo | Significado |
|---|---|
| `id_contrato` | Identificador único (ej. `LCC-WSF-L1`) |
| `concepto_objetivo` | Qué debe aprender el usuario aquí |
| `contexto_garantizado` | Qué condiciones de mercado están pre-generadas y son inmutables |
| `libertad_del_usuario` | Qué decisiones quedan totalmente en manos del usuario |
| `resultado_NO_garantizado` | Declaración explícita de que ganar o perder depende de la ejecución |
| `trampa_pedagógica` | Qué comportamiento típico de principiante hará probable un mal resultado |
| `salida_honorable` | Qué ejecución correcta (incluyendo NO entrar) produce un buen resultado educativo |
| `criterio_de_éxito` | Cómo se mide el aprendizaje (métricas de proceso, no P/L) |

### 4.2 LCC por lección

**LCC-WSF-L1 — "La misma señal, dos destinos"**
- Concepto objetivo: la señal es idéntica; la ejecución es lo que difiere.
- Contexto garantizado: camino de mercado fijo donde la señal alcanza el TP **solo si** la entrada ocurre dentro de la ventana temprana y el stop respeta la invalidación original. El camino incluye un retroceso intermedio que toca la zona donde entraría un seguidor tardío.
- Libertad del usuario: momento de entrada, tamaño, stop, salida.
- Resultado NO garantizado: entrar tarde o con stop apretado convierte la misma señal en pérdida.
- Trampa pedagógica: el retroceso intermedio barre los stops de quien entró tarde con stop pegado.
- Salida honorable: entrada puntual con stop en la invalidación original → la operación funciona como al "Trader A".
- Criterio de éxito: el usuario explica (quiz post-escenario) por qué el Trader B perdió con la misma señal.

**LCC-WSF-L2 — "Tu broker no es su broker"**
- Concepto objetivo: spread y diferencias de precio entre brokers alteran el fill (precio de ejecución).
- Contexto garantizado: el escenario simula dos condiciones de broker sobre el mismo camino: broker de spread bajo vs. broker de spread alto (parámetro del escenario, no manipulación del precio).
- Libertad del usuario: aceptar o no el fill, ajustar el TP/SL a su precio real de entrada.
- Resultado NO garantizado: con spread alto, el mismo TP nominal puede no alcanzarse nunca.
- Trampa pedagógica: copiar el TP exacto de la señal sin ajustarlo al propio precio de entrada.
- Salida honorable: recalcular TP/SL desde el fill propio, o rechazar la operación si el spread destruye el R/R.
- Criterio de éxito: el usuario recalcula el R/R real después del spread en el quiz.

**LCC-WSF-L3 — "Llegaste tarde"**
- Concepto objetivo: el entry delay destruye el risk/reward; saltarse la señal es una decisión válida y puntuable.
- Contexto garantizado: la señal se muestra cuando el precio ya recorrió ~60% del trayecto hacia el TP.
- Libertad del usuario: entrar igual, entrar con tamaño reducido, o **no entrar**.
- Resultado NO garantizado: el camino sigue hasta el TP original, pero el R/R restante es ≤ 1:0.5; un retroceso normal saca a quien entró tarde.
- Trampa pedagógica: el FOMO ("ya va a llegar al TP, todavía alcanzo").
- Salida honorable: NO entrar. El escenario otorga el puntaje de proceso más alto a quien se abstiene y lo justifica.
- Criterio de éxito: el usuario identifica el R/R restante como la razón para abstenerse.

**LCC-WSF-L4 — "El apalancamiento no se copia"**
- Concepto objetivo: la misma señal con 1x es manejable; con 20x liquida la cuenta antes de que la idea madure.
- Contexto garantizado: camino con drawdown intermedio de -4.5% desde la entrada antes de girar hacia el TP.
- Libertad del usuario: elegir leverage (1x a 50x) y tamaño.
- Resultado NO garantizado: con 20x, el drawdown de -4.5% excede el margen → liquidación forzosa **antes** del giro. Con 1x–3x, el drawdown es incómodo pero sobrevivible.
- Trampa pedagógica: la señal del "proveedor" menciona que él usa 20x.
- Salida honorable: leverage bajo + tamaño acorde al 1–2% de riesgo → la operación llega al TP.
- Criterio de éxito: el usuario explica que la liquidación ocurrió por margen, no porque "la señal estaba mal".

**LCC-WSF-L5 — "Sin invalidación no hay plan"**
- Concepto objetivo: una señal sin stop loss definido no es operable; operar sin stop expone a pérdidas abiertas.
- Contexto garantizado: la señal no incluye stop loss. El camino va en contra de forma sostenida (tendencia adversa de varias velas) antes de un rebote parcial que no recupera la entrada.
- Libertad del usuario: definir su propio stop, operar sin stop, o no entrar.
- Resultado NO garantizado: sin stop, la pérdida flotante crece hasta niveles de drawdown severo; con stop propio bien colocado, la pérdida es pequeña y controlada.
- Trampa pedagógica: "aguantar" la pérdida esperando el rebote; mover mentalmente la invalidación.
- Salida honorable: definir stop propio antes de entrar (pérdida pequeña) o rechazar la señal por no tener invalidación.
- Criterio de éxito: el usuario formula la regla "si no sé dónde estoy equivocado, no entro".

**LCC-WSF-L6 — "El enemigo eres tú"**
- Concepto objetivo: salidas emocionales, parciales demasiado tempranas o tardías, y mover el stop loss.
- Contexto garantizado: camino que va a favor, retrocede ~50% de la ganancia flotante (sin tocar el stop original), y luego continúa hasta el TP.
- Libertad del usuario: mantener, cerrar, tomar parciales, mover stop.
- Resultado NO garantizado: quien cierra en el retroceso por miedo captura una fracción mínima; quien respeta el plan llega al TP; quien mueve el stop a breakeven demasiado pronto es expulsado en el retroceso.
- Trampa pedagógica: el retroceso está diseñado (pre-generado) para ser emocionalmente incómodo pero técnicamente normal.
- Salida honorable: respetar el plan original o tomar parciales **planificadas antes de entrar**.
- Criterio de éxito: el diario registra si las salidas fueron planificadas o reactivas; el usuario lo reconoce en la revisión.

**LCC-WSF-L7 — "Volatilidad: cuando el precio te esquiva"**
- Concepto objetivo: slippage y spread ampliado durante picos de volatilidad (noticias, aperturas).
- Contexto garantizado: el camino incluye un pico de volatilidad pre-generado en el momento en que llega la señal; el modelo de ejecución aplica slippage proporcional a la velocidad de la vela (regla del motor, definida antes, no manipulación reactiva).
- Libertad del usuario: entrar a mercado durante el pico, usar orden límite, esperar a que pase el pico, o no entrar.
- Resultado NO garantizado: entrada a mercado en el pico recibe fill notablemente peor; orden límite puede no ejecutarse; esperar da peor precio nominal pero mejor fill real.
- Trampa pedagógica: la urgencia ("¡entra YA, está volando!") escrita en la propia señal.
- Salida honorable: usar orden límite o abstenerse durante el pico.
- Criterio de éxito: el usuario distingue precio cotizado vs. precio de ejecución.

**LCC-WSF-L8 — "Proceso vs. resultado"**
- Concepto objetivo: una buena operación puede perder y una mala puede ganar; Burgundy puntúa el proceso.
- Contexto garantizado: dos escenarios encadenados con seeds fijas: (a) ejecución perfecta sobre un camino que termina en pérdida; (b) ejecución pésima sobre un camino que termina en ganancia.
- Libertad del usuario: total en ambos escenarios.
- Resultado NO garantizado: explícitamente, el escenario (a) está pre-generado para que incluso la ejecución perfecta termine en stop loss.
- Trampa pedagógica: creer que el puntaje bajo del escenario (b) "es injusto porque gané".
- Salida honorable: aceptar la pérdida del escenario (a) con riesgo correcto → puntaje de proceso máximo.
- Criterio de éxito: el usuario explica con sus palabras por qué (a) puntúa más alto que (b).

---

## 5. Escenarios interactivos con seed fija (Fixed-Seed Simulator Scenarios)

### 5.1 Reglas de determinismo del módulo

- **Identidad del escenario:** `template_id + parámetros + seed + versión_del_generador` ⇒ camino de mercado idéntico, byte a byte.
- **Seeds de tutorial:** fijas y versionadas. Convención de nomenclatura (solo identificadores lógicos, no código): `WSF-T-L{lección}-S{variante}` (ej. `WSF-T-L4-S1`).
- **Seeds de challenge:** fijas por desafío y por temporada local: `WSF-C-{challenge}-S{n}`.
- **Seeds de sandbox:** aleatorias, pero registradas en el journal para permitir replay exacto.
- **Seeds de replay:** la seed original del escenario jugado; el replay nunca regenera.
- **Seeds histórico-inspiradas (futuro):** `WSF-H-{evento}-S{n}` (ver sección 13).

### 5.2 Qué cosas varían con la ejecución del usuario (y qué no)

| Cambia con el usuario | NO cambia nunca |
|---|---|
| Precio de fill (por timing + spread + slippage según reglas fijas del motor) | El camino OHLC de las velas |
| P/L, equity, margen, drawdown de la cuenta | El momento y magnitud del pico de volatilidad |
| Si hay liquidación (depende de su leverage) | El nivel donde el camino gira |
| Errores detectados y feedback | Los parámetros de spread del "broker" del escenario |
| Puntaje de proceso y XP | La señal mostrada (texto, niveles, timing de aparición) |

### 5.3 Mecánica clave: el "fantasma de replay"

En el replay de cualquier escenario de señales, Burgundy puede mostrar (como dato, no como pantalla todavía) la ejecución anterior del usuario superpuesta a la nueva. Esto materializa la tesis del módulo: **mismo mercado, distinta ejecución, distinto resultado.** Es la prueba empírica de que el mercado no cambió: cambió el usuario.

---

## 6. Ejemplos de comparación de señales (Signal Comparison Examples)

Formato realista de señal de grupo (los escenarios usan este estilo, con activo simulado genérico, ej. "BRG/USD"):

> 🔔 **SEÑAL VIP** — BRG/USD
> COMPRA en 1.0850
> TP1: 1.0910 | TP2: 1.0950
> SL: 1.0820
> "¡Entren rápido, esto vuela! 🚀"

### 6.1 Tabla comparativa: misma señal, cinco seguidores

| Seguidor | Entrada real | Leverage | Stop | Comportamiento | Resultado | Puntaje de proceso |
|---|---|---|---|---|---|---|
| A (puntual, disciplinado) | 1.0851 (spread bajo) | 2x | 1.0820 | Respeta plan | +R/R 1:2 ✔ | Alto |
| B (tardío) | 1.0895 | 2x | 1.0875 (stop "ajustado") | FOMO | Barrido en retroceso ✘ | Bajo |
| C (apalancado) | 1.0852 | 20x | sin stop | Copia "como el VIP" | Liquidado en drawdown intermedio ✘ | Muy bajo |
| D (emocional) | 1.0853 | 2x | 1.0820 | Cierra en el retroceso a 1.0865 | +mínimo, pierde el TP | Medio-bajo |
| E (se abstiene) | — | — | — | Ve la señal tarde, calcula R/R 1:0.4, no entra | 0 P/L | **Máximo** |

Lectura pedagógica: la señal "ganó" (llegó al TP), pero solo el seguidor A capturó el movimiento completo, y el mejor puntaje fue para E, que no operó. **El P/L y el puntaje de proceso son ejes independientes.**

### 6.2 Comparación de calidad de señal (qué hace operable una señal)

| Criterio | Señal operable | Señal no operable |
|---|---|---|
| Invalidación | SL definido y lógico | "Sin SL, confíen" |
| Horizonte | Plazo declarado (intradía/swing) | Sin plazo |
| Entrada | Zona de entrada, no un punto vencido | "COMPRA YA" sin nivel |
| R/R | Calculable (≥ 1:1.5 al momento de tu entrada) | Imposible de calcular o ya destruido |
| Contexto | Razón de la idea (aunque breve) | Solo emojis y urgencia |

---

## 7. Escenarios requeridos — definición completa

Cada escenario define: template, LCC, comportamiento de seed, comportamiento del camino, detalles de la señal, buenas decisiones, malas decisiones, resultados posibles, lógica de feedback y métricas.

---

### E1 — "La misma señal gana para uno y pierde para otro"

| Campo | Definición |
|---|---|
| **Template** | `WSF-E1-dual-outcome` — escenario en dos pasadas sobre el mismo camino: el usuario lo juega como "Trader A" (en tiempo) y luego como "Trader B" (la app introduce el retraso). |
| **LCC** | LCC-WSF-L1 (sección 4.2). |
| **Seed** | Fija: `WSF-T-L1-S1`. Misma seed en ambas pasadas — esa es la gracia. |
| **Camino de mercado** | Sube hacia el TP con un retroceso intermedio que perfora la zona de entrada tardía + stops apretados, sin tocar el SL original. Luego alcanza TP1. |
| **Señal** | Compra con entrada, TP1/TP2 y SL definidos (formato 6.1). Aparece en t0 para la pasada A y en t0+15 velas para la pasada B. |
| **Buenas decisiones** | Entrar cerca de la entrada original; SL en la invalidación original; tamaño 1–2% de riesgo; respetar el plan. |
| **Malas decisiones** | Entrar tarde con SL apretado "para compensar"; agrandar tamaño para "recuperar" el trayecto perdido. |
| **Resultados posibles** | A: TP alcanzado. B: stop barrido en el retroceso; o B se abstiene (mejor puntaje de proceso de la pasada B). |
| **Feedback** | Si B pierde: "El mercado fue el mismo. Lo que cambió fue tu entrada. Mira el replay: tu stop quedó dentro del ruido normal del movimiento." |
| **Métricas** | R/R al momento de entrada de cada pasada; distancia del SL a la invalidación original; decisión de abstención. |

---

### E2 — "1x es manejable, 20x liquida"

| Campo | Definición |
|---|---|
| **Template** | `WSF-E2-leverage-mismatch`. |
| **LCC** | LCC-WSF-L4. |
| **Seed** | Fija: `WSF-T-L4-S1`. |
| **Camino** | Entrada → drawdown sostenido de -4.5% (varias velas bajistas `#802F3E` con rebotes débiles) → giro → TP a +6%. Todo pre-generado. |
| **Señal** | Incluye la frase trampa: "yo entro con 20x 😎". Niveles correctos. |
| **Buenas decisiones** | Leverage ≤ 3x; riesgo por operación ≤ 2%; SL en invalidación; tolerar el drawdown dentro del plan. |
| **Malas decisiones** | Copiar el 20x; no calcular el precio de liquidación; promediar en contra para "bajar el precio medio". |
| **Resultados posibles** | Con 1x–3x: drawdown incómodo, luego TP. Con 10x: stop o liquidación según margen. Con 20x: **liquidación garantizada por matemática de margen** antes del giro (no por manipulación: el -4.5% excede el margen disponible a 20x). |
| **Feedback** | Tras liquidación: "No te sacó el mercado: te sacó tu margen. La idea de la señal terminó siendo correcta — mira el replay — pero a 20x no tenías derecho a equivocarte ni un 5%." |
| **Métricas** | Leverage elegido vs. máximo sobrevivible; % de margen consumido en el peor punto; distancia a liquidación al entrar. |

---

### E3 — "La entrada tardía arruina el risk/reward"

| Campo | Definición |
|---|---|
| **Template** | `WSF-E3-late-entry`. |
| **LCC** | LCC-WSF-L3. |
| **Seed** | Fija: `WSF-T-L3-S1`. |
| **Camino** | La señal se emitió "hace 40 minutos" (narrativa del escenario); al mostrarse al usuario, el precio ya recorrió 60% hacia TP1. Luego: retroceso normal de 1/3 del avance y avance final a TP1. |
| **Señal** | Original con R/R 1:3. Al momento de verla el usuario, el R/R real restante es ≈ 1:0.45 (la app muestra el dato si el usuario abre la calculadora de R/R — herramienta del escenario). |
| **Buenas decisiones** | Calcular el R/R restante; abstenerse; o si entra, hacerlo con tamaño reducido aceptando el R/R real (decisión informada, penaliza menos que entrar a ciegas). |
| **Malas decisiones** | Entrar a mercado por FOMO con tamaño normal y el TP/SL originales. |
| **Resultados posibles** | Quien no entra: 0 P/L, puntaje máximo. Quien entra tarde: el retroceso normal lo deja en pérdida flotante; puede terminar en SL o en un TP minúsculo con riesgo desproporcionado. |
| **Feedback** | "La señal era buena hace 40 minutos. Cuando tú la viste, ya era otra operación: arriesgabas 1 para ganar 0.45. ¿La habrías tomado si te la presentaban así?" |
| **Métricas** | R/R al momento de la decisión; uso de la calculadora de R/R; abstención. |

---

### E4 — "El spread causa un mal fill"

| Campo | Definición |
|---|---|
| **Template** | `WSF-E4-spread-fill`. |
| **LCC** | LCC-WSF-L2. |
| **Seed** | Fija: `WSF-T-L2-S1`. Se juega dos veces: parámetro `broker_spread = bajo` y `broker_spread = alto` (mismo camino). |
| **Camino** | Movimiento que alcanza el TP nominal de la señal **por 2 pips** y retrocede al SL. |
| **Señal** | Compra con TP justo (diseñado para que el spread sea la diferencia entre ganar y perder). |
| **Buenas decisiones** | Verificar el spread antes de entrar; ajustar el TP al propio fill; rechazar la operación con spread alto. |
| **Malas decisiones** | Copiar TP/SL textuales sin ajustar; ignorar que el fill fue peor que la entrada de la señal. |
| **Resultados posibles** | Spread bajo: TP alcanzado. Spread alto: el TP propio queda 2 pips por encima del máximo del camino → la posición retrocede hasta el SL. Mismo camino, resultado opuesto. |
| **Feedback** | "Tu TP nunca fue tocado porque tu precio de entrada no era el de la señal. El proveedor ganó; tú perdiste. Ninguno mintió: tienen brokers distintos." |
| **Métricas** | Diferencia entrada-señal vs. entrada-real; ajuste (o no) de TP/SL al fill propio. |

---

### E5 — "Sin stop loss, pérdida severa"

| Campo | Definición |
|---|---|
| **Template** | `WSF-E5-no-stop`. |
| **LCC** | LCC-WSF-L5. |
| **Seed** | Fija: `WSF-T-L5-S1`. |
| **Camino** | Tendencia adversa sostenida: -2%, rebote débil, -5%, rebote débil, -9% desde la entrada. El rebote nunca recupera la entrada. Fin del escenario con la posición aún abierta en -9% (si el usuario no cerró). |
| **Señal** | Sin SL: "compren y aguanten, esto rebota 💪". |
| **Buenas decisiones** | Definir SL propio antes de entrar (pérdida ≈ -1.5%); o no entrar por falta de invalidación. |
| **Malas decisiones** | Entrar sin stop; "aguantar"; promediar en contra; mover el stop hacia abajo cuando se acerca. |
| **Resultados posibles** | Con SL propio: pérdida pequeña controlada (¡y puntaje de proceso alto a pesar de perder!). Sin SL: drawdown severo y lección de pérdida abierta. Abstención: puntaje máximo. |
| **Feedback** | Sin stop: "Nadie te dijo dónde estaba equivocada esta idea — y tú tampoco lo definiste. Sin invalidación, tu pérdida no tenía piso. El stop loss no es pesimismo: es saber dónde te equivocas." Si movió el stop: "Mover el stop no protege la operación: protege tu ego, y cuesta dinero." |
| **Métricas** | Existencia de SL al entrar; drawdown máximo de la posición; si movió el SL en contra. |

---

### E6 — "Salida emocional antes del objetivo"

| Campo | Definición |
|---|---|
| **Template** | `WSF-E6-emotional-exit`. |
| **LCC** | LCC-WSF-L6. |
| **Seed** | Fija: `WSF-T-L6-S1`. |
| **Camino** | A favor +1.8%, retroceso a +0.4% (sin tocar el SL ni breakeven con el SL original), continuación hasta TP en +3.5%. El retroceso es deliberadamente lento e incómodo (varias velas `#802F3E` consecutivas pequeñas). |
| **Señal** | Completa y correcta, con SL y TP razonables. |
| **Buenas decisiones** | Plan de salida definido **antes** de entrar (TP completo o parciales pre-declaradas); respetarlo durante el retroceso. |
| **Malas decisiones** | Cerrar todo en el retroceso; mover el stop a breakeven inmediatamente (es expulsado en el retroceso si lo hace demasiado pronto); tomar una parcial enorme por miedo. |
| **Resultados posibles** | Plan respetado: TP completo. Salida en pánico: ganancia mínima + frustración de ver llegar el TP. Stop a breakeven prematuro: salida en 0 viendo el TP después. |
| **Feedback** | "Saliste en +0.4% y el plan llegaba a +3.5%. El mercado no te sacó: te sacaste tú. Pregunta del diario: ¿qué viste en ese retroceso que invalidara la idea? Si la respuesta es 'nada, tuve miedo', anótalo: ese es el error a entrenar." |
| **Métricas** | Salidas planificadas vs. reactivas; % del movimiento capturado vs. disponible; cambios de SL/TP durante la operación. |

---

### E7 — "Copiar la señal en alta volatilidad: slippage"

| Campo | Definición |
|---|---|
| **Template** | `WSF-E7-volatility-slippage`. |
| **LCC** | LCC-WSF-L7. |
| **Seed** | Fija: `WSF-T-L7-S1`. |
| **Camino** | Pico de volatilidad pre-generado (vela de rango 5x el promedio, mechas largas) exactamente cuando aparece la señal; después, estabilización y movimiento moderado a favor. |
| **Señal** | "¡¡ENTRA YA, ESTÁ VOLANDO!! 🚀🚀" — urgencia explícita como trampa pedagógica. |
| **Buenas decisiones** | Orden límite; esperar el cierre del pico; reducir tamaño por volatilidad; no entrar. |
| **Malas decisiones** | Orden a mercado dentro del pico (regla fija del motor: slippage proporcional a la velocidad de la vela → fill muy desfavorable); SL dentro del rango de las mechas (barrido por ruido). |
| **Resultados posibles** | Orden a mercado: fill malo + posible barrido del SL por la mecha → pérdida aunque la dirección era correcta. Límite/espera: entrada decente, operación moderadamente ganadora. |
| **Feedback** | "Compraste a un precio que solo existió un instante. El slippage no es un castigo del simulador: es lo que pasa cuando exiges ejecución inmediata en un mercado que se mueve más rápido que tú." |
| **Métricas** | Diferencia precio-solicitado vs. precio-fill; tipo de orden usada; colocación del SL respecto al rango del pico. |

---

### E8 — "Saltarse la señal correcta y tardía: la mejor operación es no operar"

| Campo | Definición |
|---|---|
| **Template** | `WSF-E8-skip-signal`. |
| **LCC** | LCC-WSF-L3 (variante de abstención). |
| **Seed** | Fija: `WSF-T-L3-S2`. |
| **Camino** | El precio ya está a 85% del trayecto al TP cuando aparece la señal. Después: toca TP por margen mínimo y revierte con fuerza muy por debajo de la entrada tardía. |
| **Señal** | Técnicamente "ganadora" (tocó TP): la trampa es que el usuario tardío que entró NO gana, porque su entrada está demasiado arriba y la reversión lo atrapa. |
| **Buenas decisiones** | Calcular R/R restante (≈ 1:0.15); **no entrar**; registrar en el diario por qué se abstuvo. |
| **Malas decisiones** | Entrar "porque la señal va ganando"; perseguir el precio. |
| **Resultados posibles** | Abstención: 0 P/L + puntaje de proceso máximo + XP de disciplina. Entrada tardía: el toque de TP no alcanza su TP ajustado (o lo roza sin ejecutar por spread) y la reversión lo lleva a pérdida. |
| **Feedback** | Si se abstuvo: "El grupo celebrará esta señal como ganadora. Tú sabes la verdad: para ti, ya no existía. No operar fue tu mejor operación de hoy." |
| **Métricas** | Abstención justificada (con registro en diario); R/R calculado antes de decidir. |

---

### E9 — "Riesgo correcto, pérdida igual: puntaje de proceso alto"

| Campo | Definición |
|---|---|
| **Template** | `WSF-E9-good-process-loss`. |
| **LCC** | LCC-WSF-L8(a). |
| **Seed** | Fija: `WSF-T-L8-S1`. |
| **Camino** | Pre-generado para terminar en el SL de la señal **sin importar la calidad de la entrada**: avance breve a favor (+0.5%), luego caída directa al nivel de invalidación. |
| **Señal** | De buena calidad: entrada, SL lógico, TP realista, R/R 1:2, horizonte declarado. |
| **Buenas decisiones** | Entrar con riesgo 1–2%, SL en la invalidación, sin moverlo, aceptar el stop cuando llega. |
| **Malas decisiones** | Mover el stop para "darle aire"; promediar; convertir la pérdida pequeña en grande. |
| **Resultados posibles** | Ejecución correcta: pérdida de -1% a -2% y **puntaje de proceso alto** (este es el corazón del escenario). Ejecución incorrecta: pérdida amplificada y puntaje bajo. |
| **Feedback** | "Perdiste 1.5% e hiciste todo bien. Léelo otra vez: ambas cosas son ciertas a la vez. Una buena operación puede perder. Si repites este proceso 100 veces con señales de esta calidad, las matemáticas trabajan para ti. Si lo rompes para evitar esta pérdida, trabajan en tu contra." |
| **Métricas** | % de riesgo arriesgado; integridad del plan (SL intacto); reacción post-pérdida (quiz del diario). |

---

### E10 — "Riesgo terrible, ganancia igual: puntaje de proceso bajo"

| Campo | Definición |
|---|---|
| **Template** | `WSF-E10-bad-process-win`. |
| **LCC** | LCC-WSF-L8(b). |
| **Seed** | Fija: `WSF-T-L8-S2`. |
| **Camino** | Pre-generado para terminar a favor: drawdown inicial fuerte (-6%) que NO toca liquidación a leverage medio, y luego reversión potente hasta +8%. |
| **Señal** | Mala: sin SL, sin horizonte, pura urgencia. |
| **Buenas decisiones** | (Paradoja intencional) La decisión de proceso correcta era no entrar o entrar con SL propio — lo que en este camino produce pérdida pequeña o ganancia menor. |
| **Malas decisiones** | Entrar sin stop con tamaño grande... que en ESTE camino, gana mucho. |
| **Resultados posibles** | El temerario gana +8% con puntaje de proceso **muy bajo** y una advertencia de primer nivel. El disciplinado gana poco o pierde poco con puntaje alto. |
| **Feedback** | "Ganaste 8%. Tu puntaje de proceso: 12/100. Esto no es un error de Burgundy. Hiciste una apuesta sin invalidación y sobreviviste al -6% por suerte, no por plan. El mercado acaba de pagarte por practicar el hábito que tarde o temprano vacía cuentas. La ganancia de hoy es el anzuelo de la pérdida de mañana." |
| **Métricas** | Puntaje de proceso vs. P/L (divergencia explícita y mostrada); drawdown soportado sin plan; registro de la reflexión en el diario. |

---

## 8. Reglas de detección de errores (Mistake Detection Rules)

El detector de errores evalúa **solo decisiones del usuario contra reglas objetivas**, nunca contra el resultado. Reglas del módulo:

| ID | Regla | Condición de disparo | Severidad |
|---|---|---|---|
| MD-01 | Entrada sin stop loss | Orden abierta sin SL definido | Crítica |
| MD-02 | Entrada tardía con R/R destruido | R/R al momento del fill < 1:1 cuando la señal original era ≥ 1:2 | Alta |
| MD-03 | Leverage excesivo | Leverage tal que un movimiento adverso ≤ 5% causa liquidación | Crítica |
| MD-04 | Riesgo por operación excesivo | Riesgo > 3% del equity en una operación | Alta |
| MD-05 | Stop loss movido en contra | SL desplazado alejándolo del precio tras la entrada | Crítica |
| MD-06 | Stop a breakeven prematuro | SL movido a entrada antes de que el precio avance ≥ 1R | Media |
| MD-07 | Salida emocional (pánico) | Cierre total en retroceso < 50% del avance, sin toque de SL, sin parcial planificada | Alta |
| MD-08 | Salida emocional (codicia) | TP alcanzado pero la orden de TP fue cancelada/alejada durante la operación | Alta |
| MD-09 | Parcial no planificada | Toma de parcial sin plan de parciales declarado antes de entrar | Media |
| MD-10 | Orden a mercado en pico de volatilidad | Market order durante vela con rango > 3x el promedio del escenario | Media |
| MD-11 | TP/SL copiados sin ajustar al fill propio | TP/SL textuales de la señal cuando el fill difiere > umbral del template | Media |
| MD-12 | Promediar en contra sin plan | Nueva orden en la misma dirección con posición en pérdida y sin plan declarado | Crítica |
| MD-13 | FOMO de persecución | Entrada a mercado cuando el precio está > X% por encima de la entrada de la señal (X por template) | Alta |
| MD-14 | Confianza ciega | Entrada en señal sin SL/horizonte sin definir invalidación propia (combinación MD-01 + señal incompleta) | Crítica |
| MD-15 | Ignorar la calculadora de R/R | Entrada en escenario de retraso sin consultar el R/R restante (solo lecciones donde la herramienta existe) | Baja |

**Detecciones positivas (también se registran, suman XP):**

| ID | Conducta correcta |
|---|---|
| PD-01 | Abstención justificada (no entrar + razón registrada en el diario) |
| PD-02 | SL propio definido ante señal sin invalidación |
| PD-03 | TP/SL recalculados desde el fill propio |
| PD-04 | Riesgo ≤ 2% sostenido en toda la sesión |
| PD-05 | Plan de salida declarado antes de entrar y respetado |
| PD-06 | Orden límite o espera durante pico de volatilidad |
| PD-07 | Aceptar el stop sin moverlo (incluso si la operación pierde) |

---

## 9. Mensajes de feedback (Feedback Messages)

### 9.1 Principios de tono

- Directo, serio, sin burla y sin falsa motivación.
- Siempre conecta la consecuencia con la **decisión**, nunca con la suerte ni con "el mercado malvado".
- Nunca dice "debiste ganar". Nunca promete que el buen proceso gana siempre.
- Cierra con una pregunta o regla accionable cuando sea posible.
- En la interfaz futura, los mensajes críticos usan el resaltado `#C9A050` sobre superficies `#2E2E2E` (anotación de diseño, no pantalla).

### 9.2 Biblioteca de mensajes por categoría (ejemplos canónicos)

**Sobre el retraso (MD-02, MD-13):**
> "Cuando viste esta señal, ya era otra operación. El proveedor arriesgaba 1 para ganar 3. Tú arriesgabas 1 para ganar 0.4. Mismo mensaje, matemática opuesta."

**Sobre el stop loss (MD-01, MD-14):**
> "El stop loss no es una opinión pesimista: es la respuesta a la pregunta '¿dónde estoy equivocado?'. Si la señal no la responde y tú tampoco, no tienes una operación. Tienes una esperanza con dinero encima."

**Sobre mover el stop (MD-05):**
> "Moviste tu stop 2 veces. Cada vez compraste unos minutos de alivio a cambio de una pérdida mayor. El stop se decide antes de entrar, cuando todavía piensas con claridad."

**Sobre el leverage (MD-03):**
> "A 20x, una caída de 4.5% no es un mal rato: es el final de la cuenta. El proveedor de la señal puede usar 20x con su capital y su tolerancia. Tú copiaste su apalancamiento, no su cuenta."

**Sobre la salida emocional (MD-07):**
> "Cerraste en +0.4%. El plan llegaba a +3.5% y el precio llegó. ¿Qué cambió en el mercado cuando saliste? Si la respuesta es 'nada, tuve miedo', acabas de encontrar tu próximo entrenamiento."

**Sobre el slippage (MD-10):**
> "Pediste 'a mercado' y el mercado te dio lo que había: un precio peor. En velas rápidas, la urgencia se paga. La orden límite es tu manera de decir: a este precio sí, a otro no."

**Sobre la abstención (PD-01):**
> "No entraste, y esa fue la decisión con mejor puntaje de hoy. En los grupos nadie presume las señales que no tomó. En Burgundy, sí cuentan."

**Sobre perder bien (E9 / PD-07):**
> "Perdiste 1.5% con un proceso de 92/100. Guarda esta operación en tu diario: así se ve perder bien. Repetir esto no te garantiza ganar la próxima — te garantiza seguir vivo para cuando las probabilidades paguen."

**Sobre ganar mal (E10):**
> "Ganaste, y este puntaje bajo te va a molestar. Bien: que te moleste. La suerte te pagó por un hábito que cobra con cuentas enteras. Si no distingues esta ganancia de una buena operación, el mercado te lo explicará más caro."

**Sobre la responsabilidad (cierre de módulo):**
> "El proveedor escribió un mensaje. Tú pusiste la entrada, el tamaño, el stop, la salida y el dinero. La señal nunca fue la operación: la operación siempre fuiste tú."

### 9.3 Estructura técnica del feedback (para el futuro motor)

Cada mensaje se asocia a: `regla_disparadora (MD/PD)`, `escenario`, `severidad`, `momento de entrega` (inmediato discreto durante la operación / completo en la revisión post-escenario), y `entrada de diario sugerida`. Los mensajes críticos nunca interrumpen la simulación en curso: se entregan en la revisión, para no convertirse en ayuda dinámica que altere el experimento.

---

## 10. Métricas de evaluación (Evaluation Metrics)

### 10.1 Puntaje de Proceso (Process Score, 0–100) — métrica estrella del módulo

| Componente | Peso | Qué mide |
|---|---|---|
| Gestión de riesgo | 30% | % arriesgado, leverage vs. sobrevivible, distancia a liquidación |
| Invalidación | 25% | SL definido antes de entrar, lógico, no movido en contra |
| Calidad de entrada | 20% | R/R al momento del fill, no perseguir precio, tipo de orden adecuado a la volatilidad |
| Disciplina de salida | 15% | Plan declarado y respetado; parciales planificadas |
| Juicio de selección | 10% | Operar señales operables; abstenerse de las destruidas (PD-01 puntúa aquí al máximo) |

**El P/L no es componente del Process Score.** Se muestra al lado, deliberadamente, para entrenar la disociación resultado/proceso.

> **Integración cerrada por `SCORING_V1_LOCK` (documento 11):** el Process Score **no es un sistema de scoring paralelo**. Se implementa como el `scoringProfile` `senales_proceso` sobre las 7 categorías del sistema único de evaluación (Gestión de riesgo 30 → Gestión de riesgo · Invalidación 25 → Disciplina · Calidad de entrada 20 → Calidad de proceso · Disciplina de salida 15 → Disciplina + Consistencia · Juicio de selección 10 → Paciencia · Rentabilidad 0). Las fórmulas, thresholds y predicados son los del lock; este documento solo define el contenido pedagógico. Ante contradicción, gana el lock.

### 10.2 Métricas secundarias del módulo

- **Tasa de abstención justificada:** señales rechazadas con razón válida / señales con R/R destruido presentadas.
- **Brecha proceso-resultado:** divergencia entre Process Score y P/L por sesión (visualiza E9/E10 en la práctica del usuario).
- **Índice de salidas reactivas:** salidas no planificadas / salidas totales.
- **Supervivencia:** % de escenarios terminados sin liquidación.
- **Costo de ejecución acumulado:** spread + slippage pagados vs. el fill ideal de cada escenario (hace visibles los costos invisibles).
- **Errores críticos por sesión:** conteo de MD-01/03/05/12/14 (meta a largo plazo: cero).

### 10.3 Integración con la gamificación seria

- XP por lección completada + XP por PD detectadas. **Nunca XP por P/L positivo.**
- Racha diaria: practicar un escenario de señales al día mantiene la racha.
- Desbloqueos: completar L1–L8 desbloquea los challenges; mantener errores críticos en cero por N sesiones desbloquea variantes avanzadas.
- High scores y rankings locales por Process Score, no por dinero ganado.

---

## 11. Variantes de modo Challenge (Challenge Mode Variants)

Todos con **seeds fijas de challenge** (`WSF-C-*`), puntuación por Process Score y ranking local.

| Challenge | Mecánica | Seed | Métrica de victoria |
|---|---|---|---|
| **"Cazador de señales"** | 10 señales seguidas de calidad mixta (operables, destruidas, sin SL, tardías). El usuario decide cuáles tomar y cómo. | `WSF-C-HUNTER-S1` | Process Score promedio; bonus por abstenciones correctas |
| **"El grupo VIP"** | Sesión que recrea un día de grupo de Telegram: 6 señales con urgencia, emojis y presión social simulada en el texto. | `WSF-C-VIP-S1` | Terminar con ≥ 0 errores críticos |
| **"Sobrevive la semana"** | Challenge de supervivencia: 15 señales, cuenta inicial fija; el objetivo no es maximizar, es terminar con drawdown < 10%. | `WSF-C-SURV-S1` | Supervivencia + drawdown final |
| **"Disciplina de hierro"** | Challenge de disciplina de riesgo: cualquier operación con riesgo > 2% o sin SL termina el challenge al instante. | `WSF-C-IRON-S1` | Señales procesadas antes de fallar (estilo high score) |
| **"Réplica exacta"** | Replay competitivo: el usuario juega la misma seed 3 veces y debe mejorar su Process Score en cada pasada. | seed del propio historial | Mejora monótona del Process Score |
| **"Sin indicadores"** | Variante no-indicators: las señales llegan, pero el chart no muestra ningún indicador; solo precio y la calculadora de R/R. | `WSF-C-NAKED-S1` | Process Score promedio |

Notas de diseño:
- Los challenges reutilizan los templates E1–E10 con seeds distintas a las del tutorial (para que el conocimiento transfiera, no la memoria del camino).
- "El grupo VIP" es la síntesis del módulo: presión social + urgencia + señales mixtas, todo pre-generado.

---

## 12. Versión MVP del módulo

> **Alcance cerrado por `MVP_CONTENT_LOCK` (documento 12, sección 3).** Las 4 lecciones MVP de este módulo se empaquetan en las lecciones M5.1 (L1 + L3) y M5.2 (L4 + L5) del MVP. El challenge "Cazador de señales" se integra como mini-desafío dentro de M5 y **no** amplía el set canónico de 6 challenges del MVP. La lección L4 (apalancamiento) es **conceptual**: el leverage no es mecánica jugable del MVP y su escenario corre con parámetros fijos empaquetados (`LEVERAGE_MVP_LIMITS`, documento 12, sección 5). Ante contradicción entre este documento y esos locks, ganan los locks.

Para el MVP de Burgundy, el módulo se reduce a lo esencial sin perder la tesis:

**Incluye:**
1. **4 lecciones** (no 8): L1 (misma señal, dos destinos), L3 (llegaste tarde), L4 (apalancamiento), L5 (sin invalidación). Cubren las cuatro causas de fallo más frecuentes en LATAM.
2. **6 escenarios**: E1, E2, E3, E5, E8, E9 (los imprescindibles para la tesis proceso/resultado y abstención).
3. **Detector de errores reducido**: MD-01, MD-02, MD-03, MD-05, MD-07 + PD-01, PD-02, PD-07.
4. **Process Score completo** (la fórmula 10.1 entera, implementada como `scoringProfile` `senales_proceso` del sistema único de scoring — `SCORING_V1_LOCK`, documento 11).
5. **1 challenge**: "Cazador de señales" con una sola seed.
6. **Replay básico**: repetir escenario con la misma seed (sin fantasma superpuesto).
7. Feedback en revisión post-escenario (sin feedback en tiempo real).
8. Seeds fijas versionadas y registro de seed en el journal (obligatorio desde el día 1: el determinismo no es opcional).

**Excluye del MVP (se difiere):**
- L2, L6, L7, L8 como lecciones formales (sus conceptos aparecen mencionados en el feedback).
- E4, E6, E7, E10 como escenarios jugables.
- Fantasma de replay superpuesto.
- Challenges "VIP", "Sobrevive la semana", "Disciplina de hierro", "Réplica exacta", "Sin indicadores".
- Calculadora de R/R como herramienta interactiva (en MVP, el dato se muestra directamente en E3/E8).
- Métricas secundarias 10.2 (solo Process Score + errores críticos en MVP).
- Todo el modo histórico-inspirado.

**Criterio de la reducción:** el MVP debe poder producir el momento "ajá" central — *la misma señal con distinta ejecución da distinto resultado, y se puede perder bien y ganar mal* — con el mínimo de contenido. E1 + E9 logran eso por sí solos; el resto del MVP lo refuerza.

---

## 13. Versión avanzada (fases posteriores)

Una vez validado el MVP:

1. **Lecciones completas L1–L8** y los 10 escenarios E1–E10.
2. **Fantasma de replay**: superposición de ejecuciones pasadas sobre la misma seed.
3. **Los 6 challenges** de la sección 11, con temporadas locales y rankings.
4. **Detector de errores completo** (MD-01 a MD-15, PD-01 a PD-07) con historial de patrones: "has cometido salida emocional en 4 de tus últimas 6 operaciones".
5. **Perfil de fallo personal**: el diario agrega los errores del usuario y el módulo recomienda qué lección repetir (sin IA coach: reglas determinísticas sobre los contadores de errores).
6. **Horizonte temporal como mecánica**: escenarios donde la misma señal se juega en dos marcos temporales (el del proveedor y el del seguidor) para enseñar el time horizon mismatch de forma jugable.
7. **Modo "señal incompleta"**: el usuario aprende a completar señales (definir SL, horizonte y tamaño propios) y el sistema evalúa la calidad de su plan, no solo su ejecución.
8. **Variantes de spread/slippage por "perfil de broker"** seleccionable en sandbox, para que el usuario experimente cómo el mismo plan rinde con distintos costos de ejecución.
9. **Exportación enriquecida**: el archivo local de progreso exportable incluye seeds, decisiones y Process Scores, de modo que al importar en otro dispositivo el historial de replay sigue funcionando (coherente con offline-first y sin cuentas).

---

## 14. Escenarios histórico-inspirados de señales (futuro)

El modo histórico **no** es requisito del MVP. Cuando llegue, así se construirían escenarios de señales inspirados en eventos reales:

### 14.1 Principio

No se reproducen datos históricos crudos ni se nombra a proveedores de señales reales. Se crean **templates histórico-inspirados**: caminos generados que imitan la estructura estadística y narrativa de un episodio real conocido (volatilidad, gaps, velocidad, magnitud), con seed fija y versionada (`WSF-H-{evento}-S{n}`).

### 14.2 Proceso de creación (pipeline editorial, no técnico todavía)

1. **Selección del episodio**: eventos donde el copiado de señales fue masivo y dañino (p. ej., colapsos de criptomonedas con liquidaciones en cascada, anuncios de bancos centrales con picos de slippage, memecoins virales impulsadas por grupos).
2. **Extracción de la estructura**: qué hizo fallar a los seguidores — gap de apertura, spread ampliado x10, imposibilidad de salir, liquidaciones en cadena.
3. **Parametrización del template**: convertir esa estructura en parámetros del generador (volatilidad por tramo, momento del shock, profundidad del drawdown) sin usar la serie de precios real.
4. **Asignación de seed histórica fija**: para que toda la comunidad de usuarios juegue exactamente el mismo escenario y los rankings locales sean comparables.
5. **Capa narrativa educativa**: contexto del episodio en español claro ("en eventos como este, miles de seguidores de señales fueron liquidados en minutos"), siempre sin consejo financiero y sin glorificar ni demonizar.
6. **Validación pedagógica**: el escenario debe pasar el mismo estándar LCC — forzar contexto, no resultado; la abstención y la buena ejecución deben seguir siendo salidas honorables incluso dentro del desastre histórico.

### 14.3 Requisito de compatibilidad

El motor de seeds del MVP debe reservar desde ya el espacio de nombres `WSF-H-*` y el campo `versión_del_generador` en el formato de exportación, para que los escenarios históricos futuros no rompan la reproducibilidad de los antiguos. **Historical-ready, no historical-dependent.**

---

## 15. Cierre

Este módulo es la pieza de Burgundy que habla más directo al daño real que sufren los principiantes LATAM. Su mensaje final, repetido en cada lección con evidencia simulada y reproducible:

> **Una señal es una idea ajena. La operación, la pérdida y el aprendizaje son siempre tuyos.**

---

*Documento de especificación del proyecto Burgundy — firmado por tsuloid. Sin código, sin pantallas, sin desarrollo: solo arquitectura educativa.*
