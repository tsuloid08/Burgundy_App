# Burgundy — Bloque 10: Scenario Engine (Motor de Escenarios)

> **Proyecto:** Burgundy — Simulador educativo de trading
> **Autor / Firma:** tsuloid
> **Idioma:** Español (LATAM)
> **Estado:** Especificación de diseño — sin código, sin pantallas, sin esquemas finales de base de datos
> **Paleta Burgundy:** `#1A1617` (Deep Charcoal) · `#571324` (Matte Burgundy) · `#2E2E2E` (Surfaces) · `#C9A050` (Muted Gold) · `#4A6D56` (velas alcistas) · `#802F3E` (velas bajistas)

---

## 1. Filosofía del motor de escenarios

El Scenario Engine de Burgundy existe para una sola cosa: **forzar un contexto de aprendizaje, nunca forzar un resultado de trading.**

Principios no negociables:

1. **El camino del mercado se genera ANTES de cualquier acción del usuario.** El precio nunca reacciona a las decisiones del usuario. Si el usuario compra, vende, o no hace nada, las velas son exactamente las mismas.
2. **Las decisiones del usuario afectan solo su lado de la simulación:** órdenes, posiciones, cuenta, riesgo, P/L (ganancia/pérdida), journal, errores detectados, feedback y score. Jamás alteran el path generado.
3. **Determinismo total:** mismo template de escenario + mismos parámetros + misma seed + misma versión del generador = exactamente el mismo path de mercado, en cualquier dispositivo, en cualquier fecha.
4. **Se premia el proceso, no el resultado.** Un trade perdedor con buen proceso puede recibir mejor evaluación que un trade ganador con mal proceso. Esto es el corazón educativo de Burgundy.
5. **Historical-ready, no historical-dependent.** El motor consume velas en un formato universal y es agnóstico a la fuente. En MVP la fuente es sintética (generador determinista); en el futuro podrá ser "inspirada en datos históricos" sin cambiar el motor.

### 1.1 El Learning Context Contract (Contrato de Contexto de Aprendizaje)

Cada escenario está gobernado por un **Learning Context Contract (LCC)**: un contrato explícito que define qué debe aprender el usuario, qué condiciones de mercado lo van a poner a prueba, y cómo se evalúa su comportamiento independientemente del resultado monetario.

Componentes de todo LCC:

| Componente | Descripción |
|---|---|
| **Objetivo de aprendizaje** | La habilidad o disciplina concreta que el escenario entrena. |
| **Condición de mercado garantizada** | Lo que el path generado SÍ va a contener (ej.: "habrá un fake breakout entre el 40% y 60% de la sesión"). |
| **Trampas intencionales** | Eventos diseñados para provocar el error típico del principiante. |
| **Decisiones buenas aunque pierdan** | Lista de comportamientos que se evalúan positivo aunque el trade termine en pérdida. |
| **Decisiones malas aunque ganen** | Lista de comportamientos que se evalúan negativo aunque el trade termine en ganancia. |
| **Métricas de evaluación** | Qué se mide para calcular el score de proceso. |
| **Reglas de detección de errores** | Condiciones objetivas que disparan una entrada en el registro de errores. |
| **Condición de éxito / fracaso** | Definida sobre proceso y supervivencia, no solo sobre P/L. |

El LCC es la "rúbrica" del escenario. El evaluador de Burgundy compara el log de decisiones del usuario contra el LCC, nunca contra "¿ganaste plata?".

**Fórmula estándar de redacción (AUD-017):** toda condición garantizada de un escenario se redacta como *"el path ya contiene X condición; la consecuencia depende de la ejecución"*. El LCC fuerza contexto de aprendizaje (condiciones pre-generadas, trampas, criterios y evaluación), nunca el resultado post-trade: el path se genera completo antes de la decisión del usuario, no se altera después, y la evaluación juzga proceso, riesgo y decisión — no solo P/L.

---

## 2. Anatomía de un escenario

Todo escenario en Burgundy se compone de estas capas:

| Capa | Contenido |
|---|---|
| **Identidad** | ID, título, descripción, nivel de usuario requerido, fase (MVP / posterior). |
| **LCC** | El contrato de aprendizaje descrito arriba. |
| **Template de escenario** | Plantilla paramétrica del comportamiento del mercado (tendencia, rango, spike, etc.). |
| **Parámetros de mercado** | Tipo de mercado, volatilidad, liquidez, spread, slippage, fees, horizonte temporal, timeframe de velas. |
| **Seed** | Semilla determinista (fija o aleatoria según modo). |
| **Versión del generador** | `generatorVersion` — garantiza reproducibilidad aunque el algoritmo evolucione. |
| **Cuenta inicial** | Capital de partida, apalancamiento máximo permitido, instrumentos disponibles. |
| **Reglas de sesión** | Velocidad de simulación recomendada, pausas permitidas, indicadores permitidos o bloqueados. |
| **Trampas** | Eventos adversariales pre-generados dentro del path. |
| **Evaluación** | Métricas, condición de éxito, condición de fracaso, detección de errores. |
| **Feedback** | Mensajes de cierre, revisión de errores, conexión con lecciones de la academia. |
| **Replay** | Metadatos para reproducir la sesión exacta (seed + decisiones + versión). |
| **Desbloqueo** | Requisito de progresión (nivel, XP, escenario previo completado). |
| **Disponibilidad en modo libre** | Si puede jugarse fuera de la ruta de aprendizaje. |

### 2.1 Tipos de seed

| Tipo de seed | Uso | Comportamiento |
|---|---|---|
| **Fija (tutorial)** | Tutoriales y primeras lecciones | Todos los usuarios ven exactamente el mismo mercado. Permite feedback guiado paso a paso. |
| **Fija (challenge)** | Challenges con ranking | Mismo mercado para todos → los rankings de sesión son comparables y justos. |
| **Aleatoria (sandbox)** | Práctica libre | Seed nueva por sesión; se guarda para poder hacer replay. |
| **Replay** | Repetición de sesión | Reutiliza la seed original; el path es idéntico al de la sesión repetida. |
| **Histórica-inspirada (futuro)** | Escenarios basados en episodios reales | Seed + referencia a un dataset histórico en formato de vela universal; el motor no cambia. |

Regla de oro: **la seed se selecciona o genera ANTES de generar el path, y el path se genera completo ANTES de que el usuario vea la primera vela.** Se almacena un hash del path generado para verificar integridad en replays y rankings. La política completa de persistencia, export, replay y visibilidad de seeds está cerrada por `SEED_PATH_REPLAY_EXPORT_LOCK` (documento 08); el contrato de determinismo (PCG32, serialización canónica, corpus dorado) por `DETERMINISM_LOCK_V1` (documento 08).

### 2.2 Formato universal de vela

El motor solo entiende un formato de vela agnóstico a la fuente: timestamp, open, high, low, close, volumen, más metadatos de sesión (spread vigente, liquidez vigente, eventos como "news spike" marcados en la línea de tiempo). Sintético hoy, histórico mañana, mismo motor siempre.

---

## 3. Pipeline de generación de escenarios

Flujo completo, en orden estricto:

| Paso | Acción | Detalle |
|---|---|---|
| 1 | **Seleccionar objetivo de lección** | Qué habilidad se entrena (ej.: "respetar el stop loss"). Viene de la ruta de aprendizaje o de la elección del usuario en sandbox/challenge. |
| 2 | **Seleccionar Learning Context Contract** | Se carga el LCC asociado al objetivo. |
| 3 | **Seleccionar template de escenario** | Plantilla paramétrica (trend, range, fake breakout, spike, etc.). |
| 4 | **Seleccionar tipo de mercado** | Forex sintético, índice sintético, cripto sintética, acción sintética. Define rangos típicos de volatilidad y horarios. |
| 5 | **Seleccionar dificultad** | Ajusta ruido, claridad de la estructura, severidad de trampas, costos de fricción. |
| 6 | **Seleccionar horizonte temporal** | Duración simulada (1 sesión intradía, 1 semana, 3 meses, 1 año) y timeframe de velas. |
| 7 | **Seleccionar parámetros de volatilidad / liquidez / spread** | Curvas de spread por hora simulada, profundidad de liquidez, slippage esperado, fees. |
| 8 | **Seleccionar o generar seed** | Fija (tutorial/challenge), aleatoria (sandbox), o de replay. |
| 9 | **Seleccionar versión del generador** | `generatorVersion` queda sellada en los metadatos de la sesión. |
| 10 | **Generar el path completo de mercado** | Todas las velas, eventos, curvas de spread/liquidez y trampas se materializan ANTES de la acción del usuario. |
| 11 | **Almacenar hash del path** | SHA-256 de la serialización canónica del path (`DETERMINISM_LOCK_V1`, documento 08), sellado antes de la primera vela visible; valida replays, rankings y exports. |
| 12 | **Iniciar sesión de usuario** | Se presenta el contexto, el setup y la tarea; arranca el reloj de simulación. |
| 13 | **Registrar decisiones** | Cada orden, modificación, cancelación, cambio de tamaño, cierre, pausa y "no acción" relevante queda en el log con timestamp simulado. |
| 14 | **Evaluar contra el LCC** | Al cerrar la sesión, el evaluador cruza el log de decisiones con las reglas del contrato: errores detectados, decisiones buenas/malas, métricas de proceso. |
| 15 | **Guardar resultados** | Score, ranking de sesión, high score si aplica, entradas de journal, errores, y metadatos de replay (seed + parámetros + versión + hash + log de decisiones). Todo local, exportable. |

Notas de diseño:

- Los pasos 1–11 son **previos e inmutables**. Nada de lo que ocurra del paso 12 en adelante puede modificar el resultado de los pasos 1–11.
- El log de decisiones (paso 13) es la única entrada del evaluador (paso 14) además del path y el LCC.
- El replay reconstruye la sesión con: template + parámetros + seed + `generatorVersion` + log de decisiones. El hash del path verifica que la reconstrucción es fiel.

---

## 4. Templates de escenario (catálogo base)

Plantillas paramétricas que el generador puede materializar:

| Template | Estructura del path | Parámetros clave |
|---|---|---|
| `TPL_TREND` | Tendencia direccional con pullbacks | dirección, pendiente, profundidad de pullbacks, ruido |
| `TPL_RANGE` | Rango lateral con soportes/resistencias respetados | ancho del rango, número de toques, falsas salidas menores |
| `TPL_FAKE_BREAKOUT` | Rango → ruptura falsa → reversión | ventana de la trampa, magnitud de la ruptura, velocidad de reversión |
| `TPL_NEWS_SPIKE` | Calma → spike violento con spread ampliado → estabilización | timing del evento, magnitud, duración del spread ampliado, slippage |
| `TPL_HIGH_SPREAD` | Mercado normal con spread estructuralmente alto | multiplicador de spread, variación por hora |
| `TPL_LOW_LIQUIDITY` | Sesión ilíquida: velas erráticas, slippage alto | profundidad de libro simulada, gaps, horarios muertos |
| `TPL_GRIND` | Horizonte largo multi-régimen (tendencias + rangos + shocks) | mezcla de regímenes, frecuencia de shocks, duración total |
| `TPL_TRAP_SEQUENCE` | Secuencia diseñada para provocar un error conductual específico | tipo de trampa conductual, número de "anzuelos" |

Los 16 escenarios de la sección 6 se construyen combinando estos templates con LCCs y parámetros específicos.

---

## 5. Evaluación: proceso sobre resultado

### 5.1 Métricas de proceso estándar

Todas medibles objetivamente desde el log de decisiones:

| Métrica | Qué mide |
|---|---|
| **Uso de stop loss** | % de posiciones abiertas con stop loss definido antes o al momento de la entrada. |
| **Riesgo por trade** | % del capital arriesgado por operación (distancia al stop × tamaño). |
| **Respeto del stop** | Si el usuario movió el stop en contra de la posición o lo eliminó. |
| **Risk/reward planeado** | Relación riesgo/beneficio al momento de la entrada. |
| **Exposición al apalancamiento** | Leverage efectivo usado vs. el apropiado para el escenario. |
| **Disciplina post-pérdida** | Tiempo y tamaño de la siguiente operación después de una pérdida. |
| **Sobreoperación (overtrading)** | Número de trades vs. el rango razonable del escenario. |
| **Drawdown máximo** | Caída máxima de la cuenta desde su punto más alto. |
| **Supervivencia** | Si la cuenta terminó por encima del umbral de quiebra educativa del escenario. |
| **Paciencia de entrada** | Si esperó la condición del setup o entró por impulso. |
| **Calidad del journal** | Si registró razón de entrada/salida cuando el escenario lo pide. |

### 5.2 Reglas universales de detección de errores

Activas en todos los escenarios salvo que el LCC indique lo contrario. Los valores N/X/umbral de esta tabla quedan **resueltos por escenario en el `scoringProfile` de su LCC**, con los defaults numéricos canónicos de `SCORING_V1_LOCK` (documento 11):

| Código | Error detectado | Regla objetiva |
|---|---|---|
| `ERR_NO_SL` | Entrada sin stop loss | Posición abierta sin stop definido durante más de N velas. |
| `ERR_SL_MOVED` | Stop movido en contra | Stop loss desplazado alejándolo del precio para "darle aire" a una posición perdedora. |
| `ERR_OVERSIZE` | Tamaño excesivo | Riesgo por trade > umbral del escenario (típicamente 2%). |
| `ERR_OVERLEVERAGE` | Apalancamiento excesivo | Leverage efectivo > umbral del escenario. |
| `ERR_REVENGE` | Revenge trading | Trade abierto < X velas después de una pérdida, con tamaño ≥ al anterior, sin setup válido. |
| `ERR_CHASE` | Perseguir el precio | Entrada a mercado después de un movimiento extendido sin pullback, lejos de cualquier nivel del setup. |
| `ERR_OVERTRADE` | Sobreoperación | Número de trades > máximo razonable del escenario. |
| `ERR_NO_PLAN` | Sin plan | Entrada sin take profit ni criterio de salida registrado, cuando el escenario lo exige. |
| `ERR_ALL_IN` | Capital concentrado | > umbral del capital comprometido en una sola posición. |
| `ERR_HOLD_LOSER` | Sostener al perdedor | Posición en pérdida mantenida más allá del stop mental/declarado sin acción. |

Cada error detectado genera: una entrada en el registro de errores, una explicación en español claro, y un enlace conceptual a la lección de la academia que cubre ese error.

---

## 6. Especificaciones de los 16 escenarios

> Convenciones: capital expresado en USD simulados con cuentas pequeñas realistas para LATAM (100–1.000 USD en escenarios básicos). "Velocidad de simulación" = velocidad recomendada de reproducción de velas (el usuario puede pausar salvo que el LCC lo prohíba). Todos los paths son **pre-generados**; se indica explícitamente en cada ficha.

---

### Escenario 1 — Tu primera operación

| Campo | Especificación |
|---|---|
| **Título** | "Tu primera operación" |
| **Nivel de usuario** | 1 (inicio absoluto) |
| **Tipo de mercado** | Forex sintético (par mayor, comportamiento dócil) |
| **Velocidad recomendada** | Lenta (1 vela cada 3–4 s), con pausa automática en momentos clave |
| **Setup** | Tendencia alcista suave y clara con un pullback evidente. El tutorial señala el contexto: "el precio sube de forma ordenada". |
| **Tarea del usuario** | Abrir una posición de compra con stop loss y take profit, siguiendo la guía paso a paso. Cerrar o dejar que el TP/SL actúe. |
| **Learning Context Contract** | Objetivo: ejecutar el ciclo completo orden → posición → stop → take profit → cierre, entendiendo cada término (spread, stop loss, take profit, P/L). Condición garantizada: el mercado da una oportunidad limpia. Sin trampas. |
| **Template** | `TPL_TREND` (dificultad mínima, ruido bajo) |
| **Tipo de seed** | **Fija (tutorial)** |
| **Comportamiento de seed** | Idéntica para todos los usuarios y todas las repeticiones; permite que el tutorial guíe sobre velas conocidas. |
| **Path de mercado** | Pre-generado, inmutable, idéntico siempre. |
| **Trampa oculta** | Ninguna. Es el único escenario sin trampa. |
| **Error esperado del principiante** | Saltarse el stop loss porque "es solo práctica"; confundir spread con pérdida instantánea al abrir. |
| **Decisiones buenas (aunque el trade pierda)** | Definir stop loss antes de entrar; tamaño dentro del riesgo sugerido; seguir los pasos del tutorial. |
| **Decisiones malas (aunque el trade gane)** | Entrar sin stop; agrandar el tamaño por encima de lo guiado. |
| **Condición de éxito** | Completar el ciclo completo con stop loss definido. El P/L final es irrelevante. |
| **Condición de fracaso** | No se puede "fracasar"; solo se puede no completar. El tutorial insiste hasta que el ciclo se complete bien. |
| **Métricas de evaluación** | Uso de stop loss, comprensión verificada de spread/P/L (micro-confirmaciones del tutorial), seguimiento de pasos. |
| **Feedback** | "Completaste tu primera operación. Fíjate: pagaste el spread al entrar — esa es la diferencia entre el precio de compra y el de venta. Tu stop loss limitó lo que podías perder antes de saber si ganarías. Eso es lo que hace un trader: primero controla el riesgo, después busca la ganancia." |
| **Replay** | Disponible; mismo path siempre. Útil para repasar el flujo. |
| **Desbloqueo** | Ninguno; es el punto de entrada de Burgundy. |
| **Modo libre** | Sí, siempre disponible. |
| **Histórico-inspirado a futuro** | No necesario; su valor es didáctico, no realista. |
| **Fase** | **MVP** |

---

### Escenario 2 — Día de tendencia

| Campo | Especificación |
|---|---|
| **Título** | "Día de tendencia" |
| **Nivel de usuario** | 2 |
| **Tipo de mercado** | Acción sintética intradía *(reasignado desde índice sintético: el índice es post-MVP según `MVP_MARKET_LOCK`, documento 12)* |
| **Velocidad recomendada** | Media (1 vela/s), pausa libre |
| **Setup** | Sesión intradía con tendencia direccional sostenida y 2–3 pullbacks operables. |
| **Tarea del usuario** | Identificar la dirección, entrar en un pullback (no persiguiendo el precio) y dejar correr la ganancia con gestión de stop. |
| **Learning Context Contract** | Objetivo: operar a favor de la tendencia y aprender que entrar tarde persiguiendo precio es distinto a entrar en pullback. Garantía: la tendencia existe y los pullbacks ocurren en ventanas definidas del path. |
| **Template** | `TPL_TREND` (ruido moderado) |
| **Tipo de seed** | Fija en tutorial/lección; aleatoria en sandbox (el template garantiza la estructura). |
| **Comportamiento de seed** | En lección: misma para todos. En sandbox: nueva por sesión, guardada para replay. |
| **Path de mercado** | Pre-generado completo. |
| **Trampa oculta** | Un pullback más profundo de lo cómodo a mitad de sesión, diseñado para sacudir stops demasiado ajustados y tentar a operar contra-tendencia. |
| **Error esperado** | Entrar contra la tendencia "porque ya subió mucho"; cerrar la ganancia demasiado pronto por miedo; perseguir el precio tras un impulso. |
| **Decisiones buenas (aunque pierdan)** | Entrada en pullback con stop bajo estructura; mantener dirección a favor de tendencia aunque ese trade puntual pierda. |
| **Decisiones malas (aunque ganen)** | Contra-tendencia que por azar gana en la trampa del pullback; entrada persiguiendo precio que la tendencia rescata. El evaluador lo marca: "ganaste, pero ese hábito te quiebra la cuenta a largo plazo". |
| **Condición de éxito** | ≥1 entrada a favor de tendencia con stop definido y riesgo ≤2%; sin `ERR_CHASE`. |
| **Condición de fracaso** | Operar solo contra-tendencia, o ≥2 errores `ERR_CHASE`/`ERR_NO_SL`. |
| **Métricas** | Dirección vs. tendencia, paciencia de entrada, R/R planeado, gestión del stop. |
| **Feedback** | Compara las entradas del usuario contra las ventanas de pullback del path: "Aquí esperaste — bien. Aquí perseguiste el precio: entraste 14 velas después del impulso, sin retroceso. La tendencia te salvó esta vez; no siempre lo hará." |
| **Replay** | Misma seed → mismo día. Ideal para reintentar entradas en las mismas ventanas. |
| **Desbloqueo** | Escenario 1 completado. |
| **Modo libre** | Sí, tras desbloqueo. |
| **Histórico-inspirado a futuro** | Sí — días de tendencia reales son material natural. |
| **Fase** | **MVP** |

---

### Escenario 3 — Día de rango

| Campo | Especificación |
|---|---|
| **Título** | "Día de rango" |
| **Nivel de usuario** | 2 |
| **Tipo de mercado** | Forex sintético intradía |
| **Velocidad recomendada** | Media |
| **Setup** | Sesión lateral: soporte y resistencia respetados varias veces, sin tendencia. |
| **Tarea del usuario** | Reconocer que NO hay tendencia, y operar los extremos del rango o abstenerse — abstenerse también puntúa. |
| **Learning Context Contract** | Objetivo: distinguir rango de tendencia y aprender que "no operar" es una decisión válida y a veces la mejor. Garantía: el rango se mantiene la mayor parte de la sesión; los extremos se tocan ≥3 veces. |
| **Template** | `TPL_RANGE` |
| **Tipo de seed** | Fija en lección; aleatoria en sandbox. |
| **Comportamiento de seed** | Igual que Escenario 2. |
| **Path de mercado** | Pre-generado completo. |
| **Trampa oculta** | Dos falsas salidas menores del rango (mechas que asoman fuera y vuelven), tentando a operar "rupturas" inexistentes. |
| **Error esperado** | Comprar la falsa ruptura del techo; operar en medio del rango sin referencia; sobreoperar por aburrimiento. |
| **Decisiones buenas (aunque pierdan)** | Vender cerca de resistencia / comprar cerca de soporte con stop fuera del rango; decidir no operar y documentarlo en el journal. |
| **Decisiones malas (aunque ganen)** | Operar la falsa ruptura y ganar por rebote casual; abrir >5 trades por aburrimiento. |
| **Condición de éxito** | Cero entradas en falsas rupturas, o entradas solo en extremos con stop; o abstención justificada en journal. |
| **Condición de fracaso** | ≥2 entradas en rupturas falsas, o `ERR_OVERTRADE`. |
| **Métricas** | Ubicación de entradas respecto a extremos del rango, número de trades, journal de abstención. |
| **Feedback** | "Este día no había tendencia. El mercado tocó el techo 4 veces y el piso 3. Si operaste las falsas rupturas: eso que sentiste — 'ahora sí se escapa' — es exactamente lo que el mercado usa para atrapar principiantes." |
| **Replay** | Mismo rango, mismas falsas salidas. |
| **Desbloqueo** | Escenario 2 completado. |
| **Modo libre** | Sí. |
| **Histórico-inspirado a futuro** | Sí. |
| **Fase** | **MVP** |

---

### Escenario 4 — La ruptura falsa (fake breakout)

| Campo | Especificación |
|---|---|
| **Título** | "La ruptura que no era" |
| **Nivel de usuario** | 3 |
| **Tipo de mercado** | Cripto sintética (volatilidad media-alta) |
| **Velocidad recomendada** | Media; sin pausa automática (la trampa debe sentirse en tiempo real) |
| **Setup** | Rango comprimido prolongado → ruptura alcista convincente con volumen → reversión violenta que devuelve todo el movimiento. |
| **Tarea del usuario** | Operar la sesión. El escenario no avisa que la ruptura será falsa. |
| **Learning Context Contract** | Objetivo: vivir una fake breakout y aprender que el stop loss es lo único que separa una pérdida pequeña de una catástrofe. Garantía: la ruptura falsa ocurre entre el 40% y 60% de la sesión; la reversión es lo bastante rápida para castigar posiciones sin stop. |
| **Template** | `TPL_FAKE_BREAKOUT` |
| **Tipo de seed** | **Fija (lección y challenge)**; variantes aleatorias en sandbox una vez completada la lección. |
| **Comportamiento de seed** | La versión lección es idéntica para todos: la trampa es la misma trampa para todo el mundo. |
| **Path de mercado** | Pre-generado completo. La reversión NO se dispara por la entrada del usuario — ya estaba escrita. El feedback lo dice explícitamente para combatir el sesgo de "el mercado me vio entrar". |
| **Trampa oculta** | La ruptura misma. Trampa secundaria: tras la reversión, un rebote parcial tienta a "promediar" la pérdida. |
| **Error esperado** | Comprar la ruptura sin stop; mover el stop hacia abajo durante la reversión; promediar en contra. |
| **Decisiones buenas (aunque pierdan)** | Comprar la ruptura CON stop loss y aceptar la pérdida cuando se ejecuta — esto se califica explícitamente como buen trading: era un setup razonable que falló. Esperar confirmación y no entrar también es válido. |
| **Decisiones malas (aunque ganen)** | Vender en corto por corazonada antes de la ruptura sin setup (acertó de casualidad); quedarse sin stop y que el rebote parcial lo salve. |
| **Condición de éxito** | Si entró: stop definido y respetado, pérdida ≤ riesgo planeado. Si no entró: journal con razón. |
| **Condición de fracaso** | `ERR_NO_SL` o `ERR_SL_MOVED` durante la reversión; promediar la posición perdedora. |
| **Métricas** | Uso y respeto del stop, pérdida real vs. pérdida planeada, conducta post-reversión. |
| **Feedback** | "La ruptura era falsa, y aquí está lo importante: **el mercado ya había decidido este camino antes de que tú entraras.** Tu entrada no fue mala — comprar una ruptura es un setup legítimo. La diferencia entre perder 2% y perder 30% fue tu stop loss. Si lo respetaste, hoy operaste como profesional, aunque hayas perdido." |
| **Replay** | Misma trampa. Replay recomendado tras leer el feedback. |
| **Desbloqueo** | Escenario 3 completado. |
| **Modo libre** | Sí (variantes aleatorias). |
| **Histórico-inspirado a futuro** | Sí — fake breakouts históricas famosas son ideales. |
| **Fase** | **MVP** |

---

### Escenario 5 — Spike de noticia

| Campo | Especificación |
|---|---|
| **Título** | "El mercado explota: spike de noticia" |
| **Nivel de usuario** | 4 |
| **Tipo de mercado** | Forex sintético con calendario de eventos simulado |
| **Velocidad recomendada** | Media, con aviso de "evento programado en X velas" (parte de la lección: las noticias se anuncian) |
| **Setup** | Mercado tranquilo → evento de noticia → spike violento (dirección sellada en el path), spread se multiplica ×5–×10 durante el evento, slippage alto. |
| **Tarea del usuario** | Decidir cómo manejar el evento: cerrar antes, reducir tamaño, no operar durante el spike, o asumir el riesgo conscientemente. |
| **Learning Context Contract** | Objetivo: entender que durante noticias el spread y el slippage destrozan las condiciones normales, y que "saber la dirección" no basta. Garantía: el spike ocurre en el momento anunciado; las condiciones de ejecución se degradan de verdad (órdenes ejecutan con slippage severo). |
| **Template** | `TPL_NEWS_SPIKE` |
| **Tipo de seed** | Fija en lección; aleatoria en sandbox (dirección del spike aleatoria por seed). |
| **Comportamiento de seed** | La dirección del spike está en la seed — pre-decidida, no reactiva al usuario. |
| **Path de mercado** | Pre-generado, incluyendo la curva de spread y los parámetros de slippage por tramo. |
| **Trampa oculta** | El spike inicial revierte parcialmente: quien entra persiguiendo el movimiento queda atrapado en el retroceso, pagando spread ancho dos veces. |
| **Error esperado** | Entrar a mercado en plena vela de noticia; sorprenderse de que el precio ejecutado sea mucho peor que el visible (slippage); dejar una posición abierta con stop ajustado que el spread ampliado ejecuta. |
| **Decisiones buenas (aunque pierdan)** | Cerrar o reducir antes del evento; abstenerse durante el spike; si decide operar el evento, hacerlo con tamaño reducido y stop amplio consciente. |
| **Decisiones malas (aunque ganen)** | Entrar a mercado dentro del spike y ganar por suerte direccional — el evaluador muestra cuánto pagó de spread+slippage y cuánto habría perdido con la dirección contraria. |
| **Condición de éxito** | Gestión consciente del evento (cierre previo, reducción, o abstención documentada); sin `ERR_CHASE` dentro de la ventana del spike. |
| **Condición de fracaso** | Entrada a mercado en la ventana de spike con tamaño normal o mayor; cuenta con pérdida > 5% por el evento. |
| **Métricas** | Exposición durante el evento, costo de fricción pagado (spread+slippage), conducta pre-evento. |
| **Feedback** | "Mira tu precio de ejecución contra el precio que viste en pantalla: esa diferencia es el slippage. Durante noticias, el spread se multiplicó por 8. Aunque hubieras acertado la dirección, las condiciones de ejecución se comen al principiante. Los profesionales no predicen la noticia: gestionan la exposición antes de que llegue." |
| **Replay** | Mismo evento, mismo timing, mismas condiciones de fricción. |
| **Desbloqueo** | Escenario 4 completado. |
| **Modo libre** | Sí. |
| **Histórico-inspirado a futuro** | Sí — NFP, decisiones de tasas, etc., como inspiración. |
| **Fase** | **MVP** |

---

### Escenario 6 — Ambiente de spread alto

| Campo | Especificación |
|---|---|
| **Título** | "Cuando entrar ya cuesta caro" |
| **Nivel de usuario** | 4 |
| **Tipo de mercado** | `synthetic_fx` con perfil de spread estructural alto aplicado por el escenario *(no es un instrumento adicional del catálogo: `MVP_MARKET_LOCK` limita el MVP a 3 instrumentos; el "par exótico" es un perfil de escenario)* |
| **Velocidad recomendada** | Media-alta |
| **Setup** | Mercado operable con estructura normal, pero spread 5–8 veces mayor que en escenarios previos, visible en el panel. |
| **Tarea del usuario** | Operar una sesión completa siendo rentable DESPUÉS de costos, o concluir que el instrumento no compensa. |
| **Learning Context Contract** | Objetivo: internalizar que el spread es un costo real que mata estrategias de movimientos cortos. Garantía: el path contiene movimientos cortos (que el spread vuelve perdedores) y 1–2 movimientos amplios (que sí lo cubren). |
| **Template** | `TPL_HIGH_SPREAD` sobre base `TPL_RANGE`/`TPL_TREND` mixta |
| **Tipo de seed** | Fija en lección; aleatoria en sandbox. |
| **Path de mercado** | Pre-generado, con curva de spread por hora incluida en el path. |
| **Trampa oculta** | Varios "mini-setups" atractivos cuyo recorrido potencial es menor que el spread de ida y vuelta: operarlos garantiza pérdida por costos aunque se acierte la dirección. |
| **Error esperado** | Scalpear movimientos cortos ignorando el costo; no mirar nunca el panel de spread; frustrarse y sobreoperar para "recuperar las comisiones". |
| **Decisiones buenas (aunque pierdan)** | Filtrar trades cuyo objetivo no cubre ≥3× el spread; reducir frecuencia; anotar en journal que el instrumento es caro de operar. |
| **Decisiones malas (aunque ganen)** | Scalps que por azar ganan más que el costo una vez — el evaluador suma el costo acumulado de la sesión y lo proyecta: "a este ritmo, en 100 trades el spread se lleva X% de tu cuenta". |
| **Condición de éxito** | Costo de fricción total ≤ umbral del escenario; ratio objetivo/spread ≥3 en las entradas tomadas. |
| **Condición de fracaso** | Fricción acumulada > 10% del capital; `ERR_OVERTRADE`. |
| **Métricas** | Costo total pagado en spread, ratio recorrido-objetivo/spread por trade, frecuencia de trades. |
| **Feedback** | Desglose explícito: P/L bruto vs. P/L neto tras costos. "Ganaste 22 USD en movimientos y pagaste 31 USD de spread. El mercado no te ganó: te ganaron los costos." |
| **Replay** | Mismas condiciones; útil para reintentar con filtro de costos. |
| **Desbloqueo** | Escenario 5 completado. |
| **Modo libre** | Sí. |
| **Histórico-inspirado a futuro** | Sí (pares exóticos, horarios nocturnos). |
| **Fase** | **MVP** |

---

### Escenario 7 — Sesión de baja liquidez

| Campo | Especificación |
|---|---|
| **Título** | "El mercado fantasma: baja liquidez" |
| **Nivel de usuario** | 5 |
| **Tipo de mercado** | Cripto sintética en horario muerto / acción sintética de baja capitalización |
| **Velocidad recomendada** | Media |
| **Setup** | Sesión con liquidez mínima: velas erráticas con mechas largas, gaps pequeños, slippage alto incluso sin noticias, movimientos bruscos sin razón aparente. |
| **Tarea del usuario** | Operar (o no) en condiciones donde entrar y salir mueve el precio contra uno mismo. |
| **Learning Context Contract** | Objetivo: entender liquidez como "qué tan fácil entras y sales sin mover el precio en tu contra", y reconocer cuándo el mercado no es operable. Garantía: slippage estructural alto, ≥2 movimientos erráticos sin estructura, mechas que barren stops razonables. |
| **Template** | `TPL_LOW_LIQUIDITY` |
| **Tipo de seed** | Fija en lección; aleatoria en sandbox. |
| **Path de mercado** | Pre-generado, incluida la curva de profundidad de liquidez que alimenta el modelo de slippage. |
| **Trampa oculta** | Una mecha de barrido que ejecuta stops colocados en niveles "obvios" y devuelve el precio al punto de partida; un movimiento errático tentador que no tiene continuación. |
| **Error esperado** | Operar con el tamaño habitual sin ajustar por liquidez; interpretar ruido como señal; indignarse por el barrido del stop y reentrar más grande. |
| **Decisiones buenas (aunque pierdan)** | Reducir tamaño drásticamente o abstenerse; usar órdenes límite en vez de mercado; aceptar el stop barrido sin revancha. |
| **Decisiones malas (aunque ganen)** | Operar tamaño completo a mercado y que el azar acompañe; reentrada de revancha tras el barrido que casualmente funciona. |
| **Condición de éxito** | Tamaño ajustado a la liquidez (o abstención documentada); cero `ERR_REVENGE` tras el barrido. |
| **Condición de fracaso** | Slippage acumulado > umbral; `ERR_REVENGE` tras el barrido de stops. |
| **Métricas** | Slippage pagado, tipo de órdenes usadas, tamaño relativo a liquidez, conducta post-barrido. |
| **Feedback** | "¿Notaste que tus órdenes a mercado ejecutaron lejos del precio visible? Eso es baja liquidez: no hay suficientes compradores y vendedores. En mercados así, el primer ajuste no es la estrategia: es el tamaño. Y a veces, la mejor operación es no operar." |
| **Replay** | Mismas condiciones erráticas. |
| **Desbloqueo** | Escenario 6 completado. |
| **Modo libre** | Sí. |
| **Histórico-inspirado a futuro** | Sí. |
| **Fase** | **MVP** |

---

### Escenario 8 — La señal copiada que salió mal

| Campo | Especificación |
|---|---|
| **Título** | "La señal del grupo" |
| **Nivel de usuario** | 5 |
| **Tipo de mercado** | Cripto sintética (el hábitat natural de las señales de Telegram/TikTok en LATAM) |
| **Velocidad recomendada** | Media |
| **Setup** | El escenario presenta una "señal" simulada estilo grupo de Telegram: "COMPRA YA 🚀 entrada X, objetivo +40%, no puede fallar". Sin razón técnica, sin stop sugerido, con urgencia artificial. El path: subida breve inicial (la señal "parece funcionar") seguida de caída sostenida. |
| **Tarea del usuario** | Decidir qué hacer con la señal. No hay opción correcta única: seguirla con gestión propia, ignorarla, o analizar el gráfico por cuenta propia. |
| **Learning Context Contract** | Objetivo: experimentar el patrón real de las señales copiadas — entrada tardía, sin stop, sin contexto — y aprender que ejecutar la idea de otro sin gestión propia es apostar, no operar. Garantía: la subida inicial existe (refuerza la ilusión), la caída posterior es sostenida y profunda. Trampa conductual central del escenario. |
| **Template** | `TPL_TRAP_SEQUENCE` (variante señal externa) |
| **Tipo de seed** | **Fija (lección)** — la trampa debe ser idéntica para todos; el feedback depende de su timing exacto. |
| **Path de mercado** | Pre-generado. La caída ocurre independientemente de si el usuario siguió la señal — y el feedback lo subraya. |
| **Trampa oculta** | La subida inicial valida la señal el tiempo justo para que el usuario aumente tamaño o quite el stop "porque va bien". Trampa secundaria: durante la caída, una "segunda señal" del grupo dice "promedia, es descuento 🔥". |
| **Error esperado** | Entrar por urgencia sin stop; promediar la caída obedeciendo la segunda señal; culpar a la señal y no al proceso. |
| **Decisiones buenas (aunque pierdan)** | Seguir la señal PERO con stop propio y tamaño ≤2% (se califica bien: convirtió una apuesta en un trade gestionado); ignorarla y documentar por qué; analizar el gráfico antes de decidir. |
| **Decisiones malas (aunque ganen)** | Entrar sin stop y salir con ganancia en la subida inicial por casualidad — el evaluador ejecuta la lección clave: "esta vez saliste antes de la caída; el grupo no te avisó cuándo salir, fue azar". |
| **Condición de éxito** | Cualquier decisión con gestión propia: stop definido, tamaño controlado, o abstención razonada. |
| **Condición de fracaso** | Entrada sin stop por la señal; promediar con la segunda señal; pérdida > 10% del capital. |
| **Métricas** | Presencia de gestión propia sobre la señal, respuesta a la segunda señal, journal. |
| **Feedback** | "La señal no incluía stop loss, ni tamaño, ni cuándo salir. Eso no es una estrategia: es una orden de otra persona sobre tu dinero. Fíjate en algo más: quien publicó la señal no pierde nada si tú pierdes. Si decides seguir ideas externas alguna vez, la gestión del riesgo sigue siendo tuya. Siempre." |
| **Replay** | Misma señal, mismo path. |
| **Desbloqueo** | Escenario 4 completado (requiere haber vivido una trampa de precio antes). |
| **Modo libre** | Sí. |
| **Histórico-inspirado a futuro** | Sí — pumps históricos de cripto son inspiración directa. |
| **Fase** | **MVP** |

---

### Escenario 9 — La operación sobreapalancada

| Campo | Especificación |
|---|---|
| **Título** | "Apalancamiento: la espada de doble filo" |
| **Nivel de usuario** | 6 |
| **Tipo de mercado** | Forex sintético con apalancamiento disponible hasta 1:500 (deliberadamente excesivo, como ofrecen brokers dirigidos a LATAM) |
| **Velocidad recomendada** | Media |
| **Setup** | Cuenta pequeña (100 USD). El panel permite elegir apalancamiento libremente. El path: movimiento normal con una oscilación intermedia de ~1.5% en contra antes de resolverse. |
| **Tarea del usuario** | Tomar un trade direccional eligiendo el apalancamiento. La dirección "correcta" es alcanzable; la pregunta del escenario es el tamaño. |
| **Learning Context Contract** | Objetivo: vivir cómo el apalancamiento convierte una oscilación normal en una liquidación. Garantía: el path ya contiene una oscilación intermedia de ~1.5% en contra, sellada antes de la decisión del usuario; la consecuencia depende de la ejecución — por matemática de margen, una posición con leverage efectivo >1:60 aproximadamente no la sobrevive, aunque la dirección final fuera correcta. |
| **Template** | `TPL_TREND` con oscilación calibrada |
| **Tipo de seed** | **Fija (lección)** — la magnitud de la oscilación (~1.5%) debe ser exacta y reproducible para que la lección de margen sea idéntica para todos; el path no conoce ni ajusta nada del usuario. |
| **Path de mercado** | Pre-generado. La oscilación NO busca al usuario: está escrita; es la matemática del leverage la que decide quién sobrevive. |
| **Trampa oculta** | La interfaz "ofrece" el apalancamiento alto sin advertencias (como un broker real). La oscilación intermedia es la trampa: tener razón en la dirección no salva al sobreapalancado. |
| **Error esperado** | Elegir leverage máximo "porque la cuenta es chica y quiero que rinda"; no calcular el precio de liquidación; confundir leverage alto con ambición legítima. |
| **Decisiones buenas (aunque pierdan)** | Leverage moderado con riesgo ≤2% y stop propio antes del nivel de liquidación; calcular cuánto movimiento en contra soporta la posición antes de entrar. |
| **Decisiones malas (aunque ganen)** | En este path no existe "ganar" sobreapalancado (por matemática de margen, la oscilación ya contenida en el path liquida primero) — pero si el usuario usa leverage alto con tamaño minúsculo y sobrevive técnicamente, el evaluador marca igual el hábito. |
| **Condición de éxito** | Sobrevivir la oscilación: leverage efectivo y stop tales que la posición aguanta el 1.5% en contra. |
| **Condición de fracaso** | Liquidación de la posición; pérdida > 50% del capital en un trade. |
| **Métricas** | Leverage efectivo, distancia al precio de liquidación al entrar, riesgo real por trade. |
| **Feedback** | "Tenías razón en la dirección. El precio terminó donde pensaste. Y aun así la cuenta quedó liquidada antes, en la oscilación intermedia. Eso es el apalancamiento: controlas una posición grande con poco capital, pero cada movimiento en contra se multiplica igual que cada movimiento a favor. Tener razón no basta si no sobrevives hasta tener razón." |
| **Replay** | Mismo path; reintentar con leverage distinto es exactamente el ejercicio. |
| **Desbloqueo** | Escenario 5 completado. |
| **Modo libre** | Sí. |
| **Histórico-inspirado a futuro** | Sí. |
| **Fase** | **MVP** — única excepción de leverage del MVP: corre con **parámetros fijos empaquetados** (leverage definido por el escenario, fórmula de liquidación simplificada y determinista, sin margin engine general), según `LEVERAGE_MVP_LIMITS` (documento 12, sección 5). |

---

### Escenario 10 — La trampa del revenge trading

| Campo | Especificación |
|---|---|
| **Título** | "Después de la pérdida" |
| **Nivel de usuario** | 6 |
| **Tipo de mercado** | Acción sintética intradía *(reasignado desde índice sintético: el índice es post-MVP según `MVP_MARKET_LOCK`, documento 12)* |
| **Velocidad recomendada** | Media; **pausa deshabilitada en la ventana post-pérdida** (la presión emocional es parte del diseño) |
| **Setup** | El path ya contiene un primer setup razonable que falla (pérdida limpia con stop). Inmediatamente después aparecen 2–3 "oportunidades" de calidad mediocre, tentadoras, antes de que llegue un segundo setup genuino más tarde. |
| **Tarea del usuario** | Operar la sesión completa. El verdadero examen empieza después de la primera pérdida. |
| **Learning Context Contract** | Objetivo: reconocer y resistir el impulso de recuperar la pérdida inmediatamente. Garantía: la primera pérdida ocurre ante buen proceso (no es culpa del usuario); los anzuelos post-pérdida tienen estructura deficiente; el setup genuino tardío existe y es identificable. |
| **Template** | `TPL_TRAP_SEQUENCE` (variante post-pérdida) |
| **Tipo de seed** | **Fija (lección)** — la secuencia pérdida → anzuelos → setup real debe ser idéntica para todos. |
| **Path de mercado** | Pre-generado. La primera pérdida estaba escrita: ningún proceso la evitaba. El feedback lo declara para enseñar que pérdidas con buen proceso son parte del juego. |
| **Trampa oculta** | Los anzuelos post-pérdida: parecen oportunidades pero carecen de estructura; uno de ellos incluso "funcionaría" brevemente para reforzar el mal hábito de quien cae. |
| **Error esperado** | Reentrar en <5 velas tras la pérdida con tamaño igual o mayor; duplicar tamaño "para recuperar"; encadenar 3+ trades impulsivos. |
| **Decisiones buenas (aunque pierdan)** | Pausar tras la pérdida (el evaluador mide las velas de espera); mantener o reducir el tamaño en el siguiente trade; esperar al setup genuino aunque también pierda. |
| **Decisiones malas (aunque ganen)** | Trade de revancha en un anzuelo que casualmente gana — el feedback es directo: "recuperaste la pérdida con el peor hábito que existe; el mercado te acaba de pagar por aprender algo falso". |
| **Condición de éxito** | Cero `ERR_REVENGE`; tamaño post-pérdida ≤ tamaño previo; espera mínima respetada o journal entre trades. |
| **Condición de fracaso** | `ERR_REVENGE` detectado; pérdida acumulada > 8% del capital en la secuencia post-pérdida. |
| **Métricas** | Tiempo entre pérdida y siguiente trade, evolución del tamaño, calidad del setup elegido post-pérdida, drawdown de la secuencia. |
| **Feedback** | "Tu primera pérdida fue con buen proceso: entraste bien y el mercado fue para el otro lado. Eso va a pasar miles de veces en tu vida como trader. Lo que define tu futuro no es esa pérdida: es lo que hiciste en las velas siguientes. El deseo de recuperar 'ya' es la trampa más cara del trading." |
| **Replay** | Misma secuencia; replay recomendado para practicar la pausa. |
| **Desbloqueo** | Escenario 9 completado. |
| **Modo libre** | Sí. |
| **Histórico-inspirado a futuro** | Parcial — la secuencia conductual es sintética por diseño. |
| **Fase** | **MVP** |

---

### Escenario 11 — Gestión de riesgo correcta

| Campo | Especificación |
|---|---|
| **Título** | "El 1% que te mantiene vivo" |
| **Nivel de usuario** | 7 |
| **Tipo de mercado** | Forex sintético, multi-sesión (5 días simulados) |
| **Velocidad recomendada** | Alta (velas comprimidas), con desaceleración en zonas de decisión |
| **Setup** | Semana simulada con ~8 setups de calidad variada. El path garantiza que incluso eligiendo bien, 3–4 de los trades pierden: una semana realista. |
| **Tarea del usuario** | Operar la semana completa arriesgando un % fijo y consistente por trade (la lección propone 1–2%), sobreviviendo la racha perdedora intermedia. |
| **Learning Context Contract** | Objetivo: demostrar matemáticamente que con riesgo fijo pequeño, una racha perdedora normal no destruye la cuenta. Garantía: el path contiene una racha de 3 pérdidas consecutivas en la mitad de la semana, y termina con setups favorables que solo aprovecha quien aún tiene capital y calma. |
| **Template** | `TPL_GRIND` (corto, 5 días) |
| **Tipo de seed** | Fija en lección; aleatoria en sandbox manteniendo la garantía estructural (racha perdedora presente). |
| **Path de mercado** | Pre-generado completo (los 5 días). |
| **Trampa oculta** | La racha de 3 pérdidas seguidas: matemáticamente inofensiva al 1% (−3%), devastadora al 15% por trade (−38%). El path castiga la inconsistencia de tamaño, no la dirección. |
| **Error esperado** | Aumentar el riesgo tras pérdidas para recuperar; reducirlo a casi cero tras pérdidas y perderse los setups finales por miedo; riesgo inconsistente entre trades. |
| **Decisiones buenas (aunque pierdan)** | Riesgo constante (±0.5% de variación) en TODOS los trades, incluida la racha perdedora; respetar stops; seguir operando los setups finales con el mismo tamaño. |
| **Decisiones malas (aunque ganen)** | Una semana ganadora lograda con riesgo errático (1%, luego 8%, luego 3%) — el evaluador muestra la simulación contrafactual: "con esta inconsistencia, en 20 semanas tu probabilidad de ruina es X%". |
| **Condición de éxito** | Desviación estándar del riesgo por trade ≤ umbral; drawdown máximo ≤ 6%; superviviente al final de la semana. |
| **Condición de fracaso** | Riesgo por trade > 3% en cualquier operación; drawdown > 15%. |
| **Métricas** | Consistencia del % de riesgo, drawdown máximo, conducta durante la racha, participación en los setups finales. |
| **Feedback** | Incluye una tabla comparativa generada de la propia sesión: "así quedó tu cuenta — y así habría quedado arriesgando 10% por trade en esta misma semana exacta". La comparación usa el mismo path: la diferencia es solo la gestión. |
| **Replay** | Misma semana; ideal para repetir con disciplina de tamaño distinta y comparar. |
| **Desbloqueo** | Escenario 10 completado. |
| **Modo libre** | Sí. |
| **Histórico-inspirado a futuro** | Sí. |
| **Fase** | **MVP** |

---

### Escenario 12 — Interés compuesto a largo plazo

| Campo | Especificación |
|---|---|
| **Título** | "El juego largo: capitalización compuesta" |
| **Nivel de usuario** | 8 |
| **Tipo de mercado** | Índice sintético, horizonte de 12 meses simulados (velas diarias/semanales) |
| **Velocidad recomendada** | Muy alta (meses en minutos), con pausas en eventos clave |
| **Setup** | Un año de mercado multi-régimen: tendencias, correcciones, un susto fuerte intermedio (−15% del índice). El usuario gestiona una cuenta con aportes simulados mensuales opcionales. |
| **Tarea del usuario** | Hacer crecer la cuenta durante 12 meses con decisiones de exposición y riesgo, no de scalping: cuánto exponer, cuándo reducir, cómo reaccionar a la corrección. |
| **Learning Context Contract** | Objetivo: experimentar que el crecimiento real es compuesto y lento, y que evitar pérdidas grandes importa más que lograr ganancias grandes. Garantía: la corrección intermedia ocurre; recuperarse de ella distingue al disciplinado del apostador. |
| **Template** | `TPL_GRIND` (12 meses) |
| **Tipo de seed** | Fija en lección; aleatoria en sandbox/challenge anual. |
| **Path de mercado** | Pre-generado completo (todo el año, antes de la primera decisión). |
| **Trampa oculta** | Tras la corrección del −15%, el rebote inicial es parcial y vuelve a caer levemente antes de recuperar: castiga tanto al que vendió todo en pánico en el fondo como al que apostó todo al primer rebote. |
| **Error esperado** | Buscar "el gran trade" en lugar de consistencia; vender todo en el pánico del fondo; aburrirse y sobreoperar el mercado lateral de mitad de año. |
| **Decisiones buenas (aunque pierdan)** | Exposición gradual y consistente; reducir riesgo sin salir del todo durante la corrección; mantener el plan durante el aburrimiento. |
| **Decisiones malas (aunque ganen)** | All-in en el momento justo por azar — el evaluador muestra el cono de resultados: "tu decisión tenía un 30% de probabilidad de pérdida >40% en este tipo de path". |
| **Condición de éxito** | Terminar el año con crecimiento positivo Y drawdown máximo ≤ 20% Y sin `ERR_ALL_IN`. |
| **Condición de fracaso** | Drawdown > 35%; cuenta final < 70% del capital inicial + aportes. |
| **Métricas** | CAGR simulado, drawdown máximo, consistencia de exposición, conducta en la corrección, número de decisiones impulsivas. |
| **Feedback** | "Tu cuenta creció X% en el año. Fíjate en la curva: la mayor parte del crecimiento vino de NO perder grande en marzo, no de ningún trade brillante. El interés compuesto solo trabaja para quien sobrevive las caídas." |
| **Replay** | Mismo año; comparar curvas de equity entre intentos es la herramienta didáctica central. |
| **Desbloqueo** | Escenario 11 completado. |
| **Modo libre** | Sí. |
| **Histórico-inspirado a futuro** | Sí — años históricos completos son material ideal. |
| **Fase** | **Posterior al MVP** (requiere simulación acelerada multi-mes madura) |

---

### Escenario 13 — Challenge: Supervivencia de 3 meses

| Campo | Especificación |
|---|---|
| **Título** | "Challenge: Sobrevive 90 días" |
| **Nivel de usuario** | 8 (challenge con ranking) |
| **Tipo de mercado** | Mixto sintético (forex + índice), 3 meses simulados |
| **Velocidad recomendada** | Alta, sesiones diarias comprimidas; el challenge puede jugarse en varias sesiones de app (estado persistido localmente) |
| **Setup** | 90 días de mercado hostil: dos rachas perdedoras, un news spike, una semana de spread alto, y pocos setups de calidad. Capital inicial 500 USD. **Regla dura: si el drawdown supera 25%, el challenge termina.** |
| **Tarea del usuario** | Llegar al día 90 vivo. El score premia supervivencia y proceso, no la ganancia. |
| **Learning Context Contract** | Objetivo: consolidar todas las disciplinas previas bajo presión prolongada. Garantía: el path contiene al menos un evento de cada trampa ya aprendida (fake breakout, spike, anzuelos de revancha); ninguna es nueva — el challenge examina lo aprendido. |
| **Template** | `TPL_GRIND` (90 días, hostilidad alta) con sub-eventos de `TPL_FAKE_BREAKOUT`, `TPL_NEWS_SPIKE`, `TPL_HIGH_SPREAD` |
| **Tipo de seed** | **Fija por temporada de challenge** — todos los participantes de la temporada juegan exactamente el mismo mercado; el ranking de sesión es comparable de forma justa. |
| **Comportamiento de seed** | Cada "temporada" del challenge tiene su seed sellada y versionada. Replay usa la seed de la temporada jugada. |
| **Path de mercado** | Pre-generado completo (los 90 días) antes del día 1. |
| **Trampa oculta** | La hostilidad sostenida misma: el path ya contiene pocos setups de calidad y fricción alta; con esa estructura, la estrategia "ganar mucho rápido" rompe el límite de drawdown en casi cualquier ejecución. Sobrevivir exige operar poco y bien. |
| **Error esperado** | Intentar "ganar el challenge" maximizando retorno en vez de supervivencia; relajar la disciplina en las semanas tranquilas. |
| **Decisiones buenas (aunque pierdan)** | Semanas sin operar cuando no hay setups; riesgo constante ≤1.5%; reducir exposición ante los eventos reconocibles. |
| **Decisiones malas (aunque ganen)** | Sobrevivir con sustos de drawdown del 20% por apuestas grandes — el score de proceso lo penaliza aunque haya "pasado". |
| **Condición de éxito** | Llegar al día 90 con drawdown máximo < 25%. Niveles de logro: Bronce (sobrevivió), Plata (sobrevivió con dd < 15%), Oro (sobrevivió con dd < 10% y cero errores graves). |
| **Condición de fracaso** | Drawdown ≥ 25% en cualquier momento → fin del challenge (puede reintentar la temporada desde cero). |
| **Métricas / Score de ranking** | Score compuesto: supervivencia (peso mayor) + inverso del drawdown máximo + consistencia de riesgo + errores graves (penalización) + retorno (peso menor, deliberadamente). |
| **Feedback** | Informe de campaña completo: curva de equity, mapa de errores sobre la línea de tiempo, comparación contra los percentiles del ranking local de sesiones. |
| **Replay** | Replay completo de los 90 días con las decisiones marcadas sobre el path. High score guardado por temporada. |
| **Desbloqueo** | Escenarios 1–11 completados. |
| **Modo libre** | No — solo como challenge formal (su valor depende de las reglas duras). |
| **Histórico-inspirado a futuro** | Sí — "sobrevive el 2008" como temporada futura. |
| **Fase** | **Posterior al MVP** — movido por `MVP_CONTENT_LOCK` y `MVP_SANDBOX_LIMITS` (documento 12): el horizonte de 90 días excede el presupuesto de simulación del MVP. El rol de challenge de supervivencia del MVP lo cubre **"Supervivencia 50 velas"** (documento 12, sección 8), de horizonte corto. Este escenario regresa como challenge insignia en la fase de horizontes largos. |

---

### Escenario 14 — Challenge: Crecimiento de portafolio a 1 año

| Campo | Especificación |
|---|---|
| **Título** | "Challenge: El año del portafolio" |
| **Nivel de usuario** | 9 |
| **Tipo de mercado** | Multi-instrumento sintético (índice + forex + cripto), 12 meses simulados |
| **Velocidad recomendada** | Muy alta, jugable en múltiples sesiones de app |
| **Setup** | Cuenta de 1.000 USD repartible entre 3 instrumentos con perfiles distintos de volatilidad y correlación parcial. Un año multi-régimen con una crisis intermedia que golpea a los tres a la vez (las correlaciones suben en la crisis — lección incluida). |
| **Tarea del usuario** | Maximizar el score (no el retorno bruto) gestionando asignación, exposición y riesgo durante 12 meses. |
| **Learning Context Contract** | Objetivo: introducir gestión de portafolio — diversificación, correlación, rebalanceo — y mostrar que la diversificación reduce pero no elimina el riesgo. Garantía: la crisis intermedia ocurre y golpea a todos los instrumentos; el path premia la exposición controlada, no la predicción. |
| **Template** | `TPL_GRIND` multi-instrumento con paths correlacionados generados desde la misma seed maestra |
| **Tipo de seed** | **Fija por temporada** (ranking comparable); los 3 paths derivan de una seed maestra → determinismo total del conjunto. |
| **Path de mercado** | Los 3 paths y su estructura de correlación se pre-generan completos antes del día 1. |
| **Trampa oculta** | La falsa seguridad de la diversificación: en la crisis los tres instrumentos caen juntos. Quien creyó que diversificar lo hacía inmune sufre la lección central del escenario. |
| **Error esperado** | Concentrarlo todo en el instrumento "que va mejor" (cripto, típicamente); ignorar el rebalanceo; pánico vendiendo todo en la crisis. |
| **Decisiones buenas (aunque pierdan)** | Asignación repartida con límites por instrumento; rebalanceo periódico; reducción de exposición global (no liquidación total) en la crisis. |
| **Decisiones malas (aunque ganen)** | All-in al instrumento más volátil que por azar termina arriba — el score de proceso lo penaliza fuerte y el feedback muestra los caminos contrafactuales. |
| **Condición de éxito** | Terminar el año con score de proceso ≥ umbral y drawdown de portafolio ≤ 25%. |
| **Condición de fracaso** | Drawdown > 40%; concentración >70% en un instrumento durante más de un mes. |
| **Métricas / Score** | Retorno ajustado por drawdown, diversificación efectiva media, conducta en crisis, consistencia de rebalanceo, errores graves. |
| **Feedback** | "En marzo, tus tres mercados cayeron juntos. La diversificación amortiguó tu caída (−12% contra −19% del peor instrumento), pero no la evitó. Diversificar no es un escudo: es un amortiguador." |
| **Replay** | Año completo reproducible; high score por temporada. |
| **Desbloqueo** | Challenge 13 completado (cualquier nivel de logro). |
| **Modo libre** | No — solo challenge formal. |
| **Histórico-inspirado a futuro** | Sí — años de crisis reales como temporadas. |
| **Fase** | **Posterior al MVP** |

---

### Escenario 15 — Challenge: Price action sin indicadores

| Campo | Especificación |
|---|---|
| **Título** | "Challenge: Solo velas" |
| **Nivel de usuario** | 7 |
| **Tipo de mercado** | Forex sintético intradía, 3 sesiones |
| **Velocidad recomendada** | Media |
| **Setup** | **Todos los indicadores están bloqueados.** Solo gráfico de velas (`#4A6D56` / `#802F3E`), niveles que el usuario dibuje, y el panel de cuenta. Tres sesiones: una de tendencia, una de rango, una mixta. |
| **Tarea del usuario** | Operar tres sesiones leyendo solo estructura de precio: máximos y mínimos, soportes y resistencias, comportamiento de las velas. |
| **Learning Context Contract** | Objetivo: demostrar que la estructura del precio es legible sin indicadores y romper la dependencia del principiante de "la línea mágica". Garantía: las tres sesiones tienen estructura clara y legible a simple vista; las trampas de los escenarios 2–4 reaparecen en versión suave. |
| **Template** | `TPL_TREND` + `TPL_RANGE` + mixto, modo no-indicadores activado |
| **Tipo de seed** | Fija por temporada de challenge (ranking); aleatoria en modo práctica del challenge. |
| **Path de mercado** | Pre-generado completo para las tres sesiones. |
| **Trampa oculta** | Una fake breakout suave en la sesión mixta; sin indicadores, el usuario debe detectarla por estructura (mecha de rechazo, falta de continuación). |
| **Error esperado** | Sentirse "ciego" sin indicadores y no operar nada, o lo contrario: operar cada vela por ansiedad; ignorar los niveles que él mismo dibujó. |
| **Decisiones buenas (aunque pierdan)** | Dibujar niveles antes de operar; entradas en niveles propios con stop estructural; respetar el propio análisis. |
| **Decisiones malas (aunque ganen)** | Entradas aleatorias sin niveles dibujados que el azar premia. |
| **Condición de éxito** | ≥2 de 3 sesiones con entradas ancladas a niveles propios y stops estructurales; sin errores graves. |
| **Condición de fracaso** | Cero uso de niveles propios; `ERR_OVERTRADE` en ≥2 sesiones. |
| **Métricas / Score** | Correspondencia entrada↔nivel dibujado, calidad de stops estructurales, detección de la fake breakout (no entrar, o entrar y salir rápido). |
| **Feedback** | "Operaste tres sesiones sin un solo indicador. Tus niveles dibujados acertaron X de Y zonas relevantes del path. Los indicadores derivan del precio — nunca al revés. Ahora ya sabes leer el original." |
| **Replay** | Mismas tres sesiones; comparar lecturas entre intentos. |
| **Desbloqueo** | Escenario 11 completado. |
| **Modo libre** | Sí (versión práctica con seed aleatoria). |
| **Histórico-inspirado a futuro** | Sí — excelente con datos reales. |
| **Fase** | **MVP** |

---

### Escenario 16 — Challenge: Control de drawdown

| Campo | Especificación |
|---|---|
| **Título** | "Challenge: La caída controlada" |
| **Nivel de usuario** | 8 |
| **Tipo de mercado** | Índice sintético, 1 mes simulado |
| **Velocidad recomendada** | Alta |
| **Setup** | El usuario arranca con la cuenta YA en drawdown: 850 USD de un máximo histórico de 1.000 (−15%). El mercado del mes es difícil: pocos setups, mucha tentación. **Regla dura: si el drawdown total desde el máximo histórico llega a 25% (cuenta en 750), el challenge termina.** |
| **Tarea del usuario** | Gestionar un mes empezando desde la pérdida: recuperar lo que el mercado permita SIN profundizar la caída. El objetivo explícito no es volver a 1.000: es no llegar a 750. |
| **Learning Context Contract** | Objetivo: aprender a operar en drawdown — la situación psicológica más peligrosa del trading — reduciendo riesgo en vez de aumentarlo. Garantía: el path ya contiene una recuperación parcial de magnitud limitada y varios anzuelos de "recuperación total"; capturar la recuperación sin romper el límite depende de ejecutar con riesgo reducido. |
| **Template** | `TPL_TRAP_SEQUENCE` (variante drawdown) sobre `TPL_GRIND` corto |
| **Tipo de seed** | Fija por temporada (ranking); aleatoria en práctica. |
| **Path de mercado** | Pre-generado completo. |
| **Trampa oculta** | Dos "oportunidades de recuperación total": setups vistosos cuyo riesgo real es desproporcionado; tomarlos con tamaño grande lleva la cuenta al límite del 25% casi con certeza dentro de este path. |
| **Error esperado** | Aumentar el tamaño para "salir del hoyo rápido" — exactamente el comportamiento que convierte drawdowns recuperables en cuentas quemadas; o el opuesto: parálisis total que tampoco puntúa proceso. |
| **Decisiones buenas (aunque pierdan)** | Reducir el riesgo por trade a la mitad mientras está en drawdown (regla profesional); tomar solo setups de calidad; aceptar que recuperar toma tiempo. |
| **Decisiones malas (aunque ganen)** | Apostar grande en un anzuelo y que funcione — el evaluador muestra que en este path esa decisión rompía el límite en la mayoría de sus variantes próximas. |
| **Condición de éxito** | Terminar el mes con drawdown < 25% y score de proceso ≥ umbral. Niveles: Bronce (sobrevivió), Plata (recuperó hasta −8%), Oro (recuperó con riesgo reducido consistente y cero errores graves). |
| **Condición de fracaso** | Tocar el 25% de drawdown → fin inmediato. |
| **Métricas / Score** | Riesgo por trade durante drawdown (debe ser reducido), profundización máxima de la caída, selectividad de setups, recuperación lograda (peso menor). |
| **Feedback** | "Empezaste perdiendo 15% — recuerda esta cifra: para volver al máximo necesitas ganar 17.6%, no 15%. Por eso la primera regla del drawdown es no profundizarlo. Esta sesión mediste si puedes operar herido sin desangrarte." |
| **Replay** | Mismo mes; reintentar con disciplinas de riesgo distintas. |
| **Desbloqueo** | Challenge 13 completado. |
| **Modo libre** | Sí (versión práctica). |
| **Histórico-inspirado a futuro** | Sí. |
| **Fase** | **Posterior al MVP** — no pertenece al set canónico de 6 challenges de `MVP_CONTENT_LOCK` y su horizonte de 1 mes excede `MVP_SANDBOX_LIMITS` (documento 12). |

---

## 7. Matriz resumen de los 16 escenarios

| # | Escenario | Nivel | Template base | Seed | Trampa central | Fase |
|---|---|---|---|---|---|---|
| 1 | Tu primera operación | 1 | TREND | Fija tutorial | Ninguna | MVP |
| 2 | Día de tendencia | 2 | TREND | Fija/aleatoria | Pullback profundo | MVP |
| 3 | Día de rango | 2 | RANGE | Fija/aleatoria | Falsas salidas | MVP |
| 4 | Ruptura falsa | 3 | FAKE_BREAKOUT | Fija | La ruptura misma | MVP |
| 5 | Spike de noticia | 4 | NEWS_SPIKE | Fija/aleatoria | Reversión del spike + fricción | MVP |
| 6 | Spread alto | 4 | HIGH_SPREAD | Fija/aleatoria | Mini-setups que no cubren costos | MVP |
| 7 | Baja liquidez | 5 | LOW_LIQUIDITY | Fija/aleatoria | Barrido de stops | MVP |
| 8 | Señal copiada | 5 | TRAP_SEQUENCE | Fija | Validación inicial + "promedia" | MVP |
| 9 | Sobreapalancamiento | 6 | TREND calibrado | Fija | Oscilación que liquida | MVP |
| 10 | Revenge trading | 6 | TRAP_SEQUENCE | Fija | Anzuelos post-pérdida | MVP |
| 11 | Gestión de riesgo | 7 | GRIND 5d | Fija/aleatoria | Racha perdedora | MVP |
| 12 | Interés compuesto | 8 | GRIND 12m | Fija/aleatoria | Doble caída en corrección | Posterior |
| 13 | Supervivencia 90 días | 8 | GRIND hostil | Fija temporada | Hostilidad sostenida | Posterior |
| 14 | Portafolio 1 año | 9 | GRIND multi | Fija temporada | Correlación en crisis | Posterior |
| 15 | Solo velas | 7 | Mixto sin indicadores | Fija temporada | Fake breakout estructural | MVP |
| 16 | Control de drawdown | 8 | TRAP+GRIND | Fija temporada | Anzuelos de recuperación | Posterior |

> **Alcance MVP del catálogo cerrado por `MVP_CONTENT_LOCK` (documento 12, sección 3): MVP = escenarios 1–11 y 15; post-MVP = 12, 13, 14 y 16.** El challenge de supervivencia del MVP es "Supervivencia 50 velas" (documento 12, sección 8). Ante contradicción entre este catálogo y el lock, gana el lock.

Ruta de progresión (desbloqueos): 1 → 2 → 3 → 4 → {5, 8} → 6 → 7 → 9 → 10 → 11 → 15 *(MVP)* → {12, 13, 14, 16} *(post-MVP)*.

---

## 8. Conexión con los modos de Burgundy

### 8.1 Modo tutorial
- Usa exclusivamente **seeds fijas de tutorial** con paths conocidos: el guion del tutorial referencia velas y momentos exactos.
- Velocidad reducida y pausas automáticas en puntos de decisión.
- Los escenarios 1–4 forman el tronco del tutorial; el LCC dicta cada intervención guiada.

### 8.2 Modo sandbox
- **Seeds aleatorias** por sesión, generadas y selladas antes de la primera vela; siempre guardadas → toda sesión sandbox es reproducible.
- El usuario elige template, mercado, dificultad y parámetros desbloqueados; el pipeline (sección 3) corre igual que en lecciones. En MVP, las opciones quedan acotadas por `MVP_SANDBOX_LIMITS`, `MVP_MARKET_LOCK` y `LEVERAGE_MVP_LIMITS` (documento 12): 3 instrumentos sintéticos, horizontes Intradía/1 semana, sin leverage.
- La evaluación LCC corre en modo "informativo": detecta errores y da feedback, pero no bloquea ni puntúa para rankings.

### 8.3 Modo challenge
- **Seeds fijas por temporada**, versionadas con `generatorVersion` y hash de path: todos los participantes de una temporada juegan el mismo mercado.
- **Visibilidad de seed (`SEED_PATH_REPLAY_EXPORT_LOCK`, documento 08):** antes del intento solo se muestran `pathHash`, `seedType`, `generatorVersion` y las reglas del LCC; la seed cruda se revela al cerrar el intento. Intentos con seed conocida de antemano se marcan `seed_known = true` y no compiten en el ranking principal de primer intento.
- Reglas duras activas (límites de drawdown, fin inmediato).
- El score de challenge pondera proceso > resultado, según el LCC de cada challenge (`SCORING_V1_LOCK`, documento 11).

### 8.4 Modo replay
- Reconstruye cualquier sesión desde: template + parámetros + seed + `generatorVersion` + log de decisiones.
- Resolución cerrada por `SEED_PATH_REPLAY_EXPORT_LOCK` (documento 08): si existe path almacenado, el replay lo usa validando su hash; si no, regenera con el generador de la `generatorVersion` registrada y valida el hash. Hash no coincidente ⇒ sesión "no verificable", excluida de rankings, jamás reparada en silencio.
- Dos modos de visualización: "ver mi sesión" (decisiones sobreimpresas en el path) y "reintentar" (mismo path, nuevas decisiones, comparación lado a lado de curvas de equity).

### 8.5 Rankings de sesión y high scores
- Solo escenarios con **seed fija** alimentan rankings: la comparación es justa porque el mercado fue idéntico. El ranking principal usa el **primer intento por seed**; los intentos `seed_known = true` (replay, seed guardada) quedan fuera de esa tabla (`SEED_PATH_REPLAY_EXPORT_LOCK`, documento 08).
- Ranking local (sin nube): el usuario compite contra sus propias sesiones pasadas y contra percentiles de referencia empaquetados con la app.
- El high score guarda: score, seed, `generatorVersion`, hash de path, métricas de proceso y metadatos de replay — un high score siempre es auditable y reproducible.

### 8.6 Export / import de progreso
- El archivo de progreso incluye: estado de desbloqueos, XP, niveles, streaks, journal, registros de errores, high scores, y los **metadatos de replay** (seed + parámetros + versión + hash + log de decisiones) de las sesiones guardadas.
- Las velas viajan selectivamente según `SEED_PATH_REPLAY_EXPORT_LOCK` (documento 08): los `SeedRecord` se exportan **siempre**; el path completo solo si la sesión está en curso o su `generatorVersion` no es regenerable por el build actual. Para todo lo demás, cualquier instalación de Burgundy regenera el path idéntico desde seed + parámetros + versión, y el hash valida la regeneración tras importar.
- Si una versión futura del generador cambia, los replays antiguos siguen siendo válidos porque la `generatorVersion` original viaja en el export.

### 8.7 Escenarios histórico-inspirados (futuro)
- El motor ya está preparado: consume velas en formato universal sin importar la fuente.
- Un escenario histórico-inspirado define: referencia del episodio (ej. "inspirado en la crisis de 2008"), dataset en formato universal, y el mismo LCC/pipeline de siempre — solo cambia el paso 10 (el path se carga/adapta en lugar de generarse).
- La seed en modo histórico gobierna las variaciones (recortes de ventana, anonimización del instrumento, ruido leve para evitar memorización) manteniendo determinismo: misma seed + mismo dataset + misma versión = mismo escenario.
- Reglas educativas: nunca presentarlo como predicción; siempre etiquetar "inspirado en datos históricos"; el LCC sigue mandando — el episodio histórico es contexto, no contenido.

---

## 9. Criterios de cierre del bloque

Este bloque queda completo cuando:

1. ✔ Los 16 escenarios tienen ficha completa con LCC, seed, trampas, evaluación y feedback.
2. ✔ El pipeline de 15 pasos está definido con la regla de inmutabilidad (pasos 1–11 previos a toda acción del usuario).
3. ✔ Los tipos de seed y su comportamiento por modo (tutorial/sandbox/challenge/replay/histórico) están especificados.
4. ✔ La conexión con todos los modos de Burgundy y con export/import está definida.
5. ✔ Ningún elemento del motor permite que las decisiones del usuario alteren el path de mercado.

Siguiente bloque sugerido: motor de evaluación y scoring en detalle (formalización de las métricas de proceso y del cálculo de score por modo).

---

*Burgundy — simulador educativo de trading. Proyecto firmado por **tsuloid**. Este documento es material de diseño educativo y no constituye asesoría financiera.*
