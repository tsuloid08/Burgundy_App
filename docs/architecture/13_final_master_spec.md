# Burgundy — Bloque 13: Documento Maestro de Especificación Final (Consolidación)

**Proyecto:** Burgundy — Simulador educativo de trading
**Autor / Firma:** tsuloid
**Documento:** `docs/architecture/13_final_master_spec.md`
**Idioma:** Español (LATAM)
**Estado:** Especificación maestra consolidada — sin código, sin pantallas, previo al desarrollo
**Documentos consolidados:** 01 a 12 de la serie `docs/architecture/`
**Paleta Burgundy:** `#1A1617` (Deep Charcoal, fondos) · `#571324` (Matte Burgundy, acento) · `#2E2E2E` (superficies/divisores) · `#C9A050` (Muted Gold, énfasis crítico) · `#4A6D56` (velas alcistas) · `#802F3E` (velas bajistas)

> Este documento consolida toda la arquitectura de producto de Burgundy en una sola especificación maestra lista para implementación. Cuando una sección requiera detalle completo, se referencia el documento original de la serie.

---

## 1. Visión del producto

Burgundy es un simulador móvil educativo de trading: una **academia seria dentro de una terminal de mercado oscura con identidad Burgundy**, con progresión de largo plazo estilo juego serio.

Visión central:

> Que un principiante de LATAM pueda aprender disciplina de mercado real —gestión de riesgo, paciencia, control emocional y calidad de proceso— practicando en un entorno simulado, justo, repetible y sin riesgo de dinero real, sintiendo que progresa como en un juego serio, no como en un casino.

Burgundy no enseña a "ganar dinero rápido". Enseña a **sobrevivir, decidir bien y entender por qué los traders principiantes fracasan**, antes de que el usuario arriesgue un solo peso real.

Sentimiento objetivo del usuario:

> "Esto es un sistema de entrenamiento serio. Estoy subiendo de nivel como en un juego, pero estoy aprendiendo disciplina real de mercado."

La identidad combina cinco pilares inseparables:

1. Academia seria (lecciones estructuradas, rutas de aprendizaje, niveles, revisión de errores).
2. HUD oscuro de trading con tema Burgundy (gráfico de velas como foco, paneles de balance, equity, riesgo, P/L, spread, slippage, fees y drawdown).
3. Gamificación seria de largo plazo (racha diaria, XP, niveles, desbloqueos, desafíos, rankings de sesión, high scores).
4. Educación amigable para LATAM.
5. Terminología real de trading explicada en español claro.

Lo que Burgundy **no es**: no es broker, no es casino, no es curso, no es señalero, no es asesoría financiera, no hace ver el trading fácil y no promete riqueza.

*(Detalle completo: documento 01.)*

---

## 2. Usuario objetivo

**Principiante de LATAM, vulnerable, que busca disciplina, no entretenimiento.**

| Característica | Implicación de producto |
|---|---|
| Adulto joven (≈18–40), hispanohablante, smartphone primario (a menudo Android de gama media) | App móvil, mobile-first, presupuesto de rendimiento para gama baja |
| Conectividad intermitente o datos costosos | Offline-first total, cero llamadas de red |
| Vocabulario de trading aprendido en redes, mal entendido | Terminología real explicada en español simple |
| Capital potencial pequeño (50–500 USD hipotéticos) | Ejemplos y presets con cuentas pequeñas |
| Desconfianza razonable de apps que piden datos | Sin login, sin nube, sin monetización, privacidad por diseño |

Comportamientos típicos que Burgundy ataca directamente: copiar señales de Telegram/TikTok, usar apalancamiento (leverage) excesivo, entrar sin stop loss, confundir ganancia con habilidad, overtrading, revenge trading y FOMO. Cada uno tiene lecciones, trampas pregeneradas y reglas de detección dedicadas.

Regla de tono: el usuario es vulnerable. Burgundy nunca lo ridiculiza, nunca lo sobreprotege con pantallas legales pesadas, y nunca le promete ganancias. Le habla claro, directo y en serio.

---

## 3. Objetivo central de aprendizaje

Transformación buscada (medible dentro del simulador, nunca por promesas de resultados reales):

| Estado inicial | Estado final buscado |
|---|---|
| Copia señales sin entender | Evalúa contexto, timing y riesgo antes de cualquier entrada |
| Entra sin stop loss | Define stop loss, take profit y tamaño de posición antes de entrar |
| Confunde suerte con habilidad | Juzga sus operaciones por calidad de proceso, no por resultado |
| Usa leverage como atajo | Entiende el leverage como amplificador de riesgo y lo dosifica |
| Opera por FOMO y venganza | Reconoce sus patrones emocionales; saltar la operación es una decisión válida |
| Cree que el trading es fácil | Entiende drawdown, costos (spread, slippage, fees) y la dificultad de sobrevivir |

Glosario base (regla de terminología real — término en inglés estándar + explicación en español claro):

- **Spread:** la diferencia entre el precio de compra y el precio de venta.
- **Slippage:** recibir un precio de ejecución peor del que esperabas.
- **Drawdown:** cuánto cae tu cuenta desde su punto más alto.
- **Leverage (apalancamiento):** controlar una posición más grande con menos capital; aumenta tanto el riesgo como el retorno potencial.
- **Liquidez:** qué tan fácil es entrar o salir sin mover demasiado el precio.
- **Risk/reward:** cuánto arriesgas comparado con cuánto buscas ganar.

---

## 4. Restricciones de producto (innegociables)

| # | Restricción |
|---|---|
| 1 | Nombre de la app: **Burgundy**; proyecto firmado bajo el usuario **tsuloid** |
| 2 | Solo en español, dirigida a LATAM |
| 3 | App móvil propietaria: Android 15+ / iOS 20+ |
| 4 | Sin login, sin cuenta en la nube |
| 5 | Sin monetización, sin cursos, sin coach de IA |
| 6 | Offline-first; progreso 100% local; export/import de archivo de progreso |
| 7 | Sin integración con brokers reales; sin trading con dinero real; no es asesoría financiera |
| 8 | Encuadre educativo sin pantallas legales pesadas; cero promesas de ganancia, cero hype |
| 9 | Paleta exacta: #1A1617, #571324, #2E2E2E, #C9A050, #4A6D56, #802F3E; prohibidos los verdes/rojos estridentes y el negro puro |
| 10 | Modos: tutorial, sandbox, challenge, modo libre, modo sin indicadores, progresión con desbloqueos |
| 11 | Rankings por sesión y high scores guardados (locales); racha diaria; XP; niveles; desbloqueos |
| 12 | Reinicio al quemar la cuenta (blow-up) o llegar a cero, con revisión educativa obligatoria |
| 13 | Reglas de equidad de simulación y sistema de semillas determinista (secciones 5–7) |
| 14 | Historical-ready, no historical-dependent: el modo histórico es fuente futura, no fundamento del MVP |

---

## 5. Reglas de equidad de la simulación (Simulation Fairness Rules)

Innegociables; definen la integridad del producto:

1. **El simulador fuerza un contexto de aprendizaje, nunca un resultado de trading.** Puede crear el contexto que la lección necesita (fake breakout, FOMO trap, pico de noticias, spread alto, día de rango, día de tendencia, trampa de señal copiada, riesgo de liquidación).
2. **El camino del mercado (market path) se genera completo antes de la acción del usuario.**
3. **Prohibición absoluta de manipulación dinámica:** la app nunca mueve el precio contra el usuario tras detectar su operación, ni lo suaviza a su favor.
4. **Las decisiones del usuario afectan solo su lado:** órdenes, posiciones, cuenta, riesgo, P/L, detección de errores, journal, feedback y score. Jamás el path generado.
5. **El aprendizaje se evalúa por calidad de decisión**, no por pérdidas forzadas; ganar, perder o abstenerse son finales válidos.
6. **Determinismo:** misma plantilla + parámetros + seed + versión del generador = exactamente el mismo path, siempre.
7. **Rankings justos:** los desafíos usan seeds fijas; todos los intentos enfrentan el mismo camino.
8. **Sandbox reproducible:** seeds aleatorias, pero toda seed guardada es reproducible.
9. **Replay exacto:** el modo de repetición reproduce el mismo camino, idéntico.
10. **El modo histórico es futuro**, no requisito del MVP.

---

## 6. Sistema de Learning Context Contract (LCC)

Cada escenario educativo es un **contrato explícito y pregenerado** entre la app y el usuario: define qué se enseña y cómo se evalúa, **antes** de que el usuario actúe.

Campos de todo LCC:

| Campo | Descripción |
|---|---|
| Objetivo de la lección | Qué concepto o disciplina enseña el escenario |
| Tipo de escenario | fake breakout, FOMO trap, news spike, día de rango/tendencia, spread alto, trampa de señal, riesgo de liquidación, etc. |
| Contexto garantizado | Condiciones de mercado pregeneradas e inmutables (ej.: "habrá una ruptura falsa entre el 40% y 60% de la sesión") |
| Secuencia de regímenes | Orden de fases del mercado que el path debe contener |
| Trampas esperadas | Errores típicos que el contexto invita a cometer |
| Decisiones buenas válidas | Decisiones de alta calidad aunque el trade pierda (incluida la abstención) |
| Decisiones malas aunque ganen | Comportamientos que se evalúan negativo incluso con P/L positivo |
| Criterios de evaluación y `scoringProfile` | Cómo se puntúa: riesgo, timing, disciplina, paciencia, abstención; pesos por intención del escenario |
| Reglas de feedback | Qué retroalimentación recibe el usuario y cuándo |
| Caminos de éxito y de fracaso | Definidos sobre proceso y supervivencia, no solo sobre P/L |
| Comportamiento de seed | `fixed` (tutorial/desafío), `random_constrained` (sandbox), `from_pool` |
| Comportamiento de replay | Replay exacto, replay bloqueado para ranking, o replay educativo anotado |

Regla clave: **el contrato describe el contexto; nunca prescribe el resultado del usuario.** El mismo contrato puede terminar en ganancia, pérdida, abstención correcta o cuenta quemada — todos son finales pedagógicos válidos. Los LCC son datos declarativos (no código), versionados, empaquetados con la app, y su ID se almacena con cada simulación generada.

---

## 7. Sistema de semillas determinista (Deterministic Seed System)

**Principio:** misma plantilla de escenario + mismos parámetros + misma seed + misma versión del generador ⇒ **exactamente el mismo market path, byte a byte**, en cualquier dispositivo y fecha.

| Tipo de seed | Uso | Propiedad clave |
|---|---|---|
| Tutorial fija | Lecciones guiadas | Idéntica para todos los usuarios y repeticiones; permite guías sobre velas conocidas |
| Desafío fija | Challenges y rankings | Equidad: todos los intentos enfrentan el mismo mercado |
| Sandbox aleatoria | Práctica libre | Variedad infinita; la seed se registra siempre para poder repetir |
| Replay | Revisión de errores y reintentos | Reconstruye la sesión exacta |
| Histórico-inspirada (futura) | Escenarios calibrados con estadísticas de episodios reales | Realismo histórico sin requerir el dataset |
| Replay histórico exacto (futuro) | Reproducir datos reales | Referencia dataset + rango de fechas |

Reglas técnicas (cerradas por **`DETERMINISM_LOCK_V1`**, documento 08): PRNG = **PCG32** (decisión cerrada, sin alternativas), seed de 64 bits sin signo, sin `Math.random`, sin reloj real ni entropía externa; streams de sub-seeds con índices fijos por subsistema (régimen, velas, sub-ticks, eventos, spread, slippage); precios en enteros escalados (`priceScale` por instrumento); redondeo half-even; hash SHA-256 de la serialización canónica del path sellado antes de iniciar la sesión; corpus dorado de hashes verificado en CI.

Cada simulación almacena: seed y tipo de seed, versión del generador, ID de plantilla, ID del LCC, mercado e instrumento, dificultad, horizonte temporal, perfiles de volatilidad/spread/liquidez, secuencia de regímenes, calendario de eventos, hash del path, decision log del usuario, resultado de evaluación y metadatos de replay. Todo viaja en el archivo de export.

---

## 8. Diseño de simulación historical-ready

Burgundy está **preparada para datos históricos sin depender de ellos**:

- **Formato universal de vela** para todas las fuentes (sección 22).
- **Abstracción de fuente de datos (`MarketDataSource`):** hoy el único productor es el generador sintético/procedural; mañana se añaden `HistoricalDataSource` y `HistoricalInspiredDataSource` con el mismo contrato de salida (path inmutable + metadatos).
- **Motores indiferentes al origen:** Order, Position, Risk, Evaluation y Replay Engine operan solo sobre el formato universal; cero ramificaciones por fuente.
- **Metadatos de origen (`sourceType`):** `synthetic | procedural | fixed_tutorial | fixed_challenge | sandbox_seed | replay | historical | historical_inspired` — reservados en el esquema desde el día 1.
- **El modo histórico es una fuente de datos futura, no el fundamento del MVP.** Cero ingesta, parsing o licencias de datos reales en MVP.

---

## 9. Cobertura de mercados

| Orden | Mercado | ¿MVP? | Lección única que enseña |
|---|---|---|---|
| 1 | Mercados sintéticos educativos | ✅ Base del MVP (`synthetic_training`) | Mecánica pura sin sesgo de activo (velas, órdenes, stop, riesgo, P/L) |
| 2 | Acciones (stocks) sintéticas | ✅ (`synthetic_stock`) | Horarios de mercado, gaps, soporte/resistencia, shocks tipo earnings |
| 3 | Índices sintéticos | ⚠️ Post-MVP temprano | Pensamiento macro, tendencia persistente, "sube por la escalera, baja por el ascensor" |
| 4 | Forex sintético | ✅ (`synthetic_fx`) | Sesiones (Asia/Londres/NY), spread variable, pips; leverage solo conceptual en MVP |
| 5 | Cripto sintética | ✅ (`synthetic_crypto`, desbloqueo tardío) | Volatilidad extrema, FOMO, liquidaciones, fin de semana ilíquido |
| 6 | Materias primas (commodities) | ⚠️ Post-MVP temprano | Shocks externos y narrativas de oferta/demanda |
| 7 | Futuros | ❌ Futuro (si es viable) | Margen, tamaño de contrato, tick value, liquidación profesional |

> Matriz cerrada por **`MVP_MARKET_LOCK`** (documento 12, sección 4): base sintética + 3 instrumentos jugables, índices post-MVP. Ante contradicción, gana el lock.

Decisión pedagógica central: el orden **invierte el camino típico del principiante LATAM** — cripto y leverage alto llegan al final, como logro de disciplina, no como punto de partida. Todos los instrumentos del MVP son **ficticios pero verosímiles** (nunca tickers reales), generados por un único motor parametrizado por perfil de mercado.

*(Fichas completas por mercado: documento 04.)*

---

## 10. Modelo de simulación

Motor en **16 capas separadas** con responsabilidad única (documento 03): LCC → plantilla de escenario → tipo de mercado → dificultad → secuencia de regímenes → perfil de volatilidad → perfil de liquidez → perfil de spread → calendario de eventos → seed → versión del generador → **path generado** → **hash del path** → decision log → motor de órdenes/riesgo/posición → motor de evaluación.

Regla de flujo: las capas 1–11 son entrada; la capa 12 se genera completa; se sella con la 13; **solo entonces** empieza la sesión; las capas 14–16 operan sobre un mercado ya inmutable.

Elementos clave:

- **Reloj de simulación** desacoplado del reloj real: pausa, tiempo real, x2/x3/x5/x10, avance vela a vela; la velocidad nunca cambia los datos, solo el ritmo de revelado.
- **Regímenes base:** tendencia alcista/bajista, rango, breakout, fakeout, reversión, shock de volatilidad, mercado lento/ilíquido. La plantilla define la secuencia; la seed define el detalle.
- **Sub-ticks intra-vela (4–8 por vela en MVP — `MVP_SANDBOX_LIMITS` y `DETERMINISM_LOCK_V1`)** pregenerados y deterministas: SL/TP, órdenes pendientes y liquidaciones se evalúan a nivel de sub-tick, nunca solo al cierre de vela.
- **Generación procedural por regímenes** con clusters de volatilidad, mechas, pullbacks, falsas salidas y ruido realista (anti-"mercado de juguete").
- **Horizonte largo: post-MVP.** El MVP limita los horizontes a Intradía y 1 semana (`MVP_SANDBOX_LIMITS`, documento 12, sección 7); los horizontes de 1 mes a 2 años por compresión en marcos mayores con checkpoints de decisión llegan en fases posteriores.

Enfoque de datos elegido para el MVP: **escenarios educativos prediseñados + generación procedural basada en regímenes + seeds deterministas.** Random walk puro: rechazado. Histórico: futuro.

---

## 11. Mecánicas de realismo

| Mecánica | Regla esencial |
|---|---|
| **Spread** | `spread = base × factor_volatilidad × factor_liquidez × factor_evento`; se amplía 3x–10x en eventos; visible y enseñable en cada vela |
| **Slippage** | Función determinista de volatilidad, liquidez, tipo de orden, tamaño relativo y contexto de evento; reproducible en replay; nunca tirada aleatoria al ejecutar |
| **Fees** | Comisión por operación + spread implícito + swap nocturno en horizontes largos; siempre desglosados en el resumen del trade |
| **Stop loss / take profit** | Evaluados a nivel de sub-tick contra el precio relevante (bid/ask); en gap o shock, el stop ejecuta al primer precio disponible (el stop limita el riesgo, no lo garantiza al céntimo) |
| **Gaps** | En aperturas de sesión, eventos y fines de semana simulados; tamaño y dirección decididos en la generación, jamás en reacción al usuario |
| **Liquidez** | Perfil continuo por sesión; liquidez baja ⇒ spread más amplio, slippage mayor, ejecución parcial conceptual; cae alrededor de shocks |
| **Eventos y shocks** | Calendario pregenerado (eventos anunciados y sorpresa) con dirección y magnitud decididas por la seed; nunca inyectados en reacción a la posición del usuario |
| **Leverage y liquidación** | En MVP: leverage 1x en toda mecánica jugable; la única liquidación existente es la del escenario educativo de sobreapalancamiento, con parámetros fijos empaquetados (`LEVERAGE_MVP_LIMITS`, documento 12, sección 5). El margen simulado general — donde la liquidación es matemática de margen, no castigo — llega en Fase 5. Reset educativo al quemar la cuenta en todos los casos |

---

## 12. Currículo para principiantes

**6 niveles, 29 lecciones**, de cero absoluto a avanzado, sin saltos (documento 02):

| Nivel | Etapa | Tema central | Lecciones | Relación con el MVP |
|---|---|---|---|---|
| 1 | Fundamentos del mercado | Mercado, precio, vela, long y short | 4 | Conceptos empaquetados en M1 |
| 2 | Supervivencia y riesgo | Riesgo, stop loss, take profit, sizing, risk/reward, señales | 6 | Conceptos empaquetados en M2/M3 (corazón del producto) |
| 3 | Estructura de mercado | Soportes/resistencias, trendlines, HH/LL, volumen | 4 | Conceptos esenciales en M1/M4 |
| 4 | Indicadores y patrones | MA, RSI, MACD, breakouts, pullbacks | 5 | Solo las trampas de 4.4/4.5 vía escenarios MVP; 4.1–4.3 post-MVP (cerrado) |
| 5 | Estilos de trading | Trend-following, mean reversion, scalping, day, swing, news | 5 | ❌ Post-MVP |
| 6 | Maestría y proceso | SMC, liquidez, backtesting, journaling, psicología | 5 | ❌ Post-MVP |

Reglas de orden no negociables: el riesgo se enseña antes que la estrategia; nada de indicadores antes de completar estructura; SMC y conceptos de liquidez solo después de fundamentos completos (incluso en modo libre — única excepción al modo libre); la psicología se detecta desde el Nivel 2 y se consolida al final.

El MVP empaqueta esto en **5 módulos / 15 lecciones cortas** (M1 mercado, M2 costos invisibles, M3 riesgo primero, M4 disciplina y proceso, M5 anti-señales), cada una con explicación + mini-simulación con seed fija + quiz + registro de errores. Los IDs canónicos y el mapeo están cerrados por **`MVP_CONTENT_LOCK`** (documento 12, sección 3); ninguna lección del currículo de 29 se implementa individualmente en el MVP.

---

## 13. Modo tutorial

- **Seeds fijas por lección**, versionadas, idénticas para todos los usuarios: las pistas pueden referenciar momentos exactos del mercado.
- Guía paso a paso con pausas automáticas en momentos clave; overlay de pistas descartable y re-invocable.
- Escenarios diseñados para forzar el contexto de la lección (la lección de slippage usa baja liquidez; la de drawdown, retrocesos profundos).
- El tutorial no se puede "ganar" con suerte: el score depende del LCC, no del P/L.
- Mensaje de confianza obligatorio en cada briefing: *"Este escenario fue generado antes de tu primera operación. Tus decisiones no cambian el camino del mercado; solo afectan tus órdenes, riesgo, resultados y evaluación."*
- Completar el tutorial desbloquea sandbox y el primer set de desafíos.

---

## 14. Modo sandbox

Práctica libre con configuración sellada al generar el path:

| Opción | Valores |
|---|---|
| Mercado | Sintéticos del MVP según `MVP_MARKET_LOCK` (forex, acción, cripto; índice post-MVP) |
| Capital inicial | Presets LATAM-realistas: 100, 500, 1.000, 5.000, 10.000, 100.000 |
| Dificultad | Iniciación, Estándar, Avanzada, Hostil |
| Horizonte | MVP: Intradía, 1 semana (`MVP_SANDBOX_LIMITS`). Post-MVP (Fase 3+): 1 mes, 3 meses, 6 meses, 1 año, 2 años |
| Velocidad | Pausa, 1x, 2x, 5x, 10x, salto de vela/checkpoint |
| Indicadores | Activados / desactivados (price action puro) |
| Leverage | No disponible en MVP — 1x incluso en Modo Libre (`LEVERAGE_MVP_LIMITS`). Post-MVP (Fase 5): según desbloqueo |
| Pistas (hints) | On/off; con hints la sesión se marca "asistida" y no compite en personal bests |
| Seed | Aleatoria por defecto; manual/guardada opcional |

Libertades MVP: guardar/reproducir/reiniciar seeds e historial completo. Guardar y retomar sesión larga y exportar sesión individual: post-MVP, Fase 3+ (`MVP_SANDBOX_LIMITS`; en MVP la portabilidad es vía export del progreso completo). Límites: nunca se modifica el path generado, nunca se edita el decision log, nunca se "deshace" un trade. La evaluación y la detección de errores siguen activas siempre.

---

## 15. Modo challenge (desafíos)

Todo desafío: **seed fija y sellada + plantilla fija + LCC fijo + reglas visibles antes de iniciar + sin hints**. Violar una regla marca el intento como fallido aunque el resultado monetario sea positivo. Reintentos ilimitados; solo el mejor puntaje se conserva como high score; el ranking principal se calcula con el **primer intento por seed**.

Tipos de desafío: diario, de habilidad, de disciplina de riesgo, sin indicadores, de supervivencia, trampa de copiado de señales, interés compuesto de largo plazo y control de drawdown.

Set inicial del MVP (6 challenges con seeds fijas, cerrado por `MVP_CONTENT_LOCK`): Supervivencia 50 velas · Riesgo de hierro (≤1% por trade) · Sin indicadores · Paciencia (máx. 3 operaciones) · Stop obligatorio · Costos reales (superar el break-even neto de spread + fees). **Criterio de éxito del MVP: completar al menos 3 de los 6.** Los challenges de horizonte largo (Supervivencia 90 días, interés compuesto, portafolio a 1 año, control de drawdown) son post-MVP.

Condiciones de fallo comunes: blow-up, margin call, violación de regla, drawdown máximo tocado, abandono. **Principio de fallo pedagógico:** toda derrota dispara la revisión de errores; el reinicio nunca está a un solo toque (anti-patrón casino).

---

## 16. Rankings de sesión y high scores

- **Todos los rankings son locales** (sin nube, sin login): el rival del usuario es su propia versión anterior.
- Claves de ranking MVP: por desafío y por seed guardada (`MVP_SANDBOX_LIMITS`). Las claves por mercado, por horizonte temporal y por dificultad son post-MVP.
- Cada high score guarda contexto completo: challenge/seed/tipo de seed/plantilla/LCC/versión del generador/dificultad/mercado/horizonte/score total y sub-puntajes/resultado de cuenta/supervivencia/fecha.
- Reemplazo solo si el score nuevo es estrictamente mayor; empate → mejor proceso → menor drawdown; el high score nunca baja.
- **Separación por versión del generador:** si cambia y altera los paths, los rankings antiguos se archivan como "temporada anterior"; nunca se mezclan puntajes de caminos no comparables.
- Sesiones de Modo Libre o asistidas guardan marcas separadas y no compiten en rankings de desafío.
- Replay: nunca compite en el ranking de primer intento; su métrica estrella es "errores corregidos".

---

## 17. Módulo educativo anti-señales ("Por qué fallan las señales")

Módulo dedicado al daño real más frecuente en LATAM: copiar señales de Telegram/TikTok/WhatsApp/Discord.

Tesis central:

> **Una señal no es un resultado. La misma señal, ejecutada por dos personas distintas, puede ganar para una y perder para la otra.** La diferencia está en el spread, el slippage, el entry delay, el tamaño, el leverage, el stop loss y el comportamiento emocional.

- **8 lecciones** (L1–L8) y **10 escenarios** (E1–E10) con seeds fijas; el MVP incluye 4 lecciones (misma señal/dos destinos, entrada tardía, leverage — solo conceptual, `LEVERAGE_MVP_LIMITS` —, sin invalidación) empaquetadas en M5.1/M5.2, 6 escenarios (E1, E2, E3, E5, E8, E9), el detector de errores reducido y 1 challenge ("Cazador de señales", integrado como mini-desafío de M5: **no** amplía el set canónico de 6 challenges de `MVP_CONTENT_LOCK`).
- **Process Score (0–100)** como métrica estrella: gestión de riesgo 30% + invalidación 25% + calidad de entrada 20% + disciplina de salida 15% + juicio de selección 10%. **El P/L no es componente del Process Score.**
- Momento "ajá" central del MVP: la misma señal con distinta ejecución da distinto resultado, y **se puede perder bien (E9) y ganar mal (E10)**.
- La abstención justificada (no entrar + razón en el journal) recibe el puntaje máximo cuando el R/R está destruido.

*(Detalle completo: documento 05.)*

---

## 18. Progresión por estilos de trading

Post-MVP (Nivel 5 del currículo): el usuario conoce los estilos principales, sus exigencias y costos, y descubre cuál se ajusta a su disciplina — no "cuál gana más":

| Estilo | Lección clave |
|---|---|
| Trend-following | Sostener la posición con salida por estructura; no cortar ganancias en la primera pausa |
| Mean reversion | Operar el regreso a la media en rangos y reconocer la invalidación por ruptura |
| Scalping / day trading | El costo real (spread + fees) devora las operaciones cortas; límite de operaciones por plan |
| Swing trading | Stops amplios bien dimensionados, gaps, paciencia ante el ruido intermedio |
| News trading | Spread alto, slippage y abstención durante el pico como decisión de calidad |

Cada estilo tiene su desafío con seed fija y métricas propias (tiempo en posición, costos acumulados, operaciones por sesión).

---

## 19. Especificación de UX móvil

*(Detalle completo: documento 07.)*

- **Navegación:** barra inferior de 5 pestañas — Academia (inicio), Simulador, Desafíos, Progreso, Perfil. El simulador es un modal de sesión que siempre se abre desde un briefing de escenario, nunca sin contexto.
- **25 pantallas especificadas:** onboarding, encuadre educativo, test de conocimientos, mapa de tutorial, lección, briefing, simulador, gráfico, panel de orden, posiciones, cuenta/equity, calculadora de riesgo, journal, controles de velocidad, selector de horizonte, dashboard de desempeño, evaluación final (doble puntaje: resultado + proceso), revisión de errores con línea de tiempo, sandbox, desafío, rankings locales, export/import, gestión de seeds/replay, ajustes.
- **Layout del simulador (mobile-first, vertical, zona del pulgar):** gráfico de velas 50–60% de pantalla → barra superior compacta → franja de cuenta → botones Comprar/Vender (≥48dp, con spread visible) → panel de orden como bottom sheet con vista previa de risk/reward, spread, fees y slippage → controles de simulación → paneles colapsables. **Confirmación de operación obligatoria** con resumen completo antes de ejecutar.
- **9 advertencias beginner-safe** (superficies #2E2E2E, texto #C9A050): riesgo alto, sin stop loss, leverage peligroso, entrada post-volatilidad, señal sin ajustar, overtrading, revenge trading, mover el stop, operar en alta volatilidad. Cada advertencia ignorada alimenta la revisión de errores.
- **UX de seeds (`SEED_PATH_REPLAY_EXPORT_LOCK`, documento 08):** en desafíos, el briefing muestra el **sello de equidad** (`pathHash` + `seedType` + `generatorVersion`), nunca la seed cruda, que se revela en la evaluación al cerrar el intento; en tutorial/sandbox la seed puede ser visible. Iconografía consistente 🔒 "Semilla fija" vs 🎲 "Semilla aleatoria"; badge "REPLAY" persistente; mensaje de confianza anti-manipulación en cada briefing.
- **Accesibilidad:** contraste WCAG AA, distinción no-solo-color en velas y P/L (▲/▼, +/−), escalado de fuente +30%, lectores de pantalla, reducción de movimiento, objetivos táctiles ≥44pt/48dp.
- Estética prohibida: verdes/rojos estridentes, negro puro, visuales de casino, UI infantil, "hazte rico".

---

## 20. Recomendación de stack técnico

> **Recomendación confirmada: React Native + TypeScript**, con el motor de simulación en TypeScript puro completamente separado de la UI.

| Capa | Tecnología |
|---|---|
| Framework móvil | React Native (Nueva Arquitectura: Fabric + TurboModules), Android 15+ / iOS 20+ |
| Lenguaje | TypeScript estricto en app y motor |
| Runtime JS | Hermes |
| Render de velas | `@shopify/react-native-skia` (canvas GPU); sin librerías de charting de terceros |
| Animaciones/gestos | Reanimated + Gesture Handler |
| Estado | Zustand (UI) + el motor como fuente de verdad de la simulación |
| Persistencia | SQLite (WAL) + MMKV (preferencias) |
| Export/import | JSON versionado + gzip + checksum SHA-256, extensión `.burgundy`, vía share sheet del SO |
| Tests | Vitest/Jest (motor), RN Testing Library (UI), Maestro/Detox (E2E) |

Alternativas evaluadas y descartadas: Flutter (segunda opción válida; menos talento JS/TS en LATAM), nativo dual (duplica el motor determinista — riesgo inaceptable), Unity/Godot (identidad de juego, UI de academia costosa), WebView (rendimiento y storage frágiles).

Riesgo principal y mitigación: determinismo de punto flotante entre runtimes → PRNG entero propio, aritmética de precios en enteros escalados, corpus dorado de hashes en CI.

---

## 21. Visión general de la arquitectura

**Burgundy es dos productos en uno: un motor de simulación determinista (librería pura `@burgundy/engine`) y una app móvil que lo consume.**

```text
CAPA UI (React Native): pantallas, HUD Burgundy, chart Skia, lecciones, gamificación
        ↓ (solo lee snapshots / despacha intenciones)
CAPA DE APLICACIÓN: SessionController · PlaybackClock · stores · viewmodels
        ↓
MOTOR (TypeScript puro, sin dependencias): PRNG · ScenarioTemplate · LCC Engine ·
RegimeGen · CandleGen · TickApprox · EventScheduler · Spread/Slippage ·
PathBuilder · HashValidator · DecisionLog · Order/Position/Risk/Evaluation Engine
        ↓
PERSISTENCIA: SQLite · MMKV · Export/Import JSON versionado
```

Reglas estrictas: la UI nunca importa internals del motor ni muta el path; las acciones del usuario son **intenciones** que el motor valida contra cuenta/órdenes/posiciones, jamás contra las velas; el motor no conoce el reloj real (recibe "avanza hasta la vela N"); el motor de órdenes tiene acceso de **solo lectura** al path.

Pipeline de generación (todo antes de la primera vela visible): seed → PRNG con streams → template → LCC (restringe y valida; re-deriva con `seed + attempt` determinista si el path no cumple el contrato) → regímenes → velas OHLC (enteros escalados) → sub-ticks → eventos → spread/slippage → **path inmutable + pathHash sellado** → runtime → evaluación.

---

## 22. Modelos de datos

*(Catálogo completo de 29 modelos: documento 09.)*

Modelos núcleo:

| Modelo | Rol | MVP | Export |
|---|---|---|---|
| `UserProfile` | XP, nivel, racha, alias local (uno por instalación) | ✅ | ✅ |
| `Market` / `Instrument` | Catálogo estático (bundle): horarios, spread, leverage máximo, fees | ✅ | ❌ (se referencia) |
| `Candle` — **formato universal de vela** | `timestamp, open, high, low, close, volume, bid, ask, spread, sourceType` + tags educativos (`volatilityTag`, `regimeTag`, `eventTag`) | ✅ | ⚠️ dentro de paths exportables |
| `TickEvent` | Sub-pasos intra-vela; en MVP se regeneran desde seed, no se almacenan | ⚠️ | ❌ |
| `Order` / `Trade` / `Position` / `Account` | Intención → ejecución → exposición viva → estado financiero por sesión (con MAE/MFE, `stopMoveHistory`, drawdown) | ✅ | ✅ |
| `SimulationSession` | Agregador central: modo, seed, estado, score, evaluación, replay | ✅ | ✅ |
| `Lesson` / `TutorialProgress` | Contenido (bundle) / progreso del alumno | ✅ | ❌ / ✅ |
| `LearningContextContract` | El contrato pedagógico declarativo (sección 6) | ✅ | ❌ (referencia por id + versión) |
| `ScenarioTemplate` / `Scenario` | Plantilla paramétrica / instancia resuelta (plantilla + seed + parámetros) | ✅ | ❌ / ✅ |
| `SeedRecord` | **Pieza mínima irrenunciable del export:** seed + tipo + generatorVersion + template + LCC + pathHash + replayable | ✅ | ✅ siempre |
| `GeneratedMarketPath` | Path inmutable materializado (blob comprimido, caché regenerable) | ✅ | ⚠️ selectivo |
| `Signal` | Señal simulada estilo Telegram con `qualityClass` y `hiddenFlawEs` | ⚠️ | respuesta ✅ / definición ❌ |
| `Mistake` | Catálogo de errores (definición en bundle) + ocurrencias detectadas | ✅ | ocurrencias ✅ |
| `UserDecisionLog` | Bitácora append-only de toda acción con tiempo simulado, precio, riesgo y `emotionalTag` opcional | ✅ | ✅ |
| `Evaluation` / `JournalEntry` | Veredicto educativo / diario guiado del alumno | ✅ | ✅ |
| `Challenge` / `RankingEntry` / `Achievement` | Desafíos (bundle) / high scores locales / logros de disciplina | ✅ | resultados ✅ |
| `ExportedProgressFile` | Formato de archivo `.burgundy` | ✅ | — |
| `HistoricalSourceMetadata` / `HistoricalPatternTemplate` | Contratos futuros reservados | ❌ | ❌ |

Regla transversal: las flechas de los datos del usuario apuntan **hacia** el path/seed (lectura), nunca al revés. Nada que haga el usuario escribe sobre `GeneratedMarketPath`, `SeedRecord` ni el contenido del bundle.

---

## 23. Persistencia local y sistema de export/import

**Tres capas:** SQLite con WAL (sesiones, trades, decision logs, progreso, journal, rankings, seeds, evaluaciones) · MMKV (preferencias de UI) · Bundle de la app (lecciones, LCC, plantillas, challenges, catálogo de errores, mercados).

- **Auto-save:** snapshot cada N velas y tras cada acción de trading; snapshot inmediato al pasar a background; decision log append-only insertado al ocurrir. Todo progreso está siempre guardado; escrituras de fin de sesión en una sola transacción.
- **Export:** archivo `.burgundy` según **`BURGUNDY_FILE_FORMAT_V1`** (documento 09) = envelope sin comprimir (`formatVersion`, `appVersion`, `generatorVersion`, `schemaVersion`, `exportedAt`, checksum SHA-256) + payload gzip (perfil, progreso, sesiones completas, SeedRecords siempre, paths selectivos según `SEED_PATH_REPLAY_EXPORT_LOCK`, decision logs, journal, evaluaciones, rankings, logros), con límites de tamaño cerrados. Entrega vía share sheet; Burgundy no sube nada a ningún servidor. Sin datos personales; sin cifrado en MVP (checksum de integridad basta).
- **Import:** validación estricta antes de tocar la base (formato → versión → checksum → esquema → referencias → pathHash de cada path); resumen previo; **reemplazo total con confirmación explícita + backup automático previo obligatorio** (merge inteligente es post-MVP); transacción sobre base temporal promovida atómicamente; errores con códigos y mensajes en español de la tabla del lock.
- **Política seed/path cerrada por `SEED_PATH_REPLAY_EXPORT_LOCK` (documento 08, §8):** `SeedRecord` y `pathHash` siempre; path materializado localmente solo para sesión activa/reciente y replays guardados explícitamente; export del path completo solo si la sesión está en curso o su `generatorVersion` no es regenerable. Si existe path almacenado, el replay lo usa (verificado contra hash); la regeneración versionada es fallback; hash no coincidente ⇒ sesión "no verificable", excluida de rankings.

---

## 24. Motor de escenarios (Scenario Engine)

*(Detalle completo: documento 10.)*

- **8 templates base:** `TPL_TREND`, `TPL_RANGE`, `TPL_FAKE_BREAKOUT`, `TPL_NEWS_SPIKE`, `TPL_HIGH_SPREAD`, `TPL_LOW_LIQUIDITY`, `TPL_GRIND`, `TPL_TRAP_SEQUENCE`.
- **Pipeline de 15 pasos** en orden estricto: objetivo de lección → LCC → template → mercado → dificultad → horizonte → volatilidad/liquidez/spread → seed → generatorVersion → **generación completa del path** → **hash** → inicio de sesión → registro de decisiones → evaluación contra el LCC → guardado local exportable. Los pasos 1–11 son previos e inmutables.
- **Catálogo de 16 escenarios especificados**, entre ellos: "Tu primera operación" (único sin trampa), "Día de tendencia", "Día de rango", "La ruptura que no era" (fake breakout), "Spike de noticia", "Spread alto", "Baja liquidez", "La señal del grupo", "Apalancamiento: la espada de doble filo", "Después de la pérdida" (revenge trading), "El 1% que te mantiene vivo", más escenarios de largo plazo y challenges de supervivencia (parte post-MVP).
- Cada escenario declara: nivel requerido, template, LCC, tipo y comportamiento de seed, trampa oculta, error esperado, decisiones buenas aunque pierdan, decisiones malas aunque ganen, condiciones de éxito/fracaso, métricas, feedback canónico, replay, desbloqueo, disponibilidad en modo libre, compatibilidad histórico-inspirada y fase (MVP/posterior).

---

## 25. Sistema de evaluación y scoring

*(Detalle completo: documento 11.)*

**Siete categorías** (0–100 cada una, ponderadas): Rentabilidad, Gestión de riesgo, Disciplina, Calidad de proceso, Paciencia, Consistencia de estrategia y Supervivencia. **La rentabilidad nunca pesa más del 25%** en ningún modo; cada LCC declara su `scoringProfile` con pesos que suman exactamente 100 (en una trampa FOMO, la paciencia pesa 35% y la rentabilidad 0%). El contrato ejecutable — fórmulas por tramos con quiebres numéricos, thresholds resueltos por escenario, predicados de "setup válido"/"oportunidad válida", reglas de abstención, integración del Process Score anti-señales como perfil `senales_proceso` y casos dorados por seed fija — está cerrado por **`SCORING_V1_LOCK`** (documento 11).

Regla de oro verificable (requisito del motor, no sugerencia):

> **Una sesión rentable con violaciones graves de riesgo debe puntuar por debajo de una sesión levemente perdedora con proceso impecable.**

- **Grades S/A/B/C/D/F** con caps duros: blow-up = F automática; >50% de trades sin stop → máx. C; violar el objetivo de la lección → máx. C; revenge trading 2+ veces → máx. C.
- **Detección determinista de errores sobre el decision log** (entrada sin stop, stop movido en contra, sobreapalancamiento, revenge, FOMO, overtrading, promediar pérdidas, señal copiada sin verificar…) — reglas objetivas y auditables, con penalizaciones acumulables por evento (no promediadas) y bonus con topes (pérdida bien gestionada, trampa evitada, racha de disciplina).
- **Feedback estructurado:** proceso primero y P/L después; máximo 3 aciertos + 1 error principal con la vela exacta; el principio detrás; qué practicar después. Siempre cita evidencia del decision log; tono serio, sin humillación, sin hype.
- **Anti-exploit:** rentabilidad saturada, caps de grade, ranking principal por primer intento, detección de ganancia concentrada en 1 trade, replay etiquetado y excluido del ranking de primer intento.

---

## 26. Alcance del MVP

Tesis que el MVP debe demostrar:

> **Es posible enseñar disciplina de trading real con un simulador que fuerza el contexto de aprendizaje, no el resultado.**

| Bloque | Contenido MVP |
|---|---|
| Motor | Generación procedural por regímenes con seed determinista; pre-generación total + hash; sub-ticks; spread/slippage/fees deterministas; gaps y eventos; versionado del generador |
| Mercados | 3 instrumentos sintéticos según `MVP_MARKET_LOCK` (`synthetic_fx`, `synthetic_stock`, `synthetic_crypto` con desbloqueo tardío) sobre la base `synthetic_training`, ficticios y verosímiles; índices post-MVP |
| Contenido | 5 módulos / 15 lecciones (M1–M5) con seeds fijas, quizzes y mini-simulaciones; módulo anti-señales reducido — IDs cerrados por `MVP_CONTENT_LOCK` |
| Modos | Tutorial guiado, sandbox mínimo según `MVP_SANDBOX_LIMITS` (seeds aleatorias + guardar/repetir; horizontes Intradía/1 semana; sin leverage), 6 challenges con seeds fijas (≥3 completados = éxito), modo libre, modo sin indicadores, progresión con desbloqueos |
| Órdenes | Market, limit, stop + SL/TP; cierre manual; reset por blow-up con revisión educativa; leverage 1x (`LEVERAGE_MVP_LIMITS`) |
| Evaluación | Score completo por LCC, grades, detección de errores núcleo (10–15), journal guiado, revisión de errores con replay exacto |
| Gamificación | XP, niveles, racha diaria, desbloqueos, high scores y rankings locales (claves MVP: por desafío y por seed guardada) |
| Persistencia | 100% local; export/import `.burgundy` con validación y backup |
| UX | Las 25 pantallas esenciales, las 9 advertencias beginner-safe, accesibilidad AA |

**Excluido del MVP sin excepción:** replay histórico real e ingesta de datos, integración con brokers, dinero real, señales reales, coach de IA, monetización, login/nube, cursos, rankings online entre usuarios, otros idiomas, leverage/margen avanzado y futuros (Fase 5), índices sintéticos y commodities (post-MVP temprano), horizontes de sandbox superiores a 1 semana y export de sesión individual (Fase 3+).

**Locks vigentes (resumen, una línea por lock):**

| Lote | Lock | Documento dueño |
|---|---|---|
| A | `MVP_MARKET_LOCK` — matriz canónica de mercados MVP | `12_mvp_scope.md` |
| A | `MVP_CONTENT_LOCK` — contenido educativo canónico MVP | `12_mvp_scope.md` |
| A | `MVP_SANDBOX_LIMITS` — sandbox mínimo y presupuesto de simulación | `12_mvp_scope.md` |
| A | `LEVERAGE_MVP_LIMITS` — leverage 1x salvo escenario cerrado | `12_mvp_scope.md` |
| B | `SEED_PATH_REPLAY_EXPORT_LOCK` — política única seed/path/replay/export/visibilidad | `08_technical_architecture.md` |
| B | `DETERMINISM_LOCK_V1` — PCG32, substreams, enteros escalados, serialización canónica, corpus dorado | `08_technical_architecture.md` |
| B | `SCORING_V1_LOCK` — `scoring_v1` ejecutable, thresholds, predicados, casos dorados | `11_evaluation_scoring.md` |
| B | `BURGUNDY_FILE_FORMAT_V1` — formato `.burgundy` v1 e import transaccional | `09_data_models.md` |

Ante contradicción entre cualquier sección de este documento (o de los documentos 01–12) y un lock, gana el lock.

Criterios de éxito (verificables): tutorial + 3 challenges + sandbox completables 100% offline; todo path generado antes de la primera acción con hash verificable; mismo seed ⇒ mismo hash en Android e iOS; score explicable al 100% por el LCC; export → instalación limpia → import sin pérdida; principiantes capaces de explicar spread, slippage, stop loss, drawdown y risk/reward; cero estética prohibida.

---

## 27. Roadmap de desarrollo

| Fase | Nombre | Objetivo | Criterio de completitud clave |
|---|---|---|---|
| 1 | Prototipo: motor de seeds sintético | Probar el corazón: generación determinista + LCC + replay | Mismo seed ⇒ mismo hash en Android e iOS; un escenario jugable de inicio a score |
| 2 | **MVP**: tutorial + challenge seeds | Producto educativo completo (sección 26) | Todos los criterios de éxito del MVP |
| 3 | Sandbox + portabilidad | Seeds aleatorias, guardar/repetir, export/import completo | Export → wipe → import sin pérdida |
| 4 | Expansión educativa | Más lecciones, mercados sintéticos, plantillas y reglas de error | Agregar una lección no requiere tocar el motor |
| 5 | Mercados avanzados | Leverage/margen completo, mecánica estilo futuros, challenges avanzados | Liquidación simulada correcta; leverage inaccesible sin educación previa |
| 6 | Formato histórico-compatible | Esquema externo de velas + validación, sin ingerir datos aún | Un archivo de velas externo válido se reproduce igual que un path sintético |
| 7 | Modo replay histórico (opcional) | Operar sobre segmentos reales importados | Segmento real operable y repetible; la app sigue funcionando sin ningún dato histórico |
| 8 | Patrones histórico-inspirados | Extraer plantillas de episodios reales para el generador procedural | Plantilla extraída genera paths deterministas de calidad equivalente a las manuales |

Lógica del roadmap en una línea:

> Primero el motor determinista, luego la academia, luego la profundidad de mercado, y solo al final los datos reales — porque Burgundy enseña disciplina, y la disciplina no necesita datos históricos para entrenarse, pero sí un motor que jamás mienta sobre el precio.

---

## 28. Riesgos y limitaciones

| Riesgo | Impacto | Mitigación |
|---|---|---|
| Paths procedurales poco creíbles ("mercado de juguete") | Alto: rompe la inmersión educativa | Plantillas calibradas, clusters de volatilidad, validación estadística de los paths |
| Determinismo roto entre plataformas | Alto: replays y rankings inválidos | PRNG entero propio, enteros escalados, corpus dorado de hashes en CI cross-runtime |
| Percepción de juego de azar | Alto: contradice la identidad | Score 100% por LCC, feedback con "por qué", tono serio, anti-patrón casino en reinicio |
| Rendimiento del chart en gama baja LATAM | Medio | Skia GPU, ventana deslizante, presupuesto explícito (60 fps / ~120 velas), pruebas en hardware real |
| Corrupción de datos locales o del export | Medio | SQLite WAL, transacciones, checksums, backup rotativo, validación estricta de import |
| Alcance creciente ("¿y si agregamos margen?") | Medio: retrasa el MVP | Este documento y el 12 son el contrato de alcance; todo lo nuevo va a fases |
| Sobrecarga de texto educativo | Medio | Lecciones cortas, una idea por lección, simulación antes que teoría |
| Evolución del generador rompe replays/rankings antiguos | Medio | `generatorVersion` + paths almacenados de sesiones importantes + temporadas de ranking |

Limitaciones asumidas y declaradas: Burgundy no simula un order book real (sub-ticks + perfiles de liquidez son una aproximación honesta y suficiente para educación); el sintético enseña mecánica y disciplina, no garantiza resultados en mercados reales (y la app lo dice explícitamente); el éxito en el simulador nunca se presenta como preparación certificada para operar dinero real.

---

## 29. Learning Context, Sistema de Seeds y Diseño de Simulación Historical-Ready

Sección integral de los 20 puntos requeridos:

1. **Learning Context Contract:** contrato pedagógico declarativo y pregenerado que define objetivo, contexto garantizado, trampas, decisiones buenas/malas, criterios de evaluación, reglas de feedback y comportamiento de seed/replay — antes de que el usuario actúe (sección 6).
2. **Forzar contexto vs. forzar resultado:** Burgundy controla que el escenario *contenga* la situación a aprender (decidido antes de la sesión, en la generación); jamás controla si el usuario *gana o pierde* (decidido por sus decisiones contra un path inmutable). Ejemplo legítimo: "esta lección genera un rango con una falsa ruptura en el tramo medio". Ejemplo prohibido: "el usuario entró en largo, entonces el precio cae para enseñarle humildad". El LCC define el contexto; la seed define la instancia; el usuario define el resultado — tres responsabilidades, tres dueños, sin cruces.
3. **Sistema de seeds determinista:** misma plantilla + parámetros + seed + versión del generador ⇒ mismo path, byte a byte. PRNG entero propio con streams por subsistema; sin entropía externa (sección 7).
4. **Tipos de seed:** tutorial fija, desafío fija, sandbox aleatoria, replay, histórico-inspirada (futura) y replay histórico exacto (futuro).
5. **Lógica de replay:** path + decision log = sesión completa reproducible. Dos modos: reintento (mismo mercado, decisiones nuevas) y revisión de errores (decisiones originales anotadas sobre el gráfico). El replay valida el hash antes de iniciar; nunca compite en el ranking de primer intento; su métrica estrella es "errores corregidos".
6. **Equidad de desafíos (`SEED_PATH_REPLAY_EXPORT_LOCK`, documento 08):** seed fija + plantilla fija + versión fija ⇒ todos los intentos enfrentan exactamente el mismo mercado; el hash permite verificar que nadie jugó contra un camino distinto. Antes del intento se muestra solo el sello de equidad (`pathHash` + `seedType` + `generatorVersion` + reglas del LCC); la seed cruda se revela al cerrar el intento; intentos con seed conocida de antemano se marcan `seed_known = true` y no son elegibles para el ranking principal de primer intento por seed.
7. **Aleatorización del sandbox:** seeds aleatorias que pasan siempre por el generador de regímenes (variación impredecible pero estructuralmente realista, nunca ruido puro); en modo `random_constrained`, el LCC garantiza que los regímenes requeridos aparezcan; toda seed se registra y puede guardarse con nombre.
8. **Persistencia local de seeds:** todo `SeedRecord` (seed + tipo + generatorVersion + template + LCC + parámetros + pathHash) se guarda en SQLite y es **permanente** mientras exista una sesión que lo referencie — pesa bytes y es la garantía de replay.
9. **Export/import de sesiones con seeds:** los `SeedRecord` viajan **siempre** en el archivo `.burgundy`; las sesiones completas incluyen órdenes, trades, decision log y evaluación; los paths completos viajan solo cuando importa el replay exacto; el import valida formato, checksum y pathHash antes de aplicar.
10. **Arquitectura historical-ready:** formato universal de vela + abstracción de fuente de datos + motores indiferentes al origen + `sourceType` reservado en el esquema. Cuando exista un adaptador histórico, producirá el mismo formato y nada más cambia.
11. **Roadmap del modo histórico futuro:** Fase 6 (formato externo compatible y validación, sin ingesta), Fase 7 (replay histórico real opcional, importado por el usuario), Fase 8 (extracción de patrones e histórico-inspirados). La app funciona íntegramente sin ningún dato histórico en todas las fases.
12. **Generación de escenarios histórico-inspirados:** se extraen propiedades estadísticas y narrativas de episodios reales (volatilidad, gaps, velocidad, magnitud, secuencia de regímenes) y se parametriza el generador procedural — sin reproducir series de precios reales ni requerir el dataset en el dispositivo; con seed fija versionada y capa narrativa educativa que pasa el mismo estándar LCC.
13. **Riesgos de la manipulación post-trade (por qué está prohibida):** genera aprendizaje falso y supersticiones ("el mercado me vio entrar"), destruye la confianza cuando se descubre, invalida los rankings y contradice la transferencia al mercado real. Es el patrón que hace que las demos "tramposas" arruinen la educación del principiante; Burgundy lo bloquea por arquitectura, no por política.
14. **Regla: el market path se genera antes de la acción del usuario.** Completo — velas, sub-ticks, spreads, liquidez y eventos — antes del primer frame de la sesión, y se sella con su hash.
15. **Regla: las decisiones del usuario afectan órdenes, riesgo, P/L, score y feedback, pero nunca el path pregenerado.** El motor de órdenes tiene acceso de solo lectura al path; por diseño no existe ninguna vía por la que una orden modifique una vela.
16. **Versionado del generador:** `generatorVersion` semántico almacenado con cada sesión; cualquier cambio que altere la salida para alguna seed incrementa la versión; CI mantiene un corpus dorado (template + contrato + params + seed → pathHash esperado); los rankings separan temporadas por versión; los desafíos nunca cambian de versión a mitad de su periodo.
17. **Validación por hash del path:** SHA-256 de la serialización canónica, calculado y persistido **antes** de iniciar la reproducción; verificado en replay, import, rankings y tests de regresión; un hash que no coincide marca la sesión como no verificable y la excluye de rankings — nunca se "repara" silenciosamente.
18. **Formato universal de vela:** `timestamp, open, high, low, close, volume, bid, ask, spread, sourceType` (+ tags educativos opcionales de volatilidad/liquidez/régimen/evento), en enteros escalados. Toda fuente presente y futura entrega exactamente esta estructura.
19. **Motor de simulación agnóstico de fuente:** Order, Position, Risk, Evaluation y Replay Engine consumen exclusivamente el formato universal; cero ramificaciones por origen de datos, protegido con tests de contrato.
20. **El modo histórico es una fuente de datos futura, no el fundamento del MVP.** Los datos históricos reales no son requisito de ninguna funcionalidad del MVP; la arquitectura solo respeta el contrato para no cerrarse la puerta.

---

## 30. Recomendación final de construcción

**Construir el MVP de Burgundy con:**

- **Escenarios sintéticos/procedurales** generados por regímenes con realismo educativo calibrado (clusters de volatilidad, mechas, pullbacks, fakeouts, eventos), sin ningún dato histórico.
- **Learning Context Contracts** declarativos para toda lección, sandbox y desafío: el score se calcula exclusivamente contra el contrato, premiando disciplina, riesgo, proceso, paciencia y supervivencia por encima del P/L.
- **Sistema de seeds determinista** con PRNG entero propio, hash sellado del path y versionado del generador.
- **Seeds fijas para tutorial y desafíos** (equidad verificable y rankings justos) y **seeds aleatorias para sandbox** (variedad infinita, siempre reproducible).
- **Replay exacto** (mismo seed = mismo mercado) como herramienta central de revisión de errores y mejora entre intentos.
- **Export/import local** (`.burgundy`, versionado, con checksum) como único mecanismo de portabilidad — sin login, sin nube, offline-first total.
- **Scoring basado en proceso** con la regla de oro: una sesión rentable con violaciones graves de riesgo puntúa por debajo de una sesión levemente perdedora con proceso impecable.

**No construir el modo histórico primero.** El histórico añade costos de datos, licencias, peso y complejidad sin aportar nada a la tesis del MVP: la disciplina se entrena con un motor que jamás miente sobre el precio, no con datos reales.

**Diseñar la arquitectura para que el modo histórico se añada después sin reconstruir el simulador:** formato universal de vela, abstracción de fuente de datos, `sourceType` reservado en el esquema, motores agnósticos al origen y formato de export compatible hacia adelante. Cuando llegue el histórico (Fases 6–8), entrará como una fuente más por el mismo contrato — y nada del motor, la evaluación, el replay ni el progreso del usuario cambiará.

---

*Documento maestro de especificación final del proyecto **Burgundy**, consolidado y firmado por **tsuloid**. Burgundy es una aplicación exclusivamente educativa: no es asesoría financiera, no opera dinero real y no se conecta a brokers. Siguiente paso del proceso: inicio del desarrollo según el roadmap (Fase 1 — Prototipo del motor de seeds sintético). Este documento no incluye código ni pantallas.*
