# 17 — High Severity Mini-Audit

**Proyecto:** Burgundy — Simulador educativo de trading
**Autor / Firma:** tsuloid
**Documento:** `docs/architecture/17_high_severity_mini_audit.md`
**Fecha:** 2026-06-12 (re-ejecución post-corrección de residuales)
**Fuentes:** `14_architecture_audit.md` · `15_high_severity_correction_plan.md` · `16_high_severity_correction_changelog.md` · documentos 07, 08, 09, 12 y 13 vigentes.

---

## 1. Residuals First Check

| Residual | Archivo | Evaluación | ¿Bloquea Prompt 14? | Corrección requerida |
|---|---|---|---|---|
| AUD-002 / `MVP_MARKET_LOCK` | `09_data_models.md` | **Corregido.** Las frases antiguas ("al menos forex y cripto", "2–4 instrumentos", "2 mercados, 2–4 instrumentos") ya no existen en 09 como regla vigente (verificado por búsqueda: solo sobreviven en 14–17 como evidencia histórica, lo cual no cuenta como fallo). 09 está ahora alineado con la matriz cerrada del lock: §1.3 (línea 66) declara "Matriz cerrada de `MVP_MARKET_LOCK` (documento dueño: `12_mvp_scope.md`, §4): base `synthetic_training`, con exactamente 3 instrumentos jugables (`synthetic_fx`, `synthetic_stock`, `synthetic_crypto` con desbloqueo tardío)"; §1.4 (línea 78) repite los 3 instrumentos con desbloqueo tardío de `synthetic_crypto` y cita el lock; §3.10 fila 3 (línea 625) cierra el modelo mínimo con la misma matriz y la misma cita. El desbloqueo tardío de `synthetic_crypto` coincide con el texto del lock (12 línea 95). La cita explícita a `MVP_MARKET_LOCK` con documento dueño 12 §4 aparece en los tres puntos antes ausentes. | **No** | Ninguna. |
| AUD-003 / `MVP_CONTENT_LOCK` | `09_data_models.md` | **Corregido.** "3–5 challenges iniciales" y "3–5 challenges, ranking local" ya no existen en 09 (verificado por búsqueda; las apariciones restantes viven solo en 14–17 como evidencia histórica). §1.24 (línea 388) declara el "Set cerrado de 6 challenges según `MVP_CONTENT_LOCK` (documento dueño: `12_mvp_scope.md`, §3)" con los 6 nombres canónicos (Supervivencia 50 velas · Riesgo de hierro · Sin indicadores · Paciencia · Stop obligatorio · Costos reales) y el criterio de éxito del MVP "completar al menos 3 de los 6 (no es una cantidad implementable menor)"; §3.10 fila 16 (línea 638) repite set cerrado de 6, éxito = ≥3 de 6, ranking local, con la misma cita al lock. Todo coincide con el texto del lock (12 línea 70: 6 cerrados por nombre; éxito = al menos 3 de los 6). | **No** | Ninguna. |
| AUD-006 / `SEED_PATH_REPLAY_EXPORT_LOCK` | `07_mobile_ux.md` | **Corregido.** 07 ya distingue los dos regímenes que ordena el lock (08 §8, punto 6, línea 322). §6 punto 1 (líneas 302–304): tutorial/sandbox → seed o identificador corto legible derivado visible en briefing/barra superior/evaluación final; desafíos pre-intento → **únicamente el sello de equidad** (`pathHash`, `seedType`, `generatorVersion` y reglas del LCC), **nunca la seed cruda**; seed cruda revelada solo al cerrar el intento; seed conocida de antemano o repetida ⇒ `seed_known = true` y el intento queda fuera del ranking principal de primer intento. §2 pantalla 7 (línea 113) replica la misma distinción en la celda del briefing de escenario. Ambos puntos citan explícitamente `SEED_PATH_REPLAY_EXPORT_LOCK` con documento dueño `08_technical_architecture.md`, §8. La frase antigua sin excepción para desafíos ("la semilla… es visible en el briefing" como regla general) ya no existe. | **No** | Ninguna. |

---

## 2. High Severity Gate Check

| Criterio | Resultado | Evidencia | Riesgo residual |
|---|---|---|---|
| Cada Alta tiene lock o decisión canónica | ✅ Cumple | 14 Altas exactas (AUD-001–014) en `14_architecture_audit.md`; los 12 locks existen físicamente en sus documentos dueños (12: `MVP_CONTENT_LOCK` línea 61, `MVP_MARKET_LOCK` línea 92 · 08: `SEED_PATH_REPLAY_EXPORT_LOCK` línea 315, entre otros · 09: `BURGUNDY_FILE_FORMAT_V1` línea 564 · 07: `BEGINNER_HUD_LOCK` línea 263); AUD-001 resuelto por acta de verificación (plan 15 §C, Lote A paso 0). | Ninguno. |
| `12_mvp_scope.md` y `13_final_master_spec.md` no contradicen las correcciones | ✅ Cumple | 12 es dueño de los locks citados y su texto coincide con lo que 07 y 09 ahora declaran (12 líneas 70, 92–98); 13 alineado por referencia: línea 191 y 481 (3 instrumentos sintéticos según `MVP_MARKET_LOCK`), líneas 288 y 482–483 (6 challenges cerrados, ≥3 = éxito, `MVP_CONTENT_LOCK`), línea 347 (briefing de desafíos muestra el sello de equidad, nunca la seed cruda, `SEED_PATH_REPLAY_EXPORT_LOCK`), tabla de los 12 locks (líneas 496–500). | Ninguno. |
| No queda una Alta sin resolver explícitamente | ✅ Cumple | Las 14 Altas están tratadas explícitamente en 16 §3; los tres residuales que mantenían a AUD-002, AUD-003 y AUD-006 en estado parcial quedaron corregidos en 09 (§1.3, §1.4, §1.24, §3.10) y 07 (§2 pantalla 7, §6 punto 1), con citas a sus locks — sección 1 de este documento. Búsqueda global confirma que las frases contradictorias solo sobreviven en 14–17 como evidencia histórica. | Ninguno. |
| No se amplió el MVP | ✅ Cumple | Los locks prohíben agregar contenido/mercados sin actualizarlos (12 líneas 74 y 98); lista de exclusión intacta (12 §16; 13 línea 481+); las correcciones de 07 y 09 solo alinearon redacción con locks existentes, sin añadir instrumentos, challenges ni pantallas. | Ninguno. |
| No se habilitó Historical Mode en MVP | ✅ Cumple | 12: "Datos históricos reales ❌ Fase 6–7" (línea 86); 09 mantiene `HistoricalSourceMetadata` y `HistoricalPatternTemplate` como ❌ MVP (§1.28–§1.29); 13: excluido del MVP sin excepción. | Ninguno. |
| No se habilitó leverage general | ✅ Cumple | `LEVERAGE_MVP_LIMITS` (12 §5): leverage = 1x en toda mecánica jugable; `MVP_MARKET_LOCK` reitera "leverage máximo = 1x" por instrumento (12 línea 97); 13 alineado. | Ninguno. |
| No se contradice Android 15+ / iOS 20+ | ✅ Cumple | 09 línea 7 declara "Android 15+ / iOS 20+ (configuración verificable: `PLATFORM_TARGET_LOCK`, documento 08)"; 12 y 13 alineados vía el mismo lock. | Verificación del Podfile iOS solo posible en macOS/EAS — limitación ya declarada dentro del lock. |
| No se contradice offline-first / no login / no broker / no dinero real | ✅ Cumple | 09 §0 principio 4 y §3.7 (todo local, sin backend, sin cuenta); 07 principio 6 y §11 (sin dinero real, sin brokers, sin login, sin nube); 12 §16 y 13 alineados. | Ninguno. |

---

## 3. Gate Decision

**ALTAS RESUELTAS: PASAR A MEDIA**

14 de 14 Altas resueltas. Los tres residuales detectados por la versión previa de este documento (AUD-002 y AUD-003 en `09_data_models.md`, AUD-006 en `07_mobile_ux.md`) fueron corregidos y verificados contra el texto de sus locks en los documentos dueños (12 §4, 12 §3, 08 §8). Ningún lock requirió cambios; 12 y 13 permanecen coherentes con las correcciones.

---

## 4. Required Next Action

Next required step:
Begin Medium Severity correction phase.
