# Burgundy — Definición de Producto (Documento de Cierre)

**Documento:** 01_product_definition.md
**Proyecto:** Burgundy — Simulador educativo de trading
**Autor / Firma:** tsuloid
**Idioma:** Español (LATAM)
**Estado:** Definición de producto bloqueada (Product Definition Lock)
**Alcance de este documento:** Solo definición de producto. No incluye código, ni pantallas, ni arquitectura técnica completa.

---

## 1. Visión del producto

Burgundy es un simulador móvil educativo de trading, diseñado como una **academia seria dentro de una terminal de mercado oscura con identidad Burgundy**, con progresión de largo plazo estilo juego.

La visión central:

> Que un principiante de LATAM pueda aprender disciplina de mercado real —gestión de riesgo, paciencia, control emocional y calidad de proceso— practicando en un entorno simulado, justo, repetible y sin riesgo de dinero real, sintiendo que progresa como en un juego serio, no como en un casino.

Burgundy no enseña a "ganar dinero rápido". Enseña a **sobrevivir, decidir bien y entender por qué los traders principiantes fracasan**, antes de que el usuario arriesgue un solo peso real.

El sentimiento objetivo del usuario:

> "Esto es un sistema de entrenamiento serio. Estoy subiendo de nivel como en un juego, pero estoy aprendiendo disciplina real de mercado."

---

## 2. Posicionamiento del producto

| Dimensión | Posicionamiento de Burgundy |
|---|---|
| Categoría | Simulador educativo de trading, móvil, propietario |
| Mercado | LATAM, solo en español |
| Tono | Academia seria + terminal financiera táctil de alta gama |
| Identidad visual | HUD oscuro tema Burgundy: fondos #1A1617, acento #571324, superficies #2E2E2E, dorado #C9A050, velas alcistas #4A6D56 y bajistas #802F3E |
| Modelo de acceso | Sin login, sin cuenta en la nube, sin monetización, offline-first |
| Diferenciador clave | Equidad de simulación verificable (sistema de semillas determinista + Learning Context Contract) y recompensa por proceso, no solo por ganancia |
| Lo que NO es | No es broker, no es casino, no es curso, no es señalero, no es asesoría financiera |

Posicionamiento frente a alternativas comunes en LATAM:

- **Frente a cuentas demo de brokers:** Burgundy enseña con contexto pedagógico, retroalimentación de errores y progresión; una demo solo deja operar sin guía.
- **Frente a apps gamificadas tipo "juego de bolsa":** Burgundy es serio, no infantil, no usa verdes/rojos estridentes ni mensajes de "hazte rico".
- **Frente a cursos y señales de Telegram/TikTok:** Burgundy enseña por qué copiar señales sin contexto, timing, control de riesgo y disciplina de ejecución falla, y lo demuestra en simulación.

---

## 3. Usuario objetivo

Usuario principal: **principiante de LATAM, vulnerable, que busca disciplina, no entretenimiento.**

Características generales:

- Adulto joven (aprox. 18–40 años), hispanohablante, usuario primario de smartphone (a menudo Android de gama media).
- Conectividad intermitente o datos limitados → necesidad de offline-first.
- Poco o nulo conocimiento financiero formal; vocabulario de trading aprendido de redes sociales, a menudo mal entendido.
- Capital potencial pequeño (cuentas reales hipotéticas de 50–500 USD), por eso los ejemplos usan cuentas pequeñas.
- Desconfianza razonable hacia apps que piden registro, datos personales o pagos → sin login, sin nube, sin monetización.

---

## 4. Perfil del principiante LATAM

Comportamientos típicos que Burgundy debe abordar directamente:

| Comportamiento observado | Riesgo real | Cómo Burgundy lo trata |
|---|---|---|
| Copiar señales de Telegram/TikTok | Opera sin contexto, sin timing ni gestión de riesgo | Escenarios de "trampa de señal copiada" que evalúan la decisión, no el resultado |
| Usar apalancamiento excesivo (leverage) | Liquidación rápida de la cuenta | Escenarios de riesgo de liquidación; explicación clara de leverage en español |
| Entrar sin stop loss | Pérdidas descontroladas y drawdown profundo | Detección de error, revisión de errores y penalización en la puntuación de proceso |
| Confundir ganancia con habilidad | Sobreconfianza tras suerte | Evaluación por calidad de decisión: una ganancia con mal proceso recibe retroalimentación crítica |
| Overtrading y revenge trading | Quemar la cuenta por emoción | Desafíos de disciplina de riesgo y detección de patrones de sobreoperación |
| FOMO en picos de precio | Comprar máximos en pánico de oportunidad | Escenarios pregenerados de fake breakout y FOMO trap |

Nota de tono: el usuario es vulnerable. Burgundy nunca lo ridiculiza, nunca lo sobreprotege con pantallas legales pesadas, y nunca le promete ganancias. Le habla claro, directo y en serio.

---

## 5. Promesa educativa central

> **"Burgundy te entrena para tomar decisiones de trading disciplinadas y sobrevivir en el mercado, explicándote la terminología real en español claro, en un simulador justo que nunca manipula el precio en tu contra."**

Componentes de la promesa:

1. **Terminología real, explicada en claro.** Se usan los términos en inglés cuando son estándar, con explicación simple:
   - *Spread:* la diferencia entre el precio de compra y el precio de venta.
   - *Slippage:* recibir un precio de ejecución peor del que esperabas.
   - *Drawdown:* cuánto cae tu cuenta desde su punto más alto.
   - *Leverage (apalancamiento):* controlar una posición más grande con menos capital, aumentando tanto el riesgo como el retorno potencial.
   - *Liquidez:* qué tan fácil es entrar o salir sin mover demasiado el precio.
   - *Risk/reward (riesgo/beneficio):* cuánto arriesgas comparado con cuánto buscas ganar.
   - *Stop loss / Take profit:* órdenes que cierran tu posición automáticamente para limitar la pérdida o asegurar la ganancia.
2. **Simulación justa y verificable.** El mercado nunca se mueve en tu contra "porque sí" (ver sección 11).
3. **Recompensa por proceso.** Disciplina, paciencia, supervivencia, control de riesgo y consistencia valen más que una racha de suerte.
4. **Progresión de largo plazo.** XP, niveles, rachas, desbloqueos y rankings serios, sin estética de casino.

---

## 6. Qué es la app

Burgundy **es**:

- Un simulador educativo de trading, móvil y propietario (Android 15+, iOS 20+; stack preferente React Native salvo que otro sea objetivamente superior — decisión a cerrar en el documento de arquitectura).
- Una app **solo en español**, dirigida a LATAM.
- **Offline-first**: funciona sin conexión; todo el progreso se guarda localmente.
- **Sin login y sin cuenta en la nube**; el usuario puede **exportar un archivo de progreso/sesión e importarlo después** para continuar.
- Una academia estructurada: lecciones, rutas de aprendizaje, niveles, explicaciones guiadas, repaso de conceptos, revisión de errores, seguimiento de progreso y retroalimentación clara tras cada decisión.
- Un HUD de trading oscuro y serio: gráfico de velas como foco visual principal; paneles con balance, equity, riesgo, P/L, spread, slippage, comisiones (fees) y drawdown visibles cuando son relevantes.
- Un sistema de gamificación seria de largo plazo: racha diaria de práctica, XP, niveles, desbloqueo de habilidades, desafíos, puntuaciones máximas guardadas, rankings por sesión, desafíos de supervivencia, de disciplina de riesgo y sin indicadores.
- Un simulador con **modos**: tutorial guiado, sandbox, desafíos, modo de progresión con desbloqueos, modo libre sin restricciones de desbloqueo, y modo opcional sin indicadores para aprender price action puro.
- Un sistema que enseña: fundamentos de price action, velas, posiciones long y short, gestión de riesgo, stop loss, take profit, position sizing, ratio riesgo/beneficio, spreads, slippage, comisiones, volatilidad, drawdown, errores emocionales, overtrading, revenge trading, FOMO, y por qué las señales copiadas fallan.
- Mercados contemplados: mercados sintéticos educativos, acciones (stocks), forex, cripto, índices, materias primas (commodities), y futuros más adelante si es viable.

---

## 7. Qué NO es la app

Burgundy **no es**:

| No es | Implicación |
|---|---|
| Trading con dinero real | Cero integración con brokers reales; cero conexión a cuentas en vivo |
| Asesoría financiera | Nada en la app constituye recomendación de inversión |
| Un casino ni un juego barato | Sin estética de apuestas, sin verdes/rojos estridentes, sin gráficos infantiles |
| Una promesa de riqueza | Prohibido el mensaje "hazte rico", el hype y el lujo aspiracional |
| Una plataforma con cuentas | Sin login, sin nube, sin recolección de datos personales |
| Un producto monetizado | Sin compras, sin suscripciones, sin anuncios (en este alcance) |
| Un catálogo de cursos | La enseñanza vive dentro del simulador y sus lecciones, no como cursos vendibles (definición canónica en §7.1) |
| Un coach de IA | Sin asistente de IA conversacional |
| Un manipulador de mercado | El simulador jamás mueve el precio contra el usuario tras detectar su operación |
| Algo que hace ver el trading fácil | El producto comunica explícitamente la dificultad y el riesgo |

### 7.1 Definición canónica: "sin cursos" vs micro-lecciones (AUD-019)

**"Sin cursos" significa que Burgundy no es una plataforma de cursos.** En concreto, queda **prohibido**:

- cursos pagos y cualquier monetización educativa (paywall, suscripciones, certificaciones);
- LMS o módulos comerciales tipo academia online;
- catálogo externo o marketplace de educación;
- mentoría, profesores/coaches o comunidad de alumnos;
- contenido de terceros o módulos pasivos tipo video-curso;
- login o cuenta para acceder a contenido educativo.

**"Sin cursos" no prohíbe las micro-lecciones educativas internas**, que sí son parte del producto porque enseñan a usar la simulación y a entender decisiones de trading. Quedan **permitidos**:

- tutoriales integrados y onboarding educativo;
- explicaciones contextuales dentro de la app;
- los módulos de aprendizaje del MVP (las 15 lecciones M1–M5 de `MVP_CONTENT_LOCK`);
- micro-lecciones vinculadas a escenarios y sus Learning Context Contracts;
- feedback posterior a decisiones;
- conceptos mínimos para principiantes (glosario y repasos).

Esta definición vive **una sola vez aquí**; los documentos 02, 07 y 13 la citan, no la redefinen. Regla para implementación (Prompt 14): las micro-lecciones internas se implementan como parte del simulador; nunca se modelan, nombran ni presentan como "course product", plataforma de cursos, LMS ni contenido monetizado.

---

## 8. Recorrido principal del usuario (Core User Journey)

1. **Primera apertura (sin fricción):** sin registro ni login. Breve presentación de la identidad de Burgundy y del marco educativo (sin pantallas legales pesadas, sin promesas de ganancia).
2. **Tutorial guiado:** escenarios con semilla fija y repetible que enseñan velas, long/short, stop loss, take profit y lectura básica del HUD. Retroalimentación inmediata tras cada decisión.
3. **Primeras lecciones de riesgo:** position sizing, risk/reward, spread, slippage, comisiones y drawdown, con cuentas pequeñas y ejemplos prácticos LATAM.
4. **Sandbox:** práctica libre con semillas aleatorias (opcionalmente guardables y reproducibles). El usuario experimenta sin presión de ranking.
5. **Desafíos:** escenarios con semilla fija (idéntica para todos los intentos) — supervivencia, disciplina de riesgo, sin indicadores, trampas de FOMO/señales. Puntuación máxima guardada y ranking por sesión.
6. **Progresión:** XP, niveles, desbloqueo de mercados y herramientas en el modo de progresión; el modo libre permite acceder a todo sin restricciones para quien lo prefiera.
7. **Error y reinicio:** si el usuario quema la cuenta o llega a cero, la app aplica el comportamiento de reset/reinicio con revisión educativa de lo ocurrido (replay exacto del mismo mercado gracias a la semilla).
8. **Revisión de errores y diario:** el usuario revisa sus decisiones, los errores detectados (overtrading, revenge trading, FOMO, falta de stop) y puede reintentar el mismo escenario exacto con decisiones distintas.
9. **Continuidad:** racha diaria de práctica; exportación del archivo de progreso para respaldo o cambio de dispositivo, e importación para continuar.

---

## 9. Transformación principal de aprendizaje

| Estado inicial del usuario | Estado final buscado |
|---|---|
| Copia señales sin entender | Evalúa contexto, timing y riesgo antes de cualquier entrada |
| Entra sin stop loss | Define stop loss, take profit y tamaño de posición antes de entrar |
| Confunde suerte con habilidad | Juzga sus operaciones por calidad de proceso, no por resultado |
| Usa apalancamiento como atajo | Entiende leverage como amplificador de riesgo y lo dosifica |
| Opera por FOMO y venganza | Reconoce sus patrones emocionales y sabe abstenerse (skip es una decisión válida y a veces la mejor) |
| Cree que el trading es fácil | Entiende drawdown, costos (spread, slippage, fees) y la dificultad real de sobrevivir |

La transformación se mide por comportamiento dentro del simulador (ver sección 18), nunca por promesas de resultados con dinero real.

---

## 10. Principios de producto

1. **Proceso sobre resultado.** La app no recompensa solo la ganancia: recompensa calidad de proceso, supervivencia, disciplina, control de riesgo, paciencia y consistencia.
2. **Equidad de simulación verificable.** El mercado se genera antes de que el usuario actúe; jamás se manipula después (sección 11).
3. **Terminología real, explicada.** Nunca se infantiliza el vocabulario; siempre se explica en español claro.
4. **Seriedad visual y de tono.** Jerarquía visual seria, paleta Burgundy exacta, sensación de terminal financiera de alta gama; nada de casino ni juego barato.
5. **Privacidad por diseño.** Sin login, sin nube, datos locales, exportación controlada por el usuario.
6. **Offline-first.** Toda funcionalidad central opera sin conexión.
7. **El error es material didáctico.** Perder, quemar la cuenta o caer en una trampa genera la mejor retroalimentación, no castigo humillante.
8. **Saltar la operación es una decisión.** No operar en un mal contexto puede ser la decisión correcta y se evalúa como tal.
9. **Sin promesas de ganancia.** Encuadre educativo sin pantallas legales pesadas, pero con cero mensajes de lucro.
10. **Historical-ready, no historical-dependent.** El motor acepta futuras fuentes históricas, pero el MVP no las requiere (sección 14).

---

## 11. Reglas de equidad de la simulación (Simulation Fairness Rules)

Estas reglas son **innegociables** y definen la integridad del producto:

1. **La app fuerza un contexto de aprendizaje, no un resultado de trading.** Puede crear un contexto diseñado para enseñar una lección específica: fake breakout, FOMO trap, pico de noticias, shock de volatilidad, entorno de spread alto, día de rango, día de tendencia, trampa de señal copiada o escenario con riesgo de liquidación.
2. **El camino del precio se genera antes de la acción del usuario.** Completo, antes de que el usuario decida.
3. **Prohibición absoluta de manipulación dinámica.** La app nunca mueve el precio contra el usuario tras detectar su operación. Nunca espera a que compre para bajar el precio, ni a que venda para subirlo, ni fuerza una pérdida "para enseñar".
4. **Las decisiones del usuario afectan solo su lado de la simulación:** órdenes, posiciones, cuenta, riesgo, P/L, detección de errores, diario y puntuación. El camino de mercado generado permanece independiente de las acciones del usuario.
5. **El aprendizaje se evalúa por calidad de decisión, no por pérdidas forzadas.** El usuario puede ganar, perder, saltar la operación, gestionar bien o mal el riesgo, o quemar la cuenta; la retroalimentación final depende de su comportamiento.
6. **Determinismo:** la misma plantilla de escenario, los mismos parámetros, la misma semilla y la misma versión del generador producen exactamente el mismo camino de mercado.
7. **Rankings justos:** los desafíos usan semillas fijas; todos los intentos enfrentan el mismo camino.
8. **Sandbox reproducible:** las sesiones sandbox pueden usar semillas aleatorias, pero toda semilla guardada debe ser reproducible.
9. **Replay exacto:** el modo de repetición reproduce el mismo camino de mercado, idéntico.
10. **Modo histórico es futuro:** no es requisito del MVP.

---

## 12. Learning Context Contract (Contrato de Contexto de Aprendizaje)

El **Learning Context Contract** es un principio central de producto: cada escenario educativo es un contrato explícito y pregenerado entre la app y el usuario, que define qué se enseña y cómo se evalúa, **antes** de que el usuario actúe.

Todo Learning Context Contract debe definir:

| Campo | Descripción |
|---|---|
| Objetivo de la lección | Qué concepto o disciplina específica enseña el escenario |
| Tipo de escenario | Ej.: fake breakout, FOMO trap, pico de noticias, día de rango, día de tendencia, spread alto, trampa de señal, riesgo de liquidación |
| Secuencia de regímenes de mercado | Orden de fases del mercado (rango → ruptura → reversión, etc.) |
| Trampas esperadas | Qué errores típicos invita el contexto a cometer |
| Decisiones buenas válidas | Conjunto de decisiones consideradas de alta calidad (incluyendo no operar) |
| Caminos de error comunes | Patrones de decisión equivocada que el escenario anticipa |
| Criterios de evaluación | Cómo se puntúa la calidad de decisión (riesgo, timing, gestión, abstención) |
| Reglas de retroalimentación | Qué feedback recibe el usuario y cuándo |
| Caminos de éxito | Qué secuencias de decisión constituyen éxito pedagógico |
| Caminos de fracaso | Qué secuencias constituyen fracaso, y qué se enseña a partir de él |
| Comportamiento de la semilla | Tipo de semilla, fijeza y repetibilidad del escenario |
| Comportamiento de replay | Cómo se reproduce el escenario en revisión de errores y reintentos |

Regla clave: el contrato describe el **contexto**; nunca prescribe el **resultado** del usuario. El mismo contrato puede terminar en ganancia, pérdida, abstención correcta o cuenta quemada — y todos esos finales producen retroalimentación educativa válida.

Cada Learning Context Contract tiene un **ID** que se almacena junto con cada simulación generada (ver sección 13).

---

## 13. Sistema de semillas determinista (Deterministic Seed System)

### Definición

La misma plantilla de escenario + los mismos parámetros + la misma semilla + la misma versión del generador ⇒ **exactamente el mismo camino de mercado**, siempre.

### Lo que habilita

- Los tutoriales se repiten de forma idéntica.
- Los desafíos son justos: cada intento (y cada usuario) enfrenta el mismo camino.
- El sandbox es aleatorio por defecto, pero las semillas pueden guardarse y reproducirse.
- La revisión de errores reproduce el mercado exacto de la sesión original.
- El usuario puede reiniciar el mismo escenario y probar decisiones distintas.
- La app puede almacenar y exportar datos de semilla/sesión dentro del archivo de progreso.
- Los futuros escenarios inspirados en historia real usan el mismo pipeline de plantillas.

### Tipos de semilla

| Tipo | Comportamiento |
|---|---|
| Tutorial seed | Fija, repetible, específica de cada lección |
| Challenge seed | Fija, segura para rankings: mismo camino en todos los intentos |
| Sandbox seed | Aleatoria por defecto; opcionalmente guardada y reproducible |
| Replay seed | Reproduce una sesión previa de forma exacta |
| Historical-inspired seed | Futuro: basada en patrones extraídos de comportamiento real de mercado |
| Historical replay reference | Futuro: referencia para replay exacto de velas históricas |

### Datos que almacena cada simulación generada

- Semilla (seed) y tipo de semilla.
- Versión del generador.
- ID de plantilla de escenario.
- ID del Learning Context Contract.
- Tipo de mercado e instrumento.
- Dificultad y horizonte temporal.
- Perfil de volatilidad, perfil de spread y perfil de liquidez.
- Secuencia de regímenes y calendario de eventos.
- Hash del camino de velas generado (verificación de integridad y equidad).
- Registro de decisiones del usuario.
- Resultado de evaluación.
- Metadatos de replay.

---

## 14. Dirección de producto historical-ready

Burgundy debe estar **preparada para datos históricos sin depender de ellos**:

- El motor de simulación acepta caminos de mercado de distintas fuentes: generador sintético, generador procedural por semilla, escenarios fijos de tutorial, escenarios fijos de desafío, y en el futuro replay de velas históricas y plantillas de patrones inspirados en historia.
- La app usa un **formato universal de vela** para todas las fuentes.
- La fuente de datos de mercado está **separada** del motor de órdenes, del motor de riesgo, del motor de replay y del motor de evaluación.
- El modo histórico es una **fuente de datos futura**, no el fundamento del MVP. Los datos históricos reales **no son requisito del MVP**.

---

## 15. Restricciones innegociables

| # | Restricción |
|---|---|
| 1 | App solo en español, dirigida a LATAM |
| 2 | Sin login ni cuenta en la nube |
| 3 | Sin monetización |
| 4 | Sin cursos (definición canónica: §7.1 — micro-lecciones internas permitidas; plataforma de cursos prohibida) |
| 5 | Sin coach de IA |
| 6 | Offline-first |
| 7 | Progreso guardado localmente |
| 8 | Exportación e importación de archivo de progreso/sesión |
| 9 | Identidad: academia seria + HUD oscuro Burgundy + gamificación seria de largo plazo + educación LATAM con terminología real explicada |
| 10 | Paleta exacta: #1A1617, #571324, #2E2E2E, #C9A050, #4A6D56, #802F3E; prohibidos los verdes/rojos estridentes y el negro puro |
| 11 | Desafíos habilitados; rankings por sesión y puntuaciones máximas guardadas |
| 12 | Modo de progresión con desbloqueos y modo libre sin restricciones |
| 13 | Sin integración con brokers reales; sin trading con dinero real |
| 14 | Sin promesas de ganancia, sin hype, sin "hazte rico" |
| 15 | Reglas de equidad de simulación (sección 11) y sistema de semillas determinista (sección 13) |
| 16 | No es asesoría financiera; encuadre educativo sin pantallas legales pesadas |
| 17 | Plataformas: Android 15+ e iOS 20+ |
| 18 | Proyecto firmado bajo el usuario tsuloid; nombre de app "Burgundy" en todo el producto y documentación |

---

## 16. Límites del producto en el MVP

**Dentro del MVP:**

- Modo tutorial guiado con escenarios de semilla fija.
- Modo sandbox con semillas aleatorias y guardado/replay opcional de semillas.
- Modo desafío con semillas fijas, puntuación máxima guardada y rankings por sesión (locales).
- Modo de progresión con desbloqueos y modo libre.
- Modo opcional sin indicadores (price action puro).
- Motor de simulación con mercados **sintéticos/procedurales** (formato universal de vela; arquitectura historical-ready).
- Al menos un conjunto inicial de tipos de instrumento educativos (mercado sintético como base; la cobertura concreta de stocks/forex/cripto/índices/commodities se prioriza en el roadmap, no aquí).
- Motor de órdenes y riesgo: long/short, stop loss, take profit, position sizing, spread, slippage, fees, leverage, drawdown, liquidación, reset al llegar a cero.
- Learning Context Contracts para las lecciones y trampas centrales (FOMO, fake breakout, señal copiada, overtrading, revenge trading, sin stop, sobre-apalancamiento).
- Detección de errores, diario de decisiones, revisión de errores con replay exacto.
- Gamificación: XP, niveles, racha diaria, desbloqueos, desafíos de supervivencia/disciplina/sin indicadores.
- Persistencia local completa + exportación/importación de archivo de progreso (incluyendo semillas y sesiones).
- Todo el contenido en español LATAM.

**Comportamientos clave del MVP:**

- Reset/reinicio al quemar la cuenta, con revisión educativa.
- Retroalimentación clara tras cada decisión.
- Paneles de balance, equity, riesgo, P/L, spread, slippage, fees y drawdown visibles cuando son relevantes.

---

## 17. Funcionalidades excluidas intencionalmente del MVP

| Excluida | Motivo / Estado |
|---|---|
| Replay de datos históricos reales | Futuro; el motor es historical-ready pero el MVP no lo requiere |
| Escenarios inspirados en historia real (pattern templates) | Futuro; usará el mismo pipeline de plantillas y semillas |
| Futuros (futures) como instrumento | "Más adelante si es viable" |
| Integración con brokers reales | Excluida permanentemente del producto |
| Trading con dinero real | Excluido permanentemente |
| Monetización (pagos, suscripciones, anuncios) | Excluida en este alcance |
| Cursos | Excluidos; la enseñanza vive en el simulador como micro-lecciones internas (ver §7.1) |
| Coach de IA | Excluido |
| Login, cuentas en la nube, sincronización en servidor | Excluidos; la portabilidad se resuelve con exportación/importación local |
| Rankings online globales entre usuarios | Excluidos del MVP; los rankings son por sesión y locales |
| Mockups visuales y diseño de pantallas | Fuera de este documento; fase posterior |
| Arquitectura técnica completa | Fuera de este documento; se define en el siguiente documento de arquitectura |

---

## 18. Criterios de éxito del producto

El éxito de Burgundy **no** se mide por ganancias simuladas del usuario, sino por evidencia de transformación de comportamiento y por integridad del producto:

### Criterios de aprendizaje y comportamiento (medibles localmente, sin nube)

| Criterio | Señal de éxito |
|---|---|
| Adopción del stop loss | Porcentaje creciente de operaciones con stop loss definido antes de entrar |
| Dimensionamiento de posición | Riesgo por operación dentro de límites razonables de forma consistente |
| Reducción de errores emocionales | Caída en detecciones de overtrading, revenge trading y FOMO por sesión a lo largo del tiempo |
| Abstención de calidad | El usuario aprende a saltar escenarios trampa (skip evaluado como decisión correcta) |
| Supervivencia | Aumento de la duración de las cuentas simuladas antes de reset |
| Uso de la revisión de errores | El usuario revisa y reintenta escenarios fallidos con decisiones distintas |
| Progresión sostenida | Rachas diarias mantenidas, niveles y desbloqueos alcanzados a lo largo de semanas (gamificación de largo plazo) |
| Comprensión de terminología | Resultados de repasos de concepto (spread, slippage, drawdown, leverage, liquidez, risk/reward) |

### Criterios de integridad del producto

- **Equidad verificable:** todo camino de mercado es pregenerado, con hash almacenado; el replay reproduce caminos idénticos; misma semilla + parámetros + versión del generador = mismo camino, sin excepción.
- **Rankings justos:** todos los intentos de un desafío usan la misma semilla fija.
- **Privacidad cumplida:** cero datos enviados a servidores; progreso 100% local; exportación/importación funcional y fiel.
- **Offline real:** toda la experiencia central funciona sin conexión.
- **Tono cumplido:** ninguna superficie del producto contiene promesas de ganancia, hype, estética de casino ni lenguaje infantil; toda la terminología real está explicada en español claro.

### Criterio de percepción (cualitativo)

El usuario debe poder describir la app así:

> "Es un sistema de entrenamiento serio. Subo de nivel como en un juego, pero estoy aprendiendo disciplina real de mercado."

---

*Documento de definición de producto de Burgundy — firmado por tsuloid. Siguiente paso del proceso (no incluido aquí): definición de arquitectura técnica.*
