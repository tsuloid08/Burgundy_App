# Burgundy — Bloque 8: Arquitectura Técnica y Stack

**Proyecto:** Burgundy — Simulador educativo de trading
**Autor / firma:** tsuloid
**Idioma:** Español (LATAM)
**Estado:** Documento de arquitectura — sin código, sin pantallas, previo al desarrollo
**Plataformas objetivo:** Android 15+ / iOS 20+ (configuración verificable: `PLATFORM_TARGET_LOCK`, §2.4) · Offline-first · Sin login · Sin cuenta en la nube

---

## 1. Evaluación de opciones tecnológicas

Antes de recomendar un stack, se evalúan las seis opciones contra los requisitos reales de Burgundy: renderizado de velas en tiempo real, reproducción de mercado simulado con velocidad ajustable, generación determinista por seed, paths pre-generados, replay, estado local, offline-first, export/import de progreso, rendimiento móvil, gestos/animaciones, mantenibilidad y velocidad de desarrollo.

### 1.1 Tabla comparativa

| Criterio | React Native | Flutter | Nativo (Swift/Kotlin) | Unity | Godot | WebView/híbrido |
|---|---|---|---|---|---|---|
| Render de gráfico de velas en tiempo real | ✅ Bueno (con Skia/`react-native-skia`) | ✅ Excelente (Skia/Impeller nativo) | ✅ Excelente | ✅ Excelente | ✅ Muy bueno | ⚠️ Aceptable (canvas), riesgo en gamas bajas |
| Playback de mercado simulado | ✅ | ✅ | ✅ | ✅ | ✅ | ⚠️ |
| Velocidad de simulación ajustable | ✅ (loop en JS/TS desacoplado del frame) | ✅ | ✅ | ✅ | ✅ | ✅ |
| Generación determinista con seed | ✅ TypeScript puro, portable | ✅ Dart puro | ⚠️ Hay que duplicar lógica o compartir vía KMP/C++ | ✅ C# | ✅ GDScript/C# | ✅ JS |
| Paths pre-generados / replay | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Estado local / offline-first | ✅ (SQLite, MMKV) | ✅ (SQLite, Hive/Isar) | ✅ | ⚠️ No es su fuerte | ⚠️ | ⚠️ Storage web limitado |
| Export/import de archivo | ✅ APIs maduras | ✅ | ✅ | ⚠️ Incómodo | ⚠️ | ⚠️ Sandbox restrictivo |
| Rendimiento móvil | ✅ Bueno (Hermes + Fabric + Skia) | ✅ Muy bueno | ✅ Máximo | ⚠️ Pesado para una app de UI; batería | ⚠️ Similar | ❌ El más débil |
| Animaciones y gestos | ✅ (Reanimated + Gesture Handler) | ✅ | ✅ | ✅ pero paradigma de juego | ✅ | ⚠️ |
| UI de "academia" (texto, lecciones, navegación, formularios) | ✅ Excelente | ✅ Excelente | ✅ Excelente | ❌ Muy costosa | ❌ Costosa | ✅ |
| Mantenibilidad a largo plazo | ✅ Ecosistema enorme, TypeScript | ✅ | ⚠️ Dos codebases | ⚠️ Licencias y deriva del engine | ⚠️ Comunidad menor en móvil | ⚠️ Deuda técnica |
| Velocidad de desarrollo (1 codebase, MVP) | ✅ Alta | ✅ Alta | ❌ Baja (x2 trabajo) | ⚠️ Media | ⚠️ Media | ✅ Alta pero techo bajo |
| Android 15+ / iOS 20+ | ✅ | ✅ | ✅ | ✅ | ⚠️ Soporte iOS menos pulido | ✅ |
| Talento disponible en LATAM | ✅ Muy alto (JS/TS) | ✅ Alto | ⚠️ Dividido | ⚠️ Perfil gamedev, no fintech | ⚠️ Escaso | ✅ |

### 1.2 Veredicto por opción

- **React Native:** cubre todos los requisitos. El único punto históricamente débil (render de gráficos de alta frecuencia) está resuelto con `react-native-skia` (canvas GPU) y Reanimated (animaciones en hilo de UI). El motor de simulación puede escribirse en TypeScript puro, 100% testeable fuera del dispositivo. **Candidato principal.**
- **Flutter:** técnicamente excelente, render superior "de fábrica". Se descarta por menor disponibilidad de talento JS/TS (relevante para un proyecto firmado por un desarrollador individual en LATAM) y porque la ventaja de render no es decisiva: Burgundy reproduce velas a velocidad controlada, no streaming de ticks reales de alta frecuencia. **Segunda opción válida.**
- **Nativo Swift/Kotlin:** máximo rendimiento, pero duplica el costo: dos UIs y, peor, dos implementaciones del motor determinista (o una tercera capa compartida en C++/Kotlin Multiplatform). Duplicar un motor que debe ser bit-a-bit reproducible es un riesgo de divergencia inaceptable para el MVP. **Descartado para MVP.**
- **Unity:** es un motor de videojuegos. La UI de academia (lecciones, glosario, journal, formularios, navegación) sería lenta y antinatural de construir; binarios pesados; consumo de batería; sensación de "juego" que contradice la identidad seria de Burgundy. **Descartado.**
- **Godot:** mismos argumentos que Unity con un ecosistema móvil menos maduro. **Descartado.**
- **WebView/híbrido:** el peor rendimiento de canvas en gamas bajas (relevante en LATAM), almacenamiento local frágil (el sistema puede purgar storage web), export/import incómodo. **Descartado.**

---

## 2. Stack recomendado

> **Recomendación confirmada: React Native + TypeScript**, con motor de simulación en TypeScript puro completamente separado de la UI.

| Capa | Tecnología recomendada |
|---|---|
| Framework móvil | React Native 0.83 vía Expo SDK 55 (Nueva Arquitectura: Fabric + TurboModules) — matriz cerrada por `TECH_STACK_LOCK` (§2.3) |
| Lenguaje | TypeScript estricto (`strict: true`) en toda la app y el motor |
| Runtime JS | Hermes |
| Render de gráficos (velas) | `@shopify/react-native-skia` (canvas GPU) |
| Animaciones / gestos | `react-native-reanimated` + `react-native-gesture-handler` |
| Estado global | Zustand (estado de UI/sesión) + el motor como fuente de verdad de la simulación |
| Persistencia local | SQLite vía `expo-sqlite` en modo WAL para datos estructurados (cerrado por `TECH_STACK_LOCK`, §2.3; `op-sqlite` queda como alternativa documentada, no instalada); MMKV para preferencias |
| Export/import | Archivo JSON versionado + checksum, vía file system y share sheet del SO |
| Tests | Vitest/Jest para el motor (puro), React Native Testing Library para UI, Maestro/Detox para E2E |
| Tooling | Expo SDK 55 managed + dev build (`expo-dev-client`); prebuild solo cuando se requiera — flujo cerrado por `TECH_STACK_LOCK` (§2.3) y `WINDOWS_POWERSHELL_WORKFLOW` (§2.5) |

### 2.1 Por qué este stack es adecuado para Burgundy

1. **El motor determinista vive en TypeScript puro.** No depende de React, ni de la plataforma, ni del reloj del dispositivo. Se ejecuta igual en Node (tests, CI) que en Hermes (dispositivo). Determinismo verificable en CI con miles de seeds.
2. **Una sola codebase** para Android 15+ e iOS 20+, crítico para un proyecto firmado por tsuloid con recursos de MVP.
3. **Skia resuelve el gráfico.** El candlestick chart con paleta personalizada (#4A6D56 alcistas, #802F3E bajistas, fondo #1A1617) se dibuja en GPU; soporta crosshair, zoom, pan y playback fluido a velocidad variable.
4. **Offline-first natural:** SQLite local, cero dependencias de red, cero login. No hay backend que diseñar, asegurar ni pagar.
5. **Ecosistema y talento:** JS/TS es el pool más grande de LATAM; la mantenibilidad a 2–3 años está garantizada por la madurez del ecosistema.
6. **Identidad visual controlable:** el HUD Burgundy (#1A1617, #571324, #2E2E2E, #C9A050) se implementa con un design system propio; React Native no impone estética de plataforma.

### 2.2 Riesgos conocidos y mitigaciones

| Riesgo | Severidad | Mitigación |
|---|---|---|
| Determinismo de punto flotante entre Hermes/Node/arquitecturas | Alta | Usar PRNG entero (ver §9); evitar `Math.random` y operaciones trascendentales no controladas en el path crítico; suite de tests de reproducibilidad cross-runtime con hashes de referencia |
| Jank en el chart si el playback corre en el hilo JS junto a la UI | Media | Playback desacoplado: el motor pre-genera el path completo; el "reloj" de reproducción solo avanza un cursor; render en Skia/Reanimated fuera del ciclo de React |
| Breaking changes de la Nueva Arquitectura de RN | Media | Fijar versiones; capa de abstracción sobre librerías nativas; el motor (el activo más valioso) es independiente de RN |
| Rendimiento en gama baja LATAM (3–4 GB RAM) | Media | Presupuesto de rendimiento explícito (§14); virtualización de listas; limitar velas visibles; perfiles en dispositivos físicos de gama baja desde el inicio |
| Tamaño del archivo de export si se guardan paths completos | Baja | Estrategia híbrida seed+path con compresión (§8) |
| Corrupción de datos locales (cierre forzado, falta de espacio) | Media | SQLite con WAL, escrituras transaccionales, checksum en export, backup automático local rotativo |

### 2.3 Matriz cerrada de stack (`TECH_STACK_LOCK`)

<!-- LOCK: TECH_STACK_LOCK v1 — Documento dueño: 08_technical_architecture.md. Resuelve AUD-009. Los documentos 12 y 13 referencian este lock por nombre, sin copiarlo. Ante contradicción entre cualquier documento y este lock, gana el lock. -->

> **TECH_STACK_LOCK v1 — Matriz canónica de dependencias y flujo de build (cerrada).**
>
> **Línea base: Expo SDK 55** (línea estable vigente al cierre de este lock, 2026-06; React Native 0.83, React 19.2). SDK 55 ejecuta **exclusivamente la New Architecture** — la Legacy Architecture fue eliminada y no existe flag de retorno, lo que elimina de raíz el riesgo de instalar dependencias incompatibles con Fabric/TurboModules. Toda dependencia nativa se instala con `npx expo install` (resuelve la versión exacta compatible con el SDK); el `package-lock.json` del primer commit del proyecto congela los parches exactos y forma parte de este contrato.
>
> | Dependencia | Versión (línea cerrada) | New Architecture | Motivo | Peso aprox.* | Alternativa (descartada) |
> |---|---|---|---|---|---|
> | `expo` | `^55.0.0` (SDK 55) | Única arquitectura soportada | Tooling managed + dev builds sin bare | — | Bare RN CLI: más fricción sin beneficio para el MVP |
> | `react-native` | `0.83.x` (fijada por SDK 55) | ✅ Fabric + TurboModules | Framework móvil | — | Flutter: segunda opción válida, descartada en §1 |
> | `react` | `19.2.x` (fijada por SDK 55) | ✅ | Requerida por RN 0.83 | — | — |
> | `typescript` | `~5.9` (plantilla SDK 55), `strict: true` | n/a | Lenguaje único de app y motor | dev | — |
> | `@shopify/react-native-skia` | `2.x` (resuelta por `expo install`) | ✅ | Render GPU del chart de velas | ~2–3 MB | Victory/TradingView: no soportan playback determinista + paleta propia |
> | `react-native-reanimated` | `4.x` + `react-native-worklets` (peer obligatorio) | ✅ (requiere New Architecture) | Animaciones en hilo de UI; cursor de playback | ~1 MB | Animated core: insuficiente para 60 fps sostenidos |
> | `react-native-gesture-handler` | `2.28+` (resuelta por `expo install`) | ✅ | Zoom, pan y crosshair del chart | <1 MB | Gestos de RN core: limitados |
> | `expo-sqlite` | línea SDK 55 (resuelta por `expo install`) | ✅ | Persistencia estructurada, **modo WAL** | ~1 MB | `op-sqlite`: válida, descartada para no salir del árbol Expo |
> | `react-native-mmkv` | `3.x` | ✅ TurboModule puro | Preferencias rápidas | <1 MB | AsyncStorage: lento, en desuso |
> | `zustand` | `^5` | n/a (JS puro) | Estado de UI/app | <100 KB | Redux: sobre-ingeniería — el motor ya es el reducer determinista |
> | `vitest` | `^3` (dev) | n/a | Tests del motor TS puro en Node | dev | — |
> | `jest-expo` + RN Testing Library | línea SDK 55 (dev) | ✅ | Tests de componentes RN | dev | — |
>
> \* Peso aproximado sobre el binario Android release; son estimaciones a validar en el primer build, no compromisos.
>
> **Flujo cerrado: Expo managed + dev build (`expo-dev-client`). Sin bare.** `npx expo prebuild` se ejecuta solo cuando se requiera regenerar los proyectos nativos, siempre con `--clean` y sin ediciones manuales de `android/`/`ios/`: toda configuración nativa vive en `app.json` y config plugins (`expo-build-properties`, ver `PLATFORM_TARGET_LOCK`). **EAS Build queda documentado como opción, no requerido para el MVP local.**
>
> **Verificación obligatoria post-instalación:** `npx expo-doctor@latest` sin errores. Es un gate: ninguna dependencia se considera integrada hasta que expo-doctor pase y el corpus dorado de `DETERMINISM_LOCK_V1` siga verde en Node y Hermes.
>
> **Política de actualización:** subir de SDK (p. ej. a SDK 56) exige una nueva versión de este lock (v2), expo-doctor limpio y corpus dorado verde. Prohibido instalar versiones fuera de la resolución de `expo install` o mezclar líneas de SDK.

### 2.4 Plataforma objetivo verificable (`PLATFORM_TARGET_LOCK`)

<!-- LOCK: PLATFORM_TARGET_LOCK v1 — Documento dueño: 08_technical_architecture.md. Resuelve AUD-010. Los documentos 09, 12 y 13 referencian este lock por nombre, sin copiarlo. Ante contradicción entre cualquier documento y este lock, gana el lock. -->

> **PLATFORM_TARGET_LOCK v1 — Configuración verificable de plataforma (cerrada).**
>
> **Android — requisito comercial "Android 15+":**
> - `minSdkVersion = 35` (Android 15).
> - `targetSdkVersion = 36` y `compileSdkVersion = 36` (Android 16 — valores por defecto de Expo SDK 55; targeting 36 implica edge-to-edge obligatorio).
> - Todo se configura vía `expo-build-properties` en `app.json`; prohibido editar Gradle a mano.
>
> **iOS — acta de mapeo del requisito comercial "iOS 20+":** Apple nunca publicó iOS 19–25; el versionado saltó de iOS 18 a iOS 26 (2025). Por lo tanto **toda versión real de iOS ≥ 20 es ≥ 26**, y el requisito comercial se cierra así:
> - `ios.deploymentTarget = "26.0"` (vía `expo-build-properties`).
> - Los documentos de la serie siguen citando "iOS 20+" como requisito comercial; la verdad técnica es este mapeo. La lectura alternativa (target 18.0) se descarta porque incluiría versiones < 20 y violaría el piso declarado.
> - El default de Expo SDK 55 es iOS 15.1; el override a 26.0 es deliberado y queda registrado en esta acta.
>
> **Comandos de verificación (sintaxis PowerShell — ver `WINDOWS_POWERSHELL_WORKFLOW`):**
> - `npx expo config --type prebuild` → inspeccionar `android.minSdkVersion`, `android.targetSdkVersion`, `android.compileSdkVersion` e `ios.deploymentTarget` resueltos.
> - Tras `npx expo prebuild --platform android --clean`: `Select-String -Path .\android\build.gradle -Pattern "SdkVersion"`.
> - El `Podfile` iOS (`platform :ios, '26.0'`) solo es verificable en macOS o EAS Build; en Windows la verificación iOS se limita a `expo config` (y EAS opcional).
>
> **Checklist de emulador/dispositivo del MVP:** emulador Android API 35 y API 36 · dispositivo físico Android de gama media/baja (3–4 GB RAM, perfil LATAM) · simulador/dispositivo iOS 26 (vía macOS o EAS; opcional para el loop local del MVP) · corpus dorado de `DETERMINISM_LOCK_V1` verificado en cada plataforma que compile.

### 2.5 Flujo de trabajo en Windows PowerShell (`WINDOWS_POWERSHELL_WORKFLOW`)

<!-- LOCK: WINDOWS_POWERSHELL_WORKFLOW v1 — Documento dueño: 08_technical_architecture.md. Resuelve AUD-011. Los documentos 12 y 13 referencian este lock por nombre, sin copiarlo. Ante contradicción entre cualquier documento y este lock, gana el lock. -->

> **WINDOWS_POWERSHELL_WORKFLOW v1 — El entorno real de desarrollo es Windows + PowerShell (cerrado).**
>
> **Comandos canónicos (sintaxis PowerShell):**
>
> ```powershell
> # Crear el proyecto (una sola vez, desde la raíz del repo)
> npx create-expo-app@latest burgundy-app --template blank-typescript
> Set-Location .\burgundy-app
>
> # Instalar dependencias nativas (siempre vía expo install — TECH_STACK_LOCK)
> npx expo install expo-dev-client expo-sqlite expo-build-properties @shopify/react-native-skia react-native-reanimated react-native-worklets react-native-gesture-handler react-native-mmkv
> npm install zustand
> npm install -D vitest
>
> # Verificación obligatoria post-instalación (gate)
> npx expo-doctor@latest
>
> # Tests y lint
> npm run test
> npm run lint
>
> # Dev build local Android (emulador o dispositivo conectado)
> npx expo run:android
>
> # Prebuild — solo cuando se requiera regenerar el proyecto nativo
> npx expo prebuild --platform android --clean
> ```
>
> **Reglas obligatorias:**
> 1. **Sin `&&`:** encadenar con `;` o `if ($?) { ... }` (compatible con Windows PowerShell 5.1 y PowerShell 7).
> 2. **Variables de entorno:** `$env:NOMBRE = "valor"` — nunca `export`.
> 3. **Rutas:** separador `\` en comandos de consola; el código y los scripts usan rutas relativas portables resueltas por Node (`path.join`), nunca rutas absolutas ni separadores fijos.
> 4. **Scripts npm shell-agnósticos:** la lógica vive en scripts Node (`node scripts/<tarea>.mjs`); prohibidas las concatenaciones bash (`&&`, `|`, subshells) dentro de `package.json`.
> 5. **Prohibido asumir bash/macOS** en documentación, scripts y tooling del repo. Los builds iOS requieren macOS o EAS Build: en Windows, iOS no forma parte del loop local de desarrollo (ver `PLATFORM_TARGET_LOCK`).
> 6. **Convención de rutas del repo:** el proyecto Expo vive en `burgundy-app\` en la raíz del repo; la documentación, en `docs\architecture\`.

---

## 3. Arquitectura general: separación UI / Motor

Principio rector: **Burgundy es dos productos en uno: un motor de simulación determinista (librería pura) y una app móvil que lo consume.**

```text
┌────────────────────────────────────────────────────────────┐
│  CAPA UI (React Native)                                    │
│  Pantallas · HUD Burgundy · Chart Skia · Lecciones ·       │
│  Gamificación · Navegación · Animaciones                   │
│        │  (solo lee estado / despacha intenciones)         │
├────────▼───────────────────────────────────────────────────┤
│  CAPA DE APLICACIÓN (orquestación)                         │
│  SessionController · PlaybackClock · Stores (Zustand) ·    │
│  Mapeo motor→viewmodels                                    │
├────────▼───────────────────────────────────────────────────┤
│  MOTOR DE SIMULACIÓN (TypeScript puro, sin dependencias)   │
│  PRNG · ScenarioTemplate · LearningContextContract ·       │
│  RegimeGen · CandleGen · TickApprox · EventScheduler ·     │
│  Spread/Slippage · PathBuilder · Replay · HashValidator ·  │
│  DecisionLog · OrderEngine · PositionEngine · RiskEngine · │
│  EvaluationEngine                                          │
├────────▼───────────────────────────────────────────────────┤
│  CAPA DE PERSISTENCIA                                      │
│  SQLite (sesiones, progreso, journal, rankings) ·          │
│  MMKV (preferencias) · Export/Import JSON versionado       │
└────────────────────────────────────────────────────────────┘
```

Reglas estrictas:

- La UI **nunca** importa internals del motor; consume una API pública (`createSession`, `submitOrder`, `advanceTo`, `getSnapshot`).
- La UI **nunca muta el market path**. El path se genera completo antes de la interacción del usuario (filosofía central del producto: el simulador fuerza un contexto de aprendizaje, no un resultado).
- Las acciones del usuario son **intenciones** (abrir orden, mover stop loss, cerrar posición) que el motor valida y aplica sobre cuenta/órdenes/posiciones — jamás sobre las velas.
- El motor no conoce el reloj real: recibe "avanza hasta el índice de vela N". La velocidad de reproducción (x1, x2, x10, pausa, paso a paso) es un asunto exclusivo del `PlaybackClock` en la capa de aplicación.
- El motor es publicable como paquete interno (`@burgundy/engine`) con su propio versionado semántico, independiente de la app.

---

## 4. Enfoque de charting y render

- **Tecnología:** `react-native-skia` — canvas acelerado por GPU, mismo motor de render que usa Flutter/Chrome.
- **Qué se dibuja:** velas (cuerpo + mechas) con la paleta exacta — alcistas #4A6D56, bajistas #802F3E — sobre fondo #1A1617, ejes y divisores en #2E2E2E, crosshair y precios críticos en #C9A050, niveles de orden/SL/TP en #571324/#C9A050.
- **Modelo de datos del chart:** la UI recibe del motor un *snapshot* inmutable (`velas visibles [0..cursor]`, posiciones, órdenes, equity). El chart es una función pura de ese snapshot + viewport (zoom/pan).
- **Playback:** un timer de Reanimated avanza el cursor según la velocidad elegida; cada tick de playback solo incrementa un índice — no recalcula nada del mercado. Pausa, retroceso visual y paso-a-paso son operaciones sobre el cursor.
- **Aproximación de ticks:** dentro de una vela en formación, el motor expone sub-pasos pre-generados (ver §9.6) para que la vela "se construya" visualmente de forma realista y reproducible.
- **Presupuesto:** 60 fps con ~120 velas visibles en un dispositivo de gama media-baja; ventana deslizante + decimación para series largas. Presupuesto normativo de render y densidad del HUD: `BEGINNER_HUD_LOCK` (documento 07).
- **Sin librerías de charting de terceros** (TradingView, Victory, etc.): ninguna soporta el modelo playback-determinista + replay + colores propios sin pelear contra ella. El chart es core del producto y se construye sobre Skia.

---

## 5. Gestión de estado

Dos dominios de estado claramente separados:

| Dominio | Fuente de verdad | Herramienta |
|---|---|---|
| **Estado de simulación** (velas, órdenes, posiciones, cuenta, riesgo, P/L, log de decisiones) | El motor — único dueño | El motor expone snapshots inmutables; la capa de aplicación los publica |
| **Estado de UI/app** (pantalla actual, viewport del chart, velocidad de playback, preferencias, progreso de academia, XP, rachas) | Stores de aplicación | Zustand (ligero, sin boilerplate, testeable) |

Reglas:

- Flujo unidireccional: UI → intención → motor → nuevo snapshot → UI. Nada de two-way binding con el estado de simulación.
- Los snapshots del motor son inmutables y serializables (clave para replay, export y tests).
- No se adopta Redux: el motor ya cumple el rol de "reducer determinista" para el dominio de simulación; duplicarlo sería sobre-ingeniería.
- El progreso persistente (XP, niveles, desbloqueos, journal, rankings locales) se escribe a SQLite a través de un repositorio único; los stores se hidratan al arrancar.

---

## 6. Persistencia local

**Motor de almacenamiento: SQLite** (transaccional, robusto ante cierres forzados, consultable) + **MMKV** para preferencias simples.

Esquema lógico (tablas principales, sin DDL):

| Tabla | Contenido |
|---|---|
| `profile` | Perfil local único: XP, nivel, racha, desbloqueos, configuración. Sin datos personales. |
| `sessions` | Una fila por sesión de simulación: id, modo (tutorial/sandbox/challenge/libre/sin-indicadores), `scenarioTemplateId`, parámetros, `seed`, `generatorVersion`, `pathHash`, timestamps, resultado, score |
| `session_paths` | Path de velas generado, serializado y comprimido (ver §8 sobre cuándo se guarda) |
| `decision_logs` | Log ordenado de decisiones del usuario por sesión (ver §9.11) |
| `journal_entries` | Journal de operaciones y revisión de errores |
| `evaluations` | Salida del Evaluation Engine por sesión (disciplina, gestión de riesgo, calidad de proceso) |
| `rankings` | High scores y rankings locales por tipo de challenge |
| `curriculum_progress` | Lecciones completadas, conceptos revisados, repasos pendientes |
| `meta` | Versión de esquema, versión de app, migraciones aplicadas |

Prácticas:

- SQLite en modo WAL; toda escritura de fin de sesión es una transacción única.
- Migraciones de esquema versionadas y aplicadas al arrancar (la tabla `meta` guarda la versión).
- Backup local rotativo automático (copia del archivo de export más reciente) para recuperación ante corrupción.
- Nada sale del dispositivo. No hay sync, no hay analytics remotos en el MVP.

---

## 7. Estrategia de export/import de progreso

**Formato:** un único archivo JSON versionado, extensión propia `.burgundy` (envelope sin comprimir + payload gzip).

> **Especificación normativa cerrada: `BURGUNDY_FILE_FORMAT_V1` (documento 09).** Esta sección es un resumen arquitectónico; el esquema exacto, los límites de tamaño, la tabla de errores de import y el proceso transaccional viven en el lock. Ante contradicción, gana el lock.

Estructura lógica del archivo:

```text
envelope (sin comprimir):
  formatVersion        (versión del formato de export, independiente de la app)
  appVersion           (versión de Burgundy que exportó)
  generatorVersion     (versión del motor instalado al exportar)
  schemaVersion        (versión del esquema de la base local)
  exportedAt           (timestamp ISO 8601 UTC)
  checksum             (SHA-256 del payload, para detectar corrupción/manipulación)
payload (gzip):
  profile              (XP, nivel, rachas, desbloqueos)
  curriculumProgress
  sessions             (metadatos + seed + generatorVersion + pathHash de cada una)
  seedRecords          (siempre, todos — SEED_PATH_REPLAY_EXPORT_LOCK §8)
  sessionPaths         (paths completos solo según SEED_PATH_REPLAY_EXPORT_LOCK: sesión en curso o generatorVersion no regenerable)
  decisionLogs
  journal
  evaluations
  rankings
```

Reglas de diseño:

1. **Export:** generado bajo demanda desde "Ajustes → Exportar progreso"; se entrega vía share sheet del SO (Drive, archivos locales, etc.). Burgundy no sube nada a ningún servidor.
2. **Import:** valida en orden — (a) JSON bien formado, (b) `formatVersion` soportada (con migradores de formato hacia adelante), (c) checksum correcto, (d) `pathHash` de cada path almacenado coincide al recalcular. Si algo falla, se rechaza con el código y mensaje de la tabla de errores de `BURGUNDY_FILE_FORMAT_V1` y **no** se toca la base actual (validación completa → base temporal → promoción atómica, con backup automático previo obligatorio).
3. **Estrategia de fusión:** el MVP usa **reemplazo total con confirmación explícita** ("esto sustituirá tu progreso actual") + backup automático previo del estado actual. Merge inteligente es post-MVP.
4. **Privacidad:** el archivo no contiene datos personales — no hay login ni identidad; solo progreso pedagógico y de simulación.
5. **Compatibilidad futura:** `formatVersion` independiente de `generatorVersion`; los migradores de import nunca se eliminan.

---

## 8. Decisión: ¿guardar paths completos, solo seeds, o ambos?

<!-- LOCK: SEED_PATH_REPLAY_EXPORT_LOCK v1 — Documento dueño: 08_technical_architecture.md. Resuelve AUD-005 y AUD-006. Los documentos 03, 06, 07, 09, 10, 12 y 13 referencian este lock por nombre, sin copiarlo. Ante contradicción entre cualquier documento y este lock, gana el lock. -->

> **SEED_PATH_REPLAY_EXPORT_LOCK v1 — Política única de seed, path, replay, export y visibilidad (cerrada).**
>
> 1. **`SeedRecord` siempre.** Todo path generado produce un `SeedRecord` (seed + `seedType` + `generatorVersion` + `templateId/Version` + `lccId/Version` + parámetros resueltos + `pathHash`). Se persiste siempre, nunca se purga mientras exista una sesión que lo referencie, y se exporta **siempre** en el archivo `.burgundy`.
> 2. **`pathHash` siempre.** El SHA-256 de la serialización canónica del path (ver `DETERMINISM_LOCK_V1`, §9) se calcula y se sella **antes de la primera vela visible** de la sesión.
> 3. **Materialización local del path.** El path completo solo se conserva materializado para: (a) la sesión activa y las sesiones recientes (ventana de las últimas 10 sesiones), y (b) los replays guardados explícitamente — incluidas las sesiones que entran a rankings. Todo lo demás es purgable y se regenera desde `seed + generatorVersion`.
> 4. **Export del path completo.** El archivo `.burgundy` incluye el path completo de una sesión solo si: (a) la sesión está `en_curso`, o (b) su `generatorVersion` no es regenerable por el build actual (generador retirado/congelado). En cualquier otro caso viaja solo el `SeedRecord`; el dispositivo destino regenera el path y lo valida contra `pathHash`.
> 5. **Replay.** Si existe path almacenado, el replay lo usa, validando su hash contra `SeedRecord.pathHash`. Si no existe, regenera con el generador de la `generatorVersion` registrada y valida el hash. **Hash no coincidente ⇒ sesión "no verificable":** se excluye de rankings, se informa al usuario con lenguaje claro y jamás se repara en silencio.
> 6. **Visibilidad de seed en desafíos.** Antes del intento se muestran únicamente `pathHash`, `seedType`, `generatorVersion` y las reglas del LCC (el **sello de equidad**). La seed cruda se revela solo al cerrar el intento. Si la seed era conocida de antemano (replay, seed guardada, sesión previa con la misma seed), el intento se marca `seed_known = true` y **no es elegible para el ranking principal de primer intento**; conserva marcas personales separadas.

**Decisión para el MVP: estrategia híbrida — siempre seed + metadatos; path completo solo cuando importa el replay exacto.** La tabla siguiente es la aplicación práctica del lock anterior.

| Tipo de sesión | Qué se guarda | Por qué |
|---|---|---|
| Tutoriales (seeds fijas) | Solo `seed + generatorVersion + pathHash` | El template y la seed están empaquetados con la app; regenerar es trivial |
| Challenges oficiales (seeds fijas) | `seed + generatorVersion + pathHash` **+ path completo si la sesión queda en rankings** | Un high score debe poder reproducirse exactamente aunque el generador evolucione |
| Sandbox / modo libre (seeds aleatorias) | `seed + generatorVersion + pathHash`; path completo **solo si el usuario guarda la sesión para replay/journal** | Evita inflar el storage con sesiones descartables |
| Sesiones con replay guardado | Path completo comprimido + decision log | El replay exacto es la promesa pedagógica central |

Justificación:

- **Solo seeds** sería frágil: obliga a congelar el generador para siempre o a mantener todas las versiones históricas ejecutables (ver §10).
- **Solo paths** infla el storage y pierde la trazabilidad de "cómo se generó".
- El path de una sesión típica (300–600 velas OHLC + sub-pasos) comprimido ocupa pocas decenas de KB: guardar los que importan es barato.
- Regla de resolución en replay: **si existe path almacenado, se usa el path** (verificado contra `pathHash`); la regeneración por seed es fallback y requiere que `generatorVersion` coincida.

---

## 9. Arquitectura del motor de simulación determinista

El motor es **puro y determinista**: dada la misma combinación de *scenario template + Learning Context Contract + parámetros + seed + generatorVersion*, produce exactamente el mismo market path, byte a byte. Sin `Date.now()`, sin `Math.random()`, sin I/O, sin estado global.

Pipeline de generación (todo ocurre **antes** de que el usuario vea la primera vela):

```text
seed + template + contract + params + generatorVersion
   │
   ▼
[1] PRNG con seed ──► streams independientes por subsistema
   ▼
[2] Scenario Template Engine ──► estructura del escenario
   ▼
[3] Learning Context Contract Engine ──► garantías pedagógicas
   ▼
[4] Regime Generator ──► secuencia de regímenes (tendencia/rango/volatilidad)
   ▼
[5] Candle Generator ──► velas OHLC en formato universal
   ▼
[6] Tick Approximation Generator ──► sub-pasos intra-vela
   ▼
[7] Event Scheduler ──► eventos programados (picos de volatilidad, gaps, "noticias")
   ▼
[8] Spread/Slippage Generator ──► series de spread y parámetros de slippage
   ▼
[9] Market Path Builder ──► PATH INMUTABLE + pathHash
   ▼
════════ el path queda sellado; empieza la interacción ════════
   ▼
[Runtime] Order Engine · Position Engine · Risk Engine · Decision Log
   ▼
[Cierre] Evaluation Engine · Replay Engine · Scenario Hash Validator
```

<!-- LOCK: DETERMINISM_LOCK_V1 — Documento dueño: 08_technical_architecture.md. Resuelve AUD-007. Los documentos 03, 09, 10 y 13 referencian este lock por nombre, sin copiarlo. Ante contradicción entre cualquier documento y este lock, gana el lock. -->

> **DETERMINISM_LOCK_V1 — Contrato ejecutable de determinismo (cerrado).**
>
> 1. **PRNG: PCG32.** Decisión cerrada; la alternativa xoshiro queda eliminada de toda la documentación. Implementado en el motor con aritmética entera; prohibidos `Math.random` y cualquier RNG del lenguaje o de la plataforma.
> 2. **Seed: entero sin signo de 64 bits.** Representación canónica: decimal, sin ceros a la izquierda (`"0"` para cero), en toda serialización, export, UI técnica y entrada de hashing.
> 3. **Substreams por subsistema (índices fijos):** `0` = régimen · `1` = velas · `2` = sub-ticks · `3` = eventos · `4` = spread · `5` = slippage · `6` = resolución de parámetros del template · `7` = reintentos de contrato (`seed + attempt`). Derivación según el esquema de streams de PCG32 (mismo estado inicial = seed; incremento del stream `k` = `2k + 1`). Un subsistema nuevo toma el siguiente índice libre; los existentes jamás se reordenan.
> 4. **Precios: enteros escalados.** Cada instrumento declara `priceScale` (potencia de 10) en el catálogo del bundle (documento 09, `Instrument`); toda generación y ejecución opera sobre `precio × priceScale` como entero. **Prohibido punto flotante en el camino crítico de generación.**
> 5. **Tiempo lógico:** índice de vela + índice de sub-tick. Sin reloj real, sin `Date.now()`, sin entropía externa. Sub-ticks por vela: **4–8 en MVP** (`MVP_SANDBOX_LIMITS`, documento 12); la cantidad exacta por vela la decide determinísticamente el stream 2.
> 6. **Redondeo: half-even (banker's rounding)** sobre enteros escalados — regla única para toda división o promedio en generación y ejecución.
> 7. **Serialización canónica del path (entrada exacta del SHA-256):** orden fijo de bloques (metadatos → velas → sub-ticks → eventos → spread → slippage), orden fijo de campos por registro definido por el esquema del motor y versionado con `generatorVersion`, codificación UTF-8, sin espacios ni saltos de línea, enteros en decimal, sin floats.
> 8. **Corpus dorado:** lista versionada en el repo de (`templateId/Version`, `lccId/Version`, parámetros, seed) → `pathHash` esperado, por `generatorVersion`. CI la verifica en Node y Hermes (y en cada plataforma soportada) en cada commit; cualquier divergencia rompe el build.

### 9.1 Seeded Pseudo-Random Generator (PRNG)

- Algoritmo: **PCG32** (cerrado por `DETERMINISM_LOCK_V1`; sin alternativas), implementado en el propio motor — nunca el RNG del lenguaje.
- Solo aritmética entera de 32 bits en el núcleo del PRNG: elimina el riesgo de divergencia de punto flotante entre runtimes.
- **Streams derivados:** de la seed maestra se derivan sub-seeds independientes por subsistema (régimen, velas, ticks, eventos, spread). Así, cambiar el consumo de aleatoriedad de un subsistema no desincroniza a los demás.
- La seed maestra se registra en la sesión; las sub-seeds son derivables y no se almacenan.

### 9.2 Scenario Template Engine

- Un template define: instrumento sintético, timeframe, número de velas, rangos de parámetros, regímenes permitidos, eventos posibles, condiciones de inicio/fin y modo (tutorial/challenge/sandbox).
- Templates declarativos (datos, no código), versionados e identificados por `templateId + templateVersion`.
- Resuelve parámetros aleatorios dentro de rangos usando su stream del PRNG.

### 9.3 Learning Context Contract Engine

- Materializa la filosofía del producto: **el simulador fuerza un contexto de aprendizaje, no un resultado de trading.**
- Un contrato declara garantías estructurales del escenario: "habrá una tendencia clara con al menos dos retrocesos", "habrá un falso breakout", "el spread se ampliará durante el evento de volatilidad".
- El contrato **restringe la generación** (guía a los generadores de régimen/eventos) y **valida el path resultante**; si un path generado no cumple el contrato, el motor re-deriva con un contador de intento determinista (`seed + attempt`) hasta cumplirlo — proceso igualmente reproducible.
- El contrato jamás mira las acciones del usuario: garantiza el contexto, no el resultado de sus operaciones.

### 9.4 Regime Generator

- Genera la secuencia de regímenes de mercado del escenario: tendencia alcista/bajista, rango, compresión, expansión de volatilidad, con duraciones y transiciones extraídas de su stream del PRNG dentro de los límites del template y el contrato.
- Salida: lista ordenada de segmentos `{régimen, duración en velas, parámetros de volatilidad/drift}`.

### 9.5 Candle Generator

- Consume la secuencia de regímenes y produce velas OHLC en el **formato universal de vela** (ver §11).
- Garantiza coherencia interna (high ≥ open/close ≥ low, continuidad entre velas salvo gaps programados por el Event Scheduler).
- Aritmética de precios en **enteros escalados** (precio × 10^decimales del instrumento): cero ambigüedad de redondeo flotante en el path.

### 9.6 Tick Approximation Generator

- Para cada vela genera una secuencia corta de sub-pasos (**4–8 puntos intra-vela en MVP** — `MVP_SANDBOX_LIMITS` y `DETERMINISM_LOCK_V1`) que recorre open → extremos → close de forma plausible.
- Usos: animación de la vela en formación, y **evaluación de ejecución intra-vela** (en qué sub-paso se cruza un stop loss, un take profit o un límite), eliminando la ambigüedad "¿tocó primero el SL o el TP dentro de la vela?".
- Pre-generado con el path: el replay reproduce exactamente las mismas ejecuciones.

### 9.7 Event Scheduler

- Programa eventos discretos en índices de vela concretos: picos de volatilidad, gaps, ampliaciones de spread, "noticias" sintéticas.
- Los eventos son **parte del path pre-generado** — nunca se inyectan en reacción al usuario. Está prohibido por diseño cualquier mecanismo de "evento punitivo" posterior a la entrada del usuario.

### 9.8 Spread/Slippage Generator

- Genera la serie de spread por vela (base + componente dependiente del régimen y de eventos) y los **parámetros** deterministas de slippage por tramo.
- El slippage aplicado a una orden concreta se calcula con una función determinista de (parámetros del tramo, tamaño de la orden, sub-paso de ejecución): mismo path + mismas decisiones en los mismos instantes ⇒ mismas ejecuciones. Sin tirada aleatoria nueva en el momento de la orden.

### 9.9 Market Path Builder

- Ensambla la salida final inmutable: velas + sub-pasos + serie de spread + eventos + parámetros de slippage + metadatos (`templateId/Version`, `contractId/Version`, `seed`, `generatorVersion`).
- Calcula el `pathHash` (SHA-256 de la serialización canónica del path, según `DETERMINISM_LOCK_V1` punto 7) y lo sella antes de la primera vela visible (`SEED_PATH_REPLAY_EXPORT_LOCK` punto 2).
- Tras el build, el path es **estructuralmente inmutable**: ningún componente del runtime tiene una vía para modificarlo.

### 9.10 Scenario Hash Validator

- Verifica `pathHash` en: carga de replay, import de progreso, validación de rankings y tests de regresión del generador en CI.
- Si un hash no coincide: la sesión se marca como no verificable, se excluye de rankings y se informa al usuario con lenguaje claro; nunca se "repara" silenciosamente.

### 9.11 User Decision Log

- Registro append-only de cada intención del usuario con su posición temporal exacta: `{índiceVela, subPaso, tipo de acción, parámetros}` — abrir/modificar/cancelar orden, mover SL/TP, cerrar posición, cambiar apalancamiento, pausar, consultar información.
- Es la segunda mitad del replay: **path + decision log = sesión completa reproducible**.
- También alimenta al Evaluation Engine (calidad de proceso: ¿entró sin stop?, ¿movió el stop en contra?, ¿sobre-operó?).

### 9.12 Order Engine

- Tipos del MVP: market, limit, stop, con stop loss y take profit adjuntos.
- Valida intenciones contra el estado de la cuenta (margen, tamaño, niveles coherentes) y las ejecuta contra el path usando sub-pasos, spread y slippage deterministas.
- Reglas de ejecución explícitas y documentadas (qué precio cruza qué orden en qué sub-paso); las mismas reglas para tutorial, sandbox, challenge y replay.

### 9.13 Position Engine

- Mantiene posiciones abiertas, precio promedio, P/L flotante y realizado, exposición y el efecto del apalancamiento.
- Recalcula equity en cada avance de vela/sub-paso a partir del path — derivación pura, sin estado oculto.

### 9.14 Risk Engine

- Calcula en tiempo real: margen usado/libre, drawdown actual y máximo, riesgo por operación (distancia al SL × tamaño), exposición total y proximidad a margin call simulado.
- Dispara los avisos educativos definidos por producto ("estás arriesgando 40% de tu cuenta en una operación") y aplica las reglas duras de los challenges de disciplina (p. ej., límite de riesgo por trade).

### 9.15 Evaluation Engine

- Al cierre de la sesión consume path + decision log + historial de cuenta y produce la evaluación pedagógica: score de proceso, detección de errores tipificados (entrar sin SL, mover el SL en contra, sobre-apalancarse, sobre-operar, perseguir el precio), métricas (drawdown máximo, R múltiple promedio, adherencia al plan del escenario) y el insumo del mistake review.
- Determinista: misma sesión ⇒ misma evaluación. Separado del Risk Engine (que opera en runtime) — este evalúa a posteriori.

---

## 10. Versionado del generador

Reglas:

1. **`generatorVersion` se almacena con cada sesión**, junto con `seed`, `templateId/Version`, `contractId/Version` y `pathHash`.
2. **`pathHash` se almacena con cada path generado**, se haya persistido el path o no.
3. Cualquier cambio que altere la salida del generador para alguna seed (algoritmo, orden de consumo del PRNG, parámetros por defecto, corrección de bug en generación) **incrementa `generatorVersion`**. Cambios que no tocan la salida (refactors, rendimiento) no la incrementan — y la suite de hashes de regresión en CI lo demuestra.
4. **Si el path completo fue almacenado**, las sesiones viejas se reproducen siempre, con cualquier versión de la app: el replay consume el path almacenado (verificado por hash), no el generador.
5. **Si solo se almacenó la seed**, reproducir exige el generador original. El MVP incluye el módulo generador como componente versionado internamente (`generators/v1`, `generators/v2`…); se conservan las versiones necesarias mientras existan sesiones que solo tengan seed. La estrategia híbrida del §8 minimiza esta carga: todo lo que merece replay exacto guarda su path.
6. Los rankings registran `generatorVersion`: scores de generadores distintos no compiten en la misma tabla si el cambio afectó la dificultad.
7. CI mantiene un **corpus dorado**: lista fija de (template, contract, params, seed) → `pathHash` esperado por versión de generador. Cualquier divergencia rompe el build. (Normativo en `DETERMINISM_LOCK_V1` punto 8.)

---

## 11. Arquitectura historical-ready (futuro, no MVP)

El modo histórico es una **fuente de datos futura**, no la base del MVP. La arquitectura lo deja preparado sin construirlo:

- **Formato universal de vela:** todos los subsistemas consumen el mismo registro — `{timestamp lógico, open, high, low, close, volumen opcional}` en enteros escalados + metadatos de instrumento (decimales, unidad). Ningún consumidor conoce el origen de la vela.
- **Abstracción `MarketDataSource`:** el Market Path Builder es hoy el único productor. Mañana se añaden `HistoricalDataSource` (datos reales empaquetados) y `HistoricalInspiredDataSource` (escenarios sintéticos calibrados sobre episodios reales) implementando el mismo contrato de salida: un path inmutable + metadatos.
- **Metadatos de origen (`sourceMetadata`):** cada path declara `{sourceType: synthetic | fixed-seed | sandbox | historical | historical-inspired, sourceRef, licencia/atribución si aplica}`. La UI puede mostrar el origen; los motores lo ignoran.
- **Indiferencia de los motores:** Order Engine, Position Engine, Risk Engine, Evaluation Engine y Replay Engine operan exclusivamente sobre el formato universal — cero ramificaciones por origen de datos. Esta regla se protege con tests de contrato.
- **Sin ingesta de datos históricos en el MVP:** ni descarga, ni parsing de CSV, ni proveedores de datos. Solo se respeta el contrato para no cerrarse la puerta.
- Las seeds "inspiradas en eventos históricos" del roadmap son templates sintéticos con parámetros calibrados — entran por el pipeline normal del generador, no requieren datos reales.

---

## 12. Capacidad offline

- **100% offline por diseño:** generación, simulación, evaluación, academia, rankings, replay y export/import funcionan sin red. No existe ninguna llamada de red en el MVP.
- Sin login, sin cuenta, sin sync: el dispositivo es la única fuente de verdad; el archivo `.burgundy` es el mecanismo de portabilidad y respaldo.
- Todo el contenido (lecciones, templates, contratos, seeds fijas) viaja empaquetado en el binario de la app.
- Beneficios directos para LATAM: funciona con conectividad intermitente o costosa, cero consumo de datos, cero latencia.
- Implicación de release: corregir contenido o templates requiere actualización de app — los templates declarativos y versionados (§9.2) mantienen ese costo bajo.

---

## 13. Estrategia de testing

| Nivel | Objetivo | Herramienta | Prioridad |
|---|---|---|---|
| Unitario del motor | Cada subsistema (PRNG, generadores, order/position/risk/evaluation) en aislamiento | Vitest/Jest en Node, sin dispositivo | Máxima |
| **Determinismo/regresión (corpus dorado)** | (template, contract, params, seed, generatorVersion) → `pathHash` exacto; cross-runtime Node/Hermes | CI en cada commit | Máxima — es el test más importante del proyecto |
| Property-based | Invariantes sobre miles de seeds: OHLC coherente, contratos cumplidos, equity = derivación pura, SL/TP se ejecutan según las reglas | fast-check | Alta |
| Replay end-to-end | path + decision log reproducen exactamente las mismas ejecuciones, P/L y evaluación | Suite del motor | Alta |
| Export/import | Round-trip sin pérdida; rechazo de archivos corruptos/manipulados; migradores de formato | Suite del motor | Alta |
| Componentes UI | HUD, panel de órdenes, flujos de lección | React Native Testing Library | Media |
| E2E móvil | Flujos críticos: tutorial completo, sesión sandbox, challenge, export/import | Maestro (o Detox) en Android/iOS | Media |
| Rendimiento | fps del chart en playback, memoria, arranque en frío, en dispositivo de gama baja real | Perfilado manual + benchmarks por release | Media |

La separación motor/UI hace que ~80% del valor de testing viva en suites puras de Node: rápidas, estables y ejecutables en cualquier CI sin emuladores.

---

## 14. Consideraciones de rendimiento

- **Pre-generación como ventaja estructural:** el costo computacional pesado (generar el path) ocurre una vez al inicio de la sesión, con indicador de carga; el playback es solo avanzar un cursor y dibujar. No hay cómputo de mercado por frame.
- **Presupuestos explícitos:** generación de path de ~500 velas < 1 s en gama baja; playback 60 fps (mínimo aceptable 30 fps en gama baja con degradación elegante — normativo en `BEGINNER_HUD_LOCK`, documento 07); arranque en frío < 3 s; consumo de memoria del chart acotado por ventana deslizante.
- Render Skia fuera del ciclo de React: el avance del cursor y el crosshair no provocan re-render de árbol de componentes.
- Estructuras compactas en el motor: arrays tipados/columnas para las series (no un objeto por vela en el hot path).
- Velas históricas fuera de viewport: decimación para zoom-out; nunca se dibujan más velas de las visibles.
- Persistencia asíncrona y por lotes: las escrituras a SQLite no bloquean el playback; el decision log se vuelca en lotes transaccionales.
- Probar desde el primer mes en hardware LATAM real de gama baja, no solo en simuladores.

---

## 15. Seguridad y privacidad

- **Minimización radical de datos:** sin login, sin email, sin nombre, sin telemetría remota, sin SDKs de analytics/ads en el MVP. No se recolecta ningún dato personal — la mejor postura de privacidad posible y una ventaja de confianza ante un público vulnerable.
- Almacenamiento local protegido por el sandbox del SO; el contenido (progreso educativo y simulaciones) es de baja sensibilidad — no hay dinero real ni credenciales financieras.
- **Integridad del export:** checksum SHA-256 + validación estricta en import (formato, versión, hashes); un archivo manipulado se rechaza sin afectar los datos existentes. La validación de `pathHash` impide además falsificar sesiones para rankings locales.
- Import seguro: parsing con validación de esquema, límites de tamaño, sin ejecución de nada proveniente del archivo (es solo datos).
- **Postura ética explícita en la app:** Burgundy es educativa, no es asesoría financiera, no opera dinero real ni se conecta a brokers; los disclaimers forman parte del producto.
- Cumplimiento simplificado: sin datos personales no hay obligaciones de tratamiento de datos remoto; aun así se publicará política de privacidad clara en las tiendas declarando "no recolectamos datos".

---

## 16. Escalabilidad futura

La arquitectura deja puertas abiertas sin construirlas ahora:

| Futuro posible | Cómo está preparado |
|---|---|
| Modo histórico | Abstracción `MarketDataSource` + formato universal de vela + `sourceMetadata` (§11) |
| Nuevos instrumentos/timeframes | Templates declarativos y metadatos de instrumento (decimales, unidad) ya parametrizados |
| Nuevos tipos de challenge | Contratos de aprendizaje declarativos + Evaluation Engine extensible por reglas |
| Sync en la nube / rankings online (si algún día se decide) | El formato de export versionado con hashes ya es, en la práctica, un protocolo de estado portable y verificable |
| Versión tablet/desktop | Motor 100% portable (TS puro); solo se rehace capa de presentación |
| Más idiomas (post-LATAM) | Strings centralizados desde el día uno, aunque el MVP sea solo español |
| Evolución del generador | `generatorVersion` + corpus dorado + paths almacenados garantizan compatibilidad histórica (§10) |

El activo de largo plazo es el motor: una librería determinista, testeada y sin dependencias que sobrevivirá a cualquier cambio de framework de UI.

---

## 17. Qué NO sobre-ingenierizar en el MVP

| No construir ahora | Por qué |
|---|---|
| Backend, login, cuentas, sync en la nube | Contradice el diseño offline-first/sin login; costo y superficie de riesgo sin valor para el MVP |
| Ingesta de datos históricos reales | Explícitamente post-MVP; basta respetar el contrato de fuente de datos |
| Motor de ticks "reales" de alta frecuencia | La aproximación por sub-pasos deterministas cubre la necesidad pedagógica y de ejecución |
| Order book / profundidad de mercado simulada | Spread + slippage deterministas son suficientes para enseñar costos de ejecución |
| Más tipos de orden (trailing, OCO complejos, parciales avanzados) | Market/limit/stop + SL/TP cubren el currículo inicial; el Order Engine queda extensible |
| Merge inteligente en import | Reemplazo total con confirmación + backup es suficiente y mucho más simple de razonar |
| Multi-idioma operativo | Solo español; basta centralizar strings |
| Módulos nativos propios (C++/JSI) para el motor | TS + Hermes cumple el presupuesto de rendimiento; optimizar solo si el perfilado lo exige |
| Cifrado del archivo de export | No contiene datos sensibles; el checksum de integridad basta |
| Editor visual de templates/escenarios | Los templates declarativos se editan como datos en el repo |
| Sistema de plugins del motor | Versionado interno simple (§10) es suficiente hasta tener señal real de necesidad |
| Telemetría/analytics | Sin red en MVP; las métricas pedagógicas viven en la evaluación local |

Regla general: **todo lo que no afecte al determinismo, al replay o a la integridad del progreso del usuario puede esperar.** Lo que sí afecte a esas tres cosas se construye bien desde el día uno, porque son los cimientos no-renegociables de Burgundy.

---

*Documento 08 de la serie de arquitectura de Burgundy — proyecto firmado por **tsuloid**. Burgundy es una aplicación educativa: no es asesoría financiera, no opera dinero real y no se conecta a brokers.*
