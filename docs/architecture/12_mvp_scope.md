# Burgundy — Bloque 12: Alcance del MVP y Roadmap de Desarrollo

**Proyecto:** Burgundy — Simulador educativo de trading
**Autor:** tsuloid
**Idioma:** Español (LATAM)
**Estado:** Documento de planificación. No incluye código ni pantallas.

---

## 1. Objetivo del MVP

El MVP de Burgundy debe demostrar una sola tesis central:

> **Es posible enseñar disciplina de trading real con un simulador que fuerza el contexto de aprendizaje, no el resultado.**

Objetivos concretos:

| # | Objetivo | Cómo se valida |
|---|----------|----------------|
| 1 | El usuario completa el tutorial y entiende spread, slippage, stop loss, drawdown y risk/reward | Quiz de comprensión + decisiones registradas en el journal |
| 2 | El mercado se genera **antes** de la acción del usuario y nunca se manipula después de la entrada | Hash del market path generado y verificable tras la sesión |
| 3 | El score premia el proceso (Learning Context Contract), no la ganancia bruta | Score reproducible con el mismo seed y las mismas decisiones |
| 4 | El progreso vive 100% local y es portable | Export → borrar app → import → progreso intacto |
| 5 | La experiencia se siente como academia seria en terminal Burgundy, no como casino | Pruebas de usabilidad con principiantes LATAM |

El MVP **no** intenta ser un simulador de mercado completo. Intenta ser un **sistema de entrenamiento de disciplina** con mecánica de mercado suficiente para que las lecciones duelan de verdad (spread, slippage, fees, drawdown).

---

## 2. Experiencia de usuario del MVP

Flujo principal (sin login, sin nube):

1. **Primera apertura:** bienvenida breve en español, disclaimer educativo ("esto no es asesoría financiera, no hay dinero real"), creación silenciosa del perfil local.
2. **Ruta guiada inicial:** el tutorial es obligatorio antes de desbloquear sandbox y challenges (progresión por desbloqueo).
3. **Hub central:** pantalla principal con: racha diaria, XP, nivel, siguiente lección sugerida, challenges disponibles, acceso a sandbox y a su journal de errores.
4. **Sesión de simulación:** gráfico de velas como foco visual (velas alcistas #4A6D56, bajistas #802F3E), panel de orden, balance/equity/riesgo/P&L visibles, fondo #1A1617, superficies #2E2E2E, acentos #571324 y resaltados críticos #C9A050.
5. **Cierre de sesión:** resumen con score basado en el Learning Context Contract, errores detectados, explicación de cada error en español claro, y opción de **replay con el mismo seed**.
6. **Progresión:** XP, niveles, desbloqueos de habilidades y challenges, high scores locales.

Tono en toda la app: claro, serio, directo, educativo. Nunca infantil, nunca hype, nunca promesa de riqueza.

---

## 3. Contenido educativo del MVP

Subconjunto del currículo (Bloque 02) suficiente para un principiante vulnerable:

| Módulo | Lecciones | Conceptos clave |
|--------|-----------|-----------------|
| M1 — Qué es el mercado | 3 | Precio, oferta/demanda, velas (candlesticks), timeframes |
| M2 — Costos invisibles | 3 | **Spread** (diferencia entre precio de compra y venta), **slippage** (ejecución peor a la esperada), **fees** |
| M3 — Riesgo primero | 4 | Stop loss, tamaño de posición, **risk/reward** (cuánto arriesgas vs. cuánto buscas ganar), **drawdown** (cuánto cae la cuenta desde su punto más alto) |
| M4 — Disciplina y proceso | 3 | Plan antes de entrar, journal, por qué una ganancia no prueba habilidad |
| M5 — Anti-señales | 2 | Por qué copiar señales de Telegram/TikTok falla; sobre-apalancamiento; entrar sin stop |

Total MVP: **15 lecciones cortas**, cada una con explicación + mini-simulación con seed fijo + quiz breve + registro de errores comunes.

---

## 4. Mercados del MVP

| Mercado | Incluido en MVP | Justificación |
|---------|-----------------|---------------|
| Un par FX sintético (estilo EUR/USD) | ✅ | Spread bajo, ideal para enseñar costos |
| Una "acción" sintética | ✅ | Gaps y volatilidad media, enseña riesgo overnight conceptual |
| Una "cripto" sintética | ✅ | Alta volatilidad, enseña drawdown y disciplina |
| Futuros / margen / apalancamiento avanzado | ❌ | Fase 5 |
| Datos históricos reales | ❌ | Fase 6–7 |

Los tres instrumentos son **sintéticos y procedurales**, generados por el motor de seeds con perfiles de volatilidad, spread, slippage y liquidez distintos. Formato de vela universal y motor agnóstico a la fuente (preparado para histórico futuro).

---

## 5. Mecánica del simulador en el MVP

Incluido:

- Órdenes: market, limit, stop de entrada.
- Salidas: stop loss, take profit, cierre manual.
- Costos siempre activos: spread, slippage modelado, fee por operación.
- Cuenta: balance, equity, P&L flotante y realizado, drawdown actual y máximo.
- Tamaño de posición con cálculo de riesgo en % de cuenta.
- Velocidad de simulación: pausa, 1x, acelerado.
- Journal automático: cada decisión queda registrada (entrada, salida, motivo si el usuario lo anota, errores detectados).

Regla inquebrantable del motor:

> **El market path se genera completo antes de cualquier acción del usuario, a partir del seed. Las decisiones del usuario afectan órdenes, posiciones, cuenta, riesgo, P&L, journal, errores, feedback y score — nunca el precio.**

Excluido del MVP: apalancamiento/margen avanzado, futuros, order book profundo, múltiples posiciones complejas con hedging.

---

## 6. Modo tutorial del MVP

- **Seeds fijos por lección:** cada lección tutorial usa un seed determinista versionado, idéntico para todos los usuarios.
- Escenarios diseñados para forzar el contexto de aprendizaje: p. ej. la lección de slippage usa un escenario de baja liquidez; la de drawdown, una tendencia con retrocesos profundos.
- Guía paso a paso con explicaciones en español claro y terminología real en inglés explicada.
- El tutorial no se puede "ganar" con suerte: el score depende del contrato de aprendizaje (¿pusiste stop?, ¿respetaste el riesgo máximo?), no del P&L.
- Al completar el tutorial se desbloquean sandbox y el primer set de challenges.

---

## 7. Modo sandbox del MVP

- **Seeds aleatorios** generados localmente; cada seed queda guardado con su registro.
- El usuario elige instrumento sintético, dificultad (perfil de volatilidad) y duración de la sesión.
- Contrato de aprendizaje genérico activo (reglas base de disciplina) para que el feedback y la detección de errores funcionen siempre.
- Funciones: guardar seed, repetir seed exacto, ver historial de sesiones, exportar sesión.
- Sin presión de ranking: el sandbox es para practicar, equivocarse y repetir.

---

## 8. Modo challenge del MVP

Set inicial de **6 challenges con seeds fijos**:

| Challenge | Contrato de aprendizaje (resumen) |
|-----------|-----------------------------------|
| Supervivencia 50 velas | No superar X% de drawdown; sobrevivir toda la sesión |
| Riesgo de hierro | Ninguna operación puede arriesgar más del 1% |
| Sin indicadores | Operar solo con velas y niveles; modo no-indicators |
| Paciencia | Máximo 3 operaciones en toda la sesión |
| Stop obligatorio | Toda entrada sin stop loss invalida el score |
| Costos reales | Superar el break-even neto de spread + fees |

- Seed fijo por challenge ⇒ comparable entre intentos y entre usuarios (mismo mercado para todos).
- Cada challenge declara su contrato antes de empezar; el usuario sabe qué se evalúa.
- Completar challenges otorga XP, desbloqueos y entrada al ranking local.

---

## 9. Sistema de ranking del MVP

- **Solo local** (sin nube, sin cuentas): rankings por challenge con los mejores intentos del propio usuario.
- High scores guardados por challenge y por seed.
- Tabla de intento: score de contrato, errores cometidos, drawdown máximo, fecha.
- Los rankings comparan **intentos sobre el mismo seed**, nunca P&L entre seeds distintos (sería comparar mercados diferentes).
- Preparado para expansión futura (Fase 5) sin rediseño: el registro de intento ya guarda seed, hash del path, decisiones y score.

---

## 10. Sistema de seeds deterministas del MVP

| Tipo de seed | Origen | Uso |
|--------------|--------|-----|
| Tutorial | Fijo, empaquetado con la app, versionado | Lecciones guiadas idénticas para todos |
| Challenge | Fijo, empaquetado, versionado | Competencia justa contra el mismo mercado |
| Sandbox | Aleatorio local (PRNG con seed registrado) | Práctica libre reproducible |
| Replay | El seed de cualquier sesión previa | Repetir exactamente el mismo mercado |
| Histórico-inspirado | **Futuro (Fase 8)** | Plantillas extraídas de datos reales |
| Replay histórico | **Futuro (Fase 7)** | Segmentos reales importados |

Requisitos técnicos:

- PRNG determinista (mismo seed + misma versión del generador ⇒ mismo path, bit a bit).
- **Registro local de seeds:** cada sesión guarda seed, versión del generador, plantilla de escenario, instrumento y parámetros.
- **Hash del market path** generado y almacenado al crear la sesión, verificable después: prueba criptográfica de que el precio no se alteró tras la entrada del usuario.
- Versionado del generador: si el algoritmo cambia, los seeds antiguos declaran su versión y se reproducen con ella o se marcan como no-reproducibles.

---

## 11. Sistema de Learning Context Contract del MVP

El **Learning Context Contract (LCC)** es el contrato que define qué debe aprender el usuario en una sesión y cómo se evalúa:

- Cada escenario (tutorial, challenge, sandbox) lleva un LCC declarado **antes** de iniciar.
- Estructura del contrato: objetivo de aprendizaje, reglas evaluadas (p. ej. "stop loss obligatorio", "riesgo ≤ 1%", "máximo N operaciones"), pesos de score, errores a detectar.
- El score final se calcula **exclusivamente contra el contrato**: disciplina, paciencia, supervivencia, calidad de proceso y control de riesgo. El P&L participa solo donde el contrato lo declara (p. ej. challenge de costos reales).
- El contrato fuerza el contexto: el escenario se elige/genera para que el concepto aparezca (mercado lateral para paciencia, mecha violenta para stop loss), pero **nunca** se altera el precio en respuesta a las acciones del usuario.
- Los contratos son datos (declarativos), no código: agregar lecciones nuevas en fases futuras no requiere tocar el motor.

---

## 12. Sistema de replay del MVP

- Cualquier sesión guardada puede repetirse con **exactamente el mismo seed y el mismo market path**.
- Dos modos:
  - **Reintento:** mismo mercado, decisiones nuevas; ideal para "ahora hazlo con stop loss".
  - **Revisión de errores:** reproducción de la sesión original mostrando las decisiones del usuario sobre el gráfico, con los errores detectados anotados en el momento exacto en que ocurrieron.
- El replay valida el hash del path antes de iniciar: si no coincide, la sesión se marca como corrupta y no se compara.
- Replay de sesiones históricas reales: fuera del MVP (Fase 7).

---

## 13. Export / import de progreso del MVP

- **Export:** un único archivo local (formato versionado) que contiene: perfil, XP, nivel, racha, desbloqueos, lecciones completadas, registros de seeds, hashes de paths, logs de decisiones, journal, high scores y configuración.
- **Import:** restaura el progreso completo en una instalación nueva; valida versión de esquema e integridad (checksums) antes de aplicar.
- Conflictos: si ya existe progreso local, el usuario elige reemplazar o conservar (sin merge complejo en MVP).
- El archivo no contiene datos personales: no hay login ni identidad más allá del alias local.
- El formato es la semilla del futuro formato de importación histórico-compatible (Fase 6): mismo principio de esquema versionado.

---

## 14. Persistencia de datos del MVP

- **100% local, offline-first.** Sin backend, sin cuentas, sin telemetría obligatoria.
- Base local estructurada (SQLite vía el stack elegido) para: perfil, progreso, sesiones, seeds, decisiones, journal, scores.
- Los market paths **no** se almacenan completos por defecto: se regeneran desde seed + versión (se almacena solo el hash). Excepción: sesiones marcadas para revisión de errores guardan el log de decisiones con timestamps de vela.
- Escrituras transaccionales: una sesión interrumpida (app cerrada, batería) se recupera o se descarta limpiamente, nunca corrompe el progreso.
- Migraciones de esquema versionadas desde el día uno.

---

## 15. Stack técnico del MVP

Conforme al Bloque 08:

| Capa | Elección | Nota |
|------|----------|------|
| Framework móvil | **React Native** (Android 15+, iOS 20+) | Un solo código base, ecosistema maduro |
| Lenguaje | TypeScript estricto | Tipos para contratos, seeds y esquemas |
| Motor de simulación | Módulo TS puro, sin dependencias de UI | Determinista, testeable, agnóstico a la fuente de velas |
| Gráfico de velas | Renderizado nativo/canvas optimizado | Foco visual principal; colores #4A6D56 / #802F3E |
| Persistencia | SQLite local + archivos de export | Offline-first |
| Estado | Store ligero unidireccional | El motor nunca depende del estado de UI |
| Tema | Sistema de diseño Burgundy (paleta fija) | #1A1617, #571324, #2E2E2E, #C9A050, #4A6D56, #802F3E |

Principio arquitectónico clave: **el motor de simulación es una librería pura** — recibe seed + plantilla + contrato, produce path + eventos; la UI solo observa. Esto garantiza determinismo, replay exacto y testeo sin emulador.

---

## 16. Exclusiones explícitas del MVP

Fuera del MVP, sin excepción:

- Replay histórico real e ingesta de datos históricos.
- Extracción de patrones históricos y generador de plantillas histórico-inspiradas.
- Integración con proveedores de datos.
- Integración con brokers, trading real, dinero real.
- Señales reales de cualquier tipo.
- AI coach.
- Monetización (compras, suscripciones, anuncios).
- Login, cuentas, sincronización en la nube.
- Cursos externos o contenido de terceros.
- Rankings online entre usuarios.
- Apalancamiento/margen avanzado y mecánica de futuros (Fase 5).
- Otros idiomas además del español.

---

## 17. Riesgos del MVP

| Riesgo | Impacto | Mitigación |
|--------|---------|------------|
| El generador procedural produce mercados poco creíbles | Alto: rompe la inmersión educativa | Plantillas de escenario calibradas (Bloque 10); validación estadística de los paths (distribución de retornos, rachas, mechas) |
| El usuario percibe el simulador como juego de azar | Alto: contradice la identidad | Score 100% basado en LCC; feedback que siempre explica el "por qué"; lenguaje serio |
| Determinismo roto entre plataformas (Android vs iOS) | Alto: replays y rankings inválidos | PRNG propio en TS puro, sin float no determinista en la generación; tests de hash cross-platform |
| Alcance creciente ("¿y si agregamos margen?") | Medio: retrasa el MVP | Este documento es el contrato de alcance; todo lo nuevo va a fases |
| Rendimiento del gráfico en gama baja LATAM | Medio | Presupuesto de rendimiento desde Fase 1; pruebas en dispositivos económicos |
| Corrupción del archivo de export | Medio | Checksums, esquema versionado, validación al importar |
| Sobrecarga educativa (demasiado texto) | Medio | Lecciones cortas, una idea por lección, simulación antes que teoría |

---

## 18. Estrategia de testing del MVP

1. **Tests unitarios del motor (prioridad máxima):** mismo seed ⇒ mismo path, mismo hash, en cada plataforma y en cada build. Es el test más importante de todo el proyecto.
2. **Tests de contrato (LCC):** dado un log de decisiones fijo sobre un seed fijo, el score y los errores detectados son siempre idénticos.
3. **Tests de simulación de cuenta:** spread, slippage, fees, P&L y drawdown verificados contra cálculos manuales.
4. **Tests de persistencia:** export → import ⇒ estado equivalente; interrupciones a mitad de sesión no corrompen datos.
5. **Tests de replay:** sesión repetida con mismo seed valida hash; revisión de errores reproduce decisiones en las velas correctas.
6. **Pruebas de usabilidad con principiantes LATAM:** ¿entienden spread y slippage tras M2? ¿el tono se siente serio y no casino?
7. **Pruebas de dispositivo:** gama baja/media Android, verificación de la paleta en pantallas OLED y LCD.

---

## 19. Criterios de éxito del MVP

El MVP está completo cuando:

- [ ] Un usuario nuevo completa tutorial (15 lecciones), al menos 3 challenges y sesiones de sandbox sin conexión a internet en ningún momento.
- [ ] El 100% de las sesiones generan el market path antes de la primera acción del usuario y su hash es verificable después.
- [ ] Repetir un seed produce un path idéntico (hash igual) en Android e iOS.
- [ ] El score de cualquier sesión se explica completamente por el Learning Context Contract declarado.
- [ ] Export → instalación limpia → import restaura el 100% del progreso.
- [ ] En pruebas con principiantes, la mayoría explica con sus palabras spread, slippage, stop loss, drawdown y risk/reward después del tutorial.
- [ ] Cero pantallas con verdes/rojos estridentes; paleta Burgundy aplicada en toda la app.
- [ ] Ninguna función excluida (login, monetización, broker, AI coach, histórico) presente en el build.

---

# Roadmap por fases

## Fase 1 — Prototipo: Motor de Seeds Sintético

- **Objetivo:** probar que el corazón de Burgundy funciona: generación determinista + contrato de aprendizaje + replay.
- **Features:** plantillas de escenario; Learning Context Contracts declarativos; generador de seeds; generador procedural de velas; replay con el mismo seed; scoring básico contra contrato.
- **Trabajo técnico:** PRNG determinista propio; formato universal de vela; motor como librería TS pura; hash de paths; harness de tests de determinismo; prototipo mínimo de gráfico de velas con la paleta.
- **Contenido educativo:** 2–3 escenarios de prueba con contratos simples (stop obligatorio, riesgo máximo).
- **Riesgos:** determinismo cross-platform; calidad de los paths sintéticos.
- **Criterios de completitud:** mismo seed ⇒ mismo hash en Android e iOS; un escenario completo jugable de inicio a score; replay exacto demostrado.

## Fase 2 — MVP: Tutorial + Challenge Seeds

- **Objetivo:** convertir el prototipo en producto educativo completo (el MVP de este documento).
- **Features:** seeds fijos de tutorial (15 lecciones); 6 challenges con seeds fijos; rankings locales; high scores; replay de errores (revisión anotada); progresión XP/niveles/desbloqueos; racha diaria.
- **Trabajo técnico:** persistencia SQLite completa; registro de seeds y logs de decisiones; sistema de detección de errores; UI completa del HUD Burgundy; migraciones de esquema.
- **Contenido educativo:** los 5 módulos del MVP (M1–M5) con quizzes y mini-simulaciones.
- **Riesgos:** alcance creciente; sobrecarga de texto; rendimiento del gráfico.
- **Criterios de completitud:** todos los criterios de éxito de la sección 19.

## Fase 3 — Sandbox con Seeds Aleatorios + Portabilidad

- **Objetivo:** práctica libre reproducible y progreso portable.
- **Features:** generación aleatoria de sandbox; guardar seed; repetir seed; historial de sesiones; export/import de sesión y de progreso completo.
- **Trabajo técnico:** formato de archivo de export versionado con checksums; flujo de import con validación; gestión de conflictos (reemplazar/conservar).
- **Contenido educativo:** guías de "cómo practicar con propósito" en sandbox.
- **Riesgos:** corrupción de archivos; usuarios esperando merge de progresos.
- **Criterios de completitud:** export → wipe → import sin pérdida; seed de sandbox repetible con hash idéntico.

## Fase 4 — Expansión Educativa

- **Objetivo:** profundidad de academia.
- **Features:** más lecciones (gestión de posición, psicología de pérdidas, sesgos comunes); más tipos de mercado sintético; más plantillas de escenario; más reglas de detección de errores.
- **Trabajo técnico:** pipeline de contenido declarativo (lecciones y contratos como datos); ampliación del catálogo de plantillas del motor.
- **Contenido educativo:** módulos M6+ (análisis de velas avanzado, contexto de mercado, planes de trading completos).
- **Riesgos:** inconsistencia de tono al crecer el contenido; deuda en el sistema de detección de errores.
- **Criterios de completitud:** agregar una lección nueva no requiere cambios en el motor; catálogo de errores ampliado y testeado.

## Fase 5 — Mercados Avanzados

- **Objetivo:** mecánica de mercado más realista para usuarios avanzados.
- **Features:** apalancamiento y margen (con educación de riesgo obligatoria previa); mecánica estilo futuros; modelos adicionales de volatilidad y liquidez; challenges avanzados.
- **Trabajo técnico:** extensión del modelo de cuenta (margen, llamadas de margen simuladas); nuevos perfiles de instrumento; recalibración de slippage por liquidez.
- **Contenido educativo:** módulo completo de apalancamiento ("controlar una posición mayor con menos capital aumenta riesgo y retorno potencial") con desbloqueo condicionado a demostrar disciplina.
- **Riesgos:** que el apalancamiento gamifique la imprudencia — mitigado con desbloqueo por mérito y contratos estrictos.
- **Criterios de completitud:** liquidación simulada correcta y testeada; apalancamiento inaccesible sin completar la educación previa.

## Fase 6 — Formato de Importación Histórico-Compatible

- **Objetivo:** preparar la infraestructura para datos reales sin ingerirlos aún.
- **Features:** formato externo de datos de velas (esquema documentado); metadatos de fuente histórica; validación de paths importados; compatibilidad de replay con paths externos.
- **Trabajo técnico:** parser y validador del formato; el motor acepta paths externos además de seeds (misma interfaz de vela universal); hashes para paths importados.
- **Contenido educativo:** ninguno nuevo; documentación del formato.
- **Riesgos:** acoplar el motor al formato externo — mitigado manteniendo la interfaz de vela universal como única frontera.
- **Criterios de completitud:** un archivo de velas externo válido se reproduce en el simulador igual que un path sintético; archivos inválidos se rechazan con mensajes claros.

## Fase 7 — Modo Replay Histórico (opcional)

- **Objetivo:** operar sobre segmentos reales del mercado.
- **Features:** importar velas históricas; replay de segmentos reales; operar sobre datos históricos con la misma mecánica de cuenta; comparación de decisiones del usuario entre intentos sobre el mismo segmento.
- **Trabajo técnico:** gestión de almacenamiento de datasets locales; anonimización/atribución de la fuente en metadatos; contratos de aprendizaje aplicables a segmentos reales.
- **Contenido educativo:** lecciones de contexto ("qué pasó en este segmento y por qué es instructivo").
- **Riesgos:** licenciamiento de datos (responsabilidad del usuario que importa, la app no distribuye datos); falsa sensación de que el pasado predice el futuro — mitigado con mensajes educativos explícitos.
- **Criterios de completitud:** un segmento real importado es operable, repetible y comparable entre intentos; la app funciona íntegramente sin ningún dato histórico (sigue siendo opcional).

## Fase 8 — Extracción de Patrones e Histórico-Inspirados

- **Objetivo:** cerrar el ciclo: aprender de mercados reales para generar escenarios sintéticos mejores.
- **Features:** análisis de segmentos reales; detección de patrones (rangos, tendencias, mechas de liquidez, gaps); extracción de plantillas; generador de escenarios histórico-inspirados con seeds deterministas.
- **Trabajo técnico:** módulo de análisis estadístico local; conversión patrón → plantilla de escenario; integración con el catálogo de plantillas y el sistema de seeds existente.
- **Contenido educativo:** escenarios "inspirados en condiciones reales" con su contexto explicado.
- **Riesgos:** sobreajuste de plantillas a eventos únicos; costo computacional en móviles — mitigado con análisis offline por lotes y validación estadística.
- **Criterios de completitud:** una plantilla extraída genera paths sintéticos deterministas indistinguibles en calidad de las plantillas manuales; todo el pipeline corre localmente sin nube.

---

## Lógica del roadmap en una línea

> Primero el motor determinista (Fase 1), luego la academia (Fases 2–4), luego la profundidad de mercado (Fase 5), y solo al final los datos reales (Fases 6–8) — porque Burgundy enseña disciplina, y la disciplina no necesita datos históricos para entrenarse, pero sí un motor que jamás mienta sobre el precio.

---

*Documento del proyecto Burgundy, firmado por **tsuloid**. Aplicación educativa, sin asesoría financiera, sin dinero real, sin conexión a brokers.*
