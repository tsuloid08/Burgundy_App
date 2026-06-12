# Burgundy — Sandbox, Desafíos, Rankings y High Scores

**Documento:** 06_sandbox_challenges.md
**Proyecto:** Burgundy — Simulador educativo de trading
**Autor / Firma:** tsuloid
**Idioma:** Español (LATAM)
**Estado:** Especificación de modos de juego, desafíos y persistencia de puntajes
**Alcance de este documento:** Solo game loop, libertad del usuario, desafíos y lógica de puntajes. No incluye código, ni pantallas de UI, ni modelos finales de base de datos.

---

## 0. Principios que gobiernan este documento

Burgundy es una academia seria dentro de un terminal de mercado oscuro con tema Burgundy. Todo lo que se define aquí responde a cinco reglas heredadas del contexto global del proyecto:

1. **El simulador fuerza un contexto de aprendizaje, nunca un resultado de trading.** El camino del mercado se genera completo antes de que el usuario actúe, y jamás se manipula después de su entrada.
2. **Determinismo por semilla (seed):** misma plantilla de escenario + parámetros + seed + versión del generador = mismo camino de mercado, siempre.
3. **Offline-first, sin login, sin nube.** Todos los rankings y high scores viven en el dispositivo. El usuario puede exportar e importar su progreso mediante un archivo de sesión.
4. **El puntaje premia disciplina, gestión de riesgo, supervivencia y calidad de proceso — no solo la ganancia.** Un usuario que gana rompiendo todas las reglas debe puntuar peor que uno que pierde poco operando con disciplina.
5. **Nada de casino.** Sin confeti, sin "jackpots", sin presión por rachas de ganancia monetaria. La progresión se siente como entrenar en una academia, no como apostar.

---

## 1. Opciones de configuración del Sandbox Mode

El Sandbox es el espacio de práctica libre de Burgundy. El usuario configura su sesión antes de iniciarla; una vez generado el camino de mercado, la configuración queda sellada para esa sesión.

### 1.1 Tabla de configuración

> **Alcance MVP cerrado por `MVP_SANDBOX_LIMITS`, `MVP_MARKET_LOCK` y `LEVERAGE_MVP_LIMITS` (documento 12).** La tabla siguiente describe el sandbox completo del producto; la columna de notas marca qué queda fuera del MVP. Ante contradicción, ganan los locks.

| Opción | Valores | Notas |
|---|---|---|
| **Tipo de mercado** | Forex sintético, acción sintética, cripto sintética | Set MVP cerrado por `MVP_MARKET_LOCK` (documento 12); índice sintético es post-MVP. Histórico queda como fuente futura, no MVP. |
| **Capital inicial** | Presets LATAM-realistas: 100, 500, 1.000, 5.000, 10.000, 100.000 (unidades de cuenta simuladas) | Los montos pequeños (100–1.000) existen a propósito: el usuario LATAM promedio empieza con poco capital y debe ver cómo se siente el riesgo real a esa escala. |
| **Dificultad** | Iniciación, Estándar, Avanzada, Hostil | Escala volatilidad, frecuencia de fakeouts, ruido, severidad de eventos, spread y slippage. |
| **Horizonte temporal** | **MVP: Intradía, 1 semana.** Post-MVP (Fase 3+): 1 mes, 3 meses, 6 meses, 1 año, 2 años | Define la duración del camino generado (`MVP_SANDBOX_LIMITS`, documento 12). Los horizontes largos habilitan los desafíos de interés compuesto y control de drawdown, ambos post-MVP. |
| **Velocidad de reproducción** | Pausa, 1x, 2x, 5x, 10x, salto a siguiente vela, salto a siguiente checkpoint | La velocidad nunca altera el camino: solo cambia el ritmo de reproducción de un camino ya generado. |
| **Indicadores** | Activados (set desbloqueado) / Desactivados (modo precio puro / raw price action) | Desactivar indicadores es una opción de entrenamiento seria, no un castigo. |
| **Apalancamiento (leverage)** | **No disponible en MVP (siempre 1x), incluso en Modo Libre** (`LEVERAGE_MVP_LIMITS`, documento 12). Post-MVP (Fase 5): activado con límite por nivel | Cuando llegue, solo desbloqueado por progresión y siempre acompañado de su explicación: controlar una posición más grande con menos capital, aumentando riesgo y retorno potencial. |
| **Pistas de tutorial (hints)** | Activadas / Desactivadas | Con hints activados la sesión es válida pero se marca como "asistida" y no compite en personal bests de seed. |
| **Seed** | Aleatoria (default), manual (ingresar/reusar una seed guardada) | Ver lógica completa de seeds en la sección 11. |

### 1.2 Libertades garantizadas del Sandbox

Dentro de una sesión de Sandbox el usuario puede:

- Operar libremente: comprar, vender, abrir y cerrar posiciones, usar stop loss, take profit y órdenes pendientes según lo desbloqueado.
- Usar o desactivar indicadores en cualquier momento (el cambio queda registrado en el decision log; una sesión cuenta como "sin indicadores" solo si jamás los activó).
- Pausar, acelerar o retroceder la reproducción para revisión visual (retroceder es solo lectura: no se puede "des-ejecutar" una orden).
- Guardar la sesión en cualquier punto y retomarla después *(post-MVP, Fase 3+ — `MVP_SANDBOX_LIMITS`; en MVP los horizontes son cortos y la sesión se completa o se descarta)*.
- Exportar la sesión a archivo e importarla en otro dispositivo o tras reinstalar *(post-MVP, Fase 3+ — en MVP la portabilidad es vía export del progreso completo `.burgundy`)*.
- Revisar el historial completo de trades de la sesión y de sesiones pasadas.
- Generar una seed aleatoria nueva, guardar la seed actual, reproducir una seed previa, reiniciar la misma seed tras quebrar la cuenta, o cambiar de seed para obtener un camino de mercado nuevo e independiente.

### 1.3 Lo que el Sandbox nunca permite

- Modificar el camino de mercado una vez generado.
- Editar o borrar entradas del decision log.
- Cambiar dificultad, mercado, horizonte o capital inicial a mitad de sesión (eso requiere una sesión nueva).
- "Deshacer" un trade ejecutado. Los errores se revisan, no se borran: esa es la base pedagógica del diario de errores.

---

## 2. Tipos de desafío (Challenge Mode)

Cada desafío es un escenario con **seed fija**, plantilla fija, Learning Context Contract (LCC) fijo y reglas explícitas. Todos los intentos de todos los usuarios (y todos los intentos del mismo usuario) corren contra exactamente el mismo camino de mercado, lo que hace justo el ranking local.

| # | Tipo de desafío | Qué entrena | Ejemplo de enunciado (tono Burgundy) |
|---|---|---|---|
| 1 | **Desafío diario** | Hábito y consistencia | "Sesión corta de hoy: 1 día intradía en índice sintético. Mismo mercado para todos los intentos. Mantén tu racha." |
| 2 | **Desafío de habilidad (skill)** | Una habilidad concreta del currículo (ej. colocar stop loss correctamente, dimensionar posición) | "Tres entradas máximo. Cada entrada debe tener stop loss antes de ejecutarse. Una entrada sin stop invalida el intento." |
| 3 | **Desafío de disciplina de riesgo** | Respetar un riesgo máximo por trade y por día | "Riesgo máximo: 1% por trade, 3% por día. Superar cualquiera de los dos límites termina el desafío en fallo, aunque vayas ganando." |
| 4 | **Desafío sin indicadores** | Lectura de precio puro (price action) | "Gráfico desnudo. Sin medias móviles, sin RSI, sin nada. Solo velas, soporte, resistencia y tu paciencia." |
| 5 | **Desafío de supervivencia** | No quebrar la cuenta en condiciones hostiles | "Mercado hostil de 1 mes con eventos de alta volatilidad. Objetivo único: terminar con la cuenta viva por encima del umbral de supervivencia." |
| 6 | **Trampa de copiado de señales** | Inmunidad a señales tipo Telegram/TikTok | "Recibirás 'señales' durante la sesión, como las de un canal de señales real. Algunas parecen buenas. El desafío evalúa si las validas con tu propio análisis o las copias a ciegas." (Conecta con `05_why_signals_fail.md`.) |
| 7 | **Desafío de interés compuesto a largo plazo** | Paciencia, gestión de capital en horizonte de 1–2 años | "Dos años de mercado. Pocas operaciones buenas valen más que muchas mediocres. Se evalúa el crecimiento sostenido y el costo del sobre-trading." |
| 8 | **Desafío de control de drawdown** | Limitar la caída desde el pico de la cuenta | "Tu cuenta nunca puede caer más de 10% desde su punto más alto (drawdown máximo). Tocarlo termina la sesión. Aprende a proteger lo ganado." |

Nota sobre la trampa de señales: las "señales" del desafío forman parte del **event schedule** generado con la seed fija. No son manipulación del precio post-entrada: son información sembrada de antemano, igual para todos los intentos. Algunas señales coinciden con movimientos reales del camino y otras no — exactamente como en la vida real.

> **Set de challenges del MVP (cerrado por `MVP_CONTENT_LOCK`, documento 12):** Supervivencia 50 velas · Riesgo de hierro · Sin indicadores · Paciencia · Stop obligatorio · Costos reales. La tabla anterior describe los **tipos** de desafío del producto completo; los tipos 6 (trampa de señales), 7 (interés compuesto 1–2 años) y 8 (control de drawdown como challenge rankeado) son post-MVP (ver sección 13). Criterio de éxito del MVP: completar al menos 3 de los 6.

---

## 3. Reglas de los desafíos

Reglas comunes a todo Challenge Mode:

1. **Seed fija y sellada.** Cada desafío tiene una seed inmutable. El **sello de equidad** que se muestra antes del intento es `pathHash` + `seedType` + `generatorVersion` + reglas del LCC — **nunca la seed cruda**, que se revela solo al cerrar el intento (`SEED_PATH_REPLAY_EXPORT_LOCK`, documento 08): "este mercado es el mismo para cada intento, y nadie pudo estudiarlo antes".
2. **Configuración bloqueada.** En desafíos el usuario no elige mercado, capital, dificultad ni horizonte: vienen definidos por el desafío.
3. **Reglas visibles antes de iniciar.** Todo desafío muestra: objetivo, restricciones, condiciones de fallo, categorías de puntaje y umbral de aprobación, antes del primer tick.
4. **Sin hints.** Los desafíos siempre corren sin pistas de tutorial.
5. **Restricciones específicas activas.** Ejemplos: límite de trades, riesgo máximo por operación, indicadores prohibidos, apalancamiento fijado, obligación de stop loss.
6. **Violación de regla ≠ ganancia válida.** Si el usuario viola una regla del desafío, el intento se marca como fallido aunque el resultado monetario sea positivo. Burgundy nunca enseña que el resultado justifica el proceso.
7. **Reintentos ilimitados.** El usuario puede reintentar cuantas veces quiera; cada intento queda registrado y solo el mejor puntaje se conserva como high score.
8. **Intentos completos solamente.** Abandonar a mitad de sesión registra el intento como "abandonado" (no puntúa, no penaliza el high score existente, pero sí queda en el historial para autoconocimiento).

---

## 4. Condiciones de fallo

| Condición | Aplica en | Efecto |
|---|---|---|
| **Cuenta en cero o quiebra (blow-up)** | Sandbox y desafíos | La sesión termina de inmediato. Se muestra la revisión de errores. El usuario puede reiniciar (misma seed) o empezar de nuevo (nueva seed en Sandbox). |
| **Margin call / margen insuficiente** | Sesiones con apalancamiento *(en MVP solo el escenario educativo de sobreapalancamiento con parámetros fijos — `LEVERAGE_MVP_LIMITS`; sesiones con apalancamiento general: Fase 5)* | Cierre forzoso de posiciones según el motor de órdenes; si la equity queda bajo el mínimo, equivale a quiebra. |
| **Violación de regla del desafío** | Solo desafíos | Intento marcado como fallido al instante, con explicación pedagógica de qué regla se rompió y por qué existe esa regla. |
| **Drawdown máximo tocado** | Desafíos de drawdown (y Sandbox si el usuario activa un límite voluntario) | Fin de sesión. La revisión enfatiza en qué punto el riesgo acumulado se volvió insostenible. |
| **Tiempo de simulación agotado sin objetivo** | Desafíos con objetivo (supervivencia, compuesto) | El motor de evaluación califica con lo logrado; si no se alcanzó el objetivo mínimo, el intento falla pero igual genera retroalimentación. |
| **Abandono manual** | Todos los modos | Intento "abandonado": sin puntaje, registrado en el historial. |

**Principio de fallo pedagógico:** fallar nunca es solo "Game Over". Toda condición de fallo dispara la pantalla de revisión: qué decisión inició el problema, qué error del catálogo se cometió (sin stop, sobre-apalancado, promediar en contra, copiar señal sin validar), y qué concepto del currículo lo cubre.

---

## 5. Comportamiento de reinicio de sesión

| Situación | Comportamiento |
|---|---|
| **Reiniciar tras quiebra (misma seed)** | Nueva sesión con la misma seed, mismo camino, mismo event schedule, mismo perfil de spread/slippage, capital inicial restaurado. Decision log nuevo y vacío. El intento fallido queda guardado en el historial: no se sobreescribe. |
| **Reiniciar con seed nueva (Sandbox)** | Sesión completamente independiente: nuevo camino generado, nuevo registro, sin relación de ranking con la anterior. |
| **Reintentar un desafío** | Siempre la misma seed fija del desafío. Cada intento es un registro nuevo; el high score solo se actualiza si el nuevo puntaje supera al mejor guardado. |
| **Retomar sesión guardada** | Se restaura el estado exacto: posición en el camino, posiciones abiertas, cuenta, decision log. El camino nunca se regenera (se restaura el ya sellado, verificado contra su hash). |
| **Importar archivo de sesión/progreso** | Restaura sesiones, high scores, seeds guardadas y progresión. Si la versión del generador del archivo no coincide con la instalada, las sesiones se marcan como "de versión anterior" y sus rankings se mantienen separados (ver sección 7). |

El reinicio nunca borra evidencia: la quiebra es uno de los eventos más educativos de Burgundy y su historial es parte del progreso del usuario, no una vergüenza a ocultar.

---

## 6. Lógica de high scores (puntaje más alto)

1. Cada intento completo produce un **Score total** y sus sub-puntajes (sección 8).
2. Al terminar el intento, se compara contra el mejor puntaje guardado para esa **clave de ranking** (sección 7).
3. Si lo supera, el nuevo registro pasa a ser el high score; el anterior se conserva en el historial como intento, no se destruye.
4. Empates: se conserva el más antiguo (lograrlo primero vale; repetirlo no lo reemplaza).
5. Los intentos asistidos (hints activados) o en Modo Libre con parámetros fuera de las reglas del desafío **no compiten** por high scores de desafío; pueden guardar marcas personales separadas etiquetadas como "libre/asistido".
5 bis. Los intentos con seed conocida de antemano (replay, seed guardada, sesión previa con la misma seed) se marcan `seed_known = true` y **no son elegibles para el ranking principal de primer intento** (`SEED_PATH_REPLAY_EXPORT_LOCK`, documento 08); conservan marcas personales separadas.
6. Un high score guarda el contexto completo de cómo se logró:

| Campo del registro de high score | Descripción |
|---|---|
| `challenge_id` | Identificador del desafío (o `sandbox` + seed para marcas de Sandbox) |
| `seed` | Semilla exacta usada |
| `seed_type` | `tutorial_fija`, `desafio_fija`, `sandbox_aleatoria`, `sandbox_guardada`, `replay`, `historico_inspirado` (futuro), `historico` (futuro) |
| `scenario_template` | Plantilla de escenario usada |
| `learning_context_contract` | LCC asociado (qué debía aprenderse/demostrarse) |
| `generator_version` | Versión exacta del generador (ej. `gen-1.0.0`) |
| `difficulty` | Dificultad de la sesión |
| `market` | Tipo de mercado |
| `time_horizon` | Horizonte temporal |
| `score` | Puntaje total |
| `final_account_result` | Resultado final de la cuenta (balance/equity final, P/L) |
| `discipline_score` | Sub-puntaje de disciplina |
| `risk_score` | Sub-puntaje de gestión de riesgo |
| `process_score` | Sub-puntaje de calidad de proceso |
| `survival_result` | Resultado de supervivencia (sobrevivió / quebró / drawdown tocado) |
| `date_completed` | Fecha de finalización (local del dispositivo) |

---

## 7. Estructura del ranking local

Los rankings de Burgundy son **locales y por sesión**: sin login, sin nube, sin comparación con extraños. El "rival" del usuario es su propia versión anterior.

### 7.1 Claves de ranking

Se guarda el mejor puntaje por cada combinación relevante. **En MVP solo existen las claves "por desafío" y "por seed de Sandbox guardada" (`MVP_SANDBOX_LIMITS`, documento 12); las demás son post-MVP.**

- **Por desafío** (`challenge_id`): el high score canónico de cada desafío, justo porque la seed es fija. *(MVP)*
- **Por seed de Sandbox guardada**: personal best específico de cada seed que el usuario decidió guardar y repetir. *(MVP)*
- **Por mercado**: mejor desempeño del usuario en forex, cripto y acción sintéticos. *(Post-MVP)*
- **Por horizonte temporal**: mejor intradía, mejor mes, mejor año, etc. *(Post-MVP)*
- **Por dificultad**: mejor marca en Iniciación, Estándar, Avanzada y Hostil. *(Post-MVP)*

### 7.2 Reglas estructurales

1. **Separación por versión del generador.** Si `generator_version` cambia y altera los caminos, los rankings antiguos se archivan como "temporada anterior" y los nuevos comienzan limpios. Nunca se mezclan puntajes de caminos no comparables.
2. **Tablas por categoría, no solo por dinero.** Cada clave de ranking guarda el mejor Score total y además el mejor sub-puntaje por categoría (sección 8): un usuario puede tener su mejor "sesión disciplinada" distinta de su mejor "sesión rentable" — y Burgundy muestra ambas para enseñar la diferencia.
3. **Historial de intentos acotado.** Se conservan todos los high scores y un historial razonable de intentos recientes por desafío para la revisión de progreso (el límite exacto se define en el modelo de datos, fuera de este documento).
4. **Exportable/importable.** Todo el bloque de rankings viaja dentro del archivo de progreso exportado.

---

## 8. Categorías de puntaje

El Score total se compone de cuatro categorías más el resultado de cuenta. La ganancia importa, pero nunca domina.

| Categoría | Qué mide | Ejemplos de señales que la suben | Ejemplos de señales que la bajan |
|---|---|---|---|
| **Disciplina (`discipline_score`)** | Apego a reglas propias y del desafío | Toda entrada con stop loss; respetar el plan declarado; no operar por revancha tras una pérdida | Quitar/alejar el stop con la posición en contra; sobre-trading; entrar inmediatamente después de una pérdida sin pausa |
| **Riesgo (`risk_score`)** | Calidad de la gestión de riesgo | Riesgo por trade dentro del límite; dimensionamiento coherente con el capital; risk/reward razonable declarado antes de entrar | Riesgo excesivo por trade; apalancamiento desproporcionado; promediar en contra; exposición total descontrolada |
| **Proceso (`process_score`)** | Calidad de las decisiones, independiente del resultado | Validar antes de entrar; esperar confirmación; cerrar por invalidación de la idea (no por pánico); usar la trampa de señales con análisis propio | Copiar una señal sin validación; entrar por impulso en un evento; cerrar ganadoras por miedo sin razón técnica |
| **Supervivencia (`survival_result` + componente de score)** | Mantener la cuenta viva y el drawdown controlado | Terminar la sesión con cuenta operable; drawdown máximo bajo; recuperación ordenada tras pérdidas | Quiebra; margin call; tocar el límite de drawdown |
| **Resultado de cuenta (`final_account_result`)** | P/L final | Crecimiento sostenido y consistente | Resultado negativo severo |

**Ponderación rectora (la calibración exacta se define con el motor de evaluación):** el conjunto disciplina + riesgo + proceso + supervivencia pesa más que el resultado monetario. Regla de oro verificable: *una sesión rentable con violaciones graves de riesgo debe puntuar por debajo de una sesión levemente perdedora con proceso impecable.* Esta regla es un requisito del motor de evaluación, no una sugerencia.

---

## 9. Acceso por desbloqueo (modo progresión)

En el modo de progresión, las opciones se ganan demostrando comprensión, alineado con el currículo de `02_beginner_curriculum.md`:

| Se desbloquea | Requisito orientativo |
|---|---|
| Sandbox básico (mercado inicial, intradía, sin leverage, dificultad Iniciación) | Completar el tutorial guiado fundamental |
| Mercados adicionales | Completar el módulo del currículo correspondiente a cada mercado |
| Horizontes largos (1 mes a 2 años) *(post-MVP, Fase 3+ — `MVP_SANDBOX_LIMITS`)* | Demostrar gestión de riesgo básica en horizontes cortos |
| Dificultades Avanzada y Hostil | High scores mínimos en la dificultad anterior |
| Apalancamiento (límites crecientes) *(post-MVP, Fase 5 — `LEVERAGE_MVP_LIMITS`)* | Aprobar la lección de leverage + desafío de disciplina de riesgo correspondiente |
| Desafíos de habilidad | Lección asociada completada |
| Desafíos de supervivencia y drawdown | Nivel de XP mínimo + desafío de riesgo aprobado |
| Modo sin indicadores como desafío rankeado | Sesiones de Sandbox sin indicadores completadas |

El mensaje pedagógico del desbloqueo: en Burgundy el apalancamiento y los mercados volátiles no son premios, son responsabilidades que se ganan.

---

## 10. Acceso en Modo Libre

El Modo Libre existe para usuarios que no quieren la progresión o que ya saben lo que hacen:

- Acceso inmediato a todos los mercados, horizontes y dificultades disponibles en la fase actual del producto, **sin** desbloqueos. En MVP eso significa los 3 instrumentos de `MVP_MARKET_LOCK`, los horizontes Intradía/1 semana de `MVP_SANDBOX_LIMITS` y **sin apalancamiento** (leverage 1x incluso en Modo Libre — `LEVERAGE_MVP_LIMITS`); el acceso libre al apalancamiento llega con la Fase 5.
- Activable desde la configuración, con una advertencia seria (no moralista) de que la progresión guiada existe porque el orden de aprendizaje importa.
- Las sesiones de Modo Libre se etiquetan como tales y **no compiten** en los rankings del modo progresión ni en high scores de desafíos: guardan sus propias marcas personales separadas.
- El Modo Libre no desactiva la evaluación: el usuario sigue recibiendo puntajes de disciplina, riesgo y proceso, y sus errores siguen alimentando el diario. Libertad de configuración, nunca libertad de consecuencias simuladas.
- Cambiar entre Modo Libre y progresión no destruye nada: ambos estados coexisten en el guardado local.

---

## 11. Lógica de guardado y replay de seeds

### 11.1 Uso de seed por modo

| Modo | Seed | Comportamiento |
|---|---|---|
| **Tutorial** | Fija, definida por el contenido | Contexto de aprendizaje fijo y replayable, diseñado para enseñar UN concepto con claridad. Mismo camino en cada repetición de la lección. |
| **Desafío** | Fija por desafío, sellada | Mismo camino para todos los intentos → ranking justo. Antes del intento, la prueba de equidad visible es `pathHash` + `seedType` + `generatorVersion`; la seed cruda se revela al cerrar el intento (`SEED_PATH_REPLAY_EXPORT_LOCK`, documento 08). |
| **Sandbox** | Aleatoria por defecto | El usuario puede: guardar la seed actual con nombre, reproducir una seed guardada, reiniciar la misma seed tras una quiebra, o generar una nueva. La aleatoriedad siempre pasa por el generador de regímenes de mercado: variación impredecible pero estructuralmente realista, nunca ruido puro. |
| **Replay** | La misma seed de la sesión original | Misma seed → mismo camino, mismo event schedule, mismo perfil de spread/slippage. El decision log original se muestra superpuesto para revisión, y el usuario puede operar de nuevo y comparar decisiones viejas contra nuevas. |
| **Histórico-inspirado / histórico (futuro)** | Seeds referenciales reservadas | Post-MVP. El formato de seed y el campo `seed_type` ya los contemplan para no romper compatibilidad. |

### 11.2 Reglas de guardado de seed

1. Una seed guardada almacena: seed, plantilla de escenario, parámetros (mercado, dificultad, horizonte, capital), `generator_version` y el hash del camino generado.
2. Reproducir una seed guardada con la **misma** `generator_version` garantiza el mismo camino, verificable contra el hash.
3. Si la `generator_version` instalada cambió, la app lo informa: la seed puede regenerarse con la versión nueva (camino distinto, marcas separadas) y el personal best antiguo queda archivado con su versión.
4. Repetir una misma seed crea una línea de **mejora personal**: la app muestra la evolución de los sub-puntajes intento a intento sobre el mismo mercado — la forma más honesta de medir progreso, porque elimina la suerte del camino.
5. Generar una seed nueva crea una sesión completamente independiente, sin vínculo de ranking con las anteriores.
6. Las seeds guardadas se incluyen en el archivo de exportación de progreso.

---

## 12. Qué pertenece al MVP

| Bloque | Contenido MVP |
|---|---|
| **Sandbox** | Versión mínima según `MVP_SANDBOX_LIMITS` (documento 12): los 3 instrumentos de `MVP_MARKET_LOCK`, capital, dificultad, **horizontes Intradía y 1 semana**, velocidad, indicadores on/off, hints on/off, **sin leverage**; seed aleatoria, guardar/reproducir/reiniciar seed; historial de trades; export/import del progreso completo (sin export por sesión ni guardar/retomar sesión larga). |
| **Desafíos** | El set canónico de 6 de `MVP_CONTENT_LOCK`: Supervivencia 50 velas, Riesgo de hierro, Sin indicadores, Paciencia, Stop obligatorio y Costos reales — todos con seed fija. |
| **Rankings y scores** | High score local por desafío y personal best por seed guardada; registro completo de high score (tabla de la sección 6); separación por `generator_version`. Claves por mercado, horizonte y dificultad: post-MVP. |
| **Puntaje** | Las cuatro categorías (disciplina, riesgo, proceso, supervivencia) + resultado de cuenta, con la regla de oro de ponderación. |
| **Fallo y reinicio** | Quiebra, margin call, violación de reglas, drawdown (en su desafío), revisión pedagógica post-fallo, reinicio misma seed / nueva seed. |
| **Replay** | Replay de sesión con decision log visible y comparación básica de decisiones. |
| **Modos de acceso** | Progresión por desbloqueo + Modo Libre con marcas separadas. |

## 13. Qué pertenece a fases posteriores

| Fase posterior | Contenido |
|---|---|
| **Sandbox completo** | Los 7 horizontes (1 mes a 2 años), guardar y retomar sesión, export/import por sesión individual, índice sintético como mercado, leverage por desbloqueo (Fase 5). Movido fuera del MVP por `MVP_SANDBOX_LIMITS`, `MVP_MARKET_LOCK` y `LEVERAGE_MVP_LIMITS`. |
| **Rankings ampliados** | Claves de ranking por mercado, por horizonte temporal y por dificultad. |
| **Desafíos avanzados** | Trampa de copiado de señales (requiere el sistema de señales sembradas del event schedule maduro), interés compuesto a largo plazo y control de drawdown como desafíos rankeados completos. |
| **Replay avanzado** | Comparación lado a lado enriquecida (métricas diferenciales por decisión, "qué hubiera pasado si" sobre el mismo camino sellado). |
| **Temporadas locales** | Archivado automático de rankings por temporada cuando cambia la versión del generador, con vista de temporadas pasadas. |
| **Seeds histórico-inspiradas** | Escenarios inspirados en episodios reales de mercado, generados sintéticamente con seeds referenciales. |
| **Modo histórico** | Replay de datos históricos reales como fuente de datos adicional (la arquitectura de formato universal de vela ya lo soporta; nunca es requisito del MVP). |
| **Estadísticas longitudinales** | Tendencias de sub-puntajes a través de meses de práctica, mapa de errores recurrentes cruzado con el catálogo de errores. |
| **Desafíos comunitarios por seed compartida** | Compartir una seed + parámetros + versión por archivo/código para que otra persona juegue exactamente el mismo mercado en su propio dispositivo — competencia sin nube y sin login. |

---

*Documento 06 de la serie de arquitectura de Burgundy — firmado por tsuloid. Continúa la serie: 01 definición de producto, 02 currículo, 03 modelo de simulación, 04 cobertura de mercados, 05 por qué fallan las señales.*
