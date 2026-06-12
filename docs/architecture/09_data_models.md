# Burgundy — Bloque 9: Modelos de Datos y Persistencia Local

**Proyecto:** Burgundy — Simulador educativo de trading
**Autor / firma:** tsuloid
**Idioma:** Español (LATAM)
**Estado:** Documento de arquitectura — sin código, sin migraciones, sin pantallas, previo al desarrollo
**Plataformas objetivo:** Android  15+ / iOS 20+ · Offline-first · Sin login · Sin cuenta en la nube

---

## 0. Principios de diseño de datos

Antes de los modelos, las reglas que gobiernan todo el diseño:

1. **El path de mercado se genera antes de la acción del usuario y es inmutable.** Ningún dato del usuario (órdenes, posiciones, decisiones) puede modificar un `GeneratedMarketPath`. Las decisiones del usuario viven en modelos separados que *referencian* el path, nunca lo alteran.
2. **Determinismo como contrato de datos.** Todo lo reproducible se identifica por `seed + generatorVersion + scenarioTemplateId`. Un mismo trío produce siempre el mismo path; el `pathHash` lo verifica.
3. **Formato universal de vela, agnóstico de fuente.** Una vela sintética, procedural o histórica futura tiene exactamente la misma forma. El motor nunca pregunta "¿de dónde vino esta vela?" para simular; solo lee `sourceType` para fines educativos y de etiquetado.
4. **Local-only, sin login.** No existen IDs de usuario remotos, tokens ni sincronización. Todos los IDs son UUID generados en el dispositivo.
5. **Exportable por diseño.** Cada modelo declara si entra en el archivo de export. El archivo exportado es un snapshot versionado, verificable por checksum, suficiente para reconstruir el progreso en otro dispositivo.
6. **Histórico-ready, no histórico-dependiente.** Los modelos para datos históricos futuros (`HistoricalSourceMetadata`, `HistoricalPatternTemplate`) se definen ahora para no romper el esquema después, pero **no forman parte del MVP**.

Convenciones usadas en todas las tablas:

- **MVP:** ✅ obligatorio en MVP · ⚠️ parcial/simplificado en MVP · ❌ post-MVP.
- **Export:** ✅ se incluye en el archivo de progreso exportado · ⚠️ se incluye parcialmente · ❌ nunca se exporta (se regenera o es estático del bundle de la app).
- Todos los modelos llevan implícitos: `id` (UUID), `createdAt`, `updatedAt`, `schemaVersion` cuando aplica.
- "Persistencia" indica dónde vive: **SQLite** (datos estructurados), **MMKV** (preferencias ligeras), **Bundle** (contenido estático que viaja con la app, no se escribe), **Derivado** (se calcula, no se guarda).

---

## 1. Catálogo de modelos

### 1.1 UserProfile (Perfil del usuario)

| Aspecto | Detalle |
|---|---|
| **Propósito** | Representar a la persona que aprende: su nombre visible, su progreso global de gamificación y sus preferencias de aprendizaje. Sin login: existe exactamente un `UserProfile` por instalación. |
| **Campos requeridos** | `id`, `displayName` (alias local, editable), `createdAt`, `xpTotal`, `level`, `streakDays`, `streakLastActiveDate`, `schemaVersion` |
| **Campos opcionales** | `avatarPreset` (de un set local, sin fotos), `preferredAccountSize` (tamaño de cuenta de práctica preferido), `onboardingCompletedAt`, `selfReportedExperience` (`ninguna` / `algo` / `he_perdido_dinero`) |
| **Relaciones** | 1→1 con `LocalAppProfile`; 1→N con `SimulationSession`, `TutorialProgress`, `JournalEntry`, `Achievement`, `RankingEntry` |
| **Persistencia** | SQLite. Se crea en el primer arranque. Nunca se borra salvo reset explícito del usuario. |
| **MVP** | ✅ |
| **Export** | ✅ (es el corazón del archivo de progreso) |

### 1.2 LocalAppProfile (Perfil local de la app)

| Aspecto | Detalle |
|---|---|
| **Propósito** | Estado técnico de la instalación, separado del progreso humano: versiones, preferencias de UI, flags de dispositivo. Permite exportar progreso sin arrastrar basura técnica de un dispositivo a otro. |
| **Campos requeridos** | `id`, `appVersion`, `engineVersion`, `dbSchemaVersion`, `installId` (UUID de instalación), `lastAutoSaveAt`, `lastBackupAt` |
| **Campos opcionales** | `uiPreferences` (velocidad de playback por defecto, mostrar/ocultar tooltips, tamaño de fuente), `hapticFeedbackEnabled`, `lastImportAt`, `lastExportAt` |
| **Relaciones** | 1→1 con `UserProfile` |
| **Persistencia** | MMKV (preferencias) + SQLite (versiones y marcas de tiempo de backup). |
| **MVP** | ✅ |
| **Export** | ⚠️ Solo `engineVersion` y `dbSchemaVersion` viajan en el export (como metadatos de compatibilidad). Las preferencias de UI **no** se exportan: son del dispositivo, no del progreso. |

### 1.3 Market (Mercado)

| Aspecto | Detalle |
|---|---|
| **Propósito** | Definir un tipo de mercado simulado (forex, cripto, índices, acciones) con sus características educativas: horarios típicos, comportamiento de spread, volatilidad característica. Es contenido estático, no datos del usuario. |
| **Campos requeridos** | `id`, `code` (`forex`, `crypto`, `indices`, `stocks`), `displayNameEs`, `descriptionEs` (explicación LATAM clara), `tradingHoursModel` (`continuo_24_7`, `sesiones`, `horario_bolsa`), `defaultSpreadProfile`, `defaultVolatilityClass` |
| **Campos opcionales** | `educationalNotesEs` (peculiaridades que el alumno debe conocer: gaps de fin de semana en forex, volatilidad extrema en cripto), `unlockRequirement` (nivel/logro necesario para desbloquearlo) |
| **Relaciones** | 1→N con `Instrument`, `ScenarioTemplate` |
| **Persistencia** | Bundle (JSON estático versionado con la app). No se escribe en SQLite. |
| **MVP** | ✅ (al menos forex y cripto según Bloque 4) |
| **Export** | ❌ Se referencia por `code`; el contenido viaja con la app. |

### 1.4 Instrument (Instrumento)

| Aspecto | Detalle |
|---|---|
| **Propósito** | Un instrumento simulado concreto dentro de un mercado (ej. "EUR/USD simulado", "BTC/USD simulado"). Define la aritmética de la simulación: tamaño de pip/tick, decimales, tamaño de contrato, apalancamiento máximo permitido en el simulador. |
| **Campos requeridos** | `id`, `marketId`, `symbol` (ej. `EURUSD.sim`), `displayNameEs`, `pipOrTickSize`, `priceDecimals`, `contractSize`, `minLotSize`, `maxLotSize`, `maxLeverage`, `baseSpreadRange`, `feeModel` (comisión fija, por lote, o spread-only) |
| **Campos opcionales** | `slippageProfileDefault`, `liquidityClass` (`alta`, `media`, `baja`), `educationalNotesEs`, `unlockRequirement` |
| **Relaciones** | N→1 con `Market`; referenciado por `ScenarioTemplate`, `SeedRecord`, `SimulationSession`, `Order`, `Trade`, `Position` |
| **Persistencia** | Bundle. |
| **MVP** | ✅ (2–4 instrumentos bastan para el MVP) |
| **Export** | ❌ Se referencia por `symbol` + versión de catálogo. |

### 1.5 Candle (Vela) — Formato Universal de Vela

| Aspecto | Detalle |
|---|---|
| **Propósito** | La unidad atómica de mercado en Burgundy. Un único formato para todas las fuentes presentes y futuras: el motor de simulación, el chart, el replay y el futuro modo histórico consumen exactamente esta estructura. |
| **Campos requeridos** | `timestamp` (tiempo simulado, época ms), `open`, `high`, `low`, `close`, `volume`, `bid`, `ask`, `spread`, `sourceType` |
| **Campos opcionales** | `volatilityTag` (`baja`, `normal`, `alta`, `extrema`), `liquidityTag` (`alta`, `media`, `baja`, `seca`), `regimeTag` (`tendencia_alcista`, `tendencia_bajista`, `rango`, `expansion`, `compresion`, `news_spike`), `eventTag` (referencia a un evento del path: noticia simulada, gap, barrida de liquidez) |
| **Valores de `sourceType`** | `synthetic` · `procedural` · `fixed_tutorial` · `fixed_challenge` · `sandbox_seed` · `replay` · `historical` · `historical_inspired` |
| **Relaciones** | Pertenece a un `GeneratedMarketPath` (array ordenado por `timestamp`). No existe como fila individual en SQLite. |
| **Persistencia** | Derivado/embebido: las velas viven serializadas (y comprimidas) dentro del `GeneratedMarketPath`, o se regeneran desde el seed. Nunca una tabla `candles` con millones de filas. |
| **MVP** | ✅ |
| **Export** | ⚠️ Solo dentro de paths que deban exportarse (ver §13.11–§13.12). |

Notas de diseño del formato universal:

- `bid`/`ask`/`spread` son obligatorios aunque parezcan redundantes (`spread = ask − bid`): hacen el costo de transacción **visible y enseñable** en cada vela, que es un objetivo pedagógico central (el alumno LATAM típico ignora el spread).
- Las etiquetas (`volatilityTag`, `regimeTag`, etc.) son **metadatos educativos**, no insumos del motor de ejecución. Permiten que el feedback diga "entraste en compresión de volatilidad justo antes de la expansión" sin recalcular nada.
- Para datos históricos futuros sin bid/ask reales, el importador deberá sintetizar `bid`/`ask` desde un perfil de spread del instrumento — el formato no cambia.

### 1.6 TickEvent (Tick / micro-movimiento)

| Aspecto | Detalle |
|---|---|
| **Propósito** | Movimiento intra-vela para ejecución realista: permite que un stop loss se ejecute dentro de la vela (no solo en el close), y que el slippage tenga una base concreta. En MVP se usa una aproximación determinista (sub-pasos por vela), no ticks reales. |
| **Campos requeridos** | `timestamp`, `price`, `bid`, `ask`, `parentCandleIndex`, `sequenceInCandle` |
| **Campos opcionales** | `volumeDelta`, `eventTag` (spike, gap interno), `liquidityTag` |
| **Relaciones** | Pertenece a un `GeneratedMarketPath` (campo `ticksOrMicroMoves`); referencia su vela madre. |
| **Persistencia** | Derivado: en MVP los micro-movimientos se **regeneran deterministamente** desde el seed cuando se necesita ejecutar dentro de una vela; no se almacenan. Post-MVP podrán materializarse para paths históricos. |
| **MVP** | ⚠️ (aproximación determinista de 4–8 sub-pasos por vela; sin almacenamiento) |
| **Export** | ❌ (se regenera del seed) |

### 1.7 Order (Orden)

| Aspecto | Detalle |
|---|---|
| **Propósito** | Una instrucción del usuario al mercado simulado: market, limit, stop, con su stop loss y take profit. Es la "intención" del usuario; el motor la valida y ejecuta contra el path inmutable. |
| **Campos requeridos** | `id`, `sessionId`, `instrumentSymbol`, `type` (`market`, `limit`, `stop`), `side` (`buy`, `sell`), `size` (lotes/unidades), `status` (`pendiente`, `ejecutada`, `cancelada`, `rechazada`, `expirada`), `createdAtSimTime` (tiempo simulado), `requestedPrice` (null en market) |
| **Campos opcionales** | `stopLossPrice`, `takeProfitPrice`, `executedPrice`, `executedAtSimTime`, `slippageApplied`, `feesApplied`, `rejectionReason` (ej. margen insuficiente — momento educativo), `linkedPositionId` |
| **Relaciones** | N→1 con `SimulationSession`; 1→0..1 con `Trade` (al ejecutarse); referenciada desde `UserDecisionLog` |
| **Persistencia** | SQLite, dentro del registro de la sesión. Inmutable una vez cerrada la sesión. |
| **MVP** | ✅ |
| **Export** | ✅ (parte de la historia de la sesión; necesaria para journal y replay de decisiones) |

### 1.8 Trade (Operación ejecutada)

| Aspecto | Detalle |
|---|---|
| **Propósito** | El registro consumado de una ejecución: precio real obtenido (con spread y slippage), comisiones, y — al cerrar — el resultado completo. Es la unidad sobre la que se construyen journal, evaluación y métricas. |
| **Campos requeridos** | `id`, `sessionId`, `instrumentSymbol`, `side`, `size`, `entryPrice`, `entrySimTime`, `entryOrderId`, `status` (`abierta`, `cerrada`), `feesTotal` |
| **Campos opcionales** | `exitPrice`, `exitSimTime`, `exitReason` (`take_profit`, `stop_loss`, `cierre_manual`, `margin_call`, `fin_de_sesion`), `realizedPnl`, `realizedPnlPercent`, `riskRewardPlanned`, `riskRewardRealized`, `maxAdverseExcursion` (cuánto fue en contra antes de resolverse — oro educativo), `maxFavorableExcursion`, `holdingCandles`, `mistakeIds[]`, `journalEntryId` |
| **Relaciones** | N→1 con `SimulationSession`; 1→1 con `Position` durante su vida; N→N con `Mistake` vía `mistakeIds`; 0..1 con `JournalEntry` |
| **Persistencia** | SQLite. Inmutable tras el cierre de la sesión. |
| **MVP** | ✅ |
| **Export** | ✅ |

### 1.9 Position (Posición)

| Aspecto | Detalle |
|---|---|
| **Propósito** | El estado vivo de una exposición abierta durante la simulación: P/L flotante, margen usado, distancia al stop. Existe para que el HUD muestre riesgo en tiempo real. |
| **Campos requeridos** | `id`, `sessionId`, `tradeId`, `instrumentSymbol`, `side`, `size`, `entryPrice`, `currentStopLoss`, `currentTakeProfit`, `unrealizedPnl`, `marginUsed`, `riskAmountCurrent` (dinero en riesgo hasta el stop) |
| **Campos opcionales** | `stopMoveHistory[]` (cada movimiento de SL/TP con su sim-time — detecta el error clásico de "mover el stop para no perder"), `partialCloses[]` |
| **Relaciones** | 1→1 con `Trade`; N→1 con `Account` de la sesión |
| **Persistencia** | **Derivado en runtime** (estado del motor). Solo se materializa en SQLite dentro del snapshot de auto-save de una sesión en curso; al cerrar la sesión, lo permanente queda en `Trade` (incluido `stopMoveHistory`, que se copia al trade). |
| **MVP** | ✅ |
| **Export** | ⚠️ Solo si hay una sesión en curso incluida en el export (snapshot); las posiciones cerradas viajan como `Trade`. |

### 1.10 Account (Cuenta de la sesión)

| Aspecto | Detalle |
|---|---|
| **Propósito** | El estado financiero simulado dentro de una sesión: balance, equity, margen, drawdown. Cada sesión tiene su propia cuenta — no existe una "cuenta global", porque Burgundy enseña por sesiones/escenarios, no simula una carrera de cuenta única. |
| **Campos requeridos** | `id`, `sessionId`, `initialBalance`, `currency` (simulada, USD por defecto), `balance`, `equity`, `marginUsed`, `freeMargin`, `peakEquity`, `maxDrawdownPercent`, `leverageAllowed` |
| **Campos opcionales** | `marginCallTriggeredAtSimTime`, `equityCurve[]` (muestreada por vela, para el gráfico post-sesión), `feesPaidTotal` |
| **Relaciones** | 1→1 con `SimulationSession`; 1→N con `Position` |
| **Persistencia** | Estado vivo en runtime; snapshot en auto-save; resultado final consolidado en SQLite al cerrar la sesión (incluida la `equityCurve` comprimida). |
| **MVP** | ✅ |
| **Export** | ✅ (el resultado final; la curva de equity puede exportarse decimada para reducir tamaño) |

### 1.11 SimulationSession (Sesión de simulación)

| Aspecto | Detalle |
|---|---|
| **Propósito** | El modelo agregador central: una partida completa de práctica (tutorial, sandbox, challenge o replay), con todo lo que pasó en ella. Es la unidad de ranking, de replay y de revisión de errores. |
| **Campos requeridos** | `id`, `mode` (`tutorial`, `sandbox`, `challenge`, `replay`, `libre`), `seedRecordId`, `instrumentSymbol`, `status` (`en_curso`, `completada`, `abandonada`, `fallida`), `startedAt` (tiempo real), `accountId`, `playbackConfig` (timeframe, velocidad inicial), `indicatorsEnabled` (false en modo sin indicadores), `engineVersionAtPlay` |
| **Campos opcionales** | `lessonId` (si es tutorial), `challengeId` (si es challenge), `learningContextContractId`, `completedAt`, `finalScore`, `scoreBreakdown` (disciplina, riesgo, proceso, resultado), `evaluationId`, `replayOfSessionId` (si es replay de otra sesión), `currentCandleIndex` (cursor de auto-save de sesión en curso), `durationRealSeconds` |
| **Relaciones** | N→1 con `SeedRecord`; 1→1 con `Account`; 1→N con `Order`, `Trade`, `UserDecisionLog`; 0..1 con `Evaluation`, `Lesson`, `Challenge`, `LearningContextContract`; 0..N con `RankingEntry`, `JournalEntry` |
| **Persistencia** | SQLite. Las sesiones `en_curso` se auto-guardan (§13.2); las completadas son inmutables. Política de retención: ver §13.9. |
| **MVP** | ✅ |
| **Export** | ✅ (completas: con órdenes, trades, decision log y evaluación) |

### 1.12 Lesson (Lección)

| Aspecto | Detalle |
|---|---|
| **Propósito** | Una unidad del currículo de la academia (Bloque 2): concepto, explicación en español LATAM claro, glosario asociado, y los escenarios prácticos que la acompañan. Contenido estático. |
| **Campos requeridos** | `id`, `moduleId`, `orderInModule`, `titleEs`, `objectiveEs`, `bodyContentRef` (referencia al contenido en el bundle), `glossaryTerms[]` (término en inglés + explicación en español), `learningContextContractId`, `scenarioTemplateIds[]`, `xpReward` |
| **Campos opcionales** | `prerequisiteLessonIds[]`, `estimatedMinutes`, `quizRef`, `unlocksAchievementId` |
| **Relaciones** | N→1 con módulo del currículo; 1→1 con `LearningContextContract`; 1→N con `ScenarioTemplate`; seguida por `TutorialProgress` |
| **Persistencia** | Bundle (contenido versionado con la app). |
| **MVP** | ✅ |
| **Export** | ❌ El contenido no se exporta; el **progreso** sobre las lecciones sí (vía `TutorialProgress`). |

### 1.13 TutorialProgress (Progreso de tutorial)

| Aspecto | Detalle |
|---|---|
| **Propósito** | El estado del usuario sobre cada lección: no iniciada, en curso, completada, dominada. Es lo que hace que la academia recuerde dónde quedó el alumno. |
| **Campos requeridos** | `id`, `lessonId`, `status` (`bloqueada`, `disponible`, `en_curso`, `completada`, `dominada`), `attempts`, `bestScore`, `firstCompletedAt`, `lastAttemptAt` |
| **Campos opcionales** | `quizResults[]`, `linkedSessionIds[]`, `conceptReviewDueAt` (repaso espaciado futuro) |
| **Relaciones** | N→1 con `UserProfile`; referencia `Lesson` por id; N→N con `SimulationSession` |
| **Persistencia** | SQLite. |
| **MVP** | ✅ |
| **Export** | ✅ |

### 1.14 LearningContextContract (Contrato de contexto de aprendizaje)

El modelo pedagógico central de Burgundy: define **qué debe enseñar un escenario y cómo se evalúa**, separado de cómo se genera el mercado. Es el "contrato" entre el currículo y el motor.

| Campo | Tipo / valores | Descripción |
|---|---|---|
| `id` | UUID/slug | Identificador estable (ej. `lcc.stop_loss_basico.v1`) |
| `lessonObjective` | texto ES | Qué debe aprender el alumno (ej. "colocar el stop loss según estructura, no según dolor") |
| `scenarioType` | enum | `tendencia`, `rango`, `trampa_de_ruptura`, `news_spike`, `compresion_expansion`, `barrida_de_stops`, etc. |
| `requiredRegimes` | array | Regímenes que el path generado **debe** contener, en orden o no (ej. `[rango, ruptura_falsa, tendencia_bajista]`) |
| `expectedTraps` | array | Trampas que el escenario presenta a propósito (ruptura falsa, spike de noticia, mecha de barrida) |
| `validGoodDecisions` | array | Decisiones que cuentan como correctas **por proceso** aunque el trade pierda (no operar, esperar confirmación, riesgo ≤ X%, respetar el stop) |
| `commonMistakes` | array de `mistakeCode` | Errores que este escenario suele provocar y que el detector debe vigilar |
| `feedbackRules` | array de reglas | `condición → mensaje educativo ES` (ej. "movió SL en contra → explicar por qué eso convierte una pérdida pequeña en una grande") |
| `successCriteria` | objeto | Qué constituye éxito (criterios de proceso, no solo P/L: ej. "sobrevivir con drawdown < 10% y 0 entradas sin stop") |
| `failureCriteria` | objeto | Qué constituye fallo (margin call, > N errores críticos, violar la regla de la lección) |
| `scoringFocus` | pesos | Distribución del score: `disciplina`, `gestionRiesgo`, `calidadProceso`, `resultado` (el resultado pesa poco a propósito) |
| `seedBehavior` | enum | `fixed` (tutorial/challenge: mismo seed para todos), `random_constrained` (sandbox: seed aleatorio que cumpla `requiredRegimes`), `from_pool` (pool curado de seeds validados) |
| `replayBehavior` | enum | `replay_exacto` (mismo path, se permite rejugar), `replay_bloqueado` (challenges rankeados: una vez visto, no puntúa de nuevo), `replay_educativo` (replay con anotaciones de errores superpuestas) |

| Aspecto | Detalle |
|---|---|
| **Relaciones** | 1→N con `Lesson`, `Challenge`, `ScenarioTemplate`, `SeedRecord`, `SimulationSession`, `Evaluation` |
| **Persistencia** | Bundle (contenido curricular versionado). |
| **MVP** | ✅ (es la pieza que materializa la filosofía "forzar contexto de aprendizaje, no resultado") |
| **Export** | ❌ Se referencia por `id` + versión. |

### 1.15 Scenario (Escenario instanciado)

| Aspecto | Detalle |
|---|---|
| **Propósito** | La instancia concreta de un `ScenarioTemplate` para una sesión: plantilla + seed + parámetros resueltos. Es el puente entre "plantilla abstracta" y "path generado". |
| **Campos requeridos** | `id`, `scenarioTemplateId`, `seedRecordId`, `resolvedParameters` (los rangos de la plantilla ya colapsados a valores concretos por el seed), `instrumentSymbol`, `timeframe`, `candleCount` |
| **Campos opcionales** | `learningContextContractId`, `difficultyResolved`, `briefingTextEs` (contexto narrativo mostrado antes de empezar) |
| **Relaciones** | N→1 con `ScenarioTemplate` y `SeedRecord`; 1→1 con `GeneratedMarketPath`; 1→N con `SimulationSession` |
| **Persistencia** | SQLite (ligero; los parámetros resueltos son pequeños). |
| **MVP** | ✅ |
| **Export** | ✅ (necesario para replay fiel) |

### 1.16 ScenarioTemplate (Plantilla de escenario)

| Campo | Tipo / valores | Descripción |
|---|---|---|
| `id` | UUID/slug | Ej. `tpl.rango_con_ruptura_falsa.v2` |
| `name` | texto ES | Nombre interno/curricular |
| `marketType` | enum | `forex`, `crypto`, `indices`, `stocks` |
| `regimeSequence` | array | Secuencia (o gramática) de regímenes que el generador debe producir: `[compresion, ruptura_falsa, tendencia]` |
| `parameterRanges` | objeto | Rangos por parámetro: volatilidad base, duración de cada régimen, amplitud del rango, agresividad del spike, perfil de spread/slippage |
| `trapTypes` | array | Trampas habilitadas: `ruptura_falsa`, `barrida_de_liquidez`, `spike_noticia`, `gap` |
| `difficulty` | enum | `intro`, `básico`, `intermedio`, `avanzado`, `supervivencia` |
| `lessonTags` | array | Conceptos curriculares que cubre (`stop_loss`, `riesgo_por_operacion`, `spread`, `paciencia`) |
| `supportedModes` | array | `tutorial`, `sandbox`, `challenge`, `replay`, `libre` |
| `historicalReady` | boolean | Si la plantilla acepta ser instanciada desde un `HistoricalPatternTemplate` futuro |

| Aspecto | Detalle |
|---|---|
| **Relaciones** | N→1 con `Market`; 1→N con `Scenario`, `SeedRecord`; 0..N con `LearningContextContract`; futura 1→N desde `HistoricalPatternTemplate` |
| **Persistencia** | Bundle. |
| **MVP** | ✅ |
| **Export** | ❌ Se referencia por `id` + versión de catálogo. |

### 1.17 SeedRecord (Registro de seed)

El modelo más crítico para reproducibilidad. **Un seed sin su contexto no reproduce nada**: por eso el registro guarda todo lo necesario para regenerar el path bit a bit.

| Campo | Tipo | Descripción |
|---|---|---|
| `seed` | entero/string | La semilla del PRNG |
| `seedType` | enum | `fixed_tutorial`, `fixed_challenge`, `sandbox_random`, `replay`, `historical_inspired` (futuro), `historical_ref` (futuro) |
| `generatorVersion` | semver | Versión exacta del generador del motor (`@burgundy/engine`) que produjo/produce el path |
| `scenarioTemplateId` | ref + versión | Plantilla usada |
| `learningContextContractId` | ref + versión | Contrato pedagógico asociado (puede ser null en modo libre) |
| `marketType` | enum | Mercado |
| `difficulty` | enum | Dificultad resuelta |
| `timeHorizon` | objeto | Timeframe + número de velas |
| `createdAt` | timestamp | Creación |
| `pathHash` | hash (SHA-256) | Hash canónico del path generado: verifica que una regeneración es idéntica |
| `replayable` | boolean | Si el usuario puede rejugarlo |
| `sourceType` | enum | Mismo vocabulario que las velas |

| Aspecto | Detalle |
|---|---|
| **Relaciones** | 1→0..1 con `GeneratedMarketPath` (materializado o no); 1→N con `Scenario`, `SimulationSession` |
| **Persistencia** | SQLite. Los `SeedRecord` son **permanentes**: pesan bytes y son la garantía de replay. Nunca se purgan mientras exista una sesión que los referencie. |
| **MVP** | ✅ |
| **Export** | ✅ **Siempre.** Es la pieza mínima irrenunciable del export (ver §13.11). |

### 1.18 GeneratedMarketPath (Path de mercado generado)

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | UUID | Identificador |
| `seedRecordId` | ref | Seed que lo produjo |
| `candles` | array de `Candle` | El path completo, en formato universal, serializado y comprimido |
| `ticksOrMicroMoves` | array opcional | Solo si se materializan (MVP: no; se regeneran) |
| `events` | array | Eventos programados del path: noticias simuladas, gaps, cambios de liquidez, con su índice de vela |
| `spreadProfile` | objeto | Perfil de spread a lo largo del path (base + ensanchamientos por evento) |
| `slippageProfile` | objeto | Modelo de slippage aplicable (por volatilidad/liquidez) |
| `liquidityProfile` | objeto | Perfil de liquidez por tramo |
| `generatedAt` | timestamp | Cuándo se materializó |
| `hash` | SHA-256 | Debe coincidir con `SeedRecord.pathHash` |
| `sourceType` | enum | Vocabulario universal |

| Aspecto | Detalle |
|---|---|
| **Propósito** | La materialización inmutable del mercado de una sesión. Se genera **completo antes** de que el usuario actúe. |
| **Relaciones** | 1→1 con `SeedRecord`; consumido por `SimulationSession` |
| **Persistencia** | **Caché regenerable** en SQLite (blob comprimido). Regla MVP: se conserva mientras la sesión esté `en_curso` o sea reciente; después puede purgarse porque el `SeedRecord` permite regenerarlo. Excepción: paths cuyo `generatorVersion` ya no coincide con el motor instalado se conservan congelados (ver §13.13). |
| **MVP** | ✅ |
| **Export** | ⚠️ Solo en los casos de §13.11; por defecto se exporta el `SeedRecord`, no el path. |

### 1.19 Signal (Señal)

| Aspecto | Detalle |
|---|---|
| **Propósito** | Una "señal" simulada presentada al usuario como ejercicio educativo (Bloque 5: por qué las señales de Telegram/TikTok fallan). El alumno aprende a evaluarlas críticamente; la señal puede ser deliberadamente mala. |
| **Campos requeridos** | `id`, `sessionId` o `scenarioId`, `appearsAtCandleIndex`, `direction`, `claimedReasonEs` (la justificación que daría el "gurú"), `qualityClass` (`válida`, `incompleta`, `contradictoria`, `trampa`), `hiddenFlawEs` (el defecto que el alumno debería detectar) |
| **Campos opcionales** | `suggestedEntry`, `suggestedStopLoss` (a menudo ausente, a propósito), `suggestedTakeProfit`, `userResponse` (`siguió`, `ignoró`, `evaluó_y_rechazó`), `outcomeIfFollowed` |
| **Relaciones** | N→1 con `Scenario`/`SimulationSession`; referenciada en `UserDecisionLog` y `Evaluation` |
| **Persistencia** | Definición en Bundle (parte del escenario); la respuesta del usuario en SQLite. |
| **MVP** | ⚠️ (al menos en las lecciones del módulo de señales; sistema completo post-MVP) |
| **Export** | ✅ La respuesta del usuario; ❌ la definición. |

### 1.20 Mistake (Error catalogado)

| Aspecto | Detalle |
|---|---|
| **Propósito** | El catálogo de errores de trading que Burgundy detecta y enseña a corregir. Dos caras: la **definición** (estática) y la **ocurrencia** (instancia detectada en una sesión). |
| **Definición — campos** | `mistakeCode` (ej. `entrada_sin_stop`, `riesgo_excesivo`, `mover_stop_en_contra`, `sobreoperar`, `promediar_perdida`, `entrar_por_fomo`, `apalancamiento_excesivo`, `confundir_suerte_con_habilidad`), `titleEs`, `explanationEs`, `whyItHurtsEs`, `howToAvoidEs`, `severity` (`leve`, `grave`, `crítico`), `detectionRule` |
| **Ocurrencia — campos** | `id`, `sessionId`, `mistakeCode`, `detectedAtSimTime`, `relatedTradeId` u `orderId`, `contextSnapshot` (riesgo, drawdown y estado al momento), `acknowledgedByUser` |
| **Relaciones** | Ocurrencias: N→1 con `SimulationSession`; N→1 con `Trade`; agregadas por `Evaluation` y revisadas en Mistake Review |
| **Persistencia** | Definiciones en Bundle; ocurrencias en SQLite. |
| **MVP** | ✅ (catálogo inicial de 10–15 errores núcleo) |
| **Export** | ✅ Ocurrencias; ❌ definiciones. |

### 1.21 UserDecisionLog (Bitácora de decisiones del usuario)

El registro granular de **todo lo que el usuario hizo y cuándo**, en tiempo simulado. Es la materia prima del replay educativo, la evaluación y el mistake review.

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | UUID | Identificador |
| `sessionId` | ref | Sesión |
| `simulationTimestamp` | tiempo simulado + índice de vela | Cuándo (en el mercado simulado) ocurrió la acción |
| `actionType` | enum | `abrir_orden`, `cancelar_orden`, `modificar_sl`, `modificar_tp`, `cerrar_posicion`, `cierre_parcial`, `pausar`, `cambiar_velocidad`, `saltar_adelante`, `consultar_glosario`, `ignorar_señal`, `seguir_señal`, `no_operar_deliberado` |
| `priceAtAction` | precio (bid/ask) | Precio vigente al actuar |
| `orderDetails` | objeto opcional | Snapshot de la orden involucrada |
| `riskAtAction` | objeto | Riesgo abierto, % de cuenta en riesgo, drawdown actual al momento de actuar |
| `emotionalTag` | enum opcional | Auto-reporte del usuario: `confiado`, `ansioso`, `frustrado`, `aburrido`, `eufórico`, `con_miedo` (opt-in, nunca obligatorio) |
| `mistakeTag` | ref opcional | `mistakeCode` si el detector vinculó la acción a un error |
| `notes` | texto opcional | Nota libre del usuario |

| Aspecto | Detalle |
|---|---|
| **Relaciones** | N→1 con `SimulationSession`; referencias opcionales a `Order`, `Trade`, `Signal`, `Mistake` |
| **Persistencia** | SQLite, append-only durante la sesión (nunca se edita una entrada pasada). Volumen esperado: decenas a pocas centenas de filas por sesión — trivial. |
| **MVP** | ✅ (los `actionType` de trading; los de navegación como `consultar_glosario` son ⚠️ opcionales) |
| **Export** | ✅ (sin él, el replay educativo y el journal pierden su valor) |

### 1.22 Evaluation (Evaluación)

| Aspecto | Detalle |
|---|---|
| **Propósito** | El veredicto educativo post-sesión: score desglosado según el `scoringFocus` del contrato, errores detectados, decisiones buenas reconocidas, y feedback en español claro. Premia proceso, no resultado. |
| **Campos requeridos** | `id`, `sessionId`, `learningContextContractId` (null en modo libre → evaluación genérica), `scoreTotal`, `scoreBreakdown` (`disciplina`, `gestionRiesgo`, `calidadProceso`, `resultado`), `mistakesDetected[]` (refs a ocurrencias), `goodDecisionsDetected[]`, `verdict` (`éxito`, `éxito_parcial`, `fallo_educativo`), `feedbackItemsEs[]` (mensajes concretos generados por las `feedbackRules`) |
| **Campos opcionales** | `comparisonToPersonalBest`, `xpAwarded`, `unlocksGranted[]`, `suggestedNextLessonId`, `processVsOutcomeNoteEs` (ej. "ganaste dinero, pero violaste tu regla de riesgo: esto fue suerte, no habilidad") |
| **Relaciones** | 1→1 con `SimulationSession`; refs a `Mistake` (ocurrencias), `LearningContextContract`, `Achievement` |
| **Persistencia** | SQLite, inmutable. |
| **MVP** | ✅ |
| **Export** | ✅ |

### 1.23 JournalEntry (Entrada de journal)

| Aspecto | Detalle |
|---|---|
| **Propósito** | El diario de trading del alumno: reflexión estructurada sobre trades y sesiones. Burgundy lo guía con plantillas (¿cuál era tu plan?, ¿lo respetaste?, ¿qué sentiste?) en lugar de un campo de texto vacío. |
| **Campos requeridos** | `id`, `createdAt`, `type` (`por_trade`, `por_sesion`, `libre`), `promptedFields` (respuestas a la plantilla guiada), `freeText` |
| **Campos opcionales** | `sessionId`, `tradeId`, `emotionalTags[]`, `lessonLearnedEs`, `linkedMistakeCodes[]`, `pinned` |
| **Relaciones** | N→1 con `UserProfile`; 0..1 con `SimulationSession` y `Trade` |
| **Persistencia** | SQLite. Editable por el usuario (es suyo), con `updatedAt`. |
| **MVP** | ✅ (versión guiada simple) |
| **Export** | ✅ (es de lo más valioso para el usuario) |

### 1.24 Challenge (Desafío)

| Aspecto | Detalle |
|---|---|
| **Propósito** | Definición de un desafío estructurado (Bloque 6): supervivencia, disciplina de riesgo, sin indicadores, etc. Con seed fijo para que la comparación en rankings sea justa: todos enfrentan exactamente el mismo mercado. |
| **Campos requeridos** | `id`, `nameEs`, `descriptionEs`, `challengeType` (`supervivencia`, `disciplina_riesgo`, `sin_indicadores`, `paciencia`, `gestion_perdidas`), `seedRecordId` (seed fijo `fixed_challenge`), `learningContextContractId`, `rules` (restricciones duras: riesgo máx., nº máx. de trades, sin indicadores), `scoringConfig`, `unlockRequirement` |
| **Campos opcionales** | `rotationKey` (challenges semanales/mensuales futuros), `rewardAchievementId`, `xpReward`, `attemptsAllowedRanked` (normalmente 1 intento rankeado; reintentos como práctica no puntuada) |
| **Relaciones** | 1→1 con `SeedRecord`; 1→N con `SimulationSession`, `RankingEntry`; 0..1 con `Achievement` |
| **Persistencia** | Bundle (definiciones); el estado del usuario vive en sesiones y rankings. |
| **MVP** | ✅ (3–5 challenges iniciales) |
| **Export** | ❌ Definición; ✅ los resultados (sesiones + rankings). |

### 1.25 RankingEntry (Entrada de ranking)

| Aspecto | Detalle |
|---|---|
| **Propósito** | High scores **locales** por challenge y por tipo de sesión. Sin servidor: el ranking es contra uno mismo (historial personal), lo que refuerza la filosofía de progreso a largo plazo en vez de competencia social. |
| **Campos requeridos** | `id`, `scope` (`challenge:<id>`, `sandbox:<templateId>`, `global_disciplina`), `sessionId`, `score`, `scoreBreakdown`, `achievedAt`, `engineVersionAtPlay`, `seedRecordId` |
| **Campos opcionales** | `rankAtTime`, `isPersonalBest`, `validForRanking` (false si la sesión fue replay de un path ya visto) |
| **Relaciones** | N→1 con `Challenge` o plantilla; 1→1 con `SimulationSession` |
| **Persistencia** | SQLite. Las entradas son inmutables; se conservan las top-N por scope + todas las personal best. |
| **MVP** | ✅ |
| **Export** | ✅ |

### 1.26 Achievement (Logro / desbloqueo)

| Aspecto | Detalle |
|---|---|
| **Propósito** | Gamificación seria: logros que premian disciplina, paciencia, supervivencia y proceso ("30 días de racha", "10 sesiones sin operar sin stop", "sobrevivió al escenario de noticia"), y desbloqueos de contenido (mercados, instrumentos, modos, challenges). |
| **Definición — campos** | `id`, `nameEs`, `descriptionEs`, `category` (`disciplina`, `proceso`, `supervivencia`, `conocimiento`, `constancia`), `criteria` (regla evaluable), `unlocksContentRefs[]`, `xpReward`, `tier` (`bronce`, `plata`, `oro` — en paleta: dorado #C9A050 solo para oro) |
| **Estado del usuario — campos** | `achievementId`, `status` (`bloqueado`, `en_progreso`, `obtenido`), `progressValue`, `obtainedAt`, `obtainedInSessionId` |
| **Relaciones** | Estado: N→1 con `UserProfile`; refs a `SimulationSession`, `Challenge`, `Lesson` |
| **Persistencia** | Definiciones en Bundle; estado en SQLite. |
| **MVP** | ✅ (set inicial enfocado en disciplina y constancia) |
| **Export** | ✅ Estado; ❌ definiciones. |

### 1.27 ExportedProgressFile (Archivo de progreso exportado)

| Aspecto | Detalle |
|---|---|
| **Propósito** | El contenedor de export/import: snapshot completo del progreso del usuario, autocontenido, versionado y verificable. No es un modelo de base de datos sino un **formato de archivo** (ver §13.3–§13.6). |
| **Campos requeridos (envelope)** | `fileFormatVersion`, `appVersion`, `engineVersion`, `dbSchemaVersion`, `exportedAt`, `installId` (origen), `checksum` (SHA-256 del payload), `payload` (todos los datos exportables comprimidos) |
| **Contenido del payload** | `UserProfile`, `TutorialProgress[]`, `SimulationSession[]` completas (con `Order[]`, `Trade[]`, `UserDecisionLog[]`, `Evaluation`, `Account` final), `SeedRecord[]`, `Scenario[]`, `GeneratedMarketPath[]` selectos (§13.11), `JournalEntry[]`, `RankingEntry[]`, estado de `Achievement[]`, respuestas a `Signal[]` |
| **Campos opcionales** | `userNote` (etiqueta que el usuario pone al export), `partialExportScope` (futuro: exportar solo journal, por ejemplo) |
| **Persistencia** | Archivo en el sistema de archivos del dispositivo, compartible vía share sheet del SO. Extensión propia: `.burgundy`. |
| **MVP** | ✅ |
| **Export** | — (es el export mismo) |

### 1.28 HistoricalSourceMetadata (Metadatos de fuente histórica — futuro)

| Aspecto | Detalle |
|---|---|
| **Propósito** | Describir de dónde provendrían datos históricos futuros: proveedor, licencia, cobertura, calidad. Existe ahora solo para reservar el contrato del esquema. |
| **Campos requeridos** | `id`, `providerName`, `licenseType`, `marketsCovered[]`, `instrumentsCovered[]`, `timeframeGranularity`, `dateRangeAvailable`, `dataQualityNotes`, `importFormatVersion` |
| **Campos opcionales** | `costNotes`, `updateFrequency`, `knownGaps[]`, `bidAskAvailable` (si no, habrá que sintetizar spread) |
| **Relaciones** | 1→N con `HistoricalPatternTemplate`; futura referencia desde `SeedRecord` con `sourceType: historical` |
| **Persistencia** | Bundle o paquete de datos descargable futuro. |
| **MVP** | ❌ (solo definición del contrato; cero implementación) |
| **Export** | ❌ |

### 1.29 HistoricalPatternTemplate (Plantilla de patrón histórico — futuro)

El puente diseñado entre datos históricos y el sistema de escenarios: un episodio histórico real se **destila** en una plantilla que el generador procedural puede reproducir de forma "histórico-inspirada", sin requerir el dataset original en el dispositivo.

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | UUID/slug | Identificador |
| `sourceMarket` | enum | Mercado de origen |
| `sourceInstrument` | string | Instrumento de origen |
| `sourceTimeframe` | string | Timeframe del episodio |
| `sourceDateRange` | rango | Fechas del episodio original |
| `patternType` | enum | `crash`, `squeeze`, `rango_prolongado`, `tendencia_parabolica`, `evento_noticia`, `barrida_histórica` |
| `extractedRegimes` | array | Secuencia de regímenes extraída del episodio |
| `volatilityProfile` | objeto | Perfil de volatilidad medido |
| `liquidityProfile` | objeto | Perfil de liquidez medido/estimado |
| `eventTags` | array | Eventos relevantes del episodio |
| `educationalUseCase` | texto ES | Qué enseña este episodio (ej. "por qué el apalancamiento mata en un crash") |
| `convertedScenarioTemplateId` | ref | El `ScenarioTemplate` generado a partir de este patrón |

| Aspecto | Detalle |
|---|---|
| **Relaciones** | N→1 con `HistoricalSourceMetadata`; 1→1 con un `ScenarioTemplate` (`historicalReady: true`) |
| **Persistencia** | Bundle/paquete descargable futuro. |
| **MVP** | ❌ |
| **Export** | ❌ |

---

## 2. Mapa de relaciones (resumen)

```text
UserProfile ─1:1─ LocalAppProfile
UserProfile ─1:N─ TutorialProgress ──ref── Lesson (bundle)
UserProfile ─1:N─ JournalEntry · Achievement(estado) · RankingEntry

Lesson ─1:1─ LearningContextContract (bundle)
Lesson ─1:N─ ScenarioTemplate (bundle) ─N:1─ Market ─1:N─ Instrument

ScenarioTemplate ─1:N─ SeedRecord ─1:1─ GeneratedMarketPath (caché regenerable)
SeedRecord ─1:N─ Scenario ─1:N─ SimulationSession

SimulationSession ─1:1─ Account
                  ─1:N─ Order ─1:0..1─ Trade ─1:1─ Position (runtime)
                  ─1:N─ UserDecisionLog
                  ─1:N─ Mistake(ocurrencias)
                  ─1:1─ Evaluation
                  ─0:N─ RankingEntry · JournalEntry

Challenge (bundle) ─1:1─ SeedRecord(fixed_challenge)
HistoricalSourceMetadata ─1:N─ HistoricalPatternTemplate ─1:1─ ScenarioTemplate [futuro]
```

Regla transversal: las flechas de los datos del usuario apuntan **hacia** el path/seed (lectura), nunca al revés. Nada que haga el usuario escribe sobre `GeneratedMarketPath`, `SeedRecord`, ni contenido del bundle.

---

## 3. Estrategia de persistencia local

### 3.1 Estrategia de guardado local

Tres capas con responsabilidades separadas:

| Capa | Tecnología | Qué guarda | Por qué |
|---|---|---|---|
| **Datos estructurados** | SQLite (WAL activado) | Sesiones, trades, decision log, progreso, journal, rankings, seeds, evaluaciones, logros | Transaccional, consultable, robusto ante cierres forzados |
| **Preferencias ligeras** | MMKV | Preferencias de UI, flags, último estado de navegación | Lectura síncrona instantánea al arrancar; nada crítico vive aquí |
| **Contenido estático** | Bundle de la app (JSON versionado) | Lecciones, contratos, plantillas, challenges, catálogo de errores, mercados, instrumentos | Versionado con la app; cero riesgo de corrupción por escritura; se actualiza con cada release |

Reglas:

- **Una sola fuente de verdad por dato.** El progreso nunca se duplica entre MMKV y SQLite.
- Los blobs grandes (paths comprimidos, curvas de equity) se guardan como columnas BLOB comprimidas, no como miles de filas.
- Todas las escrituras de fin de sesión son **una transacción**: sesión + trades + decision log + evaluación + ranking + XP se confirman juntos o no se confirman.

### 3.2 Comportamiento de auto-save

- **Durante una sesión en curso:** snapshot ligero cada N velas avanzadas (ej. cada 20) **y** después de cada acción de trading del usuario (orden, modificación de SL, cierre). El snapshot guarda: `currentCandleIndex`, estado de la cuenta, posiciones abiertas, órdenes pendientes. No guarda velas (el path ya está materializado o es regenerable).
- **Al cerrar la app o pasar a background:** snapshot inmediato. Si el SO mata la app, al reabrir se ofrece "Continuar sesión donde quedaste".
- **Fuera de sesión:** cada mutación de progreso (completar lección, ganar logro, escribir journal) se escribe inmediatamente en su transacción. No existe el concepto de "guardar manualmente el progreso": en Burgundy todo progreso está siempre guardado.
- **Append-only para el decision log:** las entradas se insertan al ocurrir, nunca en lote al final — un crash no pierde la bitácora.

### 3.3 Comportamiento de export manual

1. El usuario lo dispara desde Ajustes → "Exportar progreso".
2. La app construye el `ExportedProgressFile`: serializa el payload, lo comprime (gzip), calcula SHA-256 y arma el envelope.
3. El archivo se nombra `burgundy_progreso_<displayName>_<fecha>.burgundy`.
4. Se entrega vía **share sheet del SO**: el usuario decide a dónde va (Drive, correo, cable, etc.). Burgundy **no** sube nada a ningún servidor propio.
5. Antes de exportar se muestra qué incluye el archivo (transparencia) y se advierte: "Este archivo contiene tu journal personal. Compártelo solo con quien tú decidas."
6. La export es **no destructiva**: nada cambia en el dispositivo (solo se actualiza `lastExportAt`).

### 3.4 Comportamiento de import

1. El usuario abre un archivo `.burgundy` (desde el gestor de archivos o el share sheet) o usa Ajustes → "Importar progreso".
2. Validación en orden, **antes** de tocar la base local: (a) formato y `fileFormatVersion` soportada; (b) checksum correcto; (c) `dbSchemaVersion` del archivo ≤ la local (si es menor, se migra el payload en memoria); (d) integridad referencial básica del payload.
3. Se muestra un **resumen previo**: nivel, XP, nº de sesiones, fecha de export, dispositivo de origen, y qué pasará al importar.
4. La importación corre como transacción única sobre una base temporal; solo al validarse por completo se promueve a base activa.
5. Antes de promover, la base actual se respalda automáticamente (backup local rotativo) — un import nunca destruye lo que había sin copia previa.

### 3.5 Manejo de conflictos al importar progreso más antiguo

Escenario: el dispositivo tiene progreso más nuevo (o simplemente distinto) que el archivo importado.

- **Política MVP: reemplazo explícito e informado, sin merge.** El merge automático de dos historiales de sesiones, rachas y rankings es un campo minado (¿qué racha vale?, ¿qué personal best?, ¿sesiones duplicadas?) y no aporta valor educativo proporcional a su riesgo.
- La app **detecta** el conflicto comparando `exportedAt`, XP total y nº de sesiones, y lo dice sin ambigüedad: *"Este archivo es del 3 de mayo y tiene menos progreso que el de este dispositivo (12 sesiones vs 31). Si importas, el progreso actual será reemplazado. Se creará una copia de seguridad automática antes."*
- Opciones presentadas: **(1)** Cancelar, **(2)** Reemplazar (con backup automático previo), **(3)** Exportar primero el progreso actual y luego reemplazar (recomendada y preseleccionada cuando hay conflicto).
- El backup previo al import se conserva y es restaurable desde Ajustes durante al menos 30 días o hasta los siguientes 3 imports.
- Post-MVP puede evaluarse un merge selectivo (ej. fusionar solo journal), nunca en MVP.

### 3.6 Recomendación de formato de archivo

| Decisión | Recomendación | Justificación |
|---|---|---|
| Formato base | **JSON canónico + gzip**, extensión propia `.burgundy` | Legible para depuración, portable, multiplataforma, agnóstico de stack; gzip reduce 5–10× el peso de datos tabulares repetitivos |
| Estructura | Envelope sin comprimir (versiones + checksum) + payload comprimido | Permite validar compatibilidad y integridad **sin** descomprimir todo |
| Integridad | SHA-256 del payload en el envelope | Detecta corrupción en tránsito (correo, mensajería, nube de terceros) |
| Versionado | `fileFormatVersion` semántico propio, independiente de la versión de la app | Las migraciones de import se escriben contra el formato de archivo, no contra versiones de app |
| Serialización de números | Decimales como strings o enteros escalados (precios en unidades mínimas) | Evita pérdida de precisión de punto flotante entre plataformas — crítico para que `pathHash` sea estable |
| ¿Cifrado? | No en MVP; el archivo es del usuario y no contiene credenciales | Cifrar complicaría la recuperación; se advierte que el journal es personal. Reevaluable post-MVP |

Se descartan: SQLite crudo como formato de intercambio (acopla el archivo al esquema interno y a la versión del motor de BD) y formatos binarios propietarios (dificultan depuración y auditoría sin beneficio real a este volumen de datos).

### 3.7 Consideraciones de privacidad de datos

- **Todo es local.** Burgundy no tiene backend, no envía telemetría con datos del usuario, no requiere cuenta. Es su mayor garantía de privacidad y debe declararse explícitamente en la app.
- Datos sensibles reales en el dispositivo: `displayName`, el **journal** (reflexiones personales, etiquetas emocionales) y los `emotionalTag` del decision log. Tratamiento: nunca salen del dispositivo salvo export manual del propio usuario; el export lo advierte (§3.3).
- Las etiquetas emocionales son **opt-in** siempre: la app funciona completa sin reportar emociones.
- No se recolecta: ubicación, contactos, identificadores publicitarios, datos financieros reales. La app no pide esos permisos.
- `installId` existe solo para etiquetar el origen de un export (útil al resolver conflictos); no identifica a la persona ni se transmite.
- Si en el futuro se agrega analítica de producto, será un documento de decisión aparte, opt-in, y jamás incluirá contenido de journal ni etiquetas emocionales.

### 3.8 Limitaciones de backup

Decirlas de frente, porque sin login el respaldo es responsabilidad compartida con el usuario:

1. **No hay nube de Burgundy.** Si el usuario pierde el teléfono sin haber exportado, el progreso se pierde. La app debe decirlo en onboarding y recordarlo periódicamente.
2. **Backups del SO ( Auto Backup / iCloud)** pueden cubrir la base SQLite, pero son inconsistentes entre fabricantes, tienen límites de tamaño y el usuario puede tenerlos desactivados. Se permiten (no se excluye la BD del backup del SO) pero **no se promete** nada basado en ellos.
3. **Mitigación activa:** recordatorio suave de export tras hitos (subir de nivel, racha de 7 días, completar un módulo) si han pasado >14 días del último export: *"Llevas 3 semanas de progreso sin respaldar. Exportar toma 10 segundos."* Frecuencia limitada para no fastidiar.
4. **Backup local rotativo automático:** copia interna de la BD (3 generaciones) antes de cada import, cada migración de esquema y semanalmente. Protege contra corrupción y errores de import, **no** contra pérdida del dispositivo.
5. Desinstalar la app borra todo (comportamiento estándar del SO). Advertirlo en Ajustes.

### 3.9 Cómo evitar corromper el progreso

| Vector de corrupción | Defensa |
|---|---|
| Cierre forzado / batería muerta durante escritura | SQLite en modo WAL + transacciones atómicas; nunca escrituras parciales multi-tabla fuera de transacción |
| Falta de espacio en disco | Verificar espacio antes de iniciar sesión y antes de materializar un path; degradar con aviso, nunca escribir a medias |
| Migración de esquema fallida | Backup automático pre-migración + migraciones idempotentes + verificación post-migración antes de borrar el backup |
| Import corrupto | Validación completa + base temporal + promoción atómica (§3.4); el archivo malo jamás toca la base activa |
| Bug de la app escribiendo estado inválido | Invariantes verificadas al cargar (equity coherente con trades, hashes de paths); si falla, se ofrece restaurar del backup rotativo |
| Doble escritura concurrente | Una sola conexión de escritura serializada a SQLite; el motor de simulación no escribe directamente — entrega snapshots a la capa de persistencia |
| Corrupción silenciosa de paths | `pathHash` en `SeedRecord` verificado al cargar un path materializado; si no coincide, se regenera desde el seed |

### 3.10 Modelo de datos mínimo del MVP

Lo irreducible para lanzar (todo lo demás puede simplificarse o posponerse):

| # | Modelo | Forma MVP |
|---|---|---|
| 1 | `UserProfile` | Completo |
| 2 | `LocalAppProfile` | Versiones + preferencias básicas |
| 3 | `Market` / `Instrument` | 2 mercados, 2–4 instrumentos (bundle) |
| 4 | `Candle` (formato universal) | Completo — no se recorta: es el contrato del futuro |
| 5 | `SeedRecord` | Completo — no se recorta: es la garantía de replay |
| 6 | `GeneratedMarketPath` | Completo, con caché regenerable |
| 7 | `ScenarioTemplate` + `Scenario` | Set inicial (~8–12 plantillas) |
| 8 | `LearningContextContract` | Completo para las lecciones del MVP |
| 9 | `SimulationSession` + `Account` + `Order` + `Trade` | Completos |
| 10 | `Position` | Runtime + snapshot de auto-save |
| 11 | `UserDecisionLog` | Acciones de trading (las de navegación, opcionales) |
| 12 | `Mistake` | Catálogo de 10–15 errores núcleo + ocurrencias |
| 13 | `Evaluation` | Completa |
| 14 | `TutorialProgress` + `Lesson` (bundle) | Módulos 1–N del currículo MVP |
| 15 | `JournalEntry` | Plantilla guiada simple |
| 16 | `Challenge` + `RankingEntry` | 3–5 challenges, ranking local |
| 17 | `Achievement` | Set inicial de disciplina/constancia |
| 18 | `ExportedProgressFile` | Formato v1 completo |

Quedan **fuera del MVP**: `TickEvent` materializado (se regenera), sistema completo de `Signal` (solo lecciones del módulo de señales), `HistoricalSourceMetadata`, `HistoricalPatternTemplate`, merge de imports, cifrado de export, repaso espaciado.

### 3.11 Qué seeds y paths deben incluirse en el archivo de export

Regla general: **se exportan todos los `SeedRecord`; los `GeneratedMarketPath` solo cuando el seed ya no garantiza la regeneración exacta.**

| Dato | ¿Se exporta? | Razón |
|---|---|---|
| `SeedRecord` de **toda** sesión exportada | ✅ Siempre | Pesa bytes; sin él no hay replay ni verificación |
| `pathHash` de cada seed | ✅ Siempre | Permite al dispositivo destino verificar que su regeneración es idéntica |
| `GeneratedMarketPath` de sesiones cuyo `generatorVersion` == versión del motor actual | ❌ No | El destino lo regenera del seed y lo verifica contra el hash |
| `GeneratedMarketPath` de sesiones jugadas con un `generatorVersion` **ya retirado** del motor | ✅ Sí (comprimido) | Es la única forma de mantenerlas replayables (ver §3.13) |
| `GeneratedMarketPath` de la sesión `en_curso` (si se exporta a mitad) | ✅ Sí | Garantiza continuar exactamente donde quedó, sin depender de regeneración en el destino |
| Ticks/micro-movimientos | ❌ Nunca | Deterministas desde el seed |
| Definiciones de plantillas/contratos/lecciones | ❌ Nunca | Viajan con la app; se referencian por id + versión |

Si al importar, el dispositivo destino tiene una versión de motor que no puede regenerar un seed y el archivo no incluye el path congelado, la sesión se importa como **historial de solo lectura** (métricas, journal, evaluación visibles; replay deshabilitado con explicación). Nunca se descarta.

### 3.12 ¿El MVP guarda paths completos o solo seeds + versión de generador?

**Recomendación: estrategia híbrida "seed como verdad, path como caché".**

- **La verdad permanente es `seed + generatorVersion + scenarioTemplateId + pathHash`** (el `SeedRecord`). Esto es lo que se guarda para siempre y se exporta siempre.
- **El path materializado es una caché:** se genera completo al iniciar la sesión (obligatorio por la filosofía del producto), se conserva mientras la sesión está en curso y durante una ventana de uso reciente (ej. últimas 10 sesiones, para replay instantáneo), y después es purgable — regenerarlo desde el seed toma milisegundos para los tamaños del MVP (cientos a pocos miles de velas).
- **Excepción de congelamiento:** si una actualización del motor retira o cambia un algoritmo de generación, los paths afectados se materializan y congelan **antes** de aplicar la actualización (ver §3.13).

Por qué no "solo seeds": dejaría las sesiones rehenes de la estabilidad eterna del generador — irrealista. Por qué no "solo paths completos": multiplicaría el almacenamiento y el peso del export sin necesidad, castigando justo a los dispositivos de gama baja del mercado LATAM. El híbrido da replay garantizado con almacenamiento mínimo.

### 3.13 Replay de sesiones antiguas tras actualizaciones del generador

El problema: el replay exige reproducir el path **bit a bit**; cualquier mejora del generador puede romper esa garantía. Solución en cuatro reglas:

1. **Versionado semántico del generador, separado de la app.** `generatorVersion` vive en cada `SeedRecord`. Cambios que alteren cualquier salida para cualquier seed son **major** del generador, sin excepciones — "mejorar el realismo" también rompe replay.
2. **Generadores versionados conviviendo (estrategia preferida).** El motor conserva las rutas de generación de versiones anteriores como módulos congelados (`genV1`, `genV2`…). Regenerar un path antiguo usa la versión registrada en su `SeedRecord`. Costo: algo de peso en el bundle; beneficio: replay eterno sin almacenar paths.
3. **Congelamiento en la migración (red de seguridad).** Si mantener una versión vieja del generador se vuelve inviable (bug de determinismo, deuda inaceptable), la actualización de la app ejecuta una migración que **materializa y congela** los paths de todas las sesiones afectadas antes de retirar el generador viejo. Desde entonces esas sesiones se reproducen desde el path almacenado, no desde el seed, y sus `SeedRecord` se marcan con `sourceType: replay`.
4. **Verificación por hash siempre.** Antes de cualquier replay, el path (regenerado o cargado) se valida contra `pathHash`. Si no coincide — generador cambiado sin migración, corrupción, bug — el replay se bloquea con un mensaje honesto ("Esta sesión fue jugada con una versión anterior del simulador y no puede reproducirse con exactitud") y la sesión queda como historial de solo lectura. **Burgundy nunca muestra un replay aproximado como si fuera exacto:** la integridad del replay es parte de la integridad educativa del producto.

Implicación de ranking: una `RankingEntry` registra `engineVersionAtPlay`; los rankings comparan solo dentro de la misma versión major del generador cuando el seed es fijo (un challenge con generador nuevo es, en rigor, un mercado distinto — y se trata como tal, rotando el challenge).

---

## 4. Cierre

Este documento fija el contrato de datos de Burgundy: formato universal de vela agnóstico de fuente, `SeedRecord` como unidad permanente de reproducibilidad, paths inmutables generados antes de la acción del usuario, decisiones del usuario en modelos propios que nunca tocan el mercado, persistencia local transaccional sin login, y export/import versionado con verificación de integridad. El esquema queda preparado para el modo histórico futuro sin que el MVP dependa de él.

**Siguiente bloque sugerido:** Bloque 10 — Roadmap de desarrollo y plan de releases del MVP.

---

*Burgundy — proyecto firmado por **tsuloid**. Documento de arquitectura de datos, sin código ni pantallas. Paleta: #1A1617 · #571324 · #2E2E2E · #C9A050 · #4A6D56 · #802F3E.*
