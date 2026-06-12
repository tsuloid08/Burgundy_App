# Burgundy — Bloque 11: Sistema de Evaluación y Scoring

**Proyecto:** Burgundy — Simulador educativo de trading
**Autor:** tsuloid
**Documento:** `docs/architecture/11_evaluation_scoring.md`
**Idioma:** Español (LATAM)
**Estado:** Especificación de diseño — sin código, sin pantallas

---

## 0. Principio rector

Burgundy **no evalúa al usuario solo por la ganancia**. El simulador fuerza un contexto de aprendizaje, no un resultado de trading. El mercado se genera antes de la acción del usuario (seed determinista) y nunca se manipula después de su entrada. Por lo tanto, lo único que el usuario controla —y lo único que Burgundy califica con mayor peso— es **el proceso**: gestión de riesgo, disciplina, paciencia, consistencia y supervivencia.

Regla central del sistema:

> **Un trade perdedor con proceso correcto puede recibir nota alta. Un trade ganador con proceso peligroso puede recibir nota baja.**

Esto se evalúa siempre contra el **Learning Context Contract** (contrato de contexto de aprendizaje) de cada escenario: cada sesión declara qué se está enseñando, y el score mide qué tan bien el usuario cumplió ese objetivo, no cuánto dinero ficticio ganó.

---

## 1. Categorías de score

El score total de una sesión se compone de siete categorías. Cada una se calcula de 0 a 100 y luego se pondera (ver sección 2).

| # | Categoría | Qué mide | Ejemplos de señales |
|---|-----------|----------|---------------------|
| 1 | **Rentabilidad (Profitability)** | Resultado económico del periodo simulado | Total return, retorno ajustado por riesgo |
| 2 | **Gestión de riesgo (Risk Management)** | Cuánto arriesgó por trade y en total | % arriesgado por posición, max drawdown, uso de leverage |
| 3 | **Disciplina (Discipline)** | Respeto a las reglas propias y del escenario | Uso y respeto del stop loss, tamaño de posición correcto |
| 4 | **Calidad de proceso (Process Quality)** | Si las decisiones siguieron un razonamiento válido | Setup válido antes de entrar, cumplimiento del objetivo de lección |
| 5 | **Paciencia (Patience)** | Capacidad de esperar y de no operar | Trades evitados correctamente, tiempo de espera antes de entrar, no perseguir velas |
| 6 | **Consistencia de estrategia (Strategy Consistency)** | Si el usuario opera con un criterio estable | Varianza del risk/reward, varianza del tamaño de posición, coherencia entre trades |
| 7 | **Supervivencia (Survival)** | Si la cuenta llegó viva y sana al final | Completó el periodo, no tocó el umbral de blow-up, crecimiento sostenible |

Nota educativa para el usuario (en la app, explicado en español claro): *"Tu score no es tu ganancia. Tu score es qué tan bien operaste. Un casino te premia por ganar una vez; Burgundy te entrena para no quebrar nunca."*

---

## 2. Sistema de ponderación

### 2.1 Pesos base (modo Sandbox / Modo Libre)

| Categoría | Peso base |
|-----------|-----------|
| Rentabilidad | 15% |
| Gestión de riesgo | 20% |
| Disciplina | 20% |
| Calidad de proceso | 15% |
| Paciencia | 10% |
| Consistencia de estrategia | 10% |
| Supervivencia | 10% |

La rentabilidad nunca pesa más del 25% en ningún modo. Esto hace que el sistema sea difícil de explotar: no se puede "farmear" score apostando fuerte.

### 2.2 Pesos por intención del escenario (Learning Context Contract)

Cada escenario declara un `scoringProfile` que redistribuye los pesos según lo que enseña:

| Perfil de escenario | Rentab. | Riesgo | Disciplina | Proceso | Paciencia | Consist. | Superviv. |
|---|---|---|---|---|---|---|---|
| Tutorial (lección guiada) | 5% | 15% | 20% | **35%** | 10% | 5% | 10% |
| Lección de gestión de riesgo | 5% | **35%** | 25% | 15% | 5% | 5% | 10% |
| Trampa FOMO | 0% | 15% | 20% | 20% | **35%** | 5% | 5% |
| Lección de copiar señales | 5% | 15% | 20% | **30%** | 15% | 5% | 10% |
| Desafío sin indicadores | 10% | 15% | 15% | **30%** | 10% | 10% | 10% |
| Desafío de supervivencia | 5% | 20% | 15% | 10% | 10% | 10% | **30%** |
| Desafío de control de drawdown | 10% | **30%** | 15% | 10% | 5% | 10% | 20% |
| Desafío de disciplina de riesgo | 5% | 25% | **30%** | 15% | 5% | 10% | 10% |
| Sandbox / Modo libre | 15% | 20% | 20% | 15% | 10% | 10% | 10% |

Consecuencias directas:

- En una **trampa FOMO**, no entrar puede ser la mejor decisión: la paciencia pesa 35% y la rentabilidad 0%.
- En una **lección de riesgo**, perder 1% con stop respetado puede ser un éxito (riesgo 35% + disciplina 25%).
- En una **lección de señales**, rechazar una señal mala y atrasada es una victoria de proceso (30%).
- En un **desafío de supervivencia**, evitar la destrucción de la cuenta vale más que el retorno de corto plazo.
- En un **desafío de drawdown**, un drawdown bajo puede vencer a un retorno alto.

---

## 3. Métricas y fórmulas conceptuales

Todas las fórmulas son conceptuales (sin código). Cada métrica se normaliza a una escala 0–100 antes de entrar a su categoría.

### 3.1 Métricas de resultado

| Métrica | Definición conceptual | Normalización conceptual |
|---|---|---|
| **Total return** | (Equity final − Equity inicial) / Equity inicial | Curva en S: retornos negativos castigan, retornos altísimos saturan (no dan score infinito) |
| **Retorno ajustado por riesgo** | Total return / max drawdown del periodo (concepto tipo "MAR ratio" simplificado). Si el drawdown es casi cero, se usa un piso mínimo de drawdown para evitar divisiones absurdas | Es la métrica de rentabilidad **principal**: 8% con 4% de drawdown (ratio 2.0) supera a 40% con 38% de drawdown (ratio ~1.05) |
| **Max drawdown** | Mayor caída porcentual de la equity desde su punto más alto (explicado al usuario: "cuánto cayó tu cuenta desde su mejor momento") | Score alto con drawdown bajo; cae rápido pasado el umbral del escenario (p. ej. >15%) |
| **Crecimiento sostenible** | El retorno proviene de muchos trades moderados y no de uno o dos trades gigantes. Conceptualmente: porcentaje del P/L total aportado por el mejor trade. Si un solo trade explica >60% de la ganancia, no es sostenible | Reduce el score de Supervivencia/Rentabilidad cuando la ganancia depende de un golpe de suerte |

### 3.2 Métricas de comportamiento por trade

| Métrica | Definición conceptual | Cómo afecta |
|---|---|---|
| **Win rate** | Trades ganadores / trades totales | Métrica **informativa y secundaria**: se muestra, pero pesa poco. Burgundy enseña que un win rate alto con pérdidas grandes es una trampa |
| **Average win/loss** | Ganancia promedio de los ganadores vs. pérdida promedio de los perdedores | Ratio ≥ 1.5 suma; ratio < 1 con win rate bajo resta (perfil "gano poquito, pierdo mucho") |
| **Consistencia de risk/reward** | Desviación del R:R planificado entre trades. Se mide contra el R:R declarado o implícito por stop y target | Baja varianza = score alto en Consistencia. R:R caótico (un trade 1:3, otro 1:0.2) = score bajo |
| **Número de trades** | Conteo total, comparado con el rango esperado del escenario (cada seed define un rango razonable según oportunidades reales del path) | Dentro del rango: neutro. Muy por encima: alimenta Overtrading. Muy por debajo sin trades evitados justificados: puede indicar parálisis (penalización leve, solo en escenarios que exigen operar) |
| **Overtrading score** | Frecuencia de trades vs. oportunidades válidas del path + entradas en racha rápida (varios trades en pocas velas) | Castiga Paciencia y Disciplina |
| **Detección de revenge trading** | Patrón en el decision log: pérdida → reentrada inmediata (pocas velas) → con tamaño mayor → frecuentemente en dirección contraria o sin setup | Cada episodio detectado aplica penalización fuerte a Disciplina y marca "error emocional" en el journal |
| **Disciplina de stop loss** | (a) % de trades abiertos con stop definido; (b) % de stops respetados sin moverlos en contra; (c) cierres manuales antes del stop por pánico | Mover el stop "para darle aire" a una posición perdedora es de las penalizaciones más severas del sistema |
| **Disciplina de tamaño de posición** | Riesgo por trade vs. límite del escenario (p. ej. máx. 1–2% de la cuenta por trade). Se mide el riesgo real: distancia al stop × tamaño | Cada trade que excede el límite resta; exceder sistemáticamente colapsa la categoría Riesgo |
| **Disciplina de leverage** | Apalancamiento usado vs. máximo recomendado del escenario | Leverage imprudente penaliza Riesgo aunque el trade gane |
| **Conducta de copiar señales** | En escenarios con señales simuladas (estilo Telegram/TikTok): si el usuario entró a la señal sin verificar contexto, entró tarde a una señal vencida, o la rechazó correctamente | Rechazar una señal mala = bonus de Proceso. Copiar sin verificar = penalización de Proceso aunque gane |
| **Detección de errores emocionales** | Conjunto de patrones del decision log: perseguir velas (entrar tras un movimiento extendido), promediar pérdidas sin plan, cerrar ganadores en pánico inmediato, agrandar posición tras ganar varias seguidas (euforia) | Cada patrón detectado genera una entrada en la revisión de errores con explicación educativa |
| **Supervivencia del periodo** | ¿El usuario llegó al final de la simulación con la cuenta por encima del umbral de blow-up? | Binario con matiz: sobrevivió cómodo / sobrevivió rozando el límite / no sobrevivió |

### 3.3 Cómo se detectan los errores desde el decision log

Toda decisión del usuario queda registrada con timestamp de vela, estado de cuenta y contexto: abrir, cerrar, modificar stop/target, cambiar tamaño, cambiar leverage, saltar un trade, aceptar/rechazar una señal. La detección de errores es **determinista sobre ese log**, nunca subjetiva:

| Error | Regla de detección conceptual |
|---|---|
| Entrada impulsiva | Apertura a menos de N velas de haber visto el activo, sin setup marcado, tras movimiento extendido |
| Revenge trading | Pérdida cerrada → nueva entrada en ≤ M velas con tamaño ≥ 1.5× el anterior |
| Sin stop loss | Posición abierta sin stop definido durante más de K velas |
| Stop movido en contra | Modificación del stop alejándolo del precio en posición perdedora |
| Sobreapalancamiento | Leverage > límite del escenario al abrir |
| Promediar pérdidas | Añadir tamaño a posición en pérdida sin que el plan del escenario lo contemple |
| Perseguir señal vencida | Entrada a señal simulada después de su ventana de validez |
| FOMO | Entrada inmediatamente después de una vela de expansión fuerte, en su misma dirección, sin retroceso |

Cada error detectado: (1) penaliza la categoría correspondiente, (2) crea una entrada en la **revisión de errores** con explicación en español claro, y (3) alimenta las estadísticas de progreso a largo plazo del perfil local.

---

## 4. Niveles de calificación (grades)

El score total ponderado (0–100) se traduce a una letra con lenguaje de academia seria:

| Grade | Rango | Etiqueta en la app | Significado educativo |
|---|---|---|---|
| **S** | 95–100 | Ejecución magistral | Proceso impecable + objetivo de lección cumplido + riesgo controlado. Raro a propósito |
| **A** | 85–94 | Disciplina sólida | Proceso correcto con detalles menores |
| **B** | 70–84 | Operativa competente | Buen proceso con errores puntuales identificados |
| **C** | 55–69 | En desarrollo | Proceso inconsistente; errores repetidos pero sin conducta destructiva |
| **D** | 40–54 | Proceso frágil | Errores graves de riesgo o disciplina; el resultado (gane o pierda) no es confiable |
| **F** | 0–39 | Sesión fallida | Conducta peligrosa dominante, violación del objetivo, o blow-up |

Reglas duras (caps) que ningún peso puede esquivar:

- **Blow-up = F automática**, sin importar el resto de métricas.
- **Operar sin stop loss en más del 50% de los trades** → grade máximo C.
- **Violación directa del objetivo de la lección** (p. ej. entrar en una trampa FOMO diseñada para no entrar) → grade máximo C en esa sesión.
- **Revenge trading detectado 2+ veces en la sesión** → grade máximo C.
- Estos topes existen para que el sistema sea **difícil de explotar**: ganar mucho no compra una buena nota.

---

## 5. Reglas de penalización

Las penalizaciones se aplican sobre la categoría correspondiente antes de ponderar. Valores conceptuales de referencia:

| Penalización | Severidad | Categoría afectada |
|---|---|---|
| Trade sin stop loss | Alta (−15 a −25 por trade, acumulable) | Disciplina |
| Mover stop en contra de la posición | Muy alta (−25 por evento) | Disciplina |
| Riesgo por trade > límite del escenario | Alta, proporcional al exceso | Gestión de riesgo |
| Leverage > máximo recomendado | Alta, proporcional | Gestión de riesgo |
| Episodio de revenge trading | Muy alta (−30 por episodio) | Disciplina + marca emocional |
| Overtrading (exceso sobre rango esperado) | Media, proporcional | Paciencia |
| Entrada impulsiva / FOMO | Media (−10 a −20 por evento) | Proceso |
| Copiar señal simulada sin verificar | Alta (−20) | Proceso |
| Promediar pérdidas sin plan | Alta (−20) | Riesgo + Disciplina |
| Cerrar la sesión abandonando posiciones abiertas sin gestión | Media | Proceso |
| Drawdown sobre el umbral del escenario | Alta, escalonada | Riesgo + Supervivencia |
| Ganancia concentrada en 1 trade (>60% del P/L) | Media | Rentabilidad (recorta el componente "sostenible") |

Principio anti-exploit: **las penalizaciones se acumulan por evento, no se promedian**. Diez trades buenos no "lavan" un episodio de revenge trading; el episodio queda registrado y explicado.

---

## 6. Reglas de bonus

Los bonus premian conducta que en el mercado real protege la cuenta. Topes por categoría para que no sean farmeables:

| Bonus | Condición | Categoría |
|---|---|---|
| **Pérdida bien gestionada** | Trade perdedor con setup válido, stop definido y respetado, tamaño correcto | Proceso (+10 por trade, máx. +30 por sesión) |
| **Trade evitado correctamente** | Saltar deliberadamente una trampa del escenario (FOMO, señal vencida) | Paciencia (+15 a +25 según el escenario) |
| **Racha de disciplina** | N trades consecutivos con stop, tamaño y R:R dentro de regla | Disciplina (+10) |
| **Drawdown mínimo** | Terminar con drawdown muy por debajo del umbral del escenario | Riesgo (+10) |
| **Objetivo de lección cumplido al 100%** | Checklist del Learning Context Contract completa | Proceso (+15) |
| **Supervivencia holgada** | Terminar el desafío de supervivencia sin acercarse nunca al umbral de blow-up | Supervivencia (+15) |
| **Consistencia de R:R** | Varianza de R:R baja en sesiones con 5+ trades | Consistencia (+10) |
| **Paciencia previa a la entrada** | Esperar confirmación definida por la lección antes de entrar | Paciencia (+5 por trade, con tope) |

Regla importante: **no existe bonus por ganancia grande**. La ganancia ya está medida (y saturada) en la categoría Rentabilidad.

---

## 7. Reglas de blow-up y fallo de sesión

| Concepto | Regla |
|---|---|
| **Umbral de blow-up** | Definido por escenario; por defecto, equity ≤ 30% del capital inicial (configurable por desafío; en supervivencia puede ser 50%) |
| **Efecto inmediato** | La sesión termina, grade F automática, score total con tope de 39 |
| **Margen agotado** | Si una posición apalancada consume el margen disponible, se simula la liquidación con costos realistas (spread + slippage) y cuenta como evento de blow-up parcial |
| **Fallo de objetivo** | En desafíos con condición explícita (p. ej. "mantén el drawdown bajo 10%"), romper la condición termina o invalida el desafío aunque la cuenta siga viva |
| **Registro educativo** | Todo blow-up genera una revisión obligatoria de errores: qué secuencia de decisiones llevó a la pérdida, con explicación en español claro y sin humillar al usuario |
| **No hay borrado de evidencia** | El blow-up queda en el historial local del perfil. El usuario puede reintentar el seed, pero el intento fallido cuenta en sus estadísticas de proceso (ver 8.3) |

Mensaje tipo tras blow-up (tono Burgundy: serio, directo, no fatalista): *"Tu cuenta no sobrevivió. En el mercado real, esto sería el final del capital. Revisemos las tres decisiones que destruyeron la cuenta."*

---

## 8. Lógica de guardado de high scores

### 8.1 Qué se guarda

Por cada combinación **(modo, escenario/desafío, seed, dificultad)** se guarda localmente:

- **Mejor score total** (high score) con su desglose por categoría.
- **Mejor grade** alcanzado.
- **Número de intentos** totales en ese seed.
- **Historial resumido de intentos** (score, grade, fecha, métricas clave) para la curva de progreso.
- Referencia al **decision log** del mejor intento (para replay y comparación).

### 8.2 Cuándo se reemplaza un high score

- Solo se reemplaza si el **score total nuevo es estrictamente mayor**.
- Empate de score: gana el intento con **mejor score de proceso**; si persiste el empate, el de **menor drawdown**.
- El high score **nunca baja**: un mal intento posterior no borra el mejor registro, pero sí cuenta en el historial de intentos.

### 8.3 Intentos repetidos sobre el mismo seed

Repetir un seed es una herramienta de aprendizaje legítima (replay), pero conocer el path de memoria da ventaja. Burgundy lo maneja así:

- Cada intento registra su **número de intento** (1º, 2º, 3º…).
- El ranking del seed distingue **score en primer intento** vs. **mejor score en cualquier intento**. El primer intento es el que mide lectura real del mercado; los siguientes miden ejecución y corrección de errores.
- En los rankings de desafío, la posición principal se calcula con el **primer intento por seed**; el "mejor intento" se muestra como métrica de progreso personal.
- La app celebra explícitamente la **mejora entre intentos** ("redujiste tu drawdown de 18% a 6% en el mismo escenario") porque esa es la señal educativa más valiosa.

---

## 9–14. Sistema de rankings

Todos los rankings son **locales** (no hay cuenta ni nube). Comparan los intentos del propio usuario y, en el futuro, podrían compararse archivos exportados entre amigos (import/export ya soporta llevar el historial). Cada ranking guarda score total, grade, desglose por categoría y métricas clave.

### 9. Ranking por sesión

- Lista histórica de todas las sesiones del usuario ordenadas por score total.
- Filtrable por modo (tutorial, sandbox, desafío, libre).
- Muestra la tendencia: media móvil del score de las últimas N sesiones, para que el usuario vea si su proceso mejora aunque algunos resultados sean rojos.

### 10. Ranking por desafío

- Cada desafío (supervivencia, drawdown, sin indicadores, disciplina de riesgo, etc.) tiene su tabla propia.
- Como los desafíos usan **seeds fijos**, todos los intentos de un mismo desafío enfrentaron exactamente el mismo mercado: la comparación es justa por construcción (ver sección 18).
- Columna obligatoria junto al score: **drawdown máximo** y **grade**, para reforzar que el primer puesto no es "el que más ganó".

### 11. Ranking por mercado

- Tablas separadas por tipo de activo simulado (p. ej. divisas, cripto, índices, acciones — según los mercados sintéticos definidos en el motor).
- Sirve para que el usuario descubra en qué tipo de mercado su proceso es más débil (p. ej. buen score en tendencias suaves, mal score en mercados volátiles tipo cripto).

### 12. Ranking por horizonte temporal

- Tablas por duración/timeframe de la simulación: scalping simulado, intradía, swing, posición de largo plazo.
- Las métricas se interpretan por horizonte: el rango esperado de número de trades, el umbral de overtrading y el peso de la paciencia cambian según el horizonte (más trades es normal en intradía; en swing, overtrading se detecta antes).

### 13. Ranking por dificultad

- Cada nivel de dificultad (definido por volatilidad del path, costos de spread/slippage, límites de riesgo más estrictos, menos información visible) tiene tabla separada.
- **Los scores no se mezclan entre dificultades**: un 80 en difícil no compite contra un 80 en fácil.
- Se aplica además un **multiplicador de dificultad al XP otorgado** (no al score): el score siempre es comparable dentro de su tabla; el XP refleja el mérito global.

### 14. Ranking por seed

- Tabla por seed individual: todos los intentos del usuario sobre ese path exacto.
- Distingue primer intento vs. reintentos (sección 8.3).
- Es la base de la comparación de replay (sección 15).

---

## 15. Scoring de comparación en replay

El modo replay reproduce el mismo seed con el decision log de un intento anterior visible como referencia. La comparación entre el intento original y el nuevo:

| Dimensión comparada | Qué se muestra |
|---|---|
| Score total y por categoría | Delta por categoría: dónde mejoró y dónde empeoró |
| Curva de equity superpuesta (conceptual) | Ambas trayectorias de cuenta sobre el mismo path de mercado |
| Decisiones divergentes | Velas donde el usuario tomó una decisión distinta a su intento anterior, con el efecto de cada una |
| Errores corregidos / repetidos / nuevos | Lista explícita: "Corregiste: entraste sin stop en la vela 142. Repetiste: tamaño excesivo en la vela 201." |
| Drawdown comparado | Max drawdown de cada intento |

Reglas:

- El replay **nunca compite en el ranking de primer intento** (sería información perfecta del futuro del path).
- El score del replay alimenta el "mejor intento" y las estadísticas de mejora, con la etiqueta `replay` siempre visible.
- La métrica estrella del replay es **errores corregidos**, no el score: el objetivo del modo es cerrar el ciclo error → revisión → corrección.

---

## 16. Mensajes de feedback

El feedback es la salida educativa principal del sistema de scoring. Tono: claro, serio, directo, en español LATAM, sin hype, sin humillación, sin consejo financiero. Cada mensaje conecta **decisión → consecuencia → principio**.

### 16.1 Estructura de todo feedback de fin de sesión

1. **Resultado del proceso primero, P/L después**: "Proceso: B. Resultado: −1.2%."
2. **Lo que hiciste bien** (máximo 3 puntos, concretos, citando trades del log).
3. **Tu error más costoso** (uno solo, el de mayor penalización, con la vela y la decisión exacta).
4. **El principio detrás** (explicación del término real en español simple).
5. **Qué practicar después** (vínculo al escenario o lección que entrena esa debilidad).

### 16.2 Ejemplos de mensajes (catálogo de referencia)

| Situación | Mensaje tipo |
|---|---|
| Pérdida con buen proceso | "Perdiste 1% y aun así esta fue una buena operación. Definiste tu stop, arriesgaste lo correcto y respetaste tu plan. En trading real, las pérdidas controladas son el costo de operar: no son errores." |
| Ganancia con mal proceso | "Ganaste 6%, pero entraste sin stop loss y con 5 veces el riesgo permitido. Este resultado fue suerte, no habilidad. Con este proceso, la misma decisión destruye tu cuenta tarde o temprano." |
| Trampa FOMO evitada | "No entraste después de la vela de expansión. Esa era exactamente la trampa de este escenario. Saber no operar es una habilidad: la mayoría de los principiantes pierde aquí." |
| Revenge trading detectado | "Después de tu pérdida en la vela 87, reentraste en 3 velas con el doble de tamaño. Eso es revenge trading: intentar recuperar rápido lo perdido. Es uno de los patrones que más cuentas destruye." |
| Stop movido en contra | "Moviste tu stop loss para alejarlo del precio cuando ibas perdiendo. El stop existe para limitar tu pérdida máxima; moverlo en contra convierte una pérdida pequeña en una grande." |
| Señal rechazada correctamente | "Rechazaste la señal porque llegó tarde y el contexto ya no la respaldaba. Eso es exactamente lo que este escenario entrena: verificar antes de copiar." |
| Overtrading | "Hiciste 14 operaciones cuando el escenario ofrecía 3 o 4 oportunidades válidas. Operar de más multiplica costos (spread, slippage) y errores. La paciencia también es una posición." |
| Blow-up | "Tu cuenta no sobrevivió. Tres decisiones la destruyeron: leverage de 20x en la vela 55, sin stop en la vela 61, y doblar la posición perdedora en la vela 64. Revisémoslas una por una." |
| Drawdown controlado | "Terminaste con 8% de retorno y solo 4% de drawdown. Esa relación es lo que diferencia una cuenta que crece de una que sobrevive de milagro." |

### 16.3 Reglas del feedback

- Siempre cita **evidencia del decision log** (vela, decisión, número), nunca juicios vagos tipo "fuiste impulsivo".
- Un solo error principal por resumen; el resto vive en la revisión de errores detallada.
- Términos técnicos siempre en inglés estándar con explicación en español al primer uso de la sesión (spread, slippage, drawdown, leverage, risk/reward).
- Prohibido: tono de casino, celebración de ganancias grandes con mal proceso, lenguaje de "estás listo para el mercado real", promesas de rentabilidad.

---

## 17. Ejemplos comparativos entre usuarios

### Ejemplo A — El caso central: 8% controlado vs. 40% al borde del abismo

Mismo desafío, mismo seed fijo, dificultad media, perfil sandbox de pesos.

| Métrica | Usuaria 1 ("Mariana") | Usuario 2 ("Diego") |
|---|---|---|
| Total return | +8% | +40% |
| Max drawdown | 4% | 38% (umbral de blow-up: 70% de pérdida; rozó liquidación de margen) |
| Retorno ajustado por riesgo | 2.0 | ~1.05 |
| Riesgo por trade | 1% constante | 8–15%, variable |
| Stop loss | 100% de trades, siempre respetado | 30% de trades, movido en contra 2 veces |
| Leverage | Dentro del límite | 3× el máximo recomendado |
| Revenge trading | 0 episodios | 1 episodio |
| Concentración del P/L | Repartido en 9 trades | 72% de la ganancia en 1 trade |
| **Score por categoría** | Rentab. 78 · Riesgo 95 · Disciplina 96 · Proceso 90 · Paciencia 85 · Consist. 92 · Superviv. 95 | Rentab. 70 (saturada y recortada por concentración) · Riesgo 20 · Disciplina 15 · Proceso 30 · Paciencia 40 · Consist. 25 · Superviv. 45 |
| **Score total** | **≈ 90 → A** | **≈ 36 → D (con cap C inalcanzable por stops, queda D)** |

Lectura educativa que muestra la app: *"Diego ganó 5 veces más dinero y obtuvo la mitad del score. Su proceso habría destruido la cuenta en la mayoría de los paths posibles; el de Mariana sobrevive en casi todos."*

### Ejemplo B — Perder bien vs. ganar mal (lección de gestión de riesgo)

| | Usuario 3 ("Luis") | Usuario 4 ("Carla") |
|---|---|---|
| Resultado | −1% (stop ejecutado) | +3% |
| Proceso | Setup válido, 1% de riesgo, stop respetado, objetivo de lección cumplido | Sin stop, entrada impulsiva tras vela de expansión, 6% de riesgo |
| Score | **≈ 88 → A** | **≈ 42 → D** |

Mensaje a Luis: pérdida bien gestionada (ver 16.2). Mensaje a Carla: ganancia con mal proceso.

### Ejemplo C — Trampa FOMO (perfil con paciencia al 35%)

| | Usuario 5 ("Ana") | Usuario 6 ("Pedro") |
|---|---|---|
| Acción | No entró; marcó la trampa y esperó | Entró en la expansión, ganó 2% por timing afortunado |
| Score | **≈ 93 → A** (bonus de trade evitado + objetivo cumplido) | **≈ 50 → C (cap por violar el objetivo de la lección)** |

### Ejemplo D — Mismo seed, primer intento vs. replay

| | Intento 1 | Intento 3 (replay) |
|---|---|---|
| Score | 61 (C) | 84 (B) |
| Ranking de primer intento | Cuenta: 61 | No compite |
| Mejor intento del seed | — | Cuenta: 84, etiquetado `replay` |
| Métrica destacada | — | "Errores corregidos: 3 de 4. Error repetido: tamaño excesivo en tendencia fuerte." |

---

## 18. Equidad, separación de modos y anti-exploit

### 18.1 Por qué los seeds fijos hacen justos los rankings

- En desafíos y tutoriales, el seed fijo garantiza que **todos los intentos enfrentan exactamente el mismo path de mercado**: mismas oportunidades, mismas trampas, mismos costos. La diferencia de score solo puede venir de las decisiones.
- El path se genera **antes** de cualquier acción y nunca se ajusta después de la entrada del usuario (filosofía central del simulador): nadie puede alegar que "el mercado lo persiguió".
- El rango esperado de trades, los umbrales de drawdown y las trampas del escenario se calibran **por seed**, de modo que las métricas de comportamiento se midan contra lo que ese path realmente ofrecía.

### 18.2 Sandbox con seeds aleatorios: scoring separado

- Los scores de sandbox **nunca entran** en las tablas de desafíos: cada seed aleatorio es un mercado distinto y la comparación directa sería injusta.
- El sandbox tiene su propio historial y su tendencia de proceso (media móvil de score de proceso), que es comparable entre seeds porque las categorías de comportamiento (disciplina, riesgo, paciencia) se normalizan contra los límites del escenario, no contra el path concreto.
- El seed de cada sesión sandbox queda guardado: cualquier sesión interesante puede convertirse en replay o compartirse vía export.

### 18.3 Resultado vs. proceso: la distinción operativa

| | Calidad de resultado | Calidad de proceso |
|---|---|---|
| Fuente | Curva de equity (P/L, drawdown, retorno ajustado) | Decision log (cada decisión contra las reglas del escenario) |
| Depende del path | Sí, parcialmente | No: se evalúa contra reglas, no contra el resultado |
| Peso en el score | Minoritario (≤ 25%) | Mayoritario |
| Mensaje al usuario | "Qué pasó con tu cuenta" | "Qué tan bien decidiste" |

Burgundy siempre muestra ambos por separado antes del score combinado, para que el usuario internalice que son cosas distintas.

### 18.4 Defensas anti-exploit (resumen)

1. Rentabilidad saturada y con peso máximo de 25%: apostar fuerte no compra score.
2. Caps de grade por conducta peligrosa (sección 4): no se "compensan" con buenos números.
3. Penalizaciones acumulables por evento, no promediadas.
4. Bonus con topes por sesión: la disciplina no es farmeable repitiendo micro-trades.
5. Rankings de primer intento separados: memorizar el path no infla la tabla principal.
6. Concentración del P/L detectada: un golpe de suerte no se disfraza de habilidad.
7. Parálisis penalizada solo donde corresponde: no operar nunca tampoco maximiza score en escenarios que exigen ejecutar (excepto en trampas, donde no operar es el objetivo declarado).
8. Detección determinista sobre el decision log: las reglas son auditables y consistentes; el usuario puede ver exactamente qué decisión disparó cada penalización.

---

## 19. Persistencia local y export/import

- Todo el sistema (high scores, rankings, historial de intentos, decision logs de mejores intentos, estadísticas de errores) se guarda **localmente**, sin login ni nube.
- El archivo de export de progreso incluye el historial de scoring completo, de modo que al importarlo en otro dispositivo el usuario conserva rankings, grades, rachas y curvas de mejora.
- Los decision logs se guardan en formato compacto referenciando seed + secuencia de decisiones, lo que permite reconstruir cualquier sesión (el path se regenera determinísticamente desde el seed, en formato de vela universal y agnóstico de fuente, listo para el futuro modo histórico).

---

*Documento de arquitectura del proyecto Burgundy, firmado por **tsuloid**. Material exclusivamente educativo: Burgundy es un simulador de práctica y nada en este sistema constituye asesoría financiera.*
