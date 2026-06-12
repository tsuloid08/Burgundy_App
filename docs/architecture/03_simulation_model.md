# Burgundy — Modelo de Simulación de Mercado

**Documento:** 03_simulation_model.md
**Proyecto:** Burgundy — Simulador educativo de trading
**Autor / Firma:** tsuloid
**Idioma:** Español (LATAM)
**Estado:** Especificación del modelo de simulación (Simulation Model Lock)
**Alcance de este documento:** Solo lógica de simulación y realismo educativo. No incluye código de la app, ni diseño de pantallas, ni esquemas finales de base de datos.

---

## 0. Filosofía crítica de simulación

Antes de cualquier mecánica, Burgundy fija una regla inquebrantable:

> **El simulador fuerza un contexto de aprendizaje, nunca fuerza un resultado de trading.**

Esto significa:

- El **camino del mercado (market path) se genera completo antes de que el usuario actúe**.
- La app **nunca manipula el precio después de la entrada del usuario**. El mercado no "persigue" el stop loss del usuario ni "regala" ganancias.
- Las decisiones del usuario afectan únicamente: órdenes, posiciones, cuenta, riesgo, P/L, detección de errores, diario (journal) y puntaje.
- El camino de mercado generado permanece **independiente de las acciones del usuario**, siempre.

Esta independencia es la base de la confianza pedagógica: si el usuario pierde, pierde por sus decisiones dentro de un mercado justo y verificable, no porque "la app hizo trampa". Y si gana, ganó contra un mercado que no fue suavizado para él.

---

## 1. Arquitectura del motor: separación de capas

El motor de simulación de Burgundy se compone de **16 capas separadas**, cada una con responsabilidad única. La separación es obligatoria: ninguna capa posterior puede modificar el resultado de una anterior.

| # | Capa | Responsabilidad | ¿Determinista? |
|---|---|---|---|
| 1 | **Learning Context Contract (LCC)** | Define QUÉ debe aprender el usuario en esta sesión (ej. "respetar stop loss en tendencia bajista"). Es el contrato pedagógico. | Sí (es un documento de configuración) |
| 2 | **Scenario Template (plantilla de escenario)** | Define la estructura del escenario: duración, instrumento, secuencia esperada de regímenes, eventos obligatorios, checkpoints de decisión. | Sí |
| 3 | **Market Type (tipo de mercado)** | Forex sintético, índice sintético, cripto sintético, acción sintética. Define rangos de precio, horarios, gaps posibles. | Sí |
| 4 | **Difficulty (dificultad)** | Escala parámetros: volatilidad, frecuencia de fakeouts, ruido, severidad de eventos. | Sí |
| 5 | **Regime Sequence (secuencia de regímenes)** | Lista ordenada de regímenes de mercado (tendencia, rango, ruptura, reversión) con duraciones. | Derivada de plantilla + seed |
| 6 | **Volatility Profile (perfil de volatilidad)** | Curva de volatilidad a lo largo de la sesión (baja, media, alta, clusters). | Derivada de plantilla + seed |
| 7 | **Liquidity Profile (perfil de liquidez)** | Curva de liquidez: qué tan fácil es entrar/salir sin mover el precio. | Derivada de plantilla + seed |
| 8 | **Spread Profile (perfil de spread)** | Spread base + dinámica del spread según volatilidad/liquidez/eventos. | Derivada de plantilla + seed |
| 9 | **Event Schedule (calendario de eventos)** | Lista de shocks programados (tipo "noticia"), con timestamp, dirección potencial y magnitud. | Derivada de plantilla + seed |
| 10 | **Random Seed (semilla aleatoria)** | Número que alimenta TODO el ruido pseudoaleatorio del generador. | Es la fuente del determinismo |
| 11 | **Generator Version (versión del generador)** | Versión exacta del algoritmo generador (ej. `gen-1.0.0`). | Sí |
| 12 | **Generated Candle/Tick Path (camino generado)** | El resultado: la serie completa de velas (y sub-pasos tipo tick) de toda la sesión, generada ANTES de iniciar. | Sí, dado seed + versión + parámetros |
| 13 | **Generated Path Hash (hash del camino)** | Hash criptográfico (ej. SHA-256) de la serie generada. Sello de integridad. | Sí |
| 14 | **User Decision Log (registro de decisiones)** | Registro append-only de cada acción del usuario con timestamp de simulación. | No (depende del usuario), pero es inmutable una vez escrito |
| 15 | **Order/Risk/Position Engine (motor de órdenes/riesgo/posición)** | Ejecuta órdenes CONTRA el camino ya generado: fills, slippage, fees, SL/TP, margen, drawdown. | Sí, dado el camino + el decision log |
| 16 | **Evaluation Engine (motor de evaluación)** | Califica la sesión contra el LCC: detección de errores, métricas de proceso, puntaje, XP. | Sí, dado el decision log + resultados |

**Regla de flujo:** Capas 1–11 son entrada → la capa 12 se genera por completo → se sella con la capa 13 → solo entonces empieza la sesión → capas 14–16 operan sobre un mercado ya inmutable.

### Formato universal de vela

Todas las fuentes de datos (sintética, procedural, tutorial fija, desafío fijo, sandbox, replay, futura histórica, futura inspirada-en-histórico) deben entregar el mismo formato universal:

| Campo | Descripción |
|---|---|
| `timestamp` | Tiempo de simulación (inicio de la vela) |
| `open, high, low, close` | Precios OHLC |
| `volume` | Volumen sintético (proxy de liquidez/participación) |
| `spread_hint` | Spread vigente durante la vela (puede variar intra-vela por eventos) |
| `liquidity_hint` | Nivel de liquidez vigente (afecta slippage) |
| `regime_tag` | Etiqueta interna del régimen activo (oculta al usuario en desafíos, visible en revisión educativa) |
| `event_ref` | Referencia opcional a un evento del calendario si ocurre dentro de la vela |
| `subticks[]` | Opcional: 4–16 sub-pasos de precio intra-vela para movimiento tipo tick |

El motor de órdenes y el de evaluación solo conocen este formato. Por eso Burgundy es **historical-ready**: cuando exista un adaptador de datos históricos, producirá este mismo formato y nada más cambia.

---

## 2. El reloj de simulación

### 2.1 Diseño

- Existe un **tiempo de simulación** totalmente desacoplado del reloj real del teléfono.
- El reloj avanza en **pasos discretos** (steps). El paso mínimo es el sub-tick; el paso estándar es la vela.
- La velocidad de reproducción es un **multiplicador sobre el tiempo de simulación**: tiempo real, x3, x5, x10, y replay rápido (avance vela por vela casi instantáneo).
- Cambiar la velocidad **no cambia los datos**: el camino ya existe; solo cambia qué tan rápido se revela.

| Control | Comportamiento |
|---|---|
| Tiempo real | 1 segundo de simulación = 1 segundo real (útil en tutorial y práctica de paciencia) |
| x3 / x5 / x10 | El reloj de simulación avanza 3/5/10 veces más rápido |
| Replay rápido | Avance acelerado o vela-por-vela manual |
| Pausa | Congela el reloj; el usuario puede analizar, medir, colocar órdenes pendientes |
| Avance vela a vela | Un toque = una vela nueva revelada (modo estudio) |
| Rewind (retroceso) | Solo en modos educativos; ver sección 19 |

### 2.2 Reglas

- Las órdenes del usuario se registran con el **timestamp de simulación exacto** (vela + sub-tick) en que se emitieron.
- En pausa se pueden colocar/modificar órdenes; estas se evalúan a partir del siguiente paso revelado, nunca retroactivamente.
- El reloj nunca retrocede en modos competitivos (desafíos/rankings). El rewind existe solo en contexto educativo y queda marcado en el decision log.

---

## 3. Generación de velas

### 3.1 Proceso (procedural por regímenes)

La generación ocurre **una sola vez, completa, antes de la sesión**, así:

1. **Resolver parámetros:** plantilla + tipo de mercado + dificultad fijan rangos (precio inicial, volatilidad base, spread base, etc.).
2. **Expandir la secuencia de regímenes:** la plantilla define la estructura (ej. "rango → ruptura → tendencia alcista → reversión"); el seed determina duraciones exactas, puntos de transición y variantes.
3. **Generar la curva de volatilidad:** modelo de volatilidad por clusters (la volatilidad alta tiende a agruparse, como en mercados reales), modulada por régimen y eventos.
4. **Generar el camino de precio vela a vela:** cada vela se construye con: deriva (drift) del régimen + ruido pseudoaleatorio del seed + amplitud según volatilidad vigente + reglas de coherencia OHLC (high ≥ max(open, close), etc.).
5. **Generar sub-ticks intra-vela:** ver sección 4.
6. **Inyectar eventos:** los shocks del calendario alteran las velas en sus timestamps (ver sección 17).
7. **Generar perfiles acompañantes:** spread y liquidez por vela, coherentes con volatilidad y eventos.
8. **Sellar:** calcular el hash del camino completo y guardarlo.

### 3.2 Realismo mínimo exigido (anti-"mercado de juguete")

- La volatilidad se agrupa en clusters, no es uniforme.
- Existen mechas (wicks): el precio explora más allá del cuerpo de la vela.
- Hay ruido suficiente para que ningún patrón funcione el 100% de las veces.
- Las tendencias tienen retrocesos (pullbacks); nunca son líneas rectas.
- Los rangos tienen falsas salidas ocasionales.
- Los soportes/resistencias se respetan a veces y fallan a veces, según dificultad.

---

## 4. Movimiento tipo tick (aproximación)

Burgundy no simula un libro de órdenes real (innecesario y costoso para educación). Aproxima el movimiento tipo tick así:

- Cada vela contiene **4–16 sub-ticks** pregenerados (también deterministas, derivados del mismo seed).
- Los sub-ticks forman un mini-camino dentro de la vela que: parte del `open`, visita `high` y `low` en un orden determinado por el seed, y termina en el `close`.
- La cantidad de sub-ticks por vela escala con la volatilidad vigente (más volatilidad = más sub-pasos = movimiento más "nervioso").
- **SL/TP, ejecuciones de órdenes pendientes y liquidaciones se evalúan a nivel de sub-tick**, no de cierre de vela. Esto evita el clásico engaño de los simuladores baratos donde el stop "no se tocó" porque solo se mira el close.
- En dispositivos de gama baja, la UI puede renderizar menos sub-pasos, pero el **motor de ejecución siempre evalúa todos** los sub-ticks. Renderizado y ejecución están desacoplados.

---

## 5. Regímenes de mercado

Un régimen es el "estado de comportamiento" del mercado durante un tramo. Burgundy define estos regímenes base:

| Régimen | Comportamiento | Lección típica asociada |
|---|---|---|
| **Tendencia alcista** | Drift positivo + pullbacks proporcionales | Operar a favor, no contra; dejar correr ganancias |
| **Tendencia bajista** | Drift negativo + rebotes técnicos | No "comprar barato" en caída; respetar el stop |
| **Rango (consolidación)** | Sin drift; precio oscila entre zonas | Paciencia; no sobreoperar; riesgo de falsas rupturas |
| **Ruptura (breakout)** | Salida impulsiva de un rango con expansión de volatilidad | Confirmación vs. anticipación |
| **Fakeout (falsa ruptura)** | Ruptura que falla y regresa al rango | La trampa de perseguir el precio |
| **Reversión** | Cambio de dirección de la tendencia dominante | Detectar agotamiento; no casarse con una posición |
| **Volatilidad extrema / shock** | Movimientos amplios y erráticos alrededor de eventos | Riesgo de slippage, gaps y spread amplio |
| **Mercado lento / ilíquido** | Velas pequeñas, spread relativamente mayor | El costo de operar cuando no hay nada que operar |

Reglas de los regímenes:

- La **secuencia** de regímenes la define la plantilla del escenario (estructura pedagógica); el **detalle** (duraciones exactas, magnitudes, transiciones) lo define el seed.
- Las transiciones entre regímenes son graduales o abruptas según el tipo de transición (una reversión por shock es abrupta; un rango que muere en tendencia es gradual).
- En modo sandbox, la secuencia misma puede ser sorteada por el seed dentro de probabilidades configuradas.
- El `regime_tag` queda guardado por vela para la revisión educativa posterior ("aquí el mercado estaba en rango y tú operaste como si fuera tendencia").

---

## 6. Tendencias

- Una tendencia se genera como **drift direccional + ruido + pullbacks estructurados**.
- Los pullbacks retroceden típicamente entre un 25% y un 60% del último impulso (parametrizado por dificultad y seed), de modo que existan zonas de reingreso realistas.
- La pendiente de la tendencia varía: tramos de aceleración, tramos de pausa.
- Las tendencias en dificultad alta incluyen **sacudidas (shakeouts)**: retrocesos profundos que tocan zonas de stops "obvios" antes de continuar. Esto enseña colocación de stops con criterio, no en el lugar evidente.
- Toda tendencia tiene final: termina en rango, en reversión o en shock, según la secuencia del escenario.

---

## 7. Rangos

- Un rango se genera con un **centro de gravedad** y dos zonas (no líneas exactas) de soporte y resistencia.
- El precio oscila entre zonas con toques imperfectos: a veces no llega, a veces sobrepasa ligeramente (mechas que "barren" la zona).
- La amplitud del rango y su duración dependen de plantilla + seed.
- Dentro del rango la volatilidad tiende a comprimirse hacia el final (compresión previa a ruptura), un patrón real que el usuario debe aprender a reconocer.
- Los rangos de dificultad alta incluyen mini-fakeouts internos.

---

## 8. Rupturas (breakouts) y falsas rupturas (fakeouts)

- **Breakout real:** el precio sale de la zona del rango con expansión de volatilidad y volumen sintético, y el nuevo régimen (tendencia) lo confirma. Puede incluir un retest (regreso a probar la zona rota) según seed.
- **Fakeout:** el precio rompe la zona, avanza lo suficiente para activar entradas impulsivas (parametrizado: 0.5x–1.5x el ATR vigente más allá de la zona), y luego regresa con fuerza dentro del rango.
- La **proporción breakout/fakeout** es un parámetro de dificultad: en tutorial puede ser 80/20; en dificultad alta puede acercarse a 50/50.
- Importante: la decisión de si una ruptura es real o falsa **ya está tomada en la generación**, antes de que el usuario opere. El mercado no "decide" hacer fakeout porque el usuario entró. Esto es central a la filosofía anti-manipulación.

---

## 9. Reversiones

- Una reversión se genera como secuencia: **agotamiento → distribución/transición → cambio de drift**.
- Señales sintéticas de agotamiento incluidas en la generación: impulsos cada vez más cortos, mechas de rechazo crecientes, pérdida de momentum.
- Tipos de reversión: gradual (con estructura de techo/piso) y abrupta (provocada por evento del calendario).
- En dificultad alta se generan **reversiones falsas**: el mercado aparenta girar y luego retoma la tendencia (la trampa de "adivinar el techo").

---

## 10. Cambios de volatilidad

- La volatilidad se modela como una **curva continua por sesión** con clustering: periodos calmados y periodos agitados que tienden a persistir.
- Moduladores de la volatilidad: régimen activo (ruptura > tendencia > rango), proximidad de eventos del calendario (la volatilidad sube antes y explota durante), dificultad y perfil del escenario (un escenario "cripto sintético" tiene base más alta que uno "forex mayor sintético").
- Efectos en cadena: volatilidad alta → velas más amplias, más sub-ticks, spread más ancho, más slippage, mayor riesgo de gap. Todo coherente entre sí, porque todos los perfiles derivan de la misma generación.

---

## 11. Cambios de spread

> **Spread** (explicado): la diferencia entre el precio al que puedes comprar y el precio al que puedes vender. Es un costo invisible de cada operación.

- Cada instrumento tiene un **spread base** (definido por tipo de mercado y perfil del escenario).
- El spread vigente por vela/sub-tick se calcula como: `spread = spread_base × factor_volatilidad × factor_liquidez × factor_evento`.
- Durante eventos/shocks, el spread puede ampliarse 3x–10x momentáneamente, como en mercados reales durante noticias.
- En mercado lento/ilíquido, el spread relativo es mayor.
- El spread se muestra al usuario cuando es pedagógicamente relevante, y el motor de evaluación detecta el error de "operar en spread ampliado sin razón".

---

## 12. Cálculo de slippage

> **Slippage** (explicado): recibir un precio de ejecución peor al que esperabas, porque el mercado se movió o no había suficiente liquidez en tu precio.

Fórmula conceptual (determinista, derivada del camino y del decision log — no aleatoria en el momento de ejecución):

```
slippage = f(volatilidad_vigente, liquidez_vigente, tipo_de_orden, tamaño_relativo_de_posición, contexto_de_evento)
```

Reglas:

- **Órdenes market en condiciones normales:** slippage pequeño o nulo.
- **Órdenes market durante volatilidad alta o evento:** slippage significativo y siempre adverso o neutro (nunca "slippage regalo" sistemático; puede existir slippage favorable ocasional con probabilidad baja predefinida en la generación, para realismo).
- **Órdenes límite:** sin slippage de precio (se ejecutan al precio o mejor), pero con riesgo de **no ejecución** si el precio no llega o la liquidez es insuficiente.
- **Stop loss en gap o shock:** se ejecuta al primer precio disponible después del nivel, no al nivel exacto. Esta es una lección clave: el stop limita el riesgo, no lo garantiza al céntimo.
- **Posiciones grandes vs. liquidez baja:** penalización adicional de slippage (enseña el concepto de impacto de mercado sin simular un order book).
- El slippage es **reproducible**: en un replay del mismo escenario con las mismas decisiones, el slippage es idéntico.

---

## 13. Comisiones y costos (fees)

- Modelo simple y visible: comisión por operación (fija o porcentual según instrumento sintético) + spread como costo implícito + costo de financiamiento nocturno (swap) en simulaciones de horizonte largo con apalancamiento.
- Los fees se descuentan en el momento de la ejecución y aparecen desglosados en el resumen de cada operación.
- Objetivo pedagógico: que el usuario interiorice que **sobreoperar tiene costo**, y que una estrategia "ganadora antes de costos" puede ser perdedora después de costos.
- En tutorial temprano los fees pueden estar reducidos o desactivados (lo define el LCC), y se introducen explícitamente como lección.

---

## 14. Disparo de stop loss y take profit

- SL y TP se evalúan **a nivel de sub-tick**, contra el camino pregenerado.
- **Stop loss:** se dispara cuando el precio relevante (bid para largos, ask para cortos — el spread importa) toca o cruza el nivel. La ejecución aplica slippage según las reglas de la sección 12.
- **Take profit:** se ejecuta como orden límite: al precio o mejor, sin slippage adverso, con la salvedad de gaps (si el precio salta por encima del TP en un gap favorable, se ejecuta al precio del gap, que es mejor).
- **Caso gap/shock contra el SL:** ejecución al primer precio disponible (ver sección 15). El motor de evaluación lo registra y lo explica al usuario después: "tu stop estaba en X, se ejecutó en Y, esto se llama slippage por gap".
- Si SL y TP serían tocados dentro de la misma vela, el **orden de los sub-ticks** (ya generado, determinista) decide cuál se tocó primero. Nunca se resuelve "a favor" ni "en contra" del usuario por diseño.
- Modificar SL/TP es una acción registrada en el decision log; mover el stop "para que no me saque" es un error detectable por el motor de evaluación.

---

## 15. Gaps

> **Gap** (explicado): un salto de precio entre un cierre y la siguiente apertura, sin precios intermedios negociables.

- Los gaps se generan en: aperturas de sesión (mercados sintéticos con horario, como acciones/índices), eventos/shocks programados y fines de semana simulados en horizontes largos.
- Tamaño y dirección del gap: determinados en la generación (plantilla + seed), jamás en reacción al usuario.
- Reglas de ejecución a través de un gap: las órdenes pendientes (stops, límites) atrapadas dentro del gap se ejecutan al **primer precio disponible** después del salto. No existen precios "dentro" del gap.
- Lección asociada: el riesgo overnight / de fin de semana y por qué el tamaño de posición importa incluso con stop loss.
- Los mercados sintéticos tipo "cripto" operan 24/7 con gaps raros pero con shocks; los tipo "acciones" tienen gaps frecuentes de apertura. Esto enseña diferencias estructurales entre mercados.

---

## 16. Liquidez y ejecución

> **Liquidez** (explicado): qué tan fácil es entrar o salir de una posición sin mover el precio en tu contra.

- La liquidez es un perfil continuo por sesión (`liquidity_hint` por vela), generado junto con el camino.
- Efectos de liquidez baja: spread más amplio, slippage mayor, órdenes límite con riesgo de ejecución parcial o nula, penalización creciente por tamaño de posición.
- Efectos de liquidez alta: ejecución cercana al precio visible.
- La liquidez cae típicamente: en mercado lento, alrededor de shocks (la liquidez "se esconde" justo cuando más se necesita, como en la realidad) y en horarios muertos de mercados con sesión.
- Burgundy no simula profundidad de libro de órdenes en MVP; usa el perfil de liquidez como aproximación honesta y suficiente para educación.

---

## 17. Eventos y shocks

- El **Event Schedule** se genera antes de la sesión: lista de eventos con timestamp, tipo, magnitud potencial y resolución direccional **ya decidida por el seed**.
- Tipos de eventos sintéticos: "dato económico" (shock de volatilidad con dirección), "sorpresa de mercado" (shock sin aviso), "evento anunciado" (el usuario puede ver en un calendario simulado que viene un evento, sin conocer la dirección).
- Anatomía de un shock: compresión/nerviosismo previo (opcional) → impulso violento en 1–3 velas con spread ampliado y liquidez reducida → resolución (continuación, reversión o absorción), todo predefinido en la generación.
- Los eventos anunciados enseñan la decisión real: ¿cierro antes del evento, reduzco tamaño, o asumo el riesgo?
- Los eventos sorpresa enseñan por qué existe el stop loss y el tamaño de posición prudente.
- **Nunca** se inyecta un evento en reacción a la posición del usuario.

---

## 18. Simulación de largo plazo (3 meses, 6 meses, 1 año, 2 años)

Objetivo: que el usuario experimente drawdowns largos, rachas, estacionalidad del proceso y la diferencia entre suerte y consistencia, sin sesiones de horas.

> **Drawdown** (explicado): cuánto cae la cuenta desde su punto más alto. Es la medida del dolor del trader.

Mecánica de compresión de tiempo sin perder realismo:

- **Compresión por velas de marco mayor:** un año se reproduce en velas de 4h/diarias/semanales, no en M1. El camino se genera coherente en múltiples marcos (el camino diario agrega los intradía generados, o se genera directamente en el marco mayor según el modo).
- **Velocidad adaptable + puntos de decisión:** la reproducción corre acelerada y se **pausa automáticamente en checkpoints de decisión** definidos por la plantilla (eventos, señales relevantes, revisiones semanales simuladas). El usuario decide en los momentos que importan; el tiempo muerto se comprime.
- **Lo que NO se permite:** comprimir eliminando los periodos aburridos por completo (el aburrimiento y la espera son parte de la lección), ni acelerar tanto que el usuario no pueda reaccionar en modos donde la reacción es la lección.
- En horizontes largos se activan: swaps/costos de mantenimiento, gaps de fin de semana, secuencias multi-régimen largas y métricas de proceso de largo plazo (máximo drawdown, racha de disciplina, adherencia al plan).

---

## 19. Replay y rewind educativo

- **Replay de escenario:** volver a jugar el mismo escenario exacto (mismo seed) para reintentar con lo aprendido. El motor de evaluación compara intentos: "en el intento 1 entraste en el fakeout; en el intento 2 esperaste confirmación".
- **Replay de sesión (repetición):** reproducir la sesión pasada con las decisiones del usuario superpuestas en el gráfico: entradas, salidas, movimientos de stop, momentos de pánico (cierres impulsivos). Es la herramienta central de revisión de errores.
- **Rewind (retroceso):** disponible solo en modos educativos (tutorial, sandbox de estudio). Permite retroceder N velas para re-analizar. Reglas: el rewind queda registrado en el decision log; un resultado obtenido con rewind **nunca** genera puntaje competitivo ni ranking; en modo desafío el rewind está deshabilitado.
- **Checkpoints de decisión:** la plantilla marca momentos donde la simulación se pausa y pregunta o registra la decisión del usuario explícitamente; en la revisión, cada checkpoint se evalúa contra el LCC.

---

## 20. Sistema determinista de semillas (seeds)

### 20.1 Principio

> **Misma plantilla de escenario + mismos parámetros + misma seed + misma versión del generador = exactamente el mismo camino de mercado.**

El generador usa un PRNG (generador pseudoaleatorio) con seed explícita; todo el "azar" de la simulación proviene de ahí. No se usa ninguna fuente de entropía externa (reloj real, sensores, input del usuario) durante la generación.

### 20.2 Tipos de seed

| Tipo de seed | Uso | Propiedad clave |
|---|---|---|
| **Tutorial fija** | Lecciones guiadas | Repetible para todos los usuarios: la lección siempre muestra lo que debe mostrar |
| **Desafío fija** | Challenges y rankings de sesión | Equidad: todos los participantes enfrentan exactamente el mismo mercado |
| **Sandbox aleatoria** | Práctica libre | Variedad infinita; la seed se guarda igualmente para poder repetir |
| **Replay** | Revisión de errores | Reconstruye el escenario exacto de una sesión pasada |
| **Inspirada-en-histórico** (futura) | Escenarios procedurales calibrados con estadísticas de episodios reales (ej. "volatilidad estilo 2020") | Realismo histórico sin requerir el dataset completo |
| **Replay histórico exacto** (futuro) | Reproducir datos reales | Usa el adaptador histórico; la "seed" referencia el dataset + rango de fechas |

### 20.3 Variación controlada

- Cambiar **solo la seed** con la misma plantilla produce variaciones del mismo contexto de aprendizaje: el usuario practica "rupturas de rango" diez veces con diez mercados distintos pero pedagógicamente equivalentes.
- Cambiar la plantilla cambia el contexto; cambiar la seed cambia la instancia. Esta separación es la base del entrenamiento por repetición sin memorización del camino.

### 20.4 Equidad en desafíos

- Los desafíos con ranking usan seed fija + plantilla fija + versión de generador fija → todos compiten contra el mismo mercado.
- El hash del camino permite verificar que nadie jugó contra un camino distinto.
- El decision log con timestamps de simulación permite detectar reintentos: solo el primer intento (o el intento declarado según las reglas del desafío) puntúa para ranking.
- En desafíos, la seed no se revela hasta terminar (evita pregenerar el camino y "estudiarlo" fuera de la app).

### 20.5 Metadatos almacenados por simulación

Cada simulación generada almacena obligatoriamente:

| Campo | Propósito |
|---|---|
| Seed | Reproducibilidad |
| Tipo de seed | Tutorial / desafío / sandbox / replay / histórica futura |
| Versión del generador | Compatibilidad de replays (sección 22) |
| ID de plantilla de escenario | Estructura pedagógica usada |
| ID del Learning Context Contract | Qué se estaba enseñando |
| Tipo de mercado | Forex/índice/cripto/acción sintética |
| Tipo de instrumento | Instrumento sintético concreto |
| Dificultad | Escalado de parámetros |
| Horizonte temporal | Sesión corta / 3m / 6m / 1a / 2a |
| Perfil de volatilidad | Curva usada |
| Perfil de spread | Curva usada |
| Perfil de liquidez | Curva usada |
| Secuencia de regímenes | Estructura realizada |
| Calendario de eventos | Shocks programados |
| Hash del camino de velas generado | Integridad y verificación anti-manipulación |
| Decision log del usuario | Evaluación y replay |
| Resultado de evaluación | Puntaje, errores detectados, XP |
| Metadatos de replay | Intentos, uso de rewind, comparaciones |

Estos metadatos viajan dentro del archivo de exportación/importación de progreso del usuario (offline-first, sin nube).

---

## 21. Prevención de manipulación post-trade (garantía de integridad)

Mecanismos concretos:

1. **Pre-generación total:** el camino completo (velas + sub-ticks + spreads + liquidez + eventos) existe antes del primer frame de la sesión.
2. **Hash sellado:** al terminar la generación se calcula el hash del camino y se persiste ANTES de iniciar la reproducción. Cualquier verificación posterior (replay, auditoría de desafío) regenera el camino desde seed+parámetros+versión y compara hashes.
3. **Separación de motores:** el motor de órdenes/riesgo/posición tiene acceso de **solo lectura** al camino. Por diseño de arquitectura, no existe ninguna vía por la que una orden modifique una vela.
4. **Decision log append-only:** las decisiones se agregan, nunca se editan. El par (camino sellado, decision log) reproduce la sesión completa de forma determinista, incluyendo fills y slippage.
5. **Transparencia educativa:** al final de una sesión, el usuario puede ver "este escenario fue generado antes de que operaras; aquí está su sello". Esto construye la confianza que diferencia a Burgundy de la sensación de "la demo me hizo trampa".

---

## 22. Versionado del generador (protección de replays antiguos)

Problema: si el algoritmo generador mejora, la misma seed produciría un camino distinto, rompiendo replays, rankings históricos y verificaciones de hash.

Solución:

- Toda simulación guarda su `generator_version` (versionado semántico: `gen-MAJOR.MINOR.PATCH`).
- **Regla de oro:** cualquier cambio que altere la salida para una misma seed incrementa la versión MAJOR o MINOR. PATCH se reserva para cambios que NO alteran la salida (rendimiento, refactors verificados por hash).
- La app conserva los generadores anteriores como módulos versionados (o, si el costo de mantenerlos crece, conserva los **caminos generados completos** de las sesiones importantes dentro del archivo de progreso, de modo que el replay no necesite regenerar).
- Estrategia por defecto del MVP: guardar siempre el camino generado comprimido junto con sus metadatos. Regenerar desde seed es la verificación; el camino guardado es la fuente del replay. Así, aunque un generador viejo se retire, los replays viejos sobreviven.
- Los desafíos con ranking fijan plantilla + seed + versión: un desafío nunca cambia de versión de generador a mitad de su periodo de competencia.

---

## 23. Comparación de enfoques de datos

| Enfoque | Descripción | Ventajas | Desventajas | Veredicto para Burgundy |
|---|---|---|---|---|
| **Generación aleatoria pura** (random walk simple) | Ruido sin estructura | Trivial de implementar | Irreal: sin regímenes, sin clusters de volatilidad, sin contexto pedagógico; enseña hábitos falsos | Rechazado |
| **Datos totalmente sintéticos** (modelos estadísticos sin estructura pedagógica) | Series con propiedades estadísticas correctas | Realismo estadístico | No garantiza que aparezca lo que la lección necesita enseñar | Insuficiente solo; útil como base del procedural |
| **Datos históricos** | Velas reales de mercados reales | Realismo total; credibilidad | Requiere datasets (licencias, peso, actualización), conexión o empaquetado pesado, contradice offline-first ligero del MVP; los usuarios avanzados pueden reconocer episodios famosos y "adivinar" | Futuro, no MVP |
| **Datos híbridos** | Mezclar histórico + sintético | Flexible | Hereda los costos del histórico y suma complejidad de mezcla | Futuro |
| **Generación procedural por regímenes de volatilidad** | Construir el camino por regímenes con volatilidad realista y seed determinista | Realismo educativo controlable, variedad infinita, repetibilidad, sin datasets, offline total, escenarios siempre alineados con la lección | Requiere diseñar y calibrar bien el generador | **Núcleo del MVP** |
| **Escenarios educativos prediseñados** | Plantillas curadas a mano para enseñar algo específico | Control pedagógico total, calidad garantizada en tutoriales | No escala solo: requiere autoría | **Complemento del MVP** (tutoriales y desafíos) |
| **Replay histórico exacto** | Reproducir un periodo real tal cual | Máximo realismo; "viviste el crash" | Todos los costos del histórico + diseño de contexto | Fase futura (la arquitectura ya lo soporta vía formato universal de vela) |
| **Escenarios procedurales inspirados en histórico** | Procedural calibrado con estadísticas de episodios reales | Realismo histórico sin dataset completo, sin memorización posible | Requiere trabajo de calibración por episodio | Fase futura cercana (puente natural desde el MVP) |

---

## 24. Forzar contexto de aprendizaje vs. forzar resultado de trading

Esta distinción es la columna vertebral pedagógica de Burgundy:

| | **Forzar contexto de aprendizaje** (lo que Burgundy hace) | **Forzar resultado de trading** (lo que Burgundy prohíbe) |
|---|---|---|
| Qué se controla | Que el escenario **contenga** la situación a aprender (un fakeout, una tendencia, un shock) | Que el usuario **gane o pierda** una operación concreta |
| Cuándo se decide | Antes de la sesión, en la generación | Después de la entrada del usuario, en reacción a ella |
| Ejemplo legítimo | "Esta lección genera un rango con una falsa ruptura en algún momento del tramo medio" | — |
| Ejemplo prohibido | — | "El usuario entró en largo, entonces el precio cae para enseñarle humildad" / "el usuario va perdiendo mucho, entonces el precio rebota para que no se frustre" |
| Resultado del usuario | Abierto: puede ganar o perder dentro del escenario según sus decisiones | Predeterminado o manipulado |
| Efecto pedagógico | Confianza, aprendizaje real, transferible | Aprendizaje falso, supersticiones, desconfianza cuando se descubre |

El LCC define el contexto; la seed define la instancia; el usuario define el resultado. Tres responsabilidades, tres dueños, sin cruces.

---

## 25. Alcance MVP vs. fases posteriores

### MVP (núcleo de simulación)

| Mecánica | Incluida en MVP |
|---|---|
| Reloj de simulación con pausa, tiempo real, x3/x5/x10, avance vela a vela | ✅ |
| Generación procedural por regímenes con seed determinista | ✅ |
| Pre-generación total + hash del camino | ✅ |
| Sub-ticks intra-vela y evaluación de SL/TP a nivel sub-tick | ✅ |
| Regímenes: tendencia, rango, breakout, fakeout, reversión, mercado lento | ✅ |
| Clusters de volatilidad, spread dinámico, slippage determinista, fees básicos | ✅ |
| Gaps en eventos y aperturas; eventos anunciados y sorpresa | ✅ |
| Perfil de liquidez como modulador de ejecución | ✅ |
| Escenarios prediseñados para tutorial + seeds fijas de desafío + sandbox aleatorio | ✅ |
| Replay de escenario y replay de sesión con decisiones superpuestas | ✅ |
| Rewind educativo (solo modos no competitivos) | ✅ |
| Versionado del generador + camino guardado comprimido | ✅ |
| Metadatos completos por simulación, exportables/importables offline | ✅ |
| Horizonte largo básico (3 y 6 meses en marcos mayores con checkpoints) | ✅ |

### Fase 2 (post-MVP)

- Horizontes de 1 y 2 años con swaps/costos de mantenimiento completos y estacionalidad de proceso.
- Escenarios procedurales **inspirados en histórico** (calibración con estadísticas de episodios reales).
- Más tipos de mercado sintético y más perfiles de instrumento.
- Comparador avanzado de intentos de replay (diffs de decisiones entre intentos).
- Eventos encadenados (un shock que altera el régimen del resto de la sesión de formas más ricas).

### Fase 3 (futuro)

- **Replay histórico exacto** mediante adaptador de datos históricos al formato universal de vela.
- Modo híbrido (tramos históricos + continuación procedural).
- Aproximación de microestructura más fina (profundidad de liquidez simulada por niveles), solo si el valor educativo lo justifica.

### Recomendación final de enfoque MVP

**Escenarios educativos prediseñados + generación procedural basada en regímenes + seeds deterministas.**

Por qué:

1. **Alineación pedagógica total:** cada lección garantiza su contexto de aprendizaje (lo que el histórico puro no garantiza).
2. **Offline-first real:** cero datasets, cero descargas, cero licencias; un generador de pocos KB produce variedad infinita.
3. **Equidad verificable:** seeds fijas + hashes hacen los rankings justos y auditables, el diferenciador de confianza de Burgundy.
4. **Replayabilidad perfecta:** misma seed = mismo mercado; la revisión de errores es exacta, no aproximada.
5. **Historical-ready sin deuda:** el formato universal de vela y la separación de capas permiten enchufar datos históricos después sin tocar el motor de órdenes ni el de evaluación.
6. **Costo y riesgo mínimos:** el esfuerzo se concentra en calibrar realismo educativo, no en infraestructura de datos.

---

*Documento de especificación del modelo de simulación de Burgundy. Proyecto firmado por **tsuloid**. Contenido educativo: Burgundy no constituye asesoría financiera ni opera dinero real.*
