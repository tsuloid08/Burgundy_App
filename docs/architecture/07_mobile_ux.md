# Burgundy — Especificación de UX Móvil (Prompt 7)

**Proyecto:** Burgundy — Simulador educativo de trading
**Autor / Firma:** tsuloid
**Documento:** `docs/architecture/07_mobile_ux.md`
**Idioma:** Español (LATAM)
**Alcance:** UX, arquitectura de información, propósito de pantallas y flujos de usuario. Sin código, sin mockups, sin assets finales de UI.

---

## 0. Principios rectores de UX

Burgundy debe sentirse como **una academia seria de trading dentro de una terminal de mercado oscura con tema Burgundy**, con progresión de largo plazo tipo juego. Estos principios gobiernan toda decisión de UX:

1. **Educación antes que emoción.** Cada pantalla enseña algo o evalúa una decisión. Nada está diseñado para generar adrenalina.
2. **El mercado no se manipula.** El camino del mercado se genera **antes** de la primera operación del usuario. La UX debe comunicar esto de forma explícita y constante para construir confianza.
3. **Se evalúan decisiones, no solo ganancias.** El usuario puede ganar dinero simulado con un proceso malo y la app debe señalarlo.
4. **Beginner-safe.** El usuario LATAM principiante es vulnerable: la app advierte sobre riesgo excesivo, falta de stop loss, apalancamiento peligroso y conductas emocionales.
5. **Terminología real, explicada en claro.** Términos como *spread*, *slippage*, *drawdown*, *leverage*, *liquidity* y *risk/reward* se usan en inglés (estándar) pero siempre con explicación en español simple.
6. **Offline-first, sin login, sin nube.** Todo el progreso vive en el dispositivo; la exportación/importación de progreso es una función de primera clase, no un ajuste escondido.
7. **Estética prohibida:** verdes y rojos estridentes, negro puro `#000000`, visuales de casino, UI infantil, mensajes de "hazte rico rápido".
8. **Paleta obligatoria:**

| Rol visual | Color | Uso |
|---|---|---|
| Fondo profundo | `#1A1617` Deep Charcoal | Fondos de todas las pantallas (nunca negro puro) |
| Acento primario | `#571324` Matte Burgundy | Botones primarios, navegación activa, marcadores de progreso |
| Superficies y divisores | `#2E2E2E` | Tarjetas, paneles, separadores |
| Resaltado crítico | `#C9A050` Muted Gold | Tipografía de énfasis, alertas educativas, logros, XP |
| Velas alcistas | `#4A6D56` Custom Muted Green | Cuerpo y mecha de velas alcistas |
| Velas bajistas | `#802F3E` Custom Muted Burgundy-Red | Cuerpo y mecha de velas bajistas |

---

## 1. Estructura completa de navegación de la app

### 1.1 Modelo de navegación

- **Navegación principal:** barra inferior de 5 pestañas (estándar móvil, alcanzable con el pulgar).
- **Navegación secundaria:** stacks por pestaña (pantallas apiladas con retorno).
- **Pantallas modales de pantalla completa:** simulador de trading, briefing de escenario, evaluación final, confirmación de operación. Entran como modales porque representan una "sesión" con inicio y fin claros.
- **Hojas inferiores (bottom sheets):** calculadora de riesgo, panel de orden, glosario contextual de términos, advertencias educativas.

### 1.2 Mapa de navegación (arquitectura de información)

```
Burgundy (raíz)
│
├── [Primera apertura únicamente]
│   ├── Onboarding (3–5 pasos)
│   ├── Pantalla de encuadre educativo ("esto no es asesoría financiera")
│   ├── Test de conocimientos para principiantes
│   └── Selección de mercado inicial
│
├── Pestaña 1: ACADEMIA (inicio por defecto)
│   ├── Mapa de tutorial / ruta de aprendizaje
│   ├── Pantalla de lección
│   ├── Briefing de escenario
│   ├── → Simulador de trading (modal de sesión)
│   ├── Evaluación final de sesión
│   └── Revisión de errores
│
├── Pestaña 2: SIMULADOR
│   ├── Modo Sandbox (semilla aleatoria)
│   ├── Modo Libre (configuración propia)
│   ├── Modo Sin Indicadores
│   ├── Selección de mercado y horizonte temporal
│   ├── Briefing de escenario
│   └── → Simulador de trading (modal de sesión)
│
├── Pestaña 3: DESAFÍOS
│   ├── Lista de desafíos (semilla fija, bloqueados/desbloqueados)
│   ├── Desafíos de supervivencia
│   ├── Desafíos de disciplina de riesgo
│   ├── Desafíos sin indicadores
│   ├── Briefing de desafío (con aviso de semilla bloqueada)
│   └── → Simulador de trading (modal de sesión)
│
├── Pestaña 4: PROGRESO
│   ├── Panel de desempeño (dashboard)
│   ├── Diario de operaciones (trade journal)
│   ├── Revisión de errores (histórico)
│   ├── Rankings locales de sesiones
│   ├── Puntajes máximos guardados
│   └── Gestión de semillas y repeticiones (seed/replay)
│
└── Pestaña 5: PERFIL
    ├── Nivel, XP, racha diaria, desbloqueos
    ├── Exportar / importar progreso
    ├── Ajustes
    └── Información legal y encuadre educativo (re-accesible)
```

### 1.3 Reglas de navegación

- El simulador de trading **siempre** se abre desde un briefing de escenario; nunca de forma directa sin contexto.
- Salir del simulador a mitad de sesión exige confirmación ("Tu sesión se guardará como incompleta").
- La barra inferior se oculta dentro del simulador para maximizar el área del gráfico.
- El botón "atrás" del sistema en Android nunca descarta una operación abierta sin confirmación.

---

## 2. Propósito de cada pantalla requerida

| # | Pantalla | Propósito | Elementos clave |
|---|---|---|---|
| 1 | **Onboarding** | Presentar Burgundy como academia seria; fijar expectativas; explicar que todo es local y sin login. | 3–5 pasos: identidad, filosofía de simulación, almacenamiento local, exportación de progreso. Tono sobrio, fondo `#1A1617`, acentos `#571324`. |
| 2 | **Encuadre educativo** | Declarar de forma inequívoca: simulador educativo, sin dinero real, sin asesoría financiera, sin conexión a brokers. | Texto fijo no saltable en primera apertura; checkbox "Entiendo que esto es educativo"; re-accesible desde Perfil. |
| 3 | **Test de conocimientos** | Medir el nivel inicial para calibrar la ruta de aprendizaje (no para excluir). | 8–12 preguntas; resultado en lenguaje neutro ("Vas a empezar desde los fundamentos"), nunca humillante. |
| 4 | **Selección de mercado** | Elegir el tipo de mercado simulado (ej. divisas, índices, cripto simulado). | Tarjetas `#2E2E2E` con descripción de volatilidad y liquidez típicas en español claro. |
| 5 | **Mapa de tutorial / ruta de aprendizaje** | Núcleo de la academia: progresión visual de niveles, lecciones y desbloqueos. | Nodos conectados; estados bloqueado/disponible/completado; XP y nivel visibles; estética de mapa de academia, no de juego infantil. |
| 6 | **Pantalla de lección** | Enseñar un concepto con terminología real explicada en claro. | Texto + diagramas estáticos; glosario contextual (tocar un término abre su definición); micro-quiz de cierre. |
| 7 | **Briefing de escenario** | Dar contexto antes de cada sesión: objetivo, reglas, capital inicial e identidad del escenario. | Objetivo de aprendizaje, condiciones del mercado, tipo de semilla (fija/aleatoria), mensaje de confianza sobre la generación previa del mercado. En tutorial/sandbox puede mostrarse la semilla o un identificador legible derivado; en desafíos el briefing muestra **solo el sello de equidad** (`pathHash`, `seedType`, `generatorVersion`, reglas del LCC), nunca la seed cruda — `SEED_PATH_REPLAY_EXPORT_LOCK` (documento dueño: `08_technical_architecture.md`, §8). |
| 8 | **Simulador de trading** | Pantalla central de práctica. Ver sección 10 para prioridades de layout. | Gráfico de velas dominante + paneles colapsables. |
| 9 | **Vista de gráfico** | Visualización de velas con colores propios (`#4A6D56` / `#802F3E`), zoom y desplazamiento táctil. | Crosshair con precio/tiempo; marcadores de entradas/salidas; marcador de replay si aplica. |
| 10 | **Panel de orden** | Configurar y confirmar órdenes. | Compra/venta, tamaño, stop loss, take profit, % de riesgo, vista previa de risk/reward, spread, fees estimados, slippage estimado. |
| 11 | **Panel de posiciones** | Ver y gestionar posiciones abiertas. | P/L abierto por posición, distancia a SL/TP, botón de cierre con confirmación. |
| 12 | **Panel de cuenta/equity** | Estado financiero de la sesión simulada. | Balance, equity, margen disponible (si hay leverage), drawdown actual desde el pico. |
| 13 | **Calculadora de riesgo** | Traducir % de riesgo a tamaño de posición antes de operar. | Entrada: % de riesgo y distancia del stop; salida: tamaño sugerido; advertencia dorada `#C9A050` si supera el umbral seguro. |
| 14 | **Diario de operaciones** | Registro automático + notas del usuario por operación. | Razón de entrada (selección rápida + texto libre), emoción percibida, etiquetas de error detectado por la app. |
| 15 | **Controles de velocidad de simulación** | Pausar, avanzar vela a vela, x1, x2, x5. | Siempre visibles en el simulador; pausa automática en eventos de tutorial. |
| 16 | **Selector de horizonte temporal** | Elegir duración de la sesión simulada y timeframe de velas. | Se elige en el briefing; dentro de la sesión solo cambia la vista del timeframe, no el camino generado. |
| 17 | **Panel de desempeño** | Dashboard de largo plazo: métricas de proceso, no solo de ganancia. | Win rate, risk/reward promedio, drawdown máximo, adherencia al stop loss, frecuencia de errores, evolución de XP. |
| 18 | **Evaluación final** | Cierre de cada sesión: calificación de proceso y resultado por separado. | Dos puntajes: "Resultado" y "Calidad de decisiones"; desglose de errores; XP ganada; acceso directo a revisión de errores. |
| 19 | **Revisión de errores** | Aprender de las fallas con replay puntual. | Línea de tiempo de la sesión con marcadores de error; tocar un error reproduce el momento exacto sobre el gráfico con explicación. |
| 20 | **Modo Sandbox** | Práctica libre con semilla aleatoria, sin objetivos impuestos. | Aviso explícito de semilla aleatoria; opción de guardar la semilla para repetir. |
| 21 | **Modo Desafío** | Escenarios con semilla fija, reglas estrictas y ranking local. | Aviso de semilla bloqueada; reglas visibles antes de iniciar; reinicio reinicia el mismo escenario. |
| 22 | **Rankings locales** | Comparar las propias sesiones entre sí (no hay nube ni otros usuarios). | Ranking por desafío y por modo; ordenable por puntaje de proceso o resultado. |
| 23 | **Exportar / importar progreso** | Continuidad del progreso sin cuentas en la nube. | Exportar genera un archivo local; importar valida y previsualiza antes de aplicar; advertencia clara si la importación reemplaza progreso. |
| 24 | **Gestión de semillas / replay** | Biblioteca de semillas guardadas y sesiones repetibles. | Lista de semillas con origen (tutorial/desafío/sandbox), fecha, y acciones: repetir, ver sesiones asociadas, eliminar. |
| 25 | **Ajustes** | Configuración local. | Tamaño de fuente, reducción de movimiento, haptics, retención de datos, restablecimiento (con doble confirmación), créditos firmados por **tsuloid**. |

---

## 3. Flujos primarios de usuario

### 3.1 Flujo de primera apertura

```
Abrir app → Onboarding → Encuadre educativo (aceptación obligatoria)
→ Test de conocimientos → Resultado y calibración de ruta
→ Selección de mercado → Mapa de tutorial (Academia)
```

Regla: el usuario llega a su primera lección en menos de 4 minutos. El test puede posponerse una sola vez ("Hacerlo después"), pero el encuadre educativo no.

### 3.2 Flujo de tutorial

```
Mapa de tutorial → Seleccionar nodo disponible → Lección (concepto + glosario + micro-quiz)
→ Briefing de escenario (semilla FIJA de tutorial, mensaje de confianza visible)
→ Simulador con overlay de pistas (hints paso a paso, pausas automáticas)
→ Evaluación final (proceso + resultado) → Revisión de errores (si los hubo)
→ XP / desbloqueo → Regreso al mapa con el siguiente nodo disponible
```

Reglas UX del tutorial:
- Las semillas de tutorial son **fijas y deterministas**: todos los usuarios ven el mismo mercado en la misma lección, lo que permite que las pistas referencien momentos exactos.
- El overlay de pistas es descartable pero re-invocable con un botón persistente.
- No se puede saltar la evaluación final del tutorial.

### 3.3 Flujo de sandbox

```
Pestaña Simulador → Modo Sandbox → Configurar mercado y horizonte temporal
→ Briefing breve con AVISO DE SEMILLA ALEATORIA
→ Simulador (sin pistas obligatorias, advertencias beginner-safe activas)
→ Evaluación final → Opción: guardar semilla / repetir misma semilla / nueva semilla
```

Aviso obligatorio en el briefing sandbox:
> "Este es un mercado nuevo generado al azar. Puedes guardar la semilla si quieres volver a practicar exactamente el mismo escenario."

### 3.4 Flujo de desafío

```
Pestaña Desafíos → Elegir desafío (bloqueado si falta nivel/desbloqueo)
→ Briefing de desafío: reglas, condición de victoria/derrota, AVISO DE SEMILLA BLOQUEADA
→ Simulador (reglas estrictas: ej. riesgo máximo 2%, stop loss obligatorio)
→ Evaluación final → Registro en ranking local → Comparación contra puntaje máximo previo
```

Aviso obligatorio en el briefing de desafío:
> "Este desafío usa una semilla fija. El mercado será idéntico cada vez que lo intentes: lo que cambia son tus decisiones."

### 3.5 Flujo de reinicio de sesión (incluye cuenta en cero)

```
[Durante la sesión] Equity llega a cero o se viola una regla de desafío
→ Pantalla de cierre forzado: explicación en tono educativo, sin humillar
   ("Tu cuenta simulada llegó a cero. Esto le pasa a la mayoría de los principiantes
    que operan sin control de riesgo. Revisemos qué decisiones lo causaron.")
→ Revisión de errores obligatoria (resumida)
→ Opciones: [Reiniciar mismo escenario (misma semilla)] [Nuevo escenario] [Salir al menú]
```

Regla: el reinicio nunca está a un solo toque desde la derrota; siempre pasa por la revisión resumida. Esto evita el patrón de casino de "girar de nuevo".

### 3.6 Flujo de replay con la misma semilla

```
Gestión de semillas (o Evaluación final) → "Repetir misma semilla"
→ Briefing con MARCADOR DE REPLAY visible
→ Simulador con badge persistente "REPLAY — mismo mercado, nuevas decisiones"
→ Evaluación final comparativa: sesión actual vs. mejor sesión previa con esa semilla
```

Distinción UX obligatoria: el badge de replay y la comparación contra sesiones previas dejan claro que **repetir la misma semilla** ≠ **generar un mercado nuevo**.

### 3.7 Flujo de generación de nueva semilla

```
Evaluación final o Modo Sandbox → "Generar nuevo escenario"
→ Animación sobria de generación (sin ruleta, sin slot machine)
→ Briefing con nueva semilla y mensaje de confianza
→ Simulador
```

La "animación de generación" es una barra de progreso sobria con el texto: "Generando el camino del mercado… Este camino queda fijado antes de tu primera operación."

### 3.8 Flujo de exportar / importar progreso

**Exportar:**
```
Perfil → Exportar progreso → Resumen de lo que se exporta
(nivel, XP, racha, journal, semillas guardadas, rankings, ajustes)
→ Generar archivo → Compartir/guardar con el sistema operativo
→ Confirmación con fecha y nombre del archivo
```

**Importar:**
```
Perfil → Importar progreso → Selector de archivo del sistema
→ Validación (archivo corrupto o incompatible → estado de error claro)
→ PREVISUALIZACIÓN: "Este archivo contiene: Nivel 7, 4.520 XP, 38 sesiones, exportado el 02/06/2026"
→ Advertencia: "Importar reemplazará tu progreso actual. Esta acción no se puede deshacer."
→ Confirmación de doble paso → Importación → Confirmación final
```

---

## 4. Pantalla de trading: prioridades de layout (mobile-first)

Orden de prioridad visual de arriba hacia abajo, pensado para uso con una mano en vertical:

| Prioridad | Zona | Contenido | Comportamiento |
|---|---|---|---|
| 1 | **Gráfico de velas** (50–60% de la pantalla) | Velas `#4A6D56`/`#802F3E`, precio actual con línea punteada, marcadores de entrada/salida, marcador de replay si aplica. | Zoom con pellizco, desplazamiento horizontal, crosshair con toque sostenido. |
| 2 | **Barra superior compacta** | Mercado, timeframe seleccionado, estado del escenario (ej. "Desafío — Regla: riesgo máx. 2%"), badge REPLAY, icono de info de semilla (opcional, toque abre detalle). | Una línea; nunca compite con el gráfico. |
| 3 | **Franja de cuenta** | Balance, equity, P/L abierto, margen disponible (solo si el leverage está habilitado en el escenario). | Cifras tabulares; P/L con los colores muted de la paleta, jamás verde/rojo estridente. |
| 4 | **Botones Comprar / Vender** | Dos botones grandes (mínimo 48dp de alto), comprar a la izquierda, vender a la derecha; muestran spread entre ambos. | Abren el panel de orden como bottom sheet; nunca ejecutan con un solo toque. |
| 5 | **Panel de orden** (bottom sheet) | Tamaño de orden, stop loss, take profit, % de riesgo, vista previa de risk/reward, spread, fees estimados, slippage estimado. | La vista previa de risk/reward se actualiza en vivo. Botón final: "Confirmar operación" con resumen completo. |
| 6 | **Controles de simulación** | Pausa, paso a paso (vela a vela), x1 / x2 / x5. | Fila flotante compacta sobre el borde inferior del gráfico. |
| 7 | **Paneles secundarios** (pestañas internas colapsables) | Posiciones abiertas, calculadora de riesgo, journal rápido, panel de cuenta extendido. | Colapsados por defecto para no abrumar al principiante. |
| 8 | **Overlay de tutorial** (solo en tutorial) | Pistas contextuales ancladas al elemento relevante, con pausa automática de la simulación. | Botón persistente "💡" para re-invocar pistas. |

**Confirmación de operación (obligatoria):** ningún trade se ejecuta sin un resumen previo que muestre: dirección, tamaño, riesgo en % y en dinero simulado, SL, TP, risk/reward, spread, fees y slippage estimados. En tutorial y desafíos no se puede desactivar; en sandbox el usuario puede activar "confirmación rápida" solo a partir de cierto nivel desbloqueado.

### 4.1 Presupuesto de render y densidad por defecto (`BEGINNER_HUD_LOCK`)

<!-- LOCK: BEGINNER_HUD_LOCK v1 — Documento dueño: 07_mobile_ux.md. Resuelve la mitad de render de AUD-013 (presupuesto de render y densidad por defecto del HUD del simulador). Los documentos 08, 12 y 13 referencian este lock por nombre, sin copiarlo. Ante contradicción entre cualquier documento y este lock, gana el lock. -->

> **BEGINNER_HUD_LOCK v1 — Presupuesto de render y densidad por defecto del HUD del simulador (cerrado).**
>
> **Presupuesto de render del simulador (MVP):**
> - **Ventana deslizante de ~120 velas visibles** como techo de render; las velas fuera del viewport se deciman en zoom-out y nunca se dibujan completas.
> - **Objetivo 60 fps** de playback en gama media; **fallback documentado a 30 fps** en gama baja, con degradación elegante: se reduce la frecuencia de actualización del crosshair y la animación de sub-ticks de la vela en formación. **Nunca se degradan** la exactitud de los datos, la ejecución de órdenes ni la detección de errores.
> - Benchmark mínimo de aceptación: un dispositivo Android de gama media/baja (3–4 GB RAM, perfil LATAM) sostiene el playback a ≥30 fps con la ventana de ~120 velas.
> - Los techos de generación (máximo de velas por sesión según horizonte y 4–8 sub-ticks por vela) pertenecen a `MVP_SANDBOX_LIMITS` (documento 12); este lock gobierna el render y el HUD.
>
> **Densidad por defecto del HUD (MVP, todos los modos):**
> - Por defecto, el principiante ve únicamente: **chart + franja mínima de cuenta/riesgo + acción principal (Comprar/Vender) + controles de velocidad**.
> - **Todos los paneles avanzados** (posiciones, calculadora de riesgo, journal rápido, cuenta extendida) están **colapsados por defecto** y se abren bajo demanda como bottom sheets o pestañas internas.
> - Este lock fija únicamente el presupuesto de render y la densidad por defecto. La reorganización completa de densidad Beginner/Expanded (AUD-015, gravedad Media) queda explícitamente fuera de su alcance: aquí no se rediseñan pantallas.

---

## 5. Advertencias beginner-safe (sistema de alertas educativas)

Las advertencias usan superficies `#2E2E2E` con borde y texto `#C9A050`. Nunca bloquean con pánico: explican, advierten y dejan decidir (excepto donde una regla de desafío lo prohíba). Cada advertencia tiene un enlace "¿Por qué importa?" que abre la explicación educativa.

| Disparador | Momento | Mensaje (resumen) | Comportamiento |
|---|---|---|---|
| Riesgo demasiado alto (> umbral del escenario) | Al configurar la orden | "Estás arriesgando X% de tu cuenta. Los traders disciplinados rara vez superan el 1–2% por operación." | Advertencia + confirmación adicional. En desafíos con regla de riesgo: bloqueo. |
| Sin stop loss | Al confirmar la orden | "Vas a entrar sin stop loss. Sin un límite de pérdida definido, una sola operación puede vaciar tu cuenta." | Confirmación adicional obligatoria; el error queda etiquetado en el journal. |
| Apalancamiento peligroso | Al seleccionar leverage alto | "Con este leverage, un movimiento pequeño en contra puede liquidar tu posición. El leverage amplifica pérdidas y ganancias por igual." | Advertencia + explicación del término. |
| Entrada tras volatilidad extrema | Al abrir orden justo después de velas de rango anómalo | "El mercado acaba de moverse de forma extrema. Entrar ahora suele implicar peor slippage y más riesgo." | Advertencia informativa. |
| Señal copiada sin ajustar riesgo | Al usar parámetros de un escenario de "señal" sin adaptar tamaño/SL | "Copiaste una señal sin ajustar el riesgo a tu cuenta. Una señal ajena con el riesgo equivocado sigue siendo tu pérdida." | Advertencia educativa (escenarios específicos anti-Telegram/TikTok). |
| Overtrading | N operaciones en ventana corta de la sesión | "Llevas X operaciones en poco tiempo. Operar de más suele responder a impulso, no a análisis." | Advertencia + sugerencia de pausa; queda registrado como patrón en el dashboard. |
| Revenge trading | Nueva entrada inmediata tras una pérdida, con tamaño mayor | "Acabas de perder y estás entrando de nuevo con más tamaño. Esto se llama revenge trading y destruye cuentas." | Advertencia destacada + confirmación adicional. |
| Mover el stop loss emocionalmente | Alejar el SL cuando el precio se acerca a él | "Estás alejando tu stop loss mientras pierdes. Esto convierte una pérdida pequeña en una grande." | Advertencia + el movimiento queda etiquetado como error en la revisión. |
| Operar durante alta volatilidad | Orden durante segmento marcado como alta volatilidad del escenario | "La volatilidad actual es alta: el spread y el slippage estimados aumentaron." | Informativa; actualiza la vista previa de costos. |

Regla transversal: cada advertencia ignorada se registra y aparece en la **revisión de errores** y en el **panel de desempeño** como patrón de conducta, alimentando la calificación de "Calidad de decisiones".

---

## 6. UX de semillas deterministas y contexto de aprendizaje

Requisitos obligatorios:

1. **Visualización de semilla** — regida por `SEED_PATH_REPLAY_EXPORT_LOCK` (documento dueño: `08_technical_architecture.md`, §8), con dos regímenes distintos:
   - **Tutorial / sandbox:** la semilla (o un identificador corto legible derivado de ella) es visible en el briefing, opcionalmente en la barra superior del simulador, y siempre en la evaluación final.
   - **Desafíos:** antes del intento el briefing muestra **únicamente el sello de equidad**: `pathHash`, `seedType`, `generatorVersion` y las reglas del LCC — **nunca la seed cruda**. La seed cruda se revela solo al cerrar el intento (evaluación final). Si el usuario conocía la seed de antemano o repite con una seed conocida, el intento se marca `seed_known = true` y queda fuera del ranking principal de primer intento.
2. **Botón "Repetir sesión" (replay):** disponible en la evaluación final y en la gestión de semillas.
3. **Botón "Guardar semilla":** disponible en sandbox y en la evaluación final.
4. **"Reiniciar mismo escenario":** misma semilla, decisiones desde cero (ver flujo 3.6).
5. **"Generar nuevo escenario":** semilla nueva, claramente diferenciado del reinicio (ver flujo 3.7).
6. **Aviso de semilla bloqueada en desafíos:** texto fijo en el briefing (ver flujo 3.4).
7. **Aviso de semilla aleatoria en sandbox:** texto fijo en el briefing (ver flujo 3.3).
8. **Línea de tiempo de replay de errores:** en la revisión de errores, una línea de tiempo de la sesión con marcadores; tocar uno reproduce ese tramo del mercado con la decisión del usuario superpuesta.
9. **Mensaje de confianza:** presente en cada briefing, textual y sin tecnicismos:

> **"Este escenario fue generado antes de tu primera operación. Tus decisiones no cambian el camino del mercado; solo afectan tus órdenes, riesgo, resultados y evaluación."**

10. **Distinción fija vs. aleatoria:** iconografía y etiqueta consistentes — 🔒 "Semilla fija" (tutorial/desafío) vs. 🎲 "Semilla aleatoria" (sandbox) — con los mismos términos en toda la app.
11. **Distinción replay vs. mercado nuevo:** el badge "REPLAY" persistente y la evaluación comparativa solo existen en replays; un mercado nuevo nunca muestra comparación contra sesiones de otra semilla.
12. **Explicación anti-manipulación:** accesible desde el icono de info de semilla: "Burgundy nunca cambia la dirección del mercado después de tu entrada. Si pierdes, no es porque la app te haya puesto en contra: el camino ya estaba fijado."
13. **Explicación de evaluación de proceso:** en la evaluación final, junto al doble puntaje: "Burgundy evalúa tus decisiones, no solo tu ganancia. Una ganancia con mal proceso es suerte; una pérdida con buen proceso es aprendizaje."

---

## 7. Reglas de usabilidad móvil

1. **Zona del pulgar:** acciones primarias (comprar/vender, confirmar, controles de velocidad) en el tercio inferior de la pantalla.
2. **Tamaños táctiles:** mínimo 44×44 pt (iOS) / 48×48 dp (Android) para todo elemento interactivo; los botones de compra/venta y de confirmación son mayores.
3. **Orientación:** vertical como modo principal; horizontal opcional solo para la vista de gráfico ampliada.
4. **Una decisión por pantalla:** los bottom sheets de orden y riesgo presentan una tarea a la vez; nunca formularios largos.
5. **Cifras tabulares:** todos los números financieros usan tipografía de dígitos de ancho fijo para evitar saltos visuales.
6. **Sin parpadeos ni pulsos:** los cambios de P/L se actualizan sin animaciones de destello; el dorado `#C9A050` se reserva para énfasis educativo y logros, no para "ganancias".
7. **Gestos con alternativa:** todo gesto (pellizco, deslizamiento) tiene un equivalente por botón.
8. **Estado persistente:** si la app se cierra a mitad de sesión, al volver se ofrece reanudar exactamente donde quedó (offline-first).
9. **Carga percibida:** ninguna espera sin indicador; la generación de escenario muestra progreso con el mensaje de confianza.
10. **Confirmaciones destructivas:** cerrar posición, abandonar sesión, restablecer datos e importar progreso requieren confirmación explícita; restablecer e importar requieren doble paso.
11. **Texto en español LATAM:** voseo neutro evitado; se usa "tú" estándar LATAM; sin jerga local de un solo país.
12. **Densidad progresiva:** el principiante ve menos paneles; los paneles avanzados (margen, slippage detallado) aparecen según el nivel o el escenario lo requiera. La densidad por defecto del MVP y el presupuesto de render están cerrados por `BEGINNER_HUD_LOCK` (sección 4.1).

---

## 8. Consideraciones de accesibilidad

1. **Contraste:** el dorado `#C9A050` y los textos primarios sobre `#1A1617` deben cumplir WCAG AA (≥ 4.5:1 para texto normal). Los textos sobre `#571324` se verifican caso por caso; si no alcanzan AA se usa texto claro.
2. **No solo color:** las velas alcistas/bajistas y el P/L positivo/negativo se distinguen también por forma o signo (▲/▼, +/−), nunca solo por el verde/rojo muted. Crítico para daltonismo, dado que la paleta ya es de baja saturación.
3. **Escalado de fuente:** soporte de tamaños dinámicos del sistema; el layout del simulador tolera al menos +30% sin romper el gráfico.
4. **Lectores de pantalla:** etiquetas en español para todos los controles; el gráfico expone un resumen hablado ("Última vela: bajista, cierre en 1.0842; tendencia reciente: bajista").
5. **Reducción de movimiento:** ajuste que elimina animaciones de transición y reproduce el replay paso a paso.
6. **Haptics opcionales:** vibración sutil en confirmación de orden y advertencias, desactivable.
7. **Objetivos táctiles separados:** comprar y vender nunca adyacentes sin separación ≥ 8dp, para evitar toques accidentales con consecuencia educativa confusa.
8. **Tiempo no punitivo:** ninguna decisión educativa tiene cuenta regresiva agresiva; los desafíos con límite de tiempo lo declaran en el briefing y permiten pausa.

---

## 9. Estados vacíos (empty states)

Todos los estados vacíos enseñan algo y ofrecen la acción siguiente. Tono sobrio, sin mascotas ni ilustraciones infantiles.

| Pantalla | Estado vacío | Mensaje y acción |
|---|---|---|
| Diario de operaciones | Sin operaciones registradas | "Tu diario está vacío. Cada operación que hagas se registrará aquí con su contexto y tus notas." → [Ir a la Academia] |
| Panel de desempeño | Sin sesiones completadas | "Aún no hay datos de desempeño. Completa tu primera sesión para empezar a medir tu proceso." → [Empezar tutorial] |
| Rankings locales | Sin puntajes | "Todavía no tienes sesiones rankeadas. Completa un desafío para registrar tu primer puntaje." → [Ver desafíos] |
| Gestión de semillas | Sin semillas guardadas | "No has guardado semillas. Guarda una semilla al final de una sesión para repetir exactamente el mismo mercado." |
| Revisión de errores | Sesión sin errores detectados | "No detectamos errores de proceso en esta sesión. Recuerda: un buen proceso no garantiza ganancia, pero la suerte no se repite." |
| Posiciones abiertas | Sin posiciones | "No tienes posiciones abiertas. Usa Comprar o Vender para abrir una." |
| Desafíos | Todo bloqueado (usuario nuevo) | "Los desafíos se desbloquean avanzando en la Academia. Completa el nivel X para abrir el primero." |

---

## 10. Estados de error

Principio: el error técnico nunca culpa al usuario y nunca pierde su progreso silenciosamente.

| Situación | Comportamiento UX |
|---|---|
| Archivo de importación corrupto o incompatible | "No pudimos leer este archivo. Puede estar dañado o pertenecer a una versión no compatible. Tu progreso actual no fue modificado." → [Elegir otro archivo] |
| Falla al exportar (sin espacio/permiso) | Mensaje específico de la causa + reintento; el progreso local nunca se altera por una exportación fallida. |
| Cierre inesperado durante una sesión | Al reabrir: "Tu última sesión se interrumpió. ¿Quieres reanudarla o cerrarla y guardar lo avanzado?" |
| Datos locales dañados (caso extremo) | Recuperación parcial si es posible + explicación honesta; sugerencia de importar el último archivo exportado. Nunca un reseteo silencioso. |
| Semilla guardada no reproducible (cambio de versión del generador) | "Esta semilla pertenece a una versión anterior del simulador y ya no puede reproducirse de forma idéntica." → opciones: conservar como registro histórico o eliminar. |
| Acción inválida en simulador (ej. tamaño mayor al margen) | Validación en línea dentro del panel de orden, en el campo afectado, con explicación del término involucrado (ej. margen). |
| Regla de desafío violada | No es un "error técnico": pantalla de cierre educativo (ver flujo 3.5) explicando qué regla se violó y por qué existe. |

---

## 11. Alcance UX del MVP

### Incluido en el MVP

| Área | Alcance MVP |
|---|---|
| Onboarding + encuadre educativo + test inicial | Completo |
| Academia | Mapa de ruta con el primer arco de lecciones; lecciones + briefing + semillas fijas de tutorial |
| Simulador | Pantalla completa de trading: gráfico de velas, panel de orden con SL/TP/riesgo/RR/spread/fees/slippage, panel de posiciones, cuenta/equity, calculadora de riesgo, controles de velocidad, confirmación de operación |
| Modos | Tutorial, Sandbox (semilla aleatoria + guardar semilla), Desafíos iniciales (semilla fija), reinicio por cuenta en cero |
| Sistema de semillas | Semilla fija, semilla aleatoria, guardar semilla, repetir misma semilla, generar nueva, mensajes de confianza |
| Evaluación | Evaluación final con doble puntaje (resultado + proceso), revisión de errores con línea de tiempo |
| Advertencias beginner-safe | Las 9 advertencias de la sección 5 |
| Progreso | XP, niveles, racha diaria, desbloqueos básicos, dashboard de desempeño esencial, rankings locales por desafío |
| Persistencia | 100% local, exportar/importar progreso con previsualización y validación |
| Accesibilidad | Contraste AA, no-solo-color, escalado de fuente, objetivos táctiles |

### Diferido (post-MVP)

- Modo de datos históricos / replay histórico (la arquitectura de semillas ya lo contempla como fuente futura; la UX reserva el espacio en la gestión de semillas como "Semillas históricas — próximamente").
- Modo sin indicadores como desafío avanzado adicional (la UX lo define; su contenido completo puede llegar después del primer arco).
- Modo horizontal de gráfico ampliado.
- Resumen hablado avanzado del gráfico para lectores de pantalla (MVP incluye etiquetas básicas).
- Densidad progresiva avanzada por nivel (MVP usa dos densidades: principiante y estándar).

### Explícitamente fuera de alcance (por diseño del producto)

Sin dinero real, sin brokers, sin login, sin nube, sin monetización, sin cursos pagos, sin coach de IA, sin ranking entre usuarios, sin mensajería ni social.

---

*Documento de especificación UX de Burgundy — proyecto firmado por **tsuloid**. Material exclusivamente educativo; no constituye asesoría financiera.*
