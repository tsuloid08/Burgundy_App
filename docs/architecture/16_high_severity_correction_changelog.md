# 16 — High Severity Correction Changelog

**Proyecto:** Burgundy — Simulador educativo de trading
**Autor / Firma:** tsuloid
**Documento:** `docs/architecture/16_high_severity_correction_changelog.md`
**Fecha:** 2026-06-12
**Fuentes:** `14_architecture_audit.md` (hallazgos) · `15_high_severity_correction_plan.md` (plan ejecutado, Lotes A → B → C)

---

## 1. Purpose

Este documento registra la resolución de los **bloqueos de gravedad Alta** (AUD-001 a AUD-014) detectados en la auditoría de arquitectura (`14_architecture_audit.md`), aplicados sobre los documentos 02–13 según el plan de corrección (`15_high_severity_correction_plan.md`). Es el acta de cierre de los Lotes A (scope MVP), B (seeds/determinismo/export/scoring) y C (stack/plataforma/PowerShell/HUD), **antes** de pasar a los hallazgos Media/Baja o a Prompt 14. No introduce cambios: solo documenta los ya aplicados.

---

## 2. Gate Status

**High Severity Gate: PASSED FOR MINI-AUDIT**

Aclaración: este changelog **no aprueba Prompt 14 todavía**. Solo documenta que los lotes A, B y C fueron aplicados y que el siguiente paso es ejecutar la **mini-auditoría de Altas**, que verificará cada corrección contra los documentos (incluidos los riesgos residuales anotados en la sección 3).

---

## 3. Correction Summary Table

Cobertura por grupos: **Scope MVP / mercados / contenido / sandbox / leverage** (AUD-001 a AUD-004, AUD-014 — Lote A) · **Seeds / replay / pathHash / determinismo / export-import / scoring** (AUD-005 a AUD-008, AUD-012 — Lote B) · **Stack / plataforma / PowerShell / performance móvil / HUD beginner-safe** (AUD-009 a AUD-011, AUD-013 — Lote C).

| ID | Gravedad original | Bloqueo original | Locks aplicados | Archivos modificados | Cambio realizado | Estado | Riesgo residual |
|---|---|---|---|---|---|---|---|
| AUD-001 | Alta | `13_final_master_spec.md` no presente con nombre exacto | — (acta de verificación, plan 15 §C Lote A paso 0) | Ninguno | Verificado (2026-06-12) que los 13 documentos 01–13 existen en `docs/architecture/` con nomenclatura exacta; no requirió ediciones | Resuelto (verificación) | Ninguno |
| AUD-002 | Alta | Cobertura de mercados MVP contradictoria entre 04, 09, 12 y 13 | `MVP_MARKET_LOCK` | 04, 12, 13 | Matriz única: base `synthetic_training` + 3 instrumentos jugables (`synthetic_fx`, `synthetic_stock`, `synthetic_crypto` con desbloqueo tardío); índices y commodities pasan a post-MVP temprano; 04 §2 y 13 §9 citan el lock | Aplicado | 09 conserva "al menos forex y cripto" (§1.3) y "2 mercados, 2–4 instrumentos" (§3.10) sin cita a `MVP_MARKET_LOCK`; no contradice el tope de 3, pero la mini-auditoría debe confirmar o alinear esa redacción |
| AUD-003 | Alta | Tamaño educativo del MVP contradictorio (02/05/10/12/13) | `MVP_CONTENT_LOCK` | 02, 05, 06, 10, 12, 13 | Tabla canónica por ID: 15 lecciones M1.1–M5.2; anti-señales reducido a 4 lecciones + 6 escenarios (E1, E2, E3, E5, E8, E9) + 1 challenge integrado en M5; 6 challenges cerrados por nombre (éxito MVP = completar ≥3); catálogo del documento 10: escenarios 1–11 y 15 MVP, 12/13/14/16 post-MVP; las 29 lecciones de 02 quedan como currículo de referencia | Aplicado | Bajo |
| AUD-004 | Alta | Sandbox MVP sobredimensionado (7 horizontes, long sessions, export por sesión, rankings ampliados) | `MVP_SANDBOX_LIMITS` | 06, 12, 13 | Horizontes MVP solo Intradía y 1 semana; sin leverage; sin guardar/retomar sesión larga; sin export de sesión individual; rankings sandbox solo personal best por seed guardada; sandbox completo → Fase 3+ | Aplicado | Bajo |
| AUD-005 | Alta | Política seed/path/export sin una sola verdad (03 vs 08 vs 09 vs 10 vs 12) | `SEED_PATH_REPLAY_EXPORT_LOCK` | 03, 08, 09, 10, 12, 13 | Política única: `SeedRecord` siempre persistido y exportado; `pathHash` siempre sellado antes de la primera vela; path materializado solo para sesión activa/reciente y replays guardados; export de path completo solo si la sesión está en curso o su `generatorVersion` no es regenerable; replay usa path almacenado validando hash, con regeneración versionada como fallback; hash no coincidente ⇒ "no verificable", excluida de rankings | Aplicado | Bajo |
| AUD-006 | Alta | Visibilidad de seed en desafíos contradictoria (03/06/07/10) | `SEED_PATH_REPLAY_EXPORT_LOCK` (punto 6) | 03, 06, 10, 13 | Sello de equidad pre-intento: solo `pathHash`, `seedType`, `generatorVersion` y reglas del LCC; la seed cruda se revela al cerrar el intento; seed conocida de antemano ⇒ `seed_known = true`, no elegible para el ranking principal de primer intento | Aplicado | 07 §6.1 aún dice que "la semilla (o un identificador corto…) es visible en el briefing" sin distinguir desafíos ni citar el lock (13 §19 sí lo aplica); la mini-auditoría debe confirmar o alinear esa frase |
| AUD-007 | Alta | Contrato de determinismo no ejecutable ("PCG32 o xoshiro", sin serialización canónica ni escala entera) | `DETERMINISM_LOCK_V1` | 03, 08, 09, 10, 12, 13 | PRNG cerrado en PCG32 (xoshiro eliminado); seed uint64 con representación canónica decimal; substreams fijos por subsistema (índices 0–7); precios en enteros escalados con `priceScale` por instrumento; tiempo lógico (vela + sub-tick); redondeo half-even; serialización canónica byte-a-byte como entrada del SHA-256; corpus dorado de hashes verificado en CI (Node y Hermes) | Aplicado | El corpus dorado es mandato documental: se materializa en el repo al iniciar Fase 1 |
| AUD-008 | Alta | Scoring conceptual no implementable; Process Score anti-señales como sistema paralelo | `SCORING_V1_LOCK` | 05, 10, 11, 13 | Fórmulas por tramos con puntos de quiebre numéricos (sustituyen las "curvas en S"); thresholds N/M/K resueltos por LCC con defaults canónicos; predicados operativos de "setup válido" y "oportunidad válida" sobre el decision log; pesos por LCC con suma = 100 y rentabilidad ≤ 25; reglas de abstención; caps de grade integrados a la fórmula; Process Score implementado como `scoringProfile` `senales_proceso` (un solo motor de scoring); casos dorados por seed fija | Aplicado | Los casos dorados son mandato documental: se materializan en el repo al iniciar Fase 1 |
| AUD-009 | Alta | Stack Expo/RN aprobado sin matriz cerrada de versiones ni flujo de build decidido | `TECH_STACK_LOCK` | 08, 12, 13 | Matriz cerrada: Expo SDK 55 / RN 0.83 / React 19.2 / TS ~5.9 / Skia 2.x / Reanimated 4 + worklets / Gesture Handler 2.28+ / expo-sqlite WAL / MMKV 3 / Zustand 5 / Vitest 3 + jest-expo, con columnas New Architecture, motivo, peso y alternativa; flujo Expo managed + dev build (`expo-dev-client`), sin bare, prebuild solo `--clean`; gate obligatorio `npx expo-doctor@latest`; EAS opcional | Aplicado | Versiones por línea (no parche exacto) hasta que el `package-lock.json` del primer commit congele los parches — declarado en el propio lock |
| AUD-010 | Alta | Plataforma Android 15+ / iOS 20+ declarada pero no verificable; "iOS 20+" no existe como versión publicada | `PLATFORM_TARGET_LOCK` | 08, 09, 12, 13 | `minSdkVersion = 35`, `targetSdkVersion = 36`, `compileSdkVersion = 36` vía `expo-build-properties`; acta de mapeo iOS: Apple saltó de iOS 18 a 26 ⇒ "iOS 20+" → `ios.deploymentTarget = "26.0"`; comandos de verificación (expo config / Gradle / Podfile) y checklist de emulador/dispositivo; metadata de 09, 12 y 13 cita el lock | Aplicado | Verificación del Podfile iOS solo posible en macOS o EAS (limitación declarada en el lock); target/compile 36 son los defaults explícitos de SDK 55, documentados como deliberados |
| AUD-011 | Alta | No existía flujo Windows PowerShell; riesgo de comandos bash/macOS | `WINDOWS_POWERSHELL_WORKFLOW` | 08, 12, 13 | Comandos canónicos en PowerShell (crear proyecto, instalar, test, lint, dev build, prebuild); reglas: sin `&&`, variables `$env:`, rutas Windows, scripts npm shell-agnósticos con lógica en Node, prohibido asumir bash/macOS, convención de rutas del repo (`burgundy-app\`, `docs\architecture\`) | Aplicado | Ninguno — no quedan comandos bash/macOS como fuente operativa en la serie |
| AUD-012 | Alta | `.burgundy` sin especificación cerrada (esquema, límites, errores, atomicidad, backup) | `BURGUNDY_FILE_FORMAT_V1` | 08, 09, 12 | Envelope JSON sin comprimir + payload gzip; esquema JSON Schema/Zod único versionado; límites duros (archivo ≤ 50 MB, path ≤ 2 MB, ≤ 200 paths); import transaccional de 9 pasos con backup automático previo obligatorio y promoción atómica; tabla de errores IMP-001…IMP-008 con mensajes en español; política de migraciones de `formatVersion` | Aplicado | El esquema Zod es mandato documental: se materializa en desarrollo |
| AUD-013 | Alta | Horizontes largos sin presupuesto de velas/sub-ticks/render para gama media/baja LATAM | `MVP_SANDBOX_LIMITS` (límites) + `BEGINNER_HUD_LOCK` (render) | 06, 07, 10, 12, 13 | Horizontes largos y desafíos de 1–2 años fuera del MVP; 4–8 sub-ticks por vela; ventana deslizante de render ~120 velas con decimación; objetivo 60 fps, fallback 30 fps documentado con degradación que nunca toca exactitud de datos ni ejecución; benchmark mínimo en Android 3–4 GB RAM; densidad por defecto del HUD: chart + riesgo + acción principal + velocidad, paneles avanzados colapsados | Aplicado | Benchmarks a medir en hardware real desde Fase 1; la reorganización completa de densidad (AUD-015, Media) queda explícitamente fuera |
| AUD-014 | Alta | Leverage/liquidación demasiado cerca del MVP | `LEVERAGE_MVP_LIMITS` | 04, 05, 10, 12 | Leverage = 1x en toda mecánica jugable del MVP (tutorial, sandbox, challenges, Modo Libre); única excepción: escenario educativo de sobreapalancamiento (escenario 9 / lección M5.2) con parámetros fijos empaquetados y liquidación simplificada determinista; margin engine general, límites crecientes y futuros → Fase 5; 04 deja de listar leverage como mecánica jugable | Aplicado | Ninguno |

---

## 4. Canonical Locks Created or Updated

Convención vigente (plan 15 §B): cada lock vive **una sola vez** en su documento dueño con encabezado `<!-- LOCK: NOMBRE v1 -->`; los demás documentos lo referencian por nombre sin copiarlo; ante contradicción, **gana el lock**. Los 12 están resumidos (una línea por lock, con dueño) en `13_final_master_spec.md` §26. Ningún lock es Future-only: los contenidos futuros (Historical Mode, margin engine, horizontes largos) no tienen lock propio — los locks del MVP los excluyen explícitamente.

| Lock | Documento dueño | Resuelve | Documentos que lo referencian | Estado |
|---|---|---|---|---|
| `MVP_MARKET_LOCK` | `12_mvp_scope.md` (§4) | AUD-002 | 04, 06, 10, 13 (09 previsto por el plan; pendiente — ver riesgo residual AUD-002) | Active / MVP-binding |
| `MVP_CONTENT_LOCK` | `12_mvp_scope.md` (§3) | AUD-003 | 02, 05, 06, 10, 13 | Active / MVP-binding |
| `MVP_SANDBOX_LIMITS` | `12_mvp_scope.md` (§7) | AUD-004 y AUD-013 (límites) | 03, 06, 07, 08, 10, 13 | Active / MVP-binding |
| `LEVERAGE_MVP_LIMITS` | `12_mvp_scope.md` (§5) | AUD-014 | 04, 05, 06, 10, 13 | Active / MVP-binding (margin engine general: Fase 5) |
| `SEED_PATH_REPLAY_EXPORT_LOCK` | `08_technical_architecture.md` (§8) | AUD-005 y AUD-006 | 03, 06, 09, 10, 12, 13 (07 previsto por el plan; pendiente — ver riesgo residual AUD-006) | Active / MVP-binding |
| `DETERMINISM_LOCK_V1` | `08_technical_architecture.md` (§9) | AUD-007 | 03, 09, 10, 11, 12, 13 | Active / MVP-binding |
| `SCORING_V1_LOCK` | `11_evaluation_scoring.md` (§2 bis) | AUD-008 | 05, 10, 12, 13 | Active / MVP-binding |
| `BURGUNDY_FILE_FORMAT_V1` | `09_data_models.md` (§3.6 bis) | AUD-012 | 08, 12, 13 | Active / MVP-binding |
| `TECH_STACK_LOCK` | `08_technical_architecture.md` (§2.3) | AUD-009 | 12, 13 | Active / MVP-binding |
| `PLATFORM_TARGET_LOCK` | `08_technical_architecture.md` (§2.4) | AUD-010 | 09, 12, 13 | Active / MVP-binding |
| `WINDOWS_POWERSHELL_WORKFLOW` | `08_technical_architecture.md` (§2.5) | AUD-011 | 12, 13 | Active / MVP-binding |
| `BEGINNER_HUD_LOCK` | `07_mobile_ux.md` (§4.1) | Soporte de AUD-013 (presupuesto de render y densidad por defecto) | 08, 12, 13 | Active / MVP-binding (alcance limitado: no ejecuta el rediseño AUD-015) |

---

## 5. Files Modified

Documentos modificados por las correcciones Alta (Lotes A + B + C). `01_product_definition.md` y `14_architecture_audit.md` no fueron tocados; AUD-001 se resolvió sin ediciones (acta de verificación en el plan 15).

| Documento | Qué cambió (una línea) |
|---|---|
| `02_beginner_curriculum.md` | Las 29 lecciones quedan como currículo de referencia con nota de alcance: el MVP implementa solo las 15 lecciones M1–M5 según `MVP_CONTENT_LOCK`. |
| `03_simulation_model.md` | PCG32 cerrado y sub-ticks 4–8 citando `DETERMINISM_LOCK_V1`/`MVP_SANDBOX_LIMITS`; la política de almacenamiento de paths se reescribió para citar `SEED_PATH_REPLAY_EXPORT_LOCK` (3 referencias). |
| `04_market_coverage.md` | Tabla de mercados alineada a `MVP_MARKET_LOCK` (índices/commodities → post-MVP temprano) y leverage retirado como mecánica jugable del MVP (`LEVERAGE_MVP_LIMITS`). |
| `05_why_signals_fail.md` | Módulo reducido a 4 lecciones + 6 escenarios + 1 challenge integrado (`MVP_CONTENT_LOCK`); Process Score integrado como `scoringProfile` `senales_proceso` (`SCORING_V1_LOCK`); L4 apalancamiento solo conceptual (`LEVERAGE_MVP_LIMITS`). |
| `06_sandbox_challenges.md` | Sandbox acotado (horizontes Intradía/1 semana, sin leverage, sin long sessions ni export por sesión) citando `MVP_SANDBOX_LIMITS`/`MVP_MARKET_LOCK`/`LEVERAGE_MVP_LIMITS`; sello de equidad en desafíos según `SEED_PATH_REPLAY_EXPORT_LOCK`. |
| `07_mobile_ux.md` | Nuevo `BEGINNER_HUD_LOCK` §4.1 (documento dueño: presupuesto de render ~120 velas / 60→30 fps + densidad por defecto del HUD) y §7.12 remite al lock. |
| `08_technical_architecture.md` | Documento dueño de 5 locks: nuevos `TECH_STACK_LOCK` (§2.3), `PLATFORM_TARGET_LOCK` (§2.4), `WINDOWS_POWERSHELL_WORKFLOW` (§2.5), `SEED_PATH_REPLAY_EXPORT_LOCK` (§8) y `DETERMINISM_LOCK_V1` (§9); export/import y performance remiten a `BURGUNDY_FILE_FORMAT_V1` y `BEGINNER_HUD_LOCK`. |
| `09_data_models.md` | Nuevo `BURGUNDY_FILE_FORMAT_V1` §3.6 bis (documento dueño); `priceScale` agregado a `Instrument` (`DETERMINISM_LOCK_V1`); §3.11–3.12 remiten a `SEED_PATH_REPLAY_EXPORT_LOCK`; metadata de plataforma cita `PLATFORM_TARGET_LOCK`. |
| `10_scenario_engine.md` | Catálogo mapeado MVP/post-MVP (`MVP_CONTENT_LOCK`); modos acotados por los locks del Lote A; visibilidad de seed y resolución de replay según `SEED_PATH_REPLAY_EXPORT_LOCK`; scoring remite a `SCORING_V1_LOCK`. |
| `11_evaluation_scoring.md` | Nuevo `SCORING_V1_LOCK` §2 bis (documento dueño): fórmulas por tramos, thresholds, predicados, pesos = 100, abstención, caps integrados y casos dorados; la sección conceptual queda subordinada al lock. |
| `12_mvp_scope.md` | Documento dueño de 4 locks nuevos: `MVP_CONTENT_LOCK` (§3), `MVP_MARKET_LOCK` (§4), `LEVERAGE_MVP_LIMITS` (§5), `MVP_SANDBOX_LIMITS` (§7); las secciones 10–15 remiten a los locks de los Lotes B y C (determinismo, seed/path, formato, stack, plataforma, PowerShell, HUD). |
| `13_final_master_spec.md` | Secciones 4, 9–12, 14–16, 19–20, 23, 25–26 y 29 alineadas a los locks por referencia; nueva tabla resumen de los 12 locks con documento dueño en §26. |

---

## 6. Restrictions Preserved

Verificado contra los documentos corregidos (en particular 12 §16 y 13 §§1, 4, 26): las restricciones innegociables siguen intactas tras los Lotes A, B y C.

- **App name:** Burgundy. ✓
- **Firma / autoría:** tsuloid (presente en todos los documentos de la serie). ✓
- **Android 15+** (verificable: `minSdk 35` — `PLATFORM_TARGET_LOCK`). ✓
- **iOS 20+** (requisito comercial mapeado a deployment target 26.0 — `PLATFORM_TARGET_LOCK`). ✓
- **LATAM Spanish-only.** ✓
- **No login.** ✓
- **No monetización.** ✓
- **No cursos.** ✓
- **No AI coach.** ✓
- **No broker integration.** ✓
- **No dinero real.** ✓
- **Offline-first.** ✓
- **Progreso local.** ✓
- **Export/import de progreso** (`.burgundy` — `BURGUNDY_FILE_FORMAT_V1`). ✓
- **MVP sin Historical Mode.** ✓
- **Historical Mode solo como update futuro** (historical-ready, no historical-dependent; Fases 6–8). ✓
- **Paleta Burgundy:** ✓
  - `#1A1617` Deep Charcoal
  - `#571324` Matte Burgundy
  - `#2E2E2E` Dark Surface Gray
  - `#C9A050` Muted Gold
  - `#4A6D56` Bullish candles
  - `#802F3E` Bearish candles

Ninguna corrección movió features futuras al MVP (índices, commodities, futuros, leverage jugable, horizontes largos, merge de import, históricos, rankings online) ni introdujo código, pantallas o proyecto.

---

## 7. Next Step

Next required step:
Run `17_high_severity_mini_audit.md` to verify that all High Severity blockers are resolved before addressing Medium/Low findings or Prompt 14.
