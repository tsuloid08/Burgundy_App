# Burgundy — Documento de Arquitectura 04: Cobertura de Mercados y Progresión

**Proyecto:** Burgundy — Simulador educativo de trading
**Autor / Firma:** tsuloid
**Idioma:** Español (LATAM)
**Estado:** Especificación de diseño — sin código, sin pantallas, sin desarrollo todavía
**Paleta de referencia:** `#1A1617` (fondo profundo), `#571324` (acento Burgundy mate), `#2E2E2E` (superficies/divisores), `#C9A050` (oro apagado, énfasis crítico), `#4A6D56` (velas alcistas), `#802F3E` (velas bajistas)

---

## 1. Propósito de este documento

Este documento define **qué mercados cubre el simulador Burgundy, en qué orden se introducen, cómo se simulan sin datos reales de broker, y cómo se comportan las semillas (seeds) y los Learning Context Contracts (LCC)** para cada tipo de mercado.

Principio rector (filosofía de simulación de Burgundy):

> El simulador fuerza un **contexto de aprendizaje**, nunca un resultado de trading. El camino del mercado se genera **antes** de cualquier acción del usuario y **no se manipula después de su entrada**. Las decisiones del usuario afectan órdenes, posiciones, cuenta, riesgo, P/L, journal, errores, feedback y score — nunca el camino de precio generado.

Cada mercado en Burgundy no existe para "operar más activos", sino porque **enseña una lección distinta** que los otros mercados no enseñan igual de bien.

---

## 2. Resumen de cobertura

| Mercado | ¿En MVP? | Rol educativo principal | Riesgo para principiantes |
|---|---|---|---|
| Mercados sintéticos educativos | ✅ Sí (es la base) | Enseñar mecánica pura sin sesgo de activo | Bajo (entorno controlado) |
| Acciones (Stocks) | ✅ Sí (`synthetic_stock`) | Gaps, tendencias, soportes/resistencias, shocks tipo earnings | Medio |
| Forex | ✅ Sí (`synthetic_fx`) | Sesiones, spread variable, valor del pip; el apalancamiento se enseña solo conceptualmente en MVP | Alto |
| Cripto | ✅ Sí (`synthetic_crypto`, desbloqueado tarde) | Volatilidad extrema, FOMO, liquidaciones, fin de semana | Muy alto |
| Índices | ⚠️ Post-MVP temprano | Movimiento macro, seguimiento de tendencia, riesgo ajustado | Medio |
| Materias primas (Commodities) | ⚠️ Post-MVP temprano | Shocks de eventos, narrativas de oferta/demanda | Medio-alto |
| Futuros | ❌ Futuro (si es viable) | Margen avanzado, tamaño de contrato, tick value, liquidación | Muy alto |

> **Alcance MVP cerrado por `MVP_MARKET_LOCK` (documento 12, sección 4):** base `synthetic_training` + 3 instrumentos jugables (`synthetic_fx`, `synthetic_stock`, `synthetic_crypto` con desbloqueo tardío). Índices y commodities son post-MVP temprano. El leverage no es mecánica jugable del MVP (`LEVERAGE_MVP_LIMITS`, documento 12, sección 5). Ante contradicción entre este documento y esos locks, ganan los locks.

**Orden de introducción en modo progresión (ruta completa del producto):**
Sintético → Acciones → Índices → Forex → Cripto → Commodities → Futuros.
En el MVP, la ruta activa es el subconjunto: Sintético → Acciones → Forex → Cripto; Índices se inserta en su posición cuando llegue post-MVP.

La justificación detallada está en la sección 5.

---

## 3. Especificación por mercado

Cada mercado se describe con dos bloques: **(A) ficha educativa** y **(B) comportamiento de seeds y Learning Context Contracts**.

---

### 3.1 Mercados sintéticos educativos

#### A. Ficha educativa

| Campo | Definición |
|---|---|
| **Qué es** | Mercados generados algorítmicamente por Burgundy. No representan ningún activo real: son "campos de entrenamiento" de precio puro, construidos para enseñar un concepto a la vez. |
| **Por qué importa** | Permite aislar una lección (ej. "qué es un pullback") sin el ruido, el sesgo emocional ni las expectativas que trae un activo conocido como Bitcoin o Apple. Es el equivalente al simulador de vuelo antes de tocar un avión real. |
| **Qué lo hace diferente** | Burgundy controla el 100% del régimen, la volatilidad, las trampas y la dificultad. Es el único mercado donde el contenido del escenario está diseñado pedagógicamente vela por vela. |
| **Volatilidad típica** | Configurable: desde muy baja (lecciones de lectura de velas) hasta extrema (lecciones de drawdown). |
| **Comportamiento por sesión** | No tiene sesiones reales; puede simular "horarios" artificiales cuando la lección lo requiere (preparación para Forex). |
| **Liquidez** | Configurable. Por defecto alta (ejecución limpia) para no contaminar lecciones básicas. |
| **Spread** | Fijo y pequeño por defecto; se amplía solo cuando la lección trata sobre spread. |
| **Slippage** | Desactivado por defecto; se activa explícitamente en la lección de slippage ("recibiste un precio peor del que esperabas — esto es slippage"). |
| **Riesgo para principiantes** | Bajo: es un entorno controlado. Riesgo educativo único: que el usuario crea que el mercado real es así de limpio. Burgundy debe decirlo explícitamente. |
| **Malentendido común** | "Si gano aquí, ya sé operar." Burgundy debe reforzar: el sintético enseña mecánica y disciplina, no garantiza resultados en mercados reales. |
| **Mejor ejercicio en el simulador** | Tutoriales guiados con seed fija: identificar tendencia, colocar stop loss, calcular riesgo por operación, sobrevivir a un drawdown planificado. |
| **¿Pertenece al MVP?** | **Sí — es la base del MVP.** Todo el modo tutorial vive aquí. |
| **Requisito de desbloqueo** | Ninguno. Disponible desde el primer minuto. |
| **Disponible en modo libre** | Sí, siempre. |

#### B. Seeds y Learning Context Contracts

| Aspecto | Comportamiento |
|---|---|
| **Plantillas de escenario** | "Tendencia limpia", "Rango lateral", "Pullback y continuación", "Ruptura falsa", "Drawdown inevitable", "Mercado aburrido (paciencia)", "Volatilidad creciente". |
| **Perfiles de volatilidad realistas** | Todos, por diseño: baja, media, alta, escalonada, con picos. Es el único mercado donde la volatilidad es 100% didáctica. |
| **Modelos de spread/slippage** | Spread fijo configurable; slippage como módulo activable por lección. |
| **Condiciones de liquidez** | Alta por defecto; "liquidez fina" solo como lección explícita. |
| **Regímenes de mercado comunes** | Cualquiera, pero los LCC del tutorial deben cubrir en orden: tendencia → rango → ruptura → reversión → caos. |
| **Trampas comunes** | Rupturas falsas, "casi llega a tu target y se devuelve" (lección de toma de ganancias), velas de barrido de stops. Todas pre-generadas, nunca reactivas al usuario. |
| **Tipos de seed útiles** | Seeds fijas de tutorial (la mayoría), seeds fijas de desafío, seeds aleatorias de sandbox con plantilla restringida, seeds de replay. |
| **Seeds fijas de tutorial** | ✅ Sí — es su razón de existir. |
| **Seeds fijas de desafío** | ✅ Sí (desafíos de disciplina y supervivencia básica). |
| **Seeds aleatorias de sandbox** | ✅ Sí, con generador restringido por plantilla. |
| **Plantillas futuras inspiradas en histórico** | ✅ Posible: comportamientos estadísticos extraídos de mercados reales pueden parametrizar el generador sintético sin necesitar datos históricos crudos. |

---

### 3.2 Acciones (Stocks)

#### A. Ficha educativa

| Campo | Definición |
|---|---|
| **Qué es** | Participaciones de empresas que cotizan en bolsa. En Burgundy son acciones **ficticias pero verosímiles** (ej. una empresa tecnológica simulada), nunca tickers reales en el MVP. |
| **Por qué importa** | Es el mercado más intuitivo para un principiante LATAM: "compro un pedazo de una empresa" se entiende sin jerga. Es ideal para enseñar tendencia, pullback, soporte y resistencia. |
| **Qué lo hace diferente** | Tiene **horario de mercado**: abre y cierra. Eso produce **gaps** (el precio abre en un nivel distinto al cierre anterior), un fenómeno que Forex y cripto casi no muestran igual. |
| **Volatilidad típica** | Media. Días normales tranquilos, con shocks puntuales tipo "resultados de la empresa" (earnings) que pueden mover 5–15% en una vela. |
| **Comportamiento por sesión** | Apertura volátil, mediodía lento, cierre activo. El gap de apertura es la lección estrella. |
| **Liquidez** | Alta en "acciones grandes" simuladas; baja en "acciones pequeñas" simuladas (lección de liquidez). |
| **Spread** | Estrecho en acciones líquidas; amplio en acciones de baja liquidez. |
| **Slippage** | Bajo en condiciones normales; alto en gaps y shocks tipo earnings (lección: el stop loss puede ejecutarse peor de lo esperado en un gap). |
| **Riesgo para principiantes** | Medio. Sin apalancamiento extremo por defecto, el daño por error es contenido. |
| **Malentendido común** | "El stop loss me protege siempre." El gap nocturno demuestra que un stop puede ejecutarse mucho más abajo del nivel definido. También: "si la empresa es buena, el precio solo sube". |
| **Mejor ejercicio en el simulador** | Escenario de tendencia con pullbacks para practicar entradas; escenario de gap bajista con posición abierta para vivir el riesgo nocturno (overnight risk) en carne propia, sin dinero real. |
| **¿Pertenece al MVP?** | **Sí.** Es el primer mercado "con nombre" tras el sintético. |
| **Requisito de desbloqueo** | Completar la ruta básica de tutoriales sintéticos (gestión de riesgo + lectura de velas + stop loss). |
| **Disponible en modo libre** | Sí, una vez desbloqueado en progresión. En modo libre puro (sin progresión) está disponible con una advertencia educativa inicial. |

#### B. Seeds y Learning Context Contracts

| Aspecto | Comportamiento |
|---|---|
| **Plantillas de escenario** | "Tendencia alcista con pullbacks", "Rango con soporte/resistencia claros", "Gap de apertura (alcista y bajista)", "Shock tipo earnings", "Distribución y caída", "Acción ilíquida". |
| **Perfiles de volatilidad realistas** | Media estable con picos puntuales; volatilidad intradía en forma de U (alta al abrir y cerrar). |
| **Modelos de spread/slippage** | Spread estrecho casi fijo en activos líquidos; modelo de slippage por gap: el fill del stop se ejecuta al precio de apertura, no al nivel del stop. |
| **Condiciones de liquidez** | Dos perfiles: "blue chip simulada" (alta) y "small cap simulada" (baja, spreads amplios, fills parciales conceptuales). |
| **Regímenes comunes** | Tendencia sostenida, rango prolongado, gap-and-go, gap-and-fade. |
| **Trampas comunes** | Falsa ruptura de resistencia, gap en contra con posición abierta, "comprar porque ya cayó mucho" (cuchillo cayendo), rebote muerto tras shock. |
| **Tipos de seed útiles** | Fijas de tutorial (gaps, soporte/resistencia), fijas de desafío (sobrevivir a un earnings shock con riesgo controlado), aleatorias de sandbox, replay. |
| **Seeds fijas de tutorial** | ✅ Sí. |
| **Seeds fijas de desafío** | ✅ Sí. |
| **Seeds aleatorias de sandbox** | ✅ Sí. |
| **Plantillas futuras inspiradas en histórico** | ✅ Sí — caídas históricas célebres y temporadas de earnings reales son material ideal para plantillas "inspiradas en" (sin requerir datos crudos en MVP). |

---

### 3.3 Forex

#### A. Ficha educativa

| Campo | Definición |
|---|---|
| **Qué es** | El mercado de divisas: se compra una moneda vendiendo otra (ej. EUR/USD). En Burgundy se simulan pares verosímiles con nomenclatura genérica o pares estándar simulados. |
| **Por qué importa** | Es la puerta de entrada más común del principiante LATAM (brokers con apalancamiento alto, señales de Telegram, cuentas pequeñas). Burgundy debe enseñar aquí **por qué la mayoría pierde**: apalancamiento mal dimensionado, spread variable y operar a cualquier hora. |
| **Qué lo hace diferente** | Mercado de 24 horas en días hábiles con **sesiones** (Asia, Londres, Nueva York) que cambian el comportamiento del precio. Introduce el **pip** como unidad y el apalancamiento como norma, no como excepción. |
| **Volatilidad típica** | Baja-media en pares mayores; picos fuertes en eventos de noticias (tipo decisión de tasas o dato de empleo, simulados). |
| **Comportamiento por sesión** | Asia: rangos estrechos. Londres: expansión y rupturas. Nueva York: continuación o reversión, con solapamiento Londres-NY como ventana más activa. Esta estructura es una lección central de Burgundy. |
| **Liquidez** | Muy alta en sesiones activas; notablemente menor en el cierre de NY / apertura de Asia y cerca de noticias. |
| **Spread** | Variable: estrecho en horas líquidas, **se amplía** en noticias y horas muertas. La ampliación del spread es una lección obligatoria. |
| **Slippage** | Bajo en condiciones normales; severo durante noticias simuladas (la lección "tu stop se ejecutó 15 pips peor" se vive aquí). |
| **Riesgo para principiantes** | Alto, casi siempre por **apalancamiento mal dimensionado**, no por el mercado en sí. |
| **Malentendido común** | "Con apalancamiento 1:500 y $50 puedo vivir del trading." También: confundir pips con dinero sin entender el valor del pip según el tamaño de la posición, y copiar señales sin contexto ni stop. |
| **Mejor ejercicio en el simulador** | Desafío de "misma señal, tres tamaños de posición": la misma entrada gana con riesgo del 1% y revienta la cuenta con riesgo del 20%. Lección directa contra la cultura de señales de Telegram/TikTok. |
| **¿Pertenece al MVP?** | **Sí.** Es demasiado relevante para LATAM como para dejarlo fuera, pero se desbloquea después de acciones (índices es post-MVP). |
| **Requisito de desbloqueo** | Completar el módulo de gestión de riesgo avanzada (tamaño de posición, riesgo por operación, drawdown) y los tutoriales de acciones. |
| **Disponible en modo libre** | Sí tras desbloqueo; en modo libre puro, disponible con advertencia educativa. En MVP sin apalancamiento jugable (`LEVERAGE_MVP_LIMITS`); el leverage limitado por defecto llega en Fase 5. |

#### B. Seeds y Learning Context Contracts

| Aspecto | Comportamiento |
|---|---|
| **Plantillas de escenario** | "Día de tres sesiones", "Ruptura de Londres", "Rango asiático", "Noticia de alto impacto", "Spread widening nocturno", "Tendencia macro de varios días", "Señal sin contexto" (el LCC reproduce una entrada típica de Telegram y enseña por qué falla sin gestión). |
| **Perfiles de volatilidad realistas** | Baja en Asia, media-alta en Londres/NY, pico extremo de 1–3 velas en noticias. |
| **Modelos de spread/slippage** | Spread dinámico por "hora de sesión" simulada; multiplicador de spread y slippage en ventanas de noticia pre-generadas en la seed. |
| **Condiciones de liquidez** | Alta por defecto; caídas de liquidez programadas en rollover y noticias. |
| **Regímenes comunes** | Rango nocturno → expansión matinal; tendencia con retrocesos ordenados; reversión post-noticia. |
| **Trampas comunes** | Falsa ruptura del rango asiático, barrido de stops antes del movimiento real, spike de noticia que toca stop y se devuelve, spread ampliado que convierte un trade marginal en pérdida. |
| **Tipos de seed útiles** | Fijas de tutorial (sesiones, pips, spread), fijas de desafío (sobrevivir a una semana con noticia incluida), aleatorias de sandbox con calendario de noticias sintético, replay. |
| **Seeds fijas de tutorial** | ✅ Sí. |
| **Seeds fijas de desafío** | ✅ Sí. |
| **Seeds aleatorias de sandbox** | ✅ Sí (el calendario de eventos se genera junto con la seed, antes de cualquier acción del usuario). |
| **Plantillas futuras inspiradas en histórico** | ✅ Sí — eventos macro reales (decisiones de tasas, crisis cambiarias relevantes para LATAM) son excelente material para plantillas inspiradas en histórico. |

---

### 3.4 Cripto

#### A. Ficha educativa

| Campo | Definición |
|---|---|
| **Qué es** | Activos digitales que cotizan 24/7. En Burgundy se simulan criptos genéricas verosímiles (una "mayor" estable tipo Bitcoin simulado y una "altcoin" volátil), sin precios reales. |
| **Por qué importa** | Es donde el principiante LATAM más se quema: FOMO, apalancamiento extremo, liquidaciones y operar de madrugada un domingo. Burgundy lo usa como **laboratorio de errores emocionales**, no como mercado aspiracional. |
| **Qué lo hace diferente** | No cierra nunca (24/7), no tiene gaps de apertura pero sí movimientos violentos de fin de semana con poca liquidez, y la cultura alrededor (redes sociales, hype) empuja a decisiones emocionales. |
| **Volatilidad típica** | Alta a extrema. Movimientos de 10–20% en un día son normales en la altcoin simulada. |
| **Comportamiento por sesión** | Sin sesiones formales, pero con patrones: más volumen en horario de EE. UU., movimientos bruscos de fin de semana y madrugada con libro delgado. |
| **Liquidez** | Alta en la cripto mayor; delgada y traicionera en la altcoin, especialmente en fines de semana simulados. |
| **Spread** | Moderado en la mayor; amplio y volátil en la altcoin. |
| **Slippage** | El más alto del simulador en eventos de cascada: las velas de liquidación masiva ejecutan stops muy lejos del nivel esperado. |
| **Riesgo para principiantes** | **Muy alto.** Combinación de volatilidad extrema, apalancamiento disponible y carga emocional. |
| **Malentendido común** | "Si sube fuerte, hay que entrar ya" (FOMO); "el apalancamiento 20x multiplica mis ganancias" (omitiendo que una caída del 5% liquida la posición); confundir un mercado alcista con habilidad propia. |
| **Mejor ejercicio en el simulador** | Desafío "FOMO trap": vela verde gigante pre-generada seguida de reversión; el score premia **no entrar** o entrar con riesgo controlado. Desafío de liquidación: demostrar matemáticamente en vivo cómo 20x convierte un retroceso normal en cuenta quemada. |
| **¿Pertenece al MVP?** | **Sí**, pero como mercado de desbloqueo tardío: llegar a cripto debe sentirse como un logro de disciplina, no como el punto de partida. |
| **Requisito de desbloqueo** | Completar Forex (sesiones + spread variable; el apalancamiento solo se enseña conceptualmente en MVP — `LEVERAGE_MVP_LIMITS`) y las lecciones de disciplina del MVP (FOMO, overtrading, revenge trading en M4/M5). |
| **Disponible en modo libre** | Sí tras desbloqueo. En modo libre puro, disponible con advertencia destacada (énfasis en `#C9A050`) sobre su rol como mercado de mayor riesgo del simulador. |

#### B. Seeds y Learning Context Contracts

| Aspecto | Comportamiento |
|---|---|
| **Plantillas de escenario** | "Pump y reversión (FOMO trap)", "Cascada de liquidaciones", "Fin de semana ilíquido", "Tendencia parabólica y colapso", "Rango aburrido post-colapso (paciencia)", "Altcoin con spread amplio". |
| **Perfiles de volatilidad realistas** | Alta base con clusters: períodos calmos interrumpidos por explosiones de volatilidad; colas gordas (movimientos extremos más frecuentes que en otros mercados). |
| **Modelos de spread/slippage** | Spread proporcional a la volatilidad instantánea; slippage agresivo en velas de cascada; simulación conceptual de liquidación forzada cuando el margen se agota. |
| **Condiciones de liquidez** | Mayor: profunda entre semana, delgada en fin de semana. Altcoin: delgada siempre. |
| **Regímenes comunes** | Parabólico, colapso, rango deprimido, recuperación escalonada. |
| **Trampas comunes** | Vela de FOMO antes de reversión, falso fondo ("ya no puede caer más"), wick de barrido en madrugada, hype simulado en el contexto narrativo del escenario. |
| **Tipos de seed útiles** | Fijas de desafío (su uso principal), fijas de tutorial (liquidación, FOMO), aleatorias de sandbox con régimen de alta volatilidad, replay (revisar el propio overtrading es especialmente valioso aquí). |
| **Seeds fijas de tutorial** | ✅ Sí (lecciones de liquidación y FOMO). |
| **Seeds fijas de desafío** | ✅ Sí — cripto es el mercado de desafíos emocionales por excelencia. |
| **Seeds aleatorias de sandbox** | ✅ Sí. |
| **Plantillas futuras inspiradas en histórico** | ✅ Sí — los grandes ciclos y colapsos cripto reales son material narrativo y estadístico ideal para plantillas inspiradas en histórico. |

---

### 3.5 Índices

#### A. Ficha educativa

| Campo | Definición |
|---|---|
| **Qué es** | Una canasta que representa muchas acciones a la vez (como un "promedio del mercado"). Burgundy simula índices genéricos verosímiles (un índice "global" y uno "tecnológico"). |
| **Por qué importa** | Enseña a pensar en **macro**: el índice se mueve por el conjunto, no por una empresa. Es el mejor terreno para seguimiento de tendencia y comportamiento ajustado al riesgo, porque tiende a moverse de forma más ordenada que un activo individual. |
| **Qué lo hace diferente** | Menos ruido idiosincrático que una acción individual; tendencias más persistentes; reacciona a datos macro (tasas, empleo, inflación simulados) más que a noticias de una sola empresa. |
| **Volatilidad típica** | Media, con días de pánico puntuales donde la correlación lo arrastra todo. |
| **Comportamiento por sesión** | Horario de mercado con gaps moderados; aperturas activas tras datos macro; tendencias intradía más legibles que en acciones individuales. |
| **Liquidez** | Muy alta de forma consistente. |
| **Spread** | Estrecho y estable — el mejor del simulador después del sintético. |
| **Slippage** | Bajo, salvo en shocks macro simulados. |
| **Riesgo para principiantes** | Medio. La trampa es la complacencia: "el índice siempre sube" funciona hasta el día de pánico. |
| **Malentendido común** | "Operar el índice es seguro porque está diversificado." La diversificación reduce el riesgo de una empresa, no el riesgo de mercado ni el de apalancarse mal. |
| **Mejor ejercicio en el simulador** | Escenario de tendencia macro de varias semanas comprimidas para practicar mantener una posición ganadora (dejar correr) con trailing del riesgo; desafío de "día de pánico" para practicar reducción de exposición. |
| **¿Pertenece al MVP?** | ⚠️ **No — post-MVP temprano** (movido fuera del MVP por `MVP_MARKET_LOCK`, documento 12). Sigue siendo el complemento natural de acciones con bajo costo de implementación (mismo motor, parámetros distintos); es el primer candidato a entrar tras el MVP junto con commodities. |
| **Requisito de desbloqueo** | Completar los tutoriales de acciones (soporte/resistencia y gaps). |
| **Disponible en modo libre** | Sí tras desbloqueo; disponible en modo libre puro con introducción breve. |

#### B. Seeds y Learning Context Contracts

| Aspecto | Comportamiento |
|---|---|
| **Plantillas de escenario** | "Tendencia macro persistente", "Corrección ordenada del 5–10%", "Día de pánico", "Rango pre-dato macro y resolución", "Rotación lenta de régimen alcista a bajista". |
| **Perfiles de volatilidad realistas** | Media estable; régimen de volatilidad creciente antes de shocks; clusters de pánico cortos. |
| **Modelos de spread/slippage** | Spread estrecho casi constante; slippage solo relevante en velas de pánico. |
| **Condiciones de liquidez** | Alta constante; reducción solo en el shock macro. |
| **Regímenes comunes** | Tendencia alcista lenta ("sube por la escalera"), caída rápida ("baja por el ascensor") — esta asimetría es en sí misma una lección. |
| **Trampas comunes** | Comprar la caída demasiado pronto en un día de pánico, sobreapalancarse por la aparente "seguridad" del índice, aburrirse de la tendencia lenta y sobreoperar. |
| **Tipos de seed útiles** | Fijas de tutorial (tendencia, riesgo ajustado), fijas de desafío (mantener una ganadora; sobrevivir al pánico), aleatorias de sandbox, replay. |
| **Seeds fijas de tutorial** | ✅ Sí. |
| **Seeds fijas de desafío** | ✅ Sí. |
| **Seeds aleatorias de sandbox** | ✅ Sí. |
| **Plantillas futuras inspiradas en histórico** | ✅ Sí — los grandes ciclos y crisis de índices reales son el material histórico más documentado que existe; ideal para plantillas futuras. |

---

### 3.6 Materias primas (Commodities)

#### A. Ficha educativa

| Campo | Definición |
|---|---|
| **Qué es** | Mercados de bienes físicos: energía (tipo petróleo), metales (tipo oro) y agrícolas. Burgundy simula commodities genéricas verosímiles ("energía", "metal precioso", "grano"). |
| **Por qué importa** | Enseña que el precio responde a **narrativas de oferta y demanda** y a shocks de eventos (clima, geopolítica, inventarios — todos simulados). Introduce un tipo de volatilidad distinta: el shock externo violento que no avisa en el gráfico. |
| **Qué lo hace diferente** | El driver no es una empresa ni una tasa de interés, sino el mundo físico. Los shocks son más bruscos y direccionales; el oro simulado además enseña el concepto de "activo refugio". |
| **Volatilidad típica** | Media-alta con picos extremos por evento. La energía simulada es el activo de shocks por excelencia. |
| **Comportamiento por sesión** | Horarios de mayor actividad ligados a datos de inventarios y sesiones específicas; movimientos nocturnos relevantes. |
| **Liquidez** | Buena en los benchmarks simulados; sensible a eventos. |
| **Spread** | Moderado; se amplía con fuerza durante shocks. |
| **Slippage** | Alto durante eventos; moderado el resto del tiempo. |
| **Riesgo para principiantes** | Medio-alto: el shock externo castiga a quien opera sin stop "porque el gráfico se veía bien". |
| **Malentendido común** | "El gráfico contiene toda la información." En commodities, un evento externo puede invalidar cualquier análisis técnico en una vela. También: confundir el contado con contratos con vencimiento. |
| **Mejor ejercicio en el simulador** | Escenario de shock de oferta pre-generado: el usuario con posición abierta vive un movimiento violento en contra y aprende por qué el stop loss y el tamaño de posición existen para lo impredecible. |
| **¿Pertenece al MVP?** | ⚠️ **No en el núcleo del MVP.** Es el primer candidato post-MVP: reutiliza el motor existente y solo añade plantillas de shock. Si el presupuesto de contenido lo permite, puede entrar al final del MVP como mercado único ("metal precioso"). |
| **Requisito de desbloqueo** | Completar índices y Forex (el usuario ya entiende macro y sesiones). |
| **Disponible en modo libre** | Sí, cuando exista, tras desbloqueo. |

#### B. Seeds y Learning Context Contracts

| Aspecto | Comportamiento |
|---|---|
| **Plantillas de escenario** | "Shock de oferta", "Narrativa de demanda creciente", "Dato de inventarios", "Refugio en pánico (metal sube cuando índices caen)", "Tendencia estacional simulada". |
| **Perfiles de volatilidad realistas** | Media con colas extremas por evento; volatilidad direccional (los shocks suelen tener dirección clara, no solo ruido). |
| **Modelos de spread/slippage** | Spread moderado con multiplicador de evento; slippage severo en la vela del shock. |
| **Condiciones de liquidez** | Buena base con vacíos puntuales de liquidez durante eventos. |
| **Regímenes comunes** | Tendencia por narrativa, rango de equilibrio, shock y nueva valoración del precio. |
| **Trampas comunes** | Operar contra un shock ("ya subió demasiado"), promediar pérdidas contra una narrativa fuerte, ignorar el evento programado en el calendario del escenario. |
| **Tipos de seed útiles** | Fijas de desafío (shocks), fijas de tutorial (oferta/demanda), aleatorias de sandbox con calendario de eventos sintético, replay. |
| **Seeds fijas de tutorial** | ✅ Sí. |
| **Seeds fijas de desafío** | ✅ Sí. |
| **Seeds aleatorias de sandbox** | ✅ Sí. |
| **Plantillas futuras inspiradas en histórico** | ✅ Sí — shocks históricos de energía y metales son casos de estudio perfectos para plantillas inspiradas en histórico. |

---

### 3.7 Futuros (fase futura, si es viable)

#### A. Ficha educativa

| Campo | Definición |
|---|---|
| **Qué es** | Contratos estandarizados para comprar o vender un activo a futuro, con margen, tamaño de contrato fijo y valor por tick. Es el instrumento "profesional" del simulador. |
| **Por qué importa** | Enseña la mecánica real del riesgo profesional: margen inicial y de mantenimiento, valor del tick, tamaño del contrato y liquidación por margen insuficiente. Cierra el círculo de la educación de riesgo de Burgundy. |
| **Qué lo hace diferente** | El riesgo no se elige en monto libre: el contrato tiene un tamaño fijo y cada tick vale una cantidad concreta. Una cuenta pequeña puede no soportar ni un contrato — esa restricción es la lección. |
| **Volatilidad típica** | La del subyacente (índice, commodity), pero el apalancamiento estructural amplifica el efecto en la cuenta. |
| **Comportamiento por sesión** | Sesiones extendidas casi 24h con ventanas de alta actividad; rollover de contratos como concepto educativo. |
| **Liquidez** | Muy alta en el contrato frontal simulado; menor fuera de horario. |
| **Spread** | Estrecho en ticks, pero cada tick vale dinero real significativo — el spread "barato" engaña. |
| **Slippage** | En ticks parece pequeño; en dinero, por contrato, es la lección. |
| **Riesgo para principiantes** | **Muy alto.** Por eso es el último mercado en desbloquearse. |
| **Malentendido común** | "Es como Forex pero con otro nombre." No entender que el tamaño del contrato fija el riesgo mínimo posible, y que no hay posiciones 'pequeñas' a voluntad. |
| **Mejor ejercicio en el simulador** | Calculadora viva de margen y tick value dentro de un escenario: el usuario debe decidir si su cuenta simulada soporta siquiera un contrato antes de poder operar. Desafío de llamada de margen pre-generada. |
| **¿Pertenece al MVP?** | ❌ **No.** Fase futura, solo si el motor de cuenta soporta margen avanzado sin comprometer la simplicidad del resto. |
| **Requisito de desbloqueo** | Completar todos los demás mercados y el módulo avanzado de gestión de riesgo. Es el "rango máximo" de la progresión. |
| **Disponible en modo libre** | Solo tras desbloqueo en progresión, incluso para usuarios de modo libre (única excepción de acceso, justificada por riesgo educativo). |

#### B. Seeds y Learning Context Contracts

| Aspecto | Comportamiento |
|---|---|
| **Plantillas de escenario** | "Primer contrato (dimensionamiento)", "Llamada de margen", "Rollover de contrato", "Tendencia con tick value real", "Día de límite de movimiento". |
| **Perfiles de volatilidad realistas** | Heredados del subyacente simulado, con énfasis en el impacto monetario por contrato. |
| **Modelos de spread/slippage** | Spread en ticks (1–2 ticks típico); slippage en ticks traducido siempre a dinero en el feedback. |
| **Condiciones de liquidez** | Alta en horario principal; delgada fuera de horario (lección de operar de madrugada). |
| **Regímenes comunes** | Los del subyacente; el régimen importa menos que la mecánica de margen en este mercado. |
| **Trampas comunes** | Operar un contrato que la cuenta no soporta, ignorar el margen de mantenimiento, mantener posición hacia el rollover sin saberlo. |
| **Tipos de seed útiles** | Fijas de tutorial (margen, tick, contrato), fijas de desafío (margen call), aleatorias de sandbox solo para usuarios que completaron los tutoriales. |
| **Seeds fijas de tutorial** | ✅ Sí — imprescindibles antes de cualquier sandbox. |
| **Seeds fijas de desafío** | ✅ Sí. |
| **Seeds aleatorias de sandbox** | ✅ Sí, con candado tras tutoriales. |
| **Plantillas futuras inspiradas en histórico** | ✅ Sí, heredando las plantillas históricas del subyacente correspondiente. |

---

## 4. Cómo se simula cada mercado sin datos reales de broker

Burgundy nunca depende de feeds de brokers ni de datos históricos crudos en el MVP. La estrategia es un **motor generativo único parametrizado por mercado**:

1. **Generador base común.** Un motor de caminos de precio (camino completo generado antes de la sesión, derivado determinísticamente de la seed) produce velas OHLC. La seed determina el 100% del camino: misma seed, mismo mercado, mismas velas, siempre.
2. **Perfil de mercado.** Cada mercado es un conjunto de parámetros sobre el motor base: distribución de volatilidad, estructura de sesiones/horarios, modelo de spread, modelo de slippage, perfil de liquidez, probabilidad de gaps, calendario de eventos sintéticos y catálogo de trampas permitidas.
3. **Capa de eventos pre-generados.** Noticias, earnings, shocks de oferta y cascadas de liquidación se insertan en el camino **durante la generación**, nunca como reacción al usuario. El usuario puede tener la mala (o buena) suerte de estar posicionado cuando ocurren — exactamente como en la realidad.
4. **Learning Context Contract (LCC).** Cada escenario declara qué debe contener el camino (régimen, trampas, eventos, dificultad) para garantizar la lección. El generador produce caminos hasta satisfacer el contrato; el resultado queda fijado por la seed.
5. **Nombres ficticios verosímiles.** Activos simulados con nombres propios de Burgundy (nunca tickers reales en MVP), lo que elimina expectativas de precio "correcto" y cualquier implicación de consejo financiero.

Esto garantiza: offline-first total, reproducibilidad (replays, rankings comparables sobre la misma seed), cero dependencia legal de datos de mercado, y el principio inviolable de que **el precio nunca responde a las acciones del usuario**.

---

## 5. Orden de introducción de mercados y justificación

### Orden en modo progresión

| Orden | Mercado | Por qué en esta posición |
|---|---|---|
| 1 | **Sintético** | Primero y obligatorio. Enseña la mecánica (velas, órdenes, stop, riesgo, P/L) sin la carga emocional ni las ideas preconcebidas de un activo conocido. Es imposible "tener una opinión sobre el activo", así que toda la atención va al proceso. |
| 2 | **Acciones** | El concepto más intuitivo para LATAM ("comprar parte de una empresa"). Introduce horarios, gaps y soporte/resistencia con volatilidad contenida. |
| 3 | **Índices** *(post-MVP)* | Extiende acciones hacia el pensamiento macro con bajo riesgo incremental. Enseña tendencia persistente y la asimetría subida-lenta / caída-rápida. Entra a la progresión cuando llegue post-MVP (`MVP_MARKET_LOCK`). |
| 4 | **Forex** | Introduce sesiones de 24h, spread variable, pips y — sobre todo — apalancamiento. Llega cuando el usuario ya domina riesgo por operación, justo antes del territorio donde más se quema el principiante LATAM. |
| 5 | **Cripto** | Volatilidad extrema, FOMO y liquidaciones. Se entrega tarde a propósito: llegar a cripto en Burgundy es un logro de disciplina, lo contrario del camino habitual del principiante real (que empieza aquí y se quema). |
| 6 | **Commodities** | Shocks externos y narrativas de oferta/demanda. Requiere madurez previa en macro (índices) y eventos (Forex). |
| 7 | **Futuros** | Mecánica profesional de margen y contratos. Cierre de la progresión; fase futura del producto. |

### Mercados que se retrasan deliberadamente

- **Cripto** se retrasa aunque sea el mercado que más atrae al público objetivo. Es una decisión pedagógica central: invertir el orden típico del principiante real (cripto primero, disciplina nunca).
- **Índices** se retrasa a post-MVP temprano por priorización de alcance (`MVP_MARKET_LOCK`), no por riesgo ni por costo de motor: reutiliza el motor con parámetros distintos y entra junto a commodities en la primera ola post-MVP.
- **Commodities** se retrasa por priorización de contenido, no por riesgo: el motor lo soporta, pero las plantillas de shock requieren diseño cuidadoso.
- **Futuros** se retrasa por complejidad del motor de cuenta (margen de mantenimiento, tick value, rollover) y por riesgo educativo.

### Disponibilidad en modo libre

El modo libre respeta los desbloqueos de la progresión (un mercado desbloqueado en progresión queda libre en sandbox). Para el usuario que quiere solo modo libre sin progresión, todos los mercados del MVP están disponibles **excepto Futuros**, cada uno precedido por una introducción educativa breve y no saltable la primera vez. Esta es la única fricción intencional del modo libre y existe para proteger al usuario vulnerable sin convertir la app en una jaula.

---

## 6. Cómo explicar las diferencias entre mercados a principiantes LATAM (en la app)

Regla de Burgundy: **término real en inglés cuando es estándar + explicación en español simple + ejemplo con cuenta pequeña**. Tono serio, directo, sin promesas. Ejemplos del texto que la app debe usar:

- **Acciones:** "Compras una parte de una empresa. El mercado abre y cierra cada día: si pasa algo mientras está cerrado, el precio puede abrir en otro nivel. Eso es un **gap**, y tu **stop loss** puede ejecutarse peor de lo que pusiste."
- **Índices:** "Un índice es como el promedio de muchas empresas a la vez. Sube lento y cae rápido. Que esté 'diversificado' no significa que no puedas perder."
- **Forex:** "Compras una moneda vendiendo otra. Está abierto casi todo el día, pero no se comporta igual a toda hora: hay **sesiones**. El **spread** (la diferencia entre el precio de compra y el de venta) se amplía cuando hay noticias o poca actividad. El **leverage** (apalancamiento) te deja controlar una posición más grande con menos capital: amplifica lo que ganas y lo que pierdes, por igual."
- **Cripto:** "Nunca cierra y se mueve fuerte. Una vela verde gigante no es una invitación: muchas veces es una trampa. Con apalancamiento alto, una caída normal puede **liquidar** tu posición: la pierdes completa aunque el precio después se recupere."
- **Commodities:** "Son cosas físicas: energía, metales, granos. El precio puede moverse violentamente por algo que no aparece en el gráfico: clima, conflictos, inventarios. Por eso existe el stop loss: para lo que no puedes predecir."
- **Futuros:** "Contratos de tamaño fijo. No eliges cuánto arriesgar libremente: cada **tick** (el movimiento mínimo del precio) vale una cantidad concreta de dinero. Si tu cuenta no soporta un contrato, la respuesta correcta es no operarlo."

Mensaje transversal anti-señales (presente en Forex y Cripto): "Una señal te dice dónde entrar. No te dice cuánto arriesgar, dónde salir si falla, ni si tiene sentido para tu cuenta. Por eso la misma señal hace ganar a uno y quiebra a otro."

---

## 7. Preparación para el modo histórico (sin requerirlo en MVP)

Burgundy es **historical-ready**: la arquitectura de seeds y LCC permite incorporar datos o comportamientos históricos después, sin rediseñar nada.

| Mecanismo | Descripción |
|---|---|
| **Plantillas inspiradas en histórico** | Se extraen propiedades estadísticas y narrativas de episodios reales (un colapso cripto, una crisis cambiaria, un shock petrolero) y se usan para parametrizar el generador sintético. No requieren los datos crudos: requieren su "forma". Disponibles como evolución natural post-MVP. |
| **Replay histórico real (futuro)** | Una fuente de datos adicional que alimenta el mismo contrato de escenario: el camino ya no se genera, se reproduce. El motor, el LCC, el journal y el sistema de score no cambian — solo cambia el origen del camino de precio. |
| **Compatibilidad por mercado** | Índices y commodities son los mejores candidatos iniciales (episodios bien documentados y educativamente ricos); Forex aporta crisis cambiarias relevantes para LATAM; cripto aporta ciclos completos de manía y colapso; acciones aporta earnings y caídas célebres; futuros hereda del subyacente. |
| **Requisito de diseño desde MVP** | El identificador de seed reserva espacio para el tipo de origen del camino (generado vs. histórico-inspirado vs. histórico-real), de modo que replays, rankings y journals futuros distingan la fuente sin migración de datos. |

---

## 8. Resumen ejecutivo

- Burgundy cubre 7 tipos de mercado; **4 en MVP** según `MVP_MARKET_LOCK` (sintético como base + acciones, Forex y cripto como instrumentos jugables), índices y commodities como post-MVP temprano, y futuros como fase futura.
- Cada mercado existe por su **lección única**: sintético enseña mecánica, acciones enseñan gaps y estructura, índices enseñan macro y paciencia, Forex enseña sesiones y apalancamiento, cripto enseña control emocional, commodities enseñan shocks externos, futuros enseñan riesgo profesional.
- El orden de progresión **invierte deliberadamente el camino típico del principiante LATAM**: cripto y el apalancamiento alto llegan al final, como logro de disciplina, no como punto de partida.
- Todos los mercados se simulan con **un único motor generativo parametrizado**, con caminos pre-generados por seed determinística y eventos insertados antes de cualquier acción del usuario. Sin datos de broker, sin manipulación post-entrada, offline-first.
- La arquitectura de seeds y Learning Context Contracts deja el sistema **listo para histórico** sin requerirlo en el MVP.

---

*Documento del proyecto Burgundy — firmado bajo el usuario **tsuloid**. Material educativo de diseño de producto; no constituye asesoría financiera.*
