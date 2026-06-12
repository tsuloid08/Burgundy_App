# 18 — Medium Severity Correction Plan

**Proyecto:** Burgundy — Simulador educativo de trading
**Autor / Firma:** tsuloid
**Documento:** `docs/architecture/18_medium_severity_correction_plan.md`
**Fecha:** 2026-06-12
**Estado:** Plan de corrección. No incluye código, ni pantallas, ni ediciones a los documentos 01–13 todavía.
**Fuente de auditoría:** `docs/architecture/14_architecture_audit.md` (única fuente de verdad para identificar hallazgos Media).
**Precondición cumplida:** `17_high_severity_mini_audit.md` cerró las 14 Altas con gate **ALTAS RESUELTAS: PASAR A MEDIA**.
**Alcance:** únicamente los hallazgos con **Gravedad = Media** (AUD-015 a AUD-019). Los hallazgos Baja (AUD-020, AUD-021) quedan explícitamente fuera. Las Altas (AUD-001 a AUD-014) no se reabren.

---

## 1. Purpose

Este documento planifica la corrección de los hallazgos de **gravedad Media** detectados en la auditoría de arquitectura (`14_architecture_audit.md`), ahora que la fase de gravedad Alta está cerrada y verificada: el plan `15_high_severity_correction_plan.md` fue ejecutado, el changelog `16_high_severity_correction_changelog.md` documentó las 14 resoluciones, y la mini-auditoría `17_high_severity_mini_audit.md` confirmó **ALTAS RESUELTAS: PASAR A MEDIA** sin residuales.

Ningún hallazgo Media bloquea Prompt 14 (columna "¿Bloquea Prompt 14?" = No en los cinco casos), pero corregirlos antes de generar código reduce riesgo de UX saturada, gamificación barata, esquema inflado, percepción de manipulación de precios y drift semántico de "sin cursos". Este documento **solo planifica**: las ediciones a los documentos 01–13 ocurrirán en un paso posterior, por lotes.

Restricción heredada de la fase Alta que gobierna todo este plan: los 12 locks canónicos creados en la fase Alta **no se reabren ni se modifican en sus decisiones**. La única interacción permitida con un lock es citarlo por nombre, o un ajuste estrictamente referencial (p. ej. añadir una referencia cruzada) si una corrección Media lo exige para mantener consistencia — nunca cambiar lo que el lock decide.

---

## 2. Medium Findings Table

Hallazgos extraídos literalmente de la tabla de `14_architecture_audit.md`, filtrando Gravedad = Media. No se asume numeración externa ni equivalencias manuales.

| ID | Gravedad | Problema | Documentos afectados | Riesgo si no se corrige | Corrección propuesta | ¿Toca MVP? | ¿Puede ejecutarse sin reabrir Altas? |
|---|---|---|---|---|---|---|---|
| AUD-015 | Media | El HUD móvil puede quedar saturado: chart, barra de cuenta, P/L, margen, drawdown, comprar/vender, spread, fees, slippage, SL/TP, R/R, controles de velocidad, journal, posiciones y advertencias. | `07_mobile_ux.md`; `12_mvp_scope.md`; `13_final_master_spec.md` | Un principiante en pantalla pequeña queda abrumado; aumenta el riesgo de errores táctiles y baja performance. | Definir niveles de densidad: `Beginner HUD` con chart + riesgo + acción principal; `Expanded HUD` por bottom sheet; campos avanzados solo por lección/desbloqueo. Probar con tamaño de fuente +30%. Se construye **sobre** el `BEGINNER_HUD_LOCK` existente (07 §4.1), que ya fijó la densidad por defecto y el presupuesto de render; este hallazgo completa la reorganización de densidad que el plan 15 (§B.11) dejó explícitamente fuera. | No (reorganiza presentación; no añade features ni pantallas nuevas al alcance) | Sí — extiende el contenido de 07 sin cambiar las decisiones de `BEGINNER_HUD_LOCK`; el default ya fijado por el lock se mantiene idéntico. |
| AUD-016 | Media | El modelo de datos sigue demasiado grande para MVP: 29 modelos, modelos históricos futuros, Achievement, Signal completo, sourceTypes históricos, rankings detallados y TickEvent aunque varios se declaren post-MVP. | `09_data_models.md` | El agente puede crear tablas/migraciones innecesarias y subir la complejidad de persistencia. | Separar `mvp_schema_v1` físico de `future_schema_notes`. Implementar solo los 18 modelos mínimos de §3.10, y dejar históricos/Signal completo/Achievement avanzado como ADR, no tablas iniciales. | No (reduce lo que se implementa; no agrega nada) | Sí — es una reorganización interna de 09; los locks que viven en 09 (`BURGUNDY_FILE_FORMAT_V1`) y los citados (`MVP_MARKET_LOCK`, `SEED_PATH_REPLAY_EXPORT_LOCK`, `PLATFORM_TARGET_LOCK`, `DETERMINISM_LOCK_V1`) se conservan intactos y referenciados desde la sección `mvp_schema_v1`. |
| AUD-017 | Media | Algunas frases de LCC parecen forzar resultado aunque la filosofía lo prohíbe, por ejemplo "el TP solo si entra temprano" o caminos diseñados para liquidar con 20x. | `05_why_signals_fail.md`; `10_scenario_engine.md` | El usuario puede percibir que la app manipula el precio para enseñar una moraleja. | Estandarizar redacción: "el path ya contiene X condición; la consecuencia depende de la ejecución". Evitar frases que parezcan outcome scripting posterior a la entrada. | No (solo redacción; ninguna mecánica cambia) | Sí — ajustes de redacción puntuales; no toca `SCORING_V1_LOCK`, `SEED_PATH_REPLAY_EXPORT_LOCK` ni el catálogo MVP/post-MVP fijado por `MVP_CONTENT_LOCK`. |
| AUD-018 | Media | Aparecen percentiles de referencia empaquetados y rankings por mercado/desafío; aunque son locales, pueden sentirse como competencia externa o métrica de status. | `10_scenario_engine.md`; `11_evaluation_scoring.md` | Riesgo de gamificación barata y desviación hacia "ganar el ranking" en vez de aprender proceso. | MVP: solo historial personal y mejora de proceso. Eliminar percentiles empaquetados y rankings por retorno; conservar "primer intento" y "mejora contra tu intento anterior". | No (elimina elementos del MVP; no agrega) | Sí, con cuidado referencial — `MVP_CONTENT_LOCK` y `MVP_SANDBOX_LIMITS` mencionan "ranking local" y "personal best por seed guardada"; la corrección debe interpretarse en coherencia con ambos: se eliminan percentiles empaquetados y rankings por retorno, se conserva el historial personal de primer intento y la mejora contra el intento anterior, sin reescribir el texto de los locks. |
| AUD-019 | Media | "Sin cursos" puede chocar semánticamente con "academia", "ruta de aprendizaje" y muchas lecciones. La intención parece ser no vender cursos ni crear catálogo externo, pero no está definido con precisión. | `01_product_definition.md`; `02_beginner_curriculum.md`; `07_mobile_ux.md`; `13_final_master_spec.md` | Marketing o implementación podrían crear sección tipo curso, violando restricción de producto. | Definir explícitamente: permitido = micro-lecciones integradas al simulador; prohibido = cursos pagos, catálogo externo, mentoría, contenido de terceros o módulos pasivos tipo video-curso. | No (precisión semántica; no cambia el contenido educativo fijado por `MVP_CONTENT_LOCK`) | Sí — define terminología; las 15 lecciones MVP y el currículo de referencia de 02 quedan exactamente como los dejó `MVP_CONTENT_LOCK`. |

---

## 3. Correction Batches

Lotes pequeños, ordenados de menor a mayor superficie de edición. Cada lote es atómico: se ejecuta, se verifica contra los locks vigentes y recién entonces se pasa al siguiente.

### Lote M3 — Schema / documentation consistency (primero: el más pequeño y sin contacto con MVP)
- **AUD-016** — En `09_data_models.md`: separar la sección física `mvp_schema_v1` (los 18 modelos mínimos ya enumerados en §3.10) de una sección `future_schema_notes` (históricos, Signal completo, Achievement avanzado, TickEvent, rankings detallados) con carácter de ADR, no de tablas iniciales. No se crea ningún modelo nuevo; solo se reorganiza y se marca con claridad qué se implementa y qué no.
- **AUD-017** — En `05_why_signals_fail.md` y `10_scenario_engine.md`: barrido de redacción de los LCC para sustituir frases con apariencia de outcome scripting por la fórmula estándar "el path ya contiene X condición; la consecuencia depende de la ejecución". Ediciones puntuales por frase, no reescritura de secciones.

### Lote M2 — Rankings / scoring / anti-gaming
- **AUD-018** — En `10_scenario_engine.md` y `11_evaluation_scoring.md`: eliminar percentiles de referencia empaquetados y rankings por retorno; conservar historial personal, métrica de primer intento y "mejora contra tu intento anterior". Verificación obligatoria de coherencia con `MVP_CONTENT_LOCK` (ranking local de challenges), `MVP_SANDBOX_LIMITS` (personal best por seed guardada) y `SEED_PATH_REPLAY_EXPORT_LOCK` (elegibilidad de primer intento): los locks no se editan; el texto de 10 y 11 se alinea con la lectura "ranking = historial personal local, nunca comparación de status ni por retorno".

### Lote M1 — UX / HUD / beginner clarity
- **AUD-015** — En `07_mobile_ux.md` (principal), con referencias mínimas en `12_mvp_scope.md` y `13_final_master_spec.md` si sus resúmenes del HUD lo requieren: definir los niveles de densidad `Beginner HUD` (chart + riesgo + acción principal) y `Expanded HUD` (bottom sheet), campos avanzados solo por lección/desbloqueo, y criterio de prueba con tamaño de fuente +30%. Se construye sobre `BEGINNER_HUD_LOCK` (07 §4.1) sin alterar el presupuesto de render ni la densidad por defecto que el lock ya fijó; la extensión documenta la reorganización completa que el plan 15 §B.11 difirió a esta fase.

### Lote M4 — Terminología / "sin cursos" vs micro-lecciones
- **AUD-019** — En `01_product_definition.md` (definición canónica), con alineación en `02_beginner_curriculum.md`, `07_mobile_ux.md` y `13_final_master_spec.md`: definir explícitamente que **permitido** = micro-lecciones integradas al simulador, y **prohibido** = cursos pagos, catálogo externo, mentoría, contenido de terceros o módulos pasivos tipo video-curso. La definición vive una sola vez (en 01) y los demás documentos la citan, siguiendo la misma convención de una-sola-verdad de la fase Alta.

**Orden de ejecución: M3 → M2 → M1 → M4.** M3 va primero por ser el lote más pequeño que no toca alcance MVP (un solo documento para AUD-016 más redacción puntual para AUD-017). M4 va último porque toca más documentos (4) aunque su edición por documento sea mínima.

---

## 4. Files Allowed To Modify

Únicamente estos archivos pueden editarse durante la ejecución de los lotes Media, y solo en las secciones que cada hallazgo exige:

| Archivo | Lotes que lo tocan | Hallazgos |
|---|---|---|
| `docs/architecture/01_product_definition.md` | M4 | AUD-019 |
| `docs/architecture/02_beginner_curriculum.md` | M4 | AUD-019 |
| `docs/architecture/05_why_signals_fail.md` | M3 | AUD-017 |
| `docs/architecture/07_mobile_ux.md` | M1, M4 | AUD-015, AUD-019 |
| `docs/architecture/09_data_models.md` | M3 | AUD-016 |
| `docs/architecture/10_scenario_engine.md` | M3, M2 | AUD-017, AUD-018 |
| `docs/architecture/11_evaluation_scoring.md` | M2 | AUD-018 |
| `docs/architecture/12_mvp_scope.md` | M1 (solo referencias mínimas, si el resumen del HUD lo requiere) | AUD-015 |
| `docs/architecture/13_final_master_spec.md` | M1, M4 (solo referencias mínimas y alineación de resúmenes) | AUD-015, AUD-019 |

Ningún otro archivo se modifica. En particular: `03_simulation_model.md`, `04_market_coverage.md`, `06_sandbox_challenges.md`, `08_technical_architecture.md` y los documentos 14–17 quedan intactos. No se toca ningún archivo fuera de `docs/architecture/`.

---

## 5. Guardrails

1. **No ampliar MVP.** Ninguna corrección Media puede agregar instrumentos, lecciones, escenarios, challenges, horizontes, pantallas ni mecánicas al alcance fijado por los locks de la fase Alta.
2. **No tocar Prompt 14.** Esta fase no inicializa proyecto, no instala dependencias, no genera código ni avanza el flujo hacia Prompt 14.
3. **No convertir features futuras en MVP.** Históricos, margin engine, horizontes largos, rankings online, índices, commodities y futuros permanecen post-MVP exactamente donde los dejó la fase Alta.
4. **No cambiar decisiones de locks Alta.** Los 12 locks canónicos (`MVP_MARKET_LOCK`, `MVP_CONTENT_LOCK`, `MVP_SANDBOX_LIMITS`, `LEVERAGE_MVP_LIMITS`, `SEED_PATH_REPLAY_EXPORT_LOCK`, `DETERMINISM_LOCK_V1`, `SCORING_V1_LOCK`, `BURGUNDY_FILE_FORMAT_V1`, `TECH_STACK_LOCK`, `PLATFORM_TARGET_LOCK`, `WINDOWS_POWERSHELL_WORKFLOW`, `BEGINNER_HUD_LOCK`) conservan sus decisiones intactas. Solo se permite el ajuste estrictamente referencial (citas, referencias cruzadas) si una corrección Media lo exige para mantener consistencia.
5. **No corregir Baja todavía.** AUD-020 y AUD-021 quedan fuera de esta fase, aunque la edición pase por la línea afectada.
6. **No modificar código.** No existe código en el repo y esta fase no lo crea: las correcciones son exclusivamente documentales.
7. **No usar dynamic workflows.** Las correcciones se ejecutan de forma directa y secuencial, por lotes, sin orquestación dinámica ni subagentes.
8. **No reescribir documentos completos si basta con ajustes puntuales.** Se conserva la regla de la fase Alta: insertar y ajustar secciones contradictorias, nunca regenerar un documento entero.

Reglas adicionales heredadas: trazabilidad por commit citando el ID de auditoría que resuelve (p. ej. "AUD-016: separar mvp_schema_v1 de future_schema_notes"); ante contradicción entre un documento y un lock, gana el lock; restricciones innegociables del producto (Burgundy, firma tsuloid, Android 15+/iOS 20+ vía `PLATFORM_TARGET_LOCK`, LATAM Spanish-only, no login, no monetización, no cursos, no AI coach, no broker, no dinero real, offline-first, progreso local, export/import `.burgundy`, sin Historical Mode en MVP, paleta Burgundy) permanecen intactas.

---

## 6. Recommended Next Step

Next required step:
Execute Medium Severity corrections by batch, starting with the smallest batch that does not touch MVP scope.

---

*Documento del proyecto Burgundy, firmado por **tsuloid**. Plan de corrección de hallazgos Media, previo a su ejecución por lotes. Este documento no incluye código ni pantallas y no edita los documentos 01–13.*
