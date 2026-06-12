# Burgundy — Bloque 15: Plan de Corrección de Bloqueos de Gravedad Alta

**Proyecto:** Burgundy — Simulador educativo de trading
**Autor / Firma:** tsuloid
**Documento:** `docs/architecture/15_high_severity_correction_plan.md`
**Idioma:** Español (LATAM)
**Estado:** Plan de corrección. No incluye código, ni pantallas, ni ediciones a los documentos 01–13 todavía.
**Fuente de auditoría:** `docs/architecture/14_architecture_audit.md` (el archivo `14_architecture_audit_merged.md` referenciado en el encargo no existe en el repositorio; la única auditoría presente es `14_architecture_audit.md` y es la fuente de este plan).
**Alcance:** únicamente los problemas con **Gravedad = Alta** y **¿Bloquea Prompt 14? = Sí** (AUD-001 a AUD-014). Los problemas Media (AUD-015 a AUD-019) y Baja (AUD-020, AUD-021) quedan explícitamente fuera, salvo intersección estrictamente necesaria declarada en este documento.

> Nota de estado sobre AUD-001: la verificación del repositorio (2026-06-12) muestra que `docs/architecture/13_final_master_spec.md` **ya existe con el nombre exacto esperado** y que los 13 documentos 01–13 están presentes con nomenclatura correcta. AUD-001 se resuelve con un acta de verificación dentro de este plan (sección C, Lote A, paso 0), sin editar ningún archivo.

---

## A. Tabla de bloqueos Alta

| ID | Documentos afectados | Decisión canónica requerida | Lock relacionado | Archivos a modificar | Riesgo si no se corrige |
|---|---|---|---|---|---|
| AUD-001 | 13 | Confirmar que `13_final_master_spec.md` existe con nombre exacto y que los 13 nombres de la serie coinciden con la lista esperada. **Verificado: ya cumple.** | — (acta de verificación en este plan) | Ninguno (solo verificación) | Un agente que lea por nombre exacto cortaría el flujo reportando archivo faltante. |
| AUD-002 | 04, 09, 12, 13 | Matriz única de mercados MVP: base `synthetic_training` + 3 instrumentos máximo — `synthetic_fx`, `synthetic_stock`, `synthetic_crypto` (desbloqueo tardío). Índices y commodities pasan a post-MVP. Documento 04 deja de marcar índices como MVP; documento 09 deja de decir "2 mercados"; documento 13 deja de decir "3–5 instrumentos; índice según recursos". | `MVP_MARKET_LOCK` | `04_market_coverage.md`, `09_data_models.md`, `12_mvp_scope.md`, `13_final_master_spec.md` | El desarrollo no sabe cuántos instrumentos, perfiles de spread/slippage, templates ni pantallas construir; el MVP se infla antes de empezar. |
| AUD-003 | 02, 05, 06, 10, 12, 13 | Tabla canónica de contenido MVP por ID: **15 lecciones (M1–M5)** según documento 12; **anti-señales reducido: 4 lecciones + 6 escenarios (E1, E2, E3, E5, E8, E9) + 1 challenge** según documento 05/13; **6 challenges cerrados por nombre** (Supervivencia 50 velas, Riesgo de hierro, Sin indicadores, Paciencia, Stop obligatorio, Costos reales) con criterio de éxito = completar al menos 3; del catálogo de 16 escenarios del documento 10, solo los referenciados por las 15 lecciones y los 6 challenges quedan MVP — el resto se marca `post_mvp`. Las 29 lecciones del documento 02 se mantienen como currículo completo, con columna explícita de mapeo a las 15 del MVP. | `MVP_CONTENT_LOCK` | `02_beginner_curriculum.md`, `05_why_signals_fail.md`, `06_sandbox_challenges.md`, `10_scenario_engine.md`, `12_mvp_scope.md`, `13_final_master_spec.md` | Prompt 14 implementa demasiado contenido o elige arbitrariamente qué lecciones/escenarios/challenges construir. |
| AUD-004 | 06, 12, 13 | Sandbox MVP mínimo: **horizontes solo Intradía y 1 semana**; sin leverage (ver AUD-014); sin guardar/retomar sesión larga; sin export de sesión individual (solo export de progreso completo); rankings sandbox limitados a personal best por seed guardada (sin claves por mercado/horizonte/dificultad). Los 7 horizontes, long sessions y portabilidad por sesión pasan a Fase 3+. | `MVP_SANDBOX_LIMITS` | `06_sandbox_challenges.md`, `12_mvp_scope.md`, `13_final_master_spec.md` | El MVP deja de ser prototipo educativo y se vuelve plataforma completa, con carga inasumible de UX, persistencia, performance y QA. |
| AUD-005 | 03, 08, 09, 10, 12, 13 | Política única seed/path: `SeedRecord` **siempre**; `pathHash` **siempre**; path materializado localmente solo para (a) sesión activa/reciente y (b) replays guardados explícitamente; export incluye path completo solo si la sesión está en curso o su `generatorVersion` no es regenerable por el build actual; replay usa path almacenado si existe (validando hash), si no regenera con el generador versionado y valida hash. El documento 03 (§"guardar siempre el path comprimido") y el documento 12 (§14 "no se almacenan por defecto") se reescriben para citar esta política única. | `SEED_PATH_REPLAY_EXPORT_LOCK` | `03_simulation_model.md`, `08_technical_architecture.md`, `09_data_models.md`, `10_scenario_engine.md`, `12_mvp_scope.md`, `13_final_master_spec.md` | Replay, import, rankings y recuperación tras updates se implementan de formas incompatibles entre módulos y se rompen. |
| AUD-006 | 03, 06, 07, 10 | Visibilidad de seed en desafíos: **antes del intento se muestran solo `pathHash`, `seedType`, `generatorVersion` y las reglas del LCC**; la seed cruda se revela únicamente al cerrar el intento. Si la seed fue revelada antes (replay, seed guardada, sandbox), el intento queda marcado `seed_known = true` y **no es elegible para el ranking principal de primer intento**. El briefing del documento 07 muestra el sello de equidad (hash + tipo + versión), no la seed. | `SEED_PATH_REPLAY_EXPORT_LOCK` (sección de visibilidad) | `03_simulation_model.md`, `06_sandbox_challenges.md`, `07_mobile_ux.md`, `10_scenario_engine.md` | Usuarios precomputan o memorizan el mercado, inflan rankings y destruyen la métrica de primer intento. |
| AUD-007 | 03, 08, 09, 10, 13 | Contrato de determinismo ejecutable: **PRNG = PCG32** (decisión cerrada, eliminar "o xoshiro"); formato de seed (entero sin signo de 64 bits, representación canónica decimal); derivación de substreams por subsistema con índices fijos documentados; escala entera de precios por instrumento (`priceScale` en el catálogo); tiempo lógico = índice de vela + índice de sub-tick (sin reloj real); reglas de redondeo half-even sobre enteros escalados; serialización canónica byte-a-byte del path (orden de campos fijo, sin espacios, enteros decimales, UTF-8) como entrada del SHA-256; **corpus dorado de hashes** (template + LCC + params + seed → pathHash esperado) versionado en el repo y verificado en CI. | `DETERMINISM_LOCK_V1` | `03_simulation_model.md`, `08_technical_architecture.md`, `09_data_models.md`, `10_scenario_engine.md`, `13_final_master_spec.md` | Android/iOS/Node/Hermes producen hashes distintos; replay y rankings quedan inválidos de forma irreparable. |
| AUD-008 | 05, 10, 11, 13 | `scoring_v1` implementable: fórmulas exactas por métrica (sin "curva en S" sin definir: funciones por tramos con puntos de quiebre numéricos), thresholds N/M/K resueltos por escenario en su LCC, definición operativa de "setup válido" y "oportunidad válida" como predicados sobre el decision log, pesos por LCC con suma = 100, reglas de abstención y de penalización por no operar, caps de grade ya definidos (blow-up = F, etc.) referenciados desde las fórmulas, eventos del decision log requeridos por cada regla, y **integración cerrada del Process Score anti-señales (documento 05) como perfil de scoring dentro de las 7 categorías** (no como sistema paralelo). Para cada seed fija de tutorial/challenge: salida determinista esperada (decision log de prueba → score exacto). | `SCORING_V1_LOCK` | `05_why_signals_fail.md`, `10_scenario_engine.md`, `11_evaluation_scoring.md`, `13_final_master_spec.md` | El agente de desarrollo inventa fórmulas; el score resulta explotable, inconsistente e imposible de auditar. |
| AUD-009 | 08, 12, 13 | Matriz cerrada de stack: versiones exactas de Expo SDK, React Native, `@shopify/react-native-skia`, Reanimated, Gesture Handler, expo-sqlite (WAL), MMKV, Zustand, Vitest/Jest, con columna de estado New Architecture, motivo, peso y alternativa. Flujo decidido: **Expo managed + dev build (expo-dev-client), sin bare, prebuild solo vía `npx expo prebuild` cuando se requiera**; paso obligatorio `npx expo-doctor@latest` tras instalar; EAS Build opcional documentado pero no requerido para el MVP local. | `TECH_STACK_LOCK` | `08_technical_architecture.md`, `12_mvp_scope.md`, `13_final_master_spec.md` | Prompt 14 instala versiones incompatibles con New Architecture o genera un proyecto que no compila en Android/iOS. |
| AUD-010 | 08, 09, 12, 13 | Plataforma traducida a configuración verificable: Android — `minSdkVersion`, `targetSdkVersion` y `compileSdkVersion` exactos correspondientes a Android 15+ (API 35); iOS — `ios.deploymentTarget` exacto. **Decisión pendiente de cierre en el lock:** "iOS 20+" no corresponde a ninguna versión publicada por Apple (el versionado saltó a iOS 26); el lock debe mapear el requisito comercial a un deployment target real y dejar constancia del mapeo. Incluye comandos de verificación (app config / Gradle / Podfile) y checklist de emulador/dispositivo. | `PLATFORM_TARGET_LOCK` | `08_technical_architecture.md`, `09_data_models.md`, `12_mvp_scope.md`, `13_final_master_spec.md` | El build soporta versiones no deseadas, falla por deployment targets o queda inconsistente con el requisito comercial. |
| AUD-011 | 08, 12, 13 | Bloque de flujo Windows PowerShell: comandos exactos de creación del proyecto, instalación, test, lint, dev build y prebuild en sintaxis PowerShell (sin `&&`, sin `export`, rutas con `\`, variables `$env:`); scripts npm escritos para ser shell-agnósticos (la lógica vive en scripts Node, no en concatenaciones bash); convención de rutas del repo; prohibición explícita de asumir bash/macOS en documentación y tooling. | `WINDOWS_POWERSHELL_WORKFLOW` | `08_technical_architecture.md`, `12_mvp_scope.md`, `13_final_master_spec.md` | Prompt 14 produce comandos bash/macOS, rutas incompatibles o scripts que fallan en Windows (el entorno real de desarrollo). |
| AUD-012 | 08, 09, 12 | Especificación cerrada de `.burgundy v1`: esquema exacto (JSON Schema/Zod) con campos obligatorios; envelope sin comprimir (`formatVersion`, `appVersion`, `generatorVersion`, `schemaVersion`, `exportedAt`, checksum SHA-256 del payload) + payload gzip; límites de tamaño (máximos por archivo y por path embebido); versionado de migraciones de formato; tabla de errores de import con códigos y mensajes en español; proceso de import transaccional (validar todo → base temporal → promoción atómica) con **backup automático previo obligatorio**. | `BURGUNDY_FILE_FORMAT_V1` | `08_technical_architecture.md`, `09_data_models.md`, `12_mvp_scope.md` | Corrupción de progreso local o import de estados incompatibles; sin login, el usuario no tiene otra vía de recuperación. |
| AUD-013 | 06, 10, 12 | Horizontes largos fuera del MVP (consecuencia de `MVP_SANDBOX_LIMITS`) **más** presupuesto duro de generación: máximo de velas por sesión, máximo de sub-ticks por vela (4–8 en MVP), política de decimación para el chart (ventana deslizante, ~120 velas visibles), benchmarks mínimos en gama media/baja y fallback a 30 fps documentado. Los desafíos de interés compuesto a 1–2 años se marcan post-MVP. | `MVP_SANDBOX_LIMITS` (límites) + `BEGINNER_HUD_LOCK` (presupuesto de render) | `06_sandbox_challenges.md`, `10_scenario_engine.md`, `12_mvp_scope.md` | Memoria desbordada, generación lenta, exports pesados y jank en Android de gama media/baja LATAM — el dispositivo objetivo. |
| AUD-014 | 04, 05, 10, 12 | Leverage en MVP: **solo conceptual y acotado**. Sin margin engine general, sin liquidación completa. Única excepción: el escenario educativo cerrado de sobreapalancamiento (documento 05/10) con **parámetros fijos empaquetados** (leverage fijo del escenario, fórmula de liquidación simplificada y determinista, sin configuración por el usuario). Sandbox MVP sin opción de leverage (coherente con AUD-004). El margin engine general, los límites crecientes por desbloqueo y la mecánica de futuros quedan en Fase 5. Documento 04 deja de listar leverage como mecánica jugable del MVP en forex/cripto. | `LEVERAGE_MVP_LIMITS` | `04_market_coverage.md`, `05_why_signals_fail.md`, `10_scenario_engine.md`, `12_mvp_scope.md` | La app parece casino o entrenamiento de riesgo agresivo, y obliga a implementar un margin engine completo antes de tiempo. |

---

## B. Locks canónicos que deben crearse o actualizarse

Convención: cada lock es un bloque normativo con encabezado `<!-- LOCK: NOMBRE_DEL_LOCK v1 -->`, vive **una sola vez** en su documento dueño, y los demás documentos solo lo **referencian por nombre** (nunca lo copian). Ante contradicción entre un documento y un lock, **gana el lock**. Todos los locks se replican en resumen (una línea por lock, con puntero al dueño) en una nueva sección de `13_final_master_spec.md`.

| Lock | Estado | Resuelve | Documento dueño | Documentos que lo referencian |
|---|---|---|---|---|
| `MVP_MARKET_LOCK` | Crear | AUD-002 | `12_mvp_scope.md` | 04, 09, 13 |
| `MVP_CONTENT_LOCK` | Crear | AUD-003 | `12_mvp_scope.md` | 02, 05, 06, 10, 13 |
| `MVP_SANDBOX_LIMITS` | Crear | AUD-004, AUD-013 | `12_mvp_scope.md` | 06, 10, 13 |
| `SEED_PATH_REPLAY_EXPORT_LOCK` | Crear | AUD-005, AUD-006 | `08_technical_architecture.md` | 03, 06, 07, 09, 10, 12, 13 |
| `DETERMINISM_LOCK_V1` | Crear | AUD-007 | `08_technical_architecture.md` | 03, 09, 10, 13 |
| `TECH_STACK_LOCK` | Crear | AUD-009 | `08_technical_architecture.md` | 12, 13 |
| `PLATFORM_TARGET_LOCK` | Crear | AUD-010 | `08_technical_architecture.md` | 09, 12, 13 |
| `WINDOWS_POWERSHELL_WORKFLOW` | Crear | AUD-011 | `08_technical_architecture.md` | 12, 13 |
| `SCORING_V1_LOCK` | Crear | AUD-008 | `11_evaluation_scoring.md` | 05, 10, 13 |
| `BURGUNDY_FILE_FORMAT_V1` | Crear | AUD-012 | `09_data_models.md` | 08, 12 |
| `BEGINNER_HUD_LOCK` | Crear | Soporte de AUD-013 (presupuesto de render y densidad mínima del HUD en gama baja) | `07_mobile_ux.md` | 12, 13 |
| `LEVERAGE_MVP_LIMITS` | Crear | AUD-014 | `12_mvp_scope.md` | 04, 05, 10 |

Contenido obligatorio de cada lock:

### B.1 `MVP_MARKET_LOCK`
- Mercado base: `synthetic_training`.
- Instrumentos MVP exactos (3): `synthetic_fx` (estilo EUR/USD), `synthetic_stock`, `synthetic_crypto` (desbloqueo tardío).
- Post-MVP explícito: índices sintéticos, commodities, futuros, históricos.
- Por instrumento: perfil de volatilidad, perfil de spread, perfil de liquidez, `priceScale`, leverage máximo = 1 (ver `LEVERAGE_MVP_LIMITS`).

### B.2 `MVP_CONTENT_LOCK`
- Tabla de IDs exactos: 15 lecciones (M1.1–M1.3, M2.1–M2.3, M3.1–M3.4, M4.1–M4.3, M5.1–M5.2).
- Anti-señales MVP: 4 lecciones (por ID del documento 05), 6 escenarios (E1, E2, E3, E5, E8, E9), 1 challenge ("Cazador de señales") integrado dentro del módulo M5, sin sumar challenges al set de 6.
- 6 challenges MVP por nombre cerrado; criterio de éxito del MVP = el usuario completa al menos 3.
- Mapeo escenarios del documento 10 → estado `mvp` / `post_mvp`; todo lo no referenciado por una lección o challenge MVP es `post_mvp`.
- Las 29 lecciones del documento 02 permanecen como currículo de referencia con columna "incluida en MVP (sí/no/parcial → ID M*)".

### B.3 `MVP_SANDBOX_LIMITS`
- Horizontes MVP: Intradía y 1 semana únicamente.
- Sin leverage en sandbox MVP; sin guardar/retomar sesión larga; sin export de sesión individual.
- Rankings sandbox: solo personal best por seed guardada.
- Presupuesto duro: máximo de velas por sesión, máximo 4–8 sub-ticks por vela, ventana de render ~120 velas, benchmark mínimo en gama media/baja, fallback 30 fps.
- Sandbox completo (7 horizontes, long sessions, export por sesión) = Fase 3+; desafíos de 1–2 años = post-MVP.

### B.4 `SEED_PATH_REPLAY_EXPORT_LOCK`
- `SeedRecord` siempre persistido y siempre exportado.
- `pathHash` siempre calculado y sellado antes de la primera vela visible.
- Path materializado local: solo sesión activa/reciente y replays guardados explícitamente; el resto se regenera por seed + `generatorVersion`.
- Export de path completo: solo sesión en curso o `generatorVersion` no regenerable por el build actual.
- Replay: usa path almacenado si existe (validando hash); fallback = regeneración versionada + validación de hash; hash no coincidente ⇒ sesión "no verificable", excluida de rankings, jamás reparada en silencio.
- Visibilidad de seed en desafíos: pre-intento solo `pathHash`, `seedType`, `generatorVersion` y reglas; seed cruda al cerrar el intento; seed conocida de antemano ⇒ `seed_known = true` ⇒ no elegible para ranking principal de primer intento.

### B.5 `DETERMINISM_LOCK_V1`
- PRNG: **PCG32** (cerrado; se elimina la alternativa xoshiro de todos los documentos).
- Seed: entero sin signo de 64 bits; representación canónica documentada.
- Substreams: tabla fija de índices por subsistema (régimen, velas, sub-ticks, eventos, spread, slippage).
- Precios: enteros escalados con `priceScale` por instrumento; prohibido float en generación.
- Tiempo: lógico (índice de vela + índice de sub-tick); sin reloj real ni entropía externa.
- Redondeo: regla única documentada (half-even sobre enteros escalados).
- Serialización canónica del path: orden de campos fijo, UTF-8, sin espacios, enteros en decimal — entrada exacta del SHA-256.
- Corpus dorado: lista versionada de (template, LCC, params, seed) → pathHash esperado, verificada en CI en cada plataforma.

### B.6 `TECH_STACK_LOCK`
- Matriz de dependencias con versión exacta, motivo, peso aproximado, alternativa y estado New Architecture: Expo SDK, React Native, TypeScript, `@shopify/react-native-skia`, Reanimated, Gesture Handler, expo-sqlite, MMKV, Zustand, framework de tests.
- Flujo: Expo managed + dev build (`expo-dev-client`); prebuild solo cuando se requiera; sin bare.
- Verificación obligatoria post-instalación: `npx expo-doctor@latest`.

### B.7 `PLATFORM_TARGET_LOCK`
- Android: `minSdkVersion = 35`, `targetSdkVersion` y `compileSdkVersion` explícitos (Android 15+).
- iOS: `ios.deploymentTarget` explícito, con acta del mapeo del requisito comercial "iOS 20+" a una versión real publicada por Apple (el versionado de Apple saltó a 26; el lock cierra este mapeo y los documentos dejan de citar un target inexistente sin nota).
- Comandos de verificación de app config / Gradle / Podfile y checklist de emulador/dispositivo.

### B.8 `WINDOWS_POWERSHELL_WORKFLOW`
- Comandos exactos en PowerShell para: crear proyecto, instalar dependencias, correr tests, lint, dev build, prebuild.
- Reglas: sin `&&` (usar `;` o `if ($?)`), variables `$env:`, rutas Windows, scripts npm shell-agnósticos (lógica en Node).
- Prohibido asumir bash/macOS en docs, scripts y tooling del repo.

### B.9 `SCORING_V1_LOCK`
- Fórmula exacta por métrica de las 7 categorías: funciones por tramos con puntos de quiebre numéricos (sustituyen toda "curva en S" conceptual).
- Thresholds N/M/K resueltos por escenario dentro de su LCC (`scoringProfile` completo).
- Predicados operativos sobre el decision log para "setup válido" y "oportunidad válida".
- Pesos por LCC con suma = 100; rentabilidad ≤ 25% siempre.
- Reglas de abstención y penalización por no operar; caps de grade integrados a la fórmula.
- Integración del Process Score anti-señales como `scoringProfile` de las 7 categorías (un solo sistema de score).
- Por cada seed fija MVP: caso dorado (decision log de prueba → score y grade exactos esperados).

### B.10 `BURGUNDY_FILE_FORMAT_V1`
- Esquema JSON Schema/Zod del envelope y el payload; campos obligatorios y opcionales.
- Envelope sin comprimir: `formatVersion`, `appVersion`, `generatorVersion`, `schemaVersion`, `exportedAt`, checksum SHA-256 del payload.
- Payload gzip: perfil, progreso, sesiones, SeedRecords (siempre), paths selectivos (según `SEED_PATH_REPLAY_EXPORT_LOCK`), decision logs, journal, evaluaciones, rankings.
- Límites de tamaño por archivo y por path embebido.
- Import transaccional: validación completa → base temporal → promoción atómica; backup automático previo obligatorio.
- Tabla de errores de import con códigos y mensajes en español.
- Política de migración entre `formatVersion`.

### B.11 `BEGINNER_HUD_LOCK`
- Alcance dentro de este plan: **solo** lo necesario para AUD-013 — presupuesto de render del simulador (ventana deslizante ~120 velas, 60 fps objetivo, fallback 30 fps documentado) y densidad mínima por defecto del HUD del simulador en MVP (chart + riesgo + acción principal; paneles avanzados colapsados por defecto).
- La reorganización completa de densidad Beginner/Expanded del HUD (AUD-015) es Media y **no** se ejecuta en este plan; el lock solo fija el presupuesto y el default, sin rediseñar pantallas.

### B.12 `LEVERAGE_MVP_LIMITS`
- Leverage MVP = 1x en toda mecánica jugable (sandbox, tutorial, challenges).
- Única excepción: escenario educativo cerrado de sobreapalancamiento, con leverage fijo del escenario, fórmula de liquidación simplificada determinista y sin configuración del usuario.
- Sin margin engine general, sin margin calls dinámicas, sin futuros: Fase 5.
- Los textos educativos sobre leverage (glosario, lecciones M5/anti-señales) se mantienen: se enseña el concepto, no se opera con él.

---

## C. Orden de corrección recomendado

Las ediciones a los documentos 01–13 ocurrirán en un paso posterior (no en este). Orden propuesto: **A → B → C**, porque el alcance (A) define qué cubren los contratos técnicos (B), y el stack/entorno (C) puede cerrarse en paralelo pero se documenta al final para referenciar los locks ya escritos.

### Lote A — Scope MVP: mercados, contenido, sandbox, leverage
0. **AUD-001 (acta de verificación, sin edición):** los 13 archivos `01`–`13` existen con nombre exacto en `docs/architecture/`; `13_final_master_spec.md` está presente. Verificado el 2026-06-12. Queda cerrado.
1. **AUD-002 → `MVP_MARKET_LOCK`** en 12; actualizar tabla de mercados de 04 (índice → post-MVP), conteo de instrumentos en 09 y sección 26 de 13.
2. **AUD-003 → `MVP_CONTENT_LOCK`** en 12; marcar `post_mvp` lo no incluido en 02, 05, 06, 10 y 13; unificar el criterio "6 challenges existen / 3 completados = éxito".
3. **AUD-004 + mitad de alcance de AUD-013 → `MVP_SANDBOX_LIMITS`** en 12; reducir la configuración del sandbox en 06 y la sección 14 de 13; mover horizontes largos y desafíos de 1–2 años a post-MVP en 06, 10 y 12.
4. **AUD-014 → `LEVERAGE_MVP_LIMITS`** en 12; ajustar 04 (leverage no jugable en MVP), 05 y 10 (escenario de sobreapalancamiento como excepción cerrada de parámetros fijos).

### Lote B — Seeds, replay, path hash, determinismo, export/import, scoring
5. **AUD-005 + AUD-006 → `SEED_PATH_REPLAY_EXPORT_LOCK`** en 08; reescribir la política de almacenamiento de 03 §"estrategia por defecto" y 12 §14 para citar el lock; alinear 09 (SeedRecord/GeneratedMarketPath), 10 (rankings/replay), 07 (briefing muestra hash, no seed), 06 (sello de equidad) y 13 (§23 y §29.6).
6. **AUD-007 → `DETERMINISM_LOCK_V1`** en 08; cerrar PCG32 en 03 y 13 (eliminar "o xoshiro"); fijar `priceScale` en 09; corpus dorado referenciado en 10 y 13.
7. **AUD-012 → `BURGUNDY_FILE_FORMAT_V1`** en 09; alinear 08 §export/import y 12 §13 para citar el lock. Depende de 5 (qué paths viajan) — por eso va después.
8. **AUD-008 → `SCORING_V1_LOCK`** en 11; integrar Process Score de 05 como `scoringProfile`; resolver thresholds por escenario en 10; actualizar resumen en 13 §25. Depende de 2 (qué escenarios son MVP determinan qué thresholds hay que cerrar).

### Lote C — Stack, plataforma, PowerShell, performance móvil, HUD beginner-safe
9. **AUD-009 → `TECH_STACK_LOCK`** en 08; actualizar tabla de stack de 12 §15 y 13 §20.
10. **AUD-010 → `PLATFORM_TARGET_LOCK`** en 08; alinear metadata de plataforma en 09, 12 y 13; cerrar el mapeo "iOS 20+" → deployment target real.
11. **AUD-011 → `WINDOWS_POWERSHELL_WORKFLOW`** en 08; referencias en 12 y 13.
12. **Mitad de render de AUD-013 → `BEGINNER_HUD_LOCK`** en 07 (presupuesto de render + densidad por defecto, sin rediseñar pantallas); referencias en 12 y 13.

Criterio de cierre del plan: las 14 Altas quedan con lock dueño + documentos alineados (o acta de verificación, en el caso de AUD-001), y ninguna pareja de documentos vuelve a afirmar dos verdades distintas sobre mercados, contenido, sandbox, seeds, paths, determinismo, formato de export, scoring, stack, plataforma o leverage.

---

## D. Reglas de edición

1. **Solo bloqueos Alta.** Se corrigen exclusivamente AUD-001 a AUD-014. Los hallazgos Media (AUD-015 a AUD-019) y Baja (AUD-020, AUD-021) no se tocan en esta pasada.
2. **Intersección Media mínima declarada.** La única intersección permitida con una Media es `BEGINNER_HUD_LOCK`, y solo en su alcance reducido de la sección B.11 (presupuesto de render y densidad por defecto, exigidos por AUD-013). No se ejecuta el rediseño de densidad de AUD-015.
3. **Insertar, no reescribir.** Cada lock se inserta una sola vez en su documento dueño; en los demás documentos solo se ajustan las secciones contradictorias para citar el lock por nombre. No se reescriben documentos completos.
4. **Una sola verdad.** Cuando dos documentos se contradicen, la versión canónica es la del lock; las demás menciones se reducen a referencia. Prohibido duplicar el contenido normativo de un lock en dos documentos.
5. **No convertir features futuras en MVP.** Ninguna corrección puede mover índices, commodities, futuros, leverage jugable, horizontes largos, merge de import, históricos ni rankings online hacia el MVP.
6. **Historical Mode fuera del MVP.** Se mantiene intacto el principio "historical-ready, no historical-dependent": Fases 6–8 no se adelantan, y ningún lock introduce dependencia de datos reales.
7. **Sin código, sin pantallas, sin proyecto.** Las correcciones son documentales: nada de inicializar proyecto, instalar dependencias, generar pantallas o escribir código.
8. **Trazabilidad.** Cada edición futura a 01–13 debe citar en su commit el ID de auditoría que resuelve (p. ej. "AUD-005: política única seed/path").

---

*Documento del proyecto Burgundy, firmado por **tsuloid**. Plan de corrección previo a la edición de los documentos 01–13. Siguiente paso: ejecutar el Lote A. Este documento no incluye código ni pantallas.*
