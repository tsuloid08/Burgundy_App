# Burgundy — Sistema Curricular para Principiantes (Curriculum System)

**Documento:** 02_beginner_curriculum.md
**Proyecto:** Burgundy — Simulador educativo de trading
**Autor / Firma:** tsuloid
**Idioma:** Español (LATAM)
**Alcance:** Solo el sistema de aprendizaje. No incluye código, ni pantallas, ni modelos de base de datos.
**Documentos relacionados:** [01_product_definition.md](01_product_definition.md) (define las reglas de equidad de simulación, el Learning Context Contract y el sistema de semillas determinista que este currículo utiliza).

---

## 1. Principios del currículo

1. **De cero absoluto a avanzado, sin saltos.** El currículo asume que el usuario no sabe qué es un mercado. Cada lección depende solo de conceptos ya enseñados.
2. **El riesgo se enseña antes que la estrategia.** El usuario domina stop loss, position sizing y risk/reward antes de tocar un solo indicador.
3. **Cada lección vive dentro del simulador.** No hay teoría suelta: todo concepto se practica en un escenario pregenerado con su Learning Context Contract (LCC).
4. **Se evalúa el proceso, no el resultado.** Una operación ganadora con mal proceso recibe retroalimentación crítica; saltar una trampa es una decisión correcta puntuable.
5. **Los errores típicos LATAM son contenido de primera clase.** Copiar señales de Telegram/TikTok, sobre-apalancarse, operar sin stop loss y confundir ganancia con habilidad tienen lecciones y trampas dedicadas.
6. **Terminología real explicada en claro.** Los términos estándar se mantienen en inglés (stop loss, spread, drawdown, breakout) y se explican en español simple la primera vez que aparecen y en el glosario de repaso.
7. **Conceptos avanzados solo después de fundamentos.** Smart Money Concepts y conceptos de liquidez quedan bloqueados hasta completar estructura de mercado y gestión de riesgo, incluso si el usuario es impaciente (con la excepción controlada del modo libre, sección 10).
8. **Psicología como eje transversal.** Overtrading, revenge trading y FOMO se detectan y retroalimentan desde el Nivel 2, y se consolidan en un módulo propio al final.

---

## 2. Estructura general de la ruta de aprendizaje

El currículo se organiza en **6 niveles (etapas)** con **29 lecciones**. Cada nivel es una etapa de aprendizaje con su propia ficha y su propio marco de Learning Context Contract; cada lección dentro del nivel tiene su escenario, sus trampas y sus criterios concretos.

| Nivel | Nombre de etapa | Tema central | Lecciones | Relación con el MVP |
|---|---|---|---|---|
| 1 | Fundamentos del mercado | Qué es el mercado, el precio y la vela; long y short | 4 | Conceptos empaquetados en M1 |
| 2 | Supervivencia y riesgo | Riesgo, stop loss, take profit, sizing, risk/reward, señales | 6 | Conceptos empaquetados en M2/M3 |
| 3 | Estructura de mercado | Soportes/resistencias, trendlines, HH/LL, volumen | 4 | Conceptos esenciales en M1/M4 |
| 4 | Indicadores y patrones | Medias móviles, RSI, MACD, breakouts, pullbacks | 5 | Solo trampas de 4.4/4.5 vía escenarios; 4.1–4.3 post-MVP |
| 5 | Estilos de trading | Trend-following, mean reversion, scalping, day, swing, news | 5 | No (post-MVP) |
| 6 | Maestría y proceso | SMC, liquidez, backtesting, journaling, psicología | 5 | No (post-MVP) |

> El MVP implementa exclusivamente las **15 lecciones cortas M1–M5** definidas por `MVP_CONTENT_LOCK` (documento 12, sección 3); este currículo de 29 lecciones es la referencia completa y la ruta de expansión post-MVP (ver sección 9).

Progresión de gamificación asociada: cada lección otorga XP; completar un nivel sube de nivel al usuario y desbloquea habilidades, herramientas y desafíos del siguiente bloque. La racha diaria de práctica se alimenta de cualquier lección, repaso o desafío.

---

## 3. Nivel 1 — Fundamentos del mercado

### Ficha de la etapa

| Campo | Definición |
|---|---|
| Nombre de etapa | Fundamentos del mercado |
| Objetivo de la etapa | Que el usuario entienda qué es un mercado, cómo se forma el precio, cómo leer una vela y qué significa estar long o short |
| Condición de desbloqueo | Ninguna (etapa inicial) |
| Disponible en modo libre | Sí (todo el Nivel 1 está siempre disponible) |
| Indicadores | Deshabilitados (solo velas; modo price action puro obligatorio) |
| Dificultad recomendada | Muy baja: volatilidad suave, spread mínimo, sin eventos, sin leverage |
| Modo de simulación | Tutorial guiado con semilla fija |

### Lecciones

| # | Lección | Objetivo de la lección | Conceptos | Ejercicio en el simulador | Error típico de principiante | Condición de éxito | Métrica de evaluación |
|---|---|---|---|---|---|---|---|
| 1.1 | ¿Qué es un mercado y qué es el precio? | Entender que el precio surge del acuerdo entre compradores y vendedores | Mercado, oferta/demanda, precio como acuerdo, bid/ask básico | Observar un mercado sintético en movimiento e identificar momentos donde "ganan" compradores o vendedores (sin operar) | Creer que el precio "lo decide alguien" o que siempre tiene una razón visible | Responder correctamente el repaso de conceptos e identificar 3 de 4 momentos de presión compradora/vendedora | % de aciertos en identificación; repaso de concepto aprobado |
| 1.2 | La vela | Leer apertura, cierre, máximo, mínimo, cuerpo y mecha | Candle (vela), OHLC, vela alcista (#4A6D56) y bajista (#802F3E), timeframe | Reconstruir la historia de 5 velas: qué pasó dentro de cada una | Leer solo el color y no el rango ni las mechas | Interpretar correctamente 4 de 5 velas | Precisión de lectura de velas |
| 1.3 | Comprar y vender: la posición long | Entender que comprar busca vender más caro después | Posición, long, entrada, salida, P/L (ganancia/pérdida) | Abrir y cerrar una posición long en un escenario tranquilo de tendencia suave alcista, observando el P/L en vivo | Cerrar por pánico ante la primera vela en contra | Completar una operación long entendiendo su P/L (ganar o perder es válido) | Comprensión demostrada en repaso; sin cierres de pánico inmediatos detectados |
| 1.4 | La posición short | Entender que se puede ganar cuando el precio baja, y qué riesgo implica | Short, venta en corto, simetría e diferencias con long | Abrir y cerrar una posición short en tendencia suave bajista | Confundir la dirección: comprar cuando quería vender; creer que short es "apostar contra alguien" | Completar una operación short en la dirección que el usuario declaró antes de entrar | Coincidencia entre intención declarada y orden ejecutada |

### Learning Context Contract de la etapa

| Campo LCC | Nivel 1 |
|---|---|
| Objetivo de lección | Alfabetización de mercado: leer antes de operar |
| Familia de escenarios | "Mercado calmado didáctico" (synthetic calm market) |
| Comportamiento de mercado requerido | Movimientos limpios y lentos; tendencias suaves; sin shocks, sin noticias, sin spreads cambiantes |
| Regímenes usados | Tendencia suave alcista, tendencia suave bajista, rango tranquilo |
| Trampas incluidas | Ninguna (etapa sin trampas; la confianza se construye primero) |
| Decisiones buenas válidas | Observar, leer correctamente, declarar intención, ejecutar la dirección declarada, cerrar con calma |
| Decisiones malas comunes | Operar sin leer, invertir la dirección, cerrar por pánico inmediato |
| Reglas de feedback | Feedback inmediato tras cada interacción, en lenguaje guiado paso a paso; nunca penaliza el P/L en esta etapa |
| Resultado | Determinista (semilla fija de tutorial; el camino es idéntico en cada repetición) |
| Semilla fija de tutorial | Sí, una por lección |
| Variaciones con semillas distintas | Sí: tras aprobar, el modo práctica ofrece variaciones sandbox de la misma plantilla |
| Convertible a escenario histórico-inspirado | Sí (baja prioridad: cualquier mercado real calmado sirve como plantilla futura) |

---

## 4. Nivel 2 — Supervivencia y riesgo

Esta es la etapa más importante del currículo. Burgundy enseña a sobrevivir antes de enseñar a "acertar".

### Ficha de la etapa

| Campo | Definición |
|---|---|
| Nombre de etapa | Supervivencia y riesgo |
| Objetivo de la etapa | Que el usuario nunca más entre a un mercado sin stop loss, sin tamaño calculado y sin saber cuánto puede perder |
| Condición de desbloqueo | Completar Nivel 1 (las 4 lecciones aprobadas) |
| Disponible en modo libre | Sí |
| Indicadores | Deshabilitados (el riesgo se aprende sin muletas visuales) |
| Dificultad recomendada | Baja-media: volatilidad moderada, spread visible, fees visibles, leverage limitado e introducido al final |
| Modo de simulación | Tutorial con semilla fija + práctica sandbox con semillas variables |

### Lecciones

| # | Lección | Objetivo de la lección | Conceptos | Ejercicio en el simulador | Error típico de principiante | Condición de éxito | Métrica de evaluación |
|---|---|---|---|---|---|---|---|
| 2.1 | El riesgo existe antes que la ganancia | Entender que toda operación puede salir mal y que el capital es finito | Riesgo, pérdida, drawdown (cuánto cae la cuenta desde su punto más alto), capital | Operar 5 escenarios cortos donde algunas operaciones pierden por diseño del contexto (camino pregenerado); observar el drawdown de la cuenta | Creer que perder significa "hacerlo mal" siempre, o que ganar significa "hacerlo bien" siempre | Terminar la serie con la cuenta viva y explicar su drawdown en el repaso | Drawdown máximo; supervivencia; repaso aprobado |
| 2.2 | Stop loss: el cinturón de seguridad | Definir la salida de pérdida antes de entrar | Stop loss (orden que cierra la posición automáticamente para limitar la pérdida), invalidación | Colocar stop loss en 5 entradas; en algunas el stop se ejecuta y salva la cuenta de una caída mayor | Quitar o mover el stop cuando el precio se acerca | 100% de las entradas con stop loss definido antes de entrar; ningún stop removido | % de operaciones con stop previo; detecciones de stop movido/removido |
| 2.3 | Take profit y el plan completo | Definir también la salida de ganancia: entrada, stop y objetivo forman un plan | Take profit (orden que asegura la ganancia), plan de trade | Planificar 3 operaciones completas (entrada + stop + objetivo) antes de ejecutar | Cerrar la ganancia demasiado pronto por miedo y dejar correr la pérdida | 3 planes completos definidos antes de ejecutar; ejecución fiel al plan | Fidelidad plan vs ejecución |
| 2.4 | Position sizing: cuánto entrar | Calcular el tamaño según la distancia al stop y el % de riesgo de la cuenta | Position sizing, riesgo por operación (ej.: 1% de la cuenta), relación tamaño-distancia del stop | Con una cuenta pequeña (ej.: $200), dimensionar 4 operaciones para arriesgar exactamente el 1–2% en cada una | Usar siempre el mismo tamaño o "todo lo que se pueda" | Las 4 operaciones dentro del rango de riesgo objetivo (±0.5%) | Desviación entre riesgo planificado y riesgo real |
| 2.5 | Risk/reward y por qué el porcentaje de ganancia no basta | Entender que se puede ganar dinero acertando pocas veces, y quebrar acertando muchas | Risk/reward (cuánto arriesgas comparado con cuánto buscas ganar), winrate, expectativa | Comparar dos series simuladas pregeneradas: una con 70% de aciertos que pierde dinero y una con 40% que gana; luego planear 3 trades con R/R mínimo 1:1.5 | Perseguir el winrate alto; presumir "8 de 10 ganadas" sin mirar el tamaño de las pérdidas | Explicar en el repaso por qué la serie de 70% perdió; planear los 3 trades con R/R válido | Repaso aprobado; R/R promedio planificado |
| 2.6 | Por qué copiar señales es peligroso | Demostrar en simulación que una señal sin contexto, timing, riesgo y disciplina propia falla | Señal, contexto, timing, ejecución, responsabilidad personal | El simulador entrega "señales" (estilo Telegram) dentro de escenarios pregenerados: algunas llegan tarde, otras sin stop, otras con tamaño absurdo; el usuario decide si las sigue, las adapta o las descarta | Ejecutar la señal tal cual, sin stop, tarde y con tamaño excesivo | Descartar o adaptar correctamente al menos 3 de 4 señales defectuosas (adaptar = añadir stop, ajustar tamaño, validar timing) | Calidad de decisión por señal; detección de ejecución ciega |

### Learning Context Contract de la etapa

| Campo LCC | Nivel 2 |
|---|---|
| Objetivo de lección | Disciplina de riesgo: el usuario controla lo único controlable (su pérdida máxima, su tamaño, su plan) |
| Familia de escenarios | "Riesgo en vivo": series cortas con pérdidas inevitables por diseño de contexto, y "Trampa de señal copiada" (signal-copying trap) |
| Comportamiento de mercado requerido | Caminos pregenerados que incluyen tanto continuaciones como reversiones tras la entrada típica; spread y fees siempre visibles |
| Regímenes usados | Tendencia, rango, reversión moderada; volatilidad moderada con un pico controlado en 2.2 |
| Trampas incluidas | Stop tentador de remover (2.2), ganancia temprana que invita a cerrar (2.3), señal tardía/sin stop/sobredimensionada (2.6) |
| Decisiones buenas válidas | Stop antes de entrar, tamaño calculado, plan completo, respetar el plan, descartar señales defectuosas, abstenerse |
| Decisiones malas comunes | Operar sin stop, mover el stop, sobredimensionar, cortar ganancias y dejar correr pérdidas, copiar señal ciegamente |
| Reglas de feedback | Feedback al cierre de cada operación: primero proceso (¿hubo stop? ¿tamaño correcto? ¿plan respetado?), después resultado; una ganancia con mal proceso recibe advertencia explícita en #C9A050 (énfasis crítico) |
| Resultado | Determinista en tutorial; semi-aleatorio en práctica (variaciones de la misma plantilla con semillas distintas) |
| Semilla fija de tutorial | Sí, una por lección |
| Variaciones con semillas distintas | Sí, esenciales: el riesgo debe practicarse en muchos caminos diferentes |
| Convertible a escenario histórico-inspirado | Sí (2.5 y 2.6 son ideales para versiones inspiradas en episodios reales de mercado) |

---

## 5. Nivel 3 — Estructura de mercado

### Ficha de la etapa

| Campo | Definición |
|---|---|
| Nombre de etapa | Estructura de mercado |
| Objetivo de la etapa | Que el usuario lea el mapa del mercado: zonas, tendencias y estructura, antes de usar cualquier indicador |
| Condición de desbloqueo | Completar Nivel 2 + mantener en práctica un historial reciente con ≥90% de operaciones con stop loss |
| Disponible en modo libre | Sí |
| Indicadores | Deshabilitados (price action puro; el volumen se muestra solo en 3.4) |
| Dificultad recomendada | Media: estructuras claras primero, luego estructuras con ruido |
| Modo de simulación | Tutorial con semilla fija + sandbox con variaciones |

### Lecciones

| # | Lección | Objetivo de la lección | Conceptos | Ejercicio en el simulador | Error típico de principiante | Condición de éxito | Métrica de evaluación |
|---|---|---|---|---|---|---|---|
| 3.1 | Soportes y resistencias | Identificar zonas donde el precio reaccionó antes | Soporte, resistencia, zona (no línea exacta), reacción | Marcar zonas en un camino pregenerado y operar solo en reacciones a zonas marcadas | Trazar líneas exactas al pixel y operar cada toque sin confirmación | Identificar 3 de 4 zonas relevantes; entradas solo en zonas | Precisión de zonas; % de entradas dentro de zona |
| 3.2 | Trendlines | Trazar líneas de tendencia válidas y usarlas como guía, no como verdad | Trendline, toques mínimos, ruptura de línea | Trazar trendlines en 3 tendencias distintas y detectar cuál ruptura fue genuina (en camino pregenerado) | Forzar la línea para que "confirme" lo que el usuario quiere ver | Trendlines con ≥2 toques válidos; ruptura genuina identificada | Validez técnica de los trazos; repaso aprobado |
| 3.3 | Estructura: higher highs y lower lows | Leer la dirección estructural del mercado | Higher high (HH), higher low (HL), lower high (LH), lower low (LL), cambio de estructura | Etiquetar la secuencia estructural de un camino y declarar el sesgo (alcista/bajista/lateral) antes de operar | Operar contra la estructura "porque ya subió mucho" | Etiquetado correcto ≥80%; operaciones alineadas con el sesgo declarado | Precisión de etiquetado; alineación sesgo-operación |
| 3.4 | Volumen básico | Usar el volumen como contexto de interés, no como señal mágica | Volumen, liquidez (qué tan fácil es entrar o salir sin mover demasiado el precio), volumen en rupturas | Comparar dos rupturas pregeneradas: una con volumen y una sin volumen; decidir cuál merece confianza | Tratar cualquier pico de volumen como señal de entrada | Elegir la ruptura con respaldo de volumen y justificar en el repaso | Decisión correcta + repaso aprobado |

### Learning Context Contract de la etapa

| Campo LCC | Nivel 3 |
|---|---|
| Objetivo de lección | Lectura estructural: dónde está el precio respecto a su propio mapa |
| Familia de escenarios | "Mapa del mercado": rangos con zonas claras, tendencias estructuradas, rupturas con y sin respaldo |
| Comportamiento de mercado requerido | Estructuras legibles primero (HH/HL nítidos, zonas respetadas), luego variantes con ruido realista |
| Regímenes usados | Día de rango, día de tendencia, transición rango→tendencia, ruptura de estructura |
| Trampas incluidas | Falsa reacción en zona débil (3.1), ruptura de trendline sin continuación (3.2), ruptura sin volumen (3.4) |
| Decisiones buenas válidas | Marcar zonas antes de operar, declarar sesgo, esperar reacción, abstenerse fuera de zona |
| Decisiones malas comunes | Operar lejos de toda estructura, contra-tendencia sin razón, forzar trazos para justificar la entrada |
| Reglas de feedback | Feedback comparativo: la app muestra el etiquetado del usuario junto al etiquetado de referencia del LCC y explica las diferencias |
| Resultado | Determinista en tutorial; semi-aleatorio en práctica |
| Semilla fija de tutorial | Sí |
| Variaciones con semillas distintas | Sí: las plantillas estructurales generan infinitas variantes para entrenar el ojo |
| Convertible a escenario histórico-inspirado | Sí, alta prioridad: la estructura es el puente natural hacia gráficos reales futuros |

---

## 6. Nivel 4 — Indicadores y patrones

### Ficha de la etapa

| Campo | Definición |
|---|---|
| Nombre de etapa | Indicadores y patrones |
| Objetivo de la etapa | Usar indicadores como herramientas de contexto subordinadas a estructura y riesgo, y dominar breakouts y pullbacks (incluyendo sus trampas) |
| Condición de desbloqueo | Completar Nivel 3 |
| Disponible en modo libre | Sí (con advertencia educativa si no completó Nivel 3) |
| Indicadores | **Habilitados por primera vez** (se desbloquean como "habilidades": MA → RSI → MACD); las lecciones 4.4 y 4.5 ofrecen variante sin indicadores |
| Dificultad recomendada | Media-alta: ruido realista, spreads variables, primeros eventos programados |
| Modo de simulación | Tutorial con semilla fija + sandbox + primeros desafíos temáticos |

### Lecciones

| # | Lección | Objetivo de la lección | Conceptos | Ejercicio en el simulador | Error típico de principiante | Condición de éxito | Métrica de evaluación |
|---|---|---|---|---|---|---|---|
| 4.1 | Medias móviles | Leer la media móvil como resumen de tendencia, no como señal automática | Moving average (MA), períodos, precio vs media, cruces | Operar un camino con MA visible usando la media solo como filtro de dirección | Comprar cada cruce de medias mecánicamente | Operaciones alineadas con el filtro declarado; sin entradas por cruce aislado | Alineación filtro-entrada |
| 4.2 | RSI | Entender sobrecompra/sobreventa como contexto, y por qué falla en tendencia | RSI, sobrecompra, sobreventa, divergencia básica | Dos caminos pregenerados: en rango (RSI útil) y en tendencia fuerte (RSI "sobrecomprado" que sigue subiendo) | Vender solo porque el RSI marca 70 | No operar contra tendencia fuerte por lectura aislada del RSI; repaso aprobado | Decisiones contra-tendencia por RSI detectadas (deben ser 0) |
| 4.3 | MACD | Leer momentum y sus retrasos | MACD, momentum, retraso (lag) de los indicadores | Comparar señales del MACD contra la estructura ya aprendida; identificar dónde el MACD llegó tarde | Tratar el cruce del MACD como gatillo infalible | Identificar correctamente las señales tardías del escenario | Precisión en identificación de lag |
| 4.4 | Breakouts y el fake breakout | Operar rupturas con confirmación y reconocer la ruptura falsa | Breakout, fake breakout (ruptura falsa), confirmación, retest | Escenario familia "fake breakout": ruptura aparente que revierte; el usuario decide entrar, esperar retest o abstenerse | Comprar la ruptura en el primer tick, sin stop, con tamaño grande | Decisión de calidad (esperar confirmación/retest o abstenerse); si entra, con stop y tamaño correctos | Puntuación de proceso del LCC; abstención puntuada como éxito |
| 4.5 | Pullbacks y la trampa FOMO | No entrar después de una vela gigante; esperar el retroceso con riesgo definido | Pullback, impulso, FOMO, entrada perseguida (chasing) | Escenario familia "FOMO trap": vela de impulso enorme seguida de retroceso; entrar tarde es la trampa | Perseguir la vela gigante por miedo a quedarse fuera | Saltar la entrada tardía o esperar el pullback y entrar con stop y tamaño definidos | Calidad de proceso > P/L (explícito en el feedback) |

### Learning Context Contract de la etapa

| Campo LCC | Nivel 4 |
|---|---|
| Objetivo de lección | Subordinar herramientas y patrones al contexto: confirmación, paciencia y riesgo definido |
| Familia de escenarios | "Fake breakout", "FOMO trap", "Indicador en su hábitat equivocado" |
| Comportamiento de mercado requerido | Impulsos fuertes seguidos de retroceso; rupturas con y sin continuación; tendencias que invalidan lecturas ingenuas de RSI/MACD — todo pregenerado antes de la acción del usuario |
| Regímenes usados | Impulso→pullback, ruptura→reversión, ruptura→continuación con retest, tendencia fuerte sostenida |
| Trampas incluidas | Vela gigante que invita a perseguir, ruptura falsa, señal de indicador fuera de contexto |
| Decisiones buenas válidas | Saltar el trade, esperar pullback/retest, entrar solo con riesgo definido, usar el indicador como filtro |
| Decisiones malas comunes | Perseguir la vela, entrar sin stop, sobredimensionar, obedecer al indicador contra la estructura |
| Reglas de feedback | Feedback de proceso con desglose: timing, confirmación, stop, tamaño, alineación estructural; la app recalca cuando una abstención fue la mejor decisión disponible |
| Resultado | Determinista en tutorial y en desafíos; semi-aleatorio en sandbox |
| Semilla fija de tutorial | Sí; además 4.4 y 4.5 generan los primeros **desafíos con semilla fija** (ranking justo) |
| Variaciones con semillas distintas | Sí: las trampas deben verse en muchas formas para generalizar el aprendizaje |
| Convertible a escenario histórico-inspirado | Sí, máxima prioridad: fake breakouts y FOMO traps reales abundan como plantillas futuras |

#### LCC detallado de la lección insignia 4.5 — «No entres después de una vela gigante»

| Campo | Definición |
|---|---|
| Lección | No entrar después de una vela gigante |
| Contexto de aprendizaje | FOMO trap |
| Comportamiento de mercado | Vela de impulso fuerte seguida de pullback (camino completo pregenerado antes de cualquier acción) |
| Decisiones buenas | Saltar el trade; esperar el pullback; entrar solo con riesgo definido (stop + tamaño calculado) |
| Decisiones malas | Perseguir la vela; entrar sin stop loss; sobredimensionar la posición |
| Comportamiento de semilla | Semilla fija en tutorial; variación con semillas aleatorias en sandbox |
| Evaluación | La calidad del proceso pesa más que el P/L: perseguir la vela y ganar por suerte genera advertencia; abstenerse correctamente genera puntuación alta |

---

## 7. Nivel 5 — Estilos de trading

### Ficha de la etapa

| Campo | Definición |
|---|---|
| Nombre de etapa | Estilos de trading |
| Objetivo de la etapa | Que el usuario conozca los estilos principales, sus exigencias y costos, y descubra cuál se ajusta a su disciplina — no "cuál gana más" |
| Condición de desbloqueo | Completar Nivel 4 + completar al menos 2 desafíos del Nivel 4 |
| Disponible en modo libre | Sí (con advertencia educativa) |
| Indicadores | Habilitados; cada estilo sugiere su set; variante sin indicadores disponible |
| Dificultad recomendada | Alta: spreads y slippage realistas por estilo, eventos de noticias programados, sesiones largas |
| Modo de simulación | Tutorial + sandbox + desafíos por estilo |

### Lecciones

| # | Lección | Objetivo de la lección | Conceptos | Ejercicio en el simulador | Error típico de principiante | Condición de éxito | Métrica de evaluación |
|---|---|---|---|---|---|---|---|
| 5.1 | Trend-following | Operar a favor de la tendencia y sostener la posición con trailing de estructura | Trend-following, dejar correr la ganancia, salida por estructura | Sostener posiciones en un día de tendencia pregenerado sin cerrar por ansiedad | Cerrar la ganancia en la primera pausa | Mantener la posición mientras la estructura lo permita | Tiempo en posición vs estructura; salidas prematuras detectadas |
| 5.2 | Mean reversion | Operar el regreso a la media en rangos, y saber cuándo NO hacerlo | Mean reversion, rango, extremos, invalidación por ruptura | Operar reversiones en un día de rango; el escenario incluye la ruptura final que invalida el estilo | Seguir comprando "barato" cuando el rango ya se rompió | Reconocer la invalidación y dejar de operar reversión tras la ruptura | Operaciones post-invalidación (deben ser 0) |
| 5.3 | Scalping y day trading | Conocer la exigencia de los plazos cortos: costos, velocidad y desgaste | Scalping, day trading, impacto del spread/fees en operaciones cortas, overtrading | Sesión corta intradía con costos realistas; el simulador muestra cuánto del P/L se fue en spread y fees | Sobreoperar: muchas entradas pequeñas que los costos devoran | Cerrar la sesión con número de operaciones dentro del límite del plan y costos entendidos | Conteo de operaciones vs plan; repaso de costos aprobado |
| 5.4 | Swing trading | Operar movimientos de varios días con paciencia y stops amplios bien dimensionados | Swing trading, horizonte temporal, gaps, paciencia | Escenario multi-día comprimido con gaps de apertura; dimensionar con stops amplios | Usar tamaño de intradía con stop de swing (riesgo gigante) | Sizing correcto para el stop amplio; soportar el ruido intermedio sin cerrar por ansiedad | Riesgo real por operación; cierres por ruido detectados |
| 5.5 | News trading | Entender por qué las noticias generan spread alto, slippage y movimientos violentos | Pico de noticias, spread alto, slippage (recibir un precio de ejecución peor del esperado), volatilidad | Escenario familia "news spike": evento programado en el calendario del escenario; el usuario decide operar antes, durante o después (o abstenerse) | Operar dentro del pico con spread enorme y sin entender el slippage | Decisión de calidad: abstenerse durante el pico o esperar a la normalización con riesgo definido | Puntuación de proceso; costo de ejecución entendido en repaso |

### Learning Context Contract de la etapa

| Campo LCC | Nivel 5 |
|---|---|
| Objetivo de lección | Ajustar comportamiento al régimen y al estilo; conocer los costos reales de cada plazo |
| Familia de escenarios | "Día de tendencia", "Día de rango con ruptura final", "Sesión intradía con costos", "Multi-día con gaps", "News spike / shock de volatilidad / spread alto" |
| Comportamiento de mercado requerido | Regímenes marcados por estilo; eventos en calendario pregenerado (event schedule); perfiles de spread y liquidez diferenciados por escenario |
| Regímenes usados | Tendencia sostenida, rango→ruptura, intradía con microestructura, multi-día con gaps, shock de noticias |
| Trampas incluidas | Pausa que invita a cerrar temprano (5.1), rango roto que invita a promediar (5.2), exceso de oportunidades pequeñas (5.3), ruido intermedio (5.4), pico de noticias con spread alto (5.5) |
| Decisiones buenas válidas | Sostener con estructura, reconocer invalidación, limitar el número de operaciones, dimensionar por stop, abstenerse en el pico |
| Decisiones malas comunes | Cortar ganancias, promediar pérdidas, overtrading, riesgo desproporcionado, operar dentro del shock |
| Reglas de feedback | Feedback por estilo con métricas propias (tiempo en posición, costos acumulados, operaciones por sesión); comparación con la ejecución de referencia del LCC |
| Resultado | Determinista en tutorial y desafíos; semi-aleatorio en sandbox |
| Semilla fija de tutorial | Sí; cada estilo tiene además su desafío con semilla fija |
| Variaciones con semillas distintas | Sí |
| Convertible a escenario histórico-inspirado | Sí: días de tendencia/rango y eventos de noticias reales son plantillas históricas naturales |

---

## 8. Nivel 6 — Maestría y proceso

### Ficha de la etapa

| Campo | Definición |
|---|---|
| Nombre de etapa | Maestría y proceso |
| Objetivo de la etapa | Consolidar el proceso profesional: conceptos avanzados (solo ahora), validación, registro y dominio psicológico |
| Condición de desbloqueo | Completar Nivel 5 + métricas de disciplina sostenidas (≥90% stops, riesgo por trade dentro de rango en las últimas 20 operaciones de práctica) |
| Disponible en modo libre | Parcial: 6.3, 6.4 y 6.5 sí; 6.1 y 6.2 (SMC y liquidez) requieren fundamentos completos incluso en modo libre (única excepción del modo libre, ver sección 10) |
| Indicadores | A elección del usuario; desafíos finales incluyen variante sin indicadores |
| Dificultad recomendada | Alta-máxima: escenarios compuestos, trampas combinadas, riesgo de liquidación con leverage |
| Modo de simulación | Tutorial + sandbox + desafíos de maestría |

### Lecciones

| # | Lección | Objetivo de la lección | Conceptos | Ejercicio en el simulador | Error típico de principiante | Condición de éxito | Métrica de evaluación |
|---|---|---|---|---|---|---|---|
| 6.1 | Smart Money Concepts (después de fundamentos) | Entender SMC como relectura de estructura y zonas, sin misticismo | SMC, order blocks, imbalance, relación con S/R y estructura ya aprendidas | Releer escenarios de Nivel 3–4 con lente SMC y comparar: ¿qué agrega y qué renombra? | Tratar SMC como sistema secreto infalible aprendido en TikTok | Mapear correctamente conceptos SMC sobre estructura conocida en el repaso | Repaso comparativo aprobado |
| 6.2 | Conceptos de liquidez | Entender dónde se acumulan stops y por qué el precio visita esas zonas | Liquidez, barrido de liquidez (liquidity sweep), zonas de stops, riesgo de liquidación con leverage | Escenario familia "liquidation-risk": posición apalancada cerca de zona de barrido pregenerada; el usuario gestiona o evita el riesgo | Apalancarse con el stop exactamente donde todos lo ponen | Gestionar el riesgo de barrido (stop mejor ubicado, menos leverage o abstención) | Puntuación de proceso; liquidaciones evitables detectadas |
| 6.3 | Backtesting | Validar una regla contra muchos caminos antes de confiar en ella | Backtesting, muestra, sobreajuste (creer que lo que funcionó en un caso funciona siempre) | Probar una regla simple sobre un lote de escenarios generados con semillas distintas de la misma plantilla y leer los resultados agregados | Validar con 3 casos elegidos a conveniencia | Completar el lote y explicar la diferencia entre un caso y una muestra | Repaso aprobado; lote completado |
| 6.4 | Journaling | Convertir el diario de decisiones en herramienta de mejora | Journal, registro de decisiones, revisión de errores, patrón personal de error | Revisar su propio journal acumulado en Burgundy, identificar su error más repetido y reintentar (replay exacto) los 3 escenarios donde lo cometió | No revisar nunca; repetir el mismo error con ánimo distinto | Identificar su patrón de error dominante y mejorar la decisión en al menos 2 de 3 replays | Mejora medida entre intento original y replay |
| 6.5 | Psicología del trading (capstone) | Dominar overtrading, revenge trading y FOMO bajo presión combinada | Overtrading, revenge trading (operar para "vengarse" de una pérdida), FOMO, disciplina bajo presión | Desafío largo de supervivencia con trampas combinadas: pérdida temprana diseñada por contexto + oportunidades tentadoras de revancha | Duplicar tamaño tras perder; encadenar entradas impulsivas | Sobrevivir la sesión con riesgo constante, sin escalada post-pérdida ni ráfagas de entradas | Detecciones de revenge/overtrading (deben ser 0); supervivencia; consistencia de riesgo |

### Learning Context Contract de la etapa

| Campo LCC | Nivel 6 |
|---|---|
| Objetivo de lección | Proceso profesional completo: relectura avanzada, validación, registro y autocontrol |
| Familia de escenarios | "Barrido de liquidez", "Riesgo de liquidación", "Lote de backtesting", "Replay de errores propios", "Supervivencia con presión emocional" |
| Comportamiento de mercado requerido | Caminos compuestos con trampas combinadas; lotes multi-semilla de una misma plantilla; replays exactos de sesiones pasadas del usuario |
| Regímenes usados | Todos los anteriores, combinados y con transiciones |
| Trampas incluidas | Zona de stops obvia, pérdida temprana que invita a revancha, racha que invita a sobreoperar, leverage tentador |
| Decisiones buenas válidas | Riesgo constante tras perder, pausar tras detección de impulso, stop fuera de la zona obvia, reducir leverage, abstención |
| Decisiones malas comunes | Escalada de tamaño post-pérdida, ráfaga de entradas, stop en zona de barrido con leverage alto |
| Reglas de feedback | Feedback longitudinal: usa el historial del journal del usuario, no solo la sesión actual; el capstone entrega un informe de patrón personal de error |
| Resultado | Determinista en tutorial, desafíos y replays; el lote de backtesting usa conjuntos de semillas registrados (reproducibles) |
| Semilla fija de tutorial | Sí; el capstone 6.5 usa semilla fija de desafío (ranking justo) |
| Variaciones con semillas distintas | Sí; en 6.3 la multi-semilla es el contenido mismo de la lección |
| Convertible a escenario histórico-inspirado | Sí: 6.2 y 6.5 son los candidatos más valiosos para versiones histórico-inspiradas futuras |

---

## 9. Orden ideal de lecciones y alcance del MVP

### Orden ideal (secuencia completa)

1.1 → 1.2 → 1.3 → 1.4 → **2.1 → 2.2 → 2.3 → 2.4 → 2.5 → 2.6** → 3.1 → 3.2 → 3.3 → 3.4 → 4.1 → 4.2 → 4.3 → 4.4 → 4.5 → 5.1 → 5.2 → 5.3 → 5.4 → 5.5 → 6.1 → 6.2 → 6.3 → 6.4 → 6.5

Reglas de orden no negociables:

- Nada de indicadores antes de completar estructura (Nivel 3).
- Nada de estilos antes de dominar trampas básicas (Nivel 4).
- SMC y liquidez (6.1, 6.2) **solo después de fundamentos completos**, sin excepción.
- La psicología se detecta y retroalimenta desde 2.1, pero su consolidación (6.5) es el cierre del currículo.

### Lecciones en el MVP vs postergadas

> **El alcance educativo del MVP está cerrado por `MVP_CONTENT_LOCK` (documento 12, sección 3).** El MVP empaqueta los conceptos esenciales de este currículo en **15 lecciones cortas (M1–M5)**; ninguna lección de este documento se implementa individualmente en el MVP. Las 29 lecciones de este documento son el **currículo completo de referencia** y la ruta de expansión post-MVP (Fase 4). Ante contradicción entre este documento y el lock, gana el lock.

| Bloque del currículo | Estado | Cobertura en el MVP |
|---|---|---|
| Nivel 1 (1.1–1.4) | **Referencia — conceptos en MVP vía M1** | Base imprescindible: precio, oferta/demanda, velas, timeframes |
| Nivel 2 (2.1–2.6) | **Referencia — conceptos en MVP vía M2 y M3** | Corazón del producto: costos invisibles, riesgo, stop loss, sizing, risk/reward, drawdown |
| Nivel 3 (3.1–3.4) | **Referencia — conceptos esenciales en MVP vía M1/M4** | La estructura completa como lecciones dedicadas llega en Fase 4 |
| 4.4 Breakouts/fake breakout y 4.5 Pullbacks/FOMO | **Referencia — sus trampas viven en los escenarios MVP** | Escenarios 4 y 10 del documento 10 y módulos M4/M5 |
| 4.1–4.3 Indicadores (MA, RSI, MACD) | **Post-MVP (primera ola)** | Valiosos pero no bloquean la promesa central; quedan fuera del MVP de forma cerrada (sin cláusula "según recursos") |
| Nivel 5 completo (5.1–5.5) | **Post-MVP (primera ola)** | Requiere perfiles de costos y eventos más ricos |
| Nivel 6 completo (6.1–6.5) | **Post-MVP (segunda ola)** | Requiere journal maduro, lotes multi-semilla y replay longitudinal; 6.4 (journaling básico) puede adelantarse si el diario del MVP lo soporta |
| Escenarios histórico-inspirados de cualquier nivel | **Futuro** | El currículo es historical-ready; ninguna lección del MVP depende de datos reales |

---

## 10. Conexión con el modo desafío, control de progresión y modo libre

### Cómo el modo desafío se conecta al currículo

- **Cada nivel desbloquea desafíos temáticos** construidos sobre sus mismas familias de escenarios y LCC: completar 4.4 desbloquea el desafío "Ruptura falsa" (semilla fija); completar 4.5, el desafío "Trampa FOMO"; el Nivel 2 desbloquea los desafíos de disciplina de riesgo; el Nivel 3 habilita los desafíos sin indicadores; el capstone 6.5 es en sí un desafío de supervivencia.
- **Toda semilla de desafío es fija** (challenge seed): cada intento y cada usuario enfrentan el mismo camino, lo que hace justos los rankings por sesión y las puntuaciones máximas guardadas.
- **La puntuación de desafío es puntuación de proceso**: riesgo respetado, trampas evitadas, abstenciones correctas, supervivencia y consistencia — nunca solo el P/L.
- Los desafíos son la **prueba de transferencia**: la lección enseña con guía; el desafío verifica que el comportamiento se sostiene sin guía.

### Cómo se evita que el principiante salte a conceptos avanzados (sin matar el modo libre)

| Mecanismo | Comportamiento |
|---|---|
| Modo progresión (por defecto) | Las lecciones y desafíos se desbloquean estrictamente en orden; las condiciones de desbloqueo incluyen métricas de disciplina (no solo "completar"), por ejemplo ≥90% de operaciones con stop |
| Modo libre | El usuario accede a mercados, herramientas e indicadores sin restricciones de desbloqueo, **para practicar**; al abrir contenido de un nivel no alcanzado, Burgundy muestra una nota educativa breve (sin pantalla legal pesada): "Este contenido asume que dominas X; tu evaluación lo tendrá en cuenta" |
| Excepción única | Las lecciones 6.1 (SMC) y 6.2 (liquidez) permanecen bloqueadas hasta completar fundamentos incluso en modo libre, por mandato del producto: "Smart Money Concepts solo después de fundamentos" |
| Salvaguarda de evaluación | En modo libre, la detección de errores y el journal siguen activos: el usuario impaciente recibe la misma retroalimentación de proceso, que naturalmente lo redirige al currículo |
| Reset educativo | Si el usuario quema la cuenta en modo libre, el reset incluye la revisión de errores y sugiere la lección del currículo que cubre su error dominante |

### Cómo las semillas fijas de tutorial sostienen la práctica repetida

- Cada lección tutorial tiene su **tutorial seed fija**: misma plantilla + parámetros + semilla + versión del generador = el mismo camino de mercado, siempre.
- El usuario puede **repetir la lección exactamente igual** y probar decisiones distintas sobre el mismo mercado: es la forma más limpia de aislar la variable "mi decisión".
- La revisión de errores usa **replay seeds** para reproducir la sesión exacta donde se cometió el error (mismo camino, decisiones nuevas).
- La equidad es verificable: el camino existe completo antes de que el usuario actúe y su hash queda almacenado (ver documento 01, secciones 11 y 13).

### Cómo la variación de semillas en sandbox sostiene el aprendizaje de largo plazo

- Tras aprobar una lección, el sandbox genera **variaciones de la misma plantilla con semillas aleatorias**: la misma trampa (FOMO, fake breakout, señal defectuosa) aparece con formas siempre nuevas, lo que obliga a generalizar el concepto en lugar de memorizar un gráfico.
- Toda semilla sandbox puede **guardarse y reproducirse** (sandbox seed guardada → replay exacto), de modo que un buen caso de práctica se convierte en material de estudio permanente y exportable en el archivo de progreso.
- La racha diaria y el XP recompensan esta práctica variada: la progresión de largo plazo de Burgundy se construye sobre repetición con variación, que es como se entrena de verdad la disciplina.
- A futuro, las mismas plantillas aceptan **semillas histórico-inspiradas**, sin cambiar el currículo: las lecciones ya están preparadas para esa fuente de datos (historical-ready, no historical-dependent).

---

*Currículo de aprendizaje de Burgundy — firmado por tsuloid. Siguiente paso del proceso (no incluido aquí): especificación de los sistemas de simulación y evaluación, y posteriormente arquitectura técnica.*
