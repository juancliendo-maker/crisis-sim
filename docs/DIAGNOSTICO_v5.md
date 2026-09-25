# Diagnóstico técnico y funcional — CrisisSim v5.0

- **Repositorio:** `juancliendo-maker/crisis-sim`
- **Rama analizada:** `main`, commit `7afacf7` («v5.0 - Conversaciones privadas + Audio + Cámara»)
- **Archivo analizado:** `index.html` (3.925 líneas, ~153 KB). También existen `README.md` (2 líneas) y `_redirects` (`/*  /index.html  200`).
- **Alcance:** solo lectura. No se modificó ningún archivo existente.
- **Convención:** las referencias `L1234` son números de línea de `index.html` en ese commit. Lo que depende de la configuración de Supabase, Render o del navegador del usuario y no puede comprobarse leyendo el código se marca **NO VERIFICABLE DESDE EL CÓDIGO**.

---

## a) Arquitectura

### a.1 Estructura del archivo

| Bloque | Líneas | Contenido |
|---|---|---|
| `<head>` | L1–L11 | Meta, título, fuentes, Leaflet, Supabase |
| `<style>` | L12–L1577 | Todo el CSS |
| `<body>` (HTML) | L1579–L1726 | Pantallas y modales |
| `<script>` | L1727–L3923 | Toda la lógica JS (89 funciones) |

**CSS (L12–L1577)**

| Líneas | Sección |
|---|---|
| L13–L34 | Variables `:root` (paleta oscura, fuentes) |
| L35–L51 | Base (`body` con `overflow:hidden; height:100vh`, efecto scanlines) |
| L52–L80 | Indicador de conexión |
| L81–L139 | Landing |
| L140–L277 | Configuración del Director |
| L278–L343 | Ingreso de participante |
| L344–L887 | App principal: topbar, pestañas, SITREP, mapa, **CSS heredado de v4 para Comunicaciones (L544–L685, ya sin uso)**, Documentos (L687–L767), panel Director (L768–L887) |
| L888–L948 | Zona de subida de archivos |
| L949–L986 | Tarjetas de archivo |
| L987–L1047 | Modal visor de archivos |
| L1048–L1102 | Avatares |
| L1103–L1210 | Layout de conversaciones v5 |
| L1211–L1339 | Vista de conversación |
| L1340–L1403 | Adjuntos inline |
| L1404–L1474 | Área de entrada v5 |
| L1475–L1513 | Indicador de grabación |
| L1514–L1564 | Modal de cámara |
| L1565–L1576 | Etiqueta de modo monitoreo |

No hay ninguna regla `@media` en todo el archivo.

**HTML (L1579–L1726)**

| Líneas | Elemento |
|---|---|
| L1581–L1584 | Indicador de conexión `#conn-status` |
| L1586–L1608 | `#landing-screen` |
| L1610–L1644 | `#director-setup` (pasos 1–3) |
| L1646–L1657 | `#participant-login` |
| L1659 | `#toast-container` |
| L1661–L1671 | `#file-viewer-modal` |
| L1673–L1683 | `#camera-modal` |
| L1685–L1725 | `#app`: topbar, pestañas y las 4 vistas (`view-sitrep`, `view-comms`, `view-docs`, `view-director`) |

**JS (L1727–L3923)**

| Líneas | Sección |
|---|---|
| L1728–L1755 | Configuración de Supabase (`SUPABASE_URL`, `SUPABASE_KEY`, `initSupabase`, `setConnStatus`) |
| L1757–L1772 | `ACCESS_TAGS` |
| L1774–L1904 | `SCENARIOS` (contenido estático de los dos escenarios) |
| L1906–L1926 | Estado global (`session`, `map`, intervalos, `realtimeChannel`, `injectedEvents`) |
| L1928–L1980 | Navegación, `cleanup`, `beforeunload` |
| L1982–L2126 | Configuración del Director |
| L2128–L2236 | Ingreso del participante |
| L2238–L2312 | `enterApp`, carga de eventos, heartbeat |
| L2314–L2471 | Suscripciones Realtime y manejadores |
| L2473–L2494 | Filtrado por etiquetas |
| L2496–L2569 | SITREP |
| L2571–L2602 | Mapa |
| L2604–L2995 | Comunicaciones v5 (lista de conversaciones, render de mensajes, adjuntos) |
| L2997–L3082 | Envío desde conversaciones y adjuntos |
| L3084–L3188 | Grabación de audio |
| L3190–L3239 | Cámara |
| L3241–L3341 | **Código heredado v4 sin uso** (`sendChatMessage`, `attachAndSend`) |
| L3343–L3412 | Documentos |
| L3414–L3584 | Panel Director, `injectEvent`, `confirmReset` |
| L3586–L3617 | Pestañas y reloj |
| L3619–L3645 | Utilidades (`escapeHtml`, `safeBody` sin uso, `showToast`) |
| L3647–L3913 | Gestión de archivos y avatares |
| L3915–L3922 | Inicialización (`window.load`) |

### a.2 Librerías externas

| Recurso | Origen | Versión | Observación |
|---|---|---|---|
| Leaflet JS/CSS | `unpkg.com/leaflet@1.9.4` (L9–L10) | 1.9.4 fija | Sin SRI (`integrity`) |
| Supabase JS | `cdn.jsdelivr.net/npm/@supabase/supabase-js@2` (L11) | **Solo la versión mayor (2.x)**; la versión menor/parche cambia sin aviso | Sin SRI |
| Google Fonts | Share Tech Mono, Barlow Condensed, Barlow (L8) | — | — |
| Teselas de mapa | `tile.openstreetmap.org` (L2578) | — | Servidores públicos de OSM, sujetos a su política de uso |
| Visor Office | `view.officeapps.live.com` (L3804) | — | La URL pública del archivo se envía a Microsoft |

No hay bundler, `package.json`, pruebas ni proceso de build. Tampoco hay `render.yaml`: la configuración de Render (sitio estático, reescrituras, cabeceras) es **NO VERIFICABLE DESDE EL CÓDIGO**. El archivo `_redirects` usa el formato de Netlify; si Render lo respeta es **NO VERIFICABLE DESDE EL CÓDIGO**.

### a.3 Inicialización de Supabase

- La URL del proyecto y la clave publicable están escritas en el código (L1731–L1732): `https://nuqteugvwhtrhyxjaxej.supabase.co` y `sb_publishable_…`.
- `initSupabase()` (L1736–L1749) llama a `window.supabase.createClient(URL, KEY)` sin opciones y guarda el cliente en `db`. La variable se llama `db` para no chocar con el global `window.supabase` (commit `94fc56c`).
- Se ejecuta en `window.load` (L3918–L3922). **Pone el indicador en «CONECTADO» sin hacer ninguna petición**: solo confirma que la librería cargó. La conexión real se confirma cuando el canal Realtime devuelve `SUBSCRIBED` (L2344–L2348).
- No hay autenticación de ningún tipo: no aparece `auth.`, `signIn` ni `rpc` en el archivo. Todas las peticiones salen con el rol anónimo de la clave publicable.

---

## b) Inventario funcional

### b.1 Inicio (landing)
| Funcionalidad | Implementación |
|---|---|
| Elegir perfil Director | `goToDirectorSetup()` L1931 |
| Elegir perfil Participante | `goToParticipantLogin()` L1938 |
| Indicador de conexión | `setConnStatus()` L1751 |

### b.2 Configuración del Director
| Funcionalidad | Implementación |
|---|---|
| Ver y seleccionar escenario | `renderScenarioGrid()` L1985, `selectScenario()` L2003 |
| Plantillas de roles (completa: 10, mínima: 4, limpiar) | `loadRolePreset()` L2011 |
| Editar roles: nombre y etiquetas | `renderRoleBuilder()` L2038, `updateRoleName()` L2060, `toggleTag()` L2064 |
| Agregar rol (máx. 10) y quitar rol | `addRole()` L2072, `removeRole()` L2078 |
| Crear sesión y mostrar código | `launchSession()` L2083, `generateSessionCode()` L2118 (formato `AAA-1234`) |
| Volver | `backToLanding()` L1943 |

A los 1,5 s de crear la sesión el Director entra automáticamente a la app (L2105–L2109).

### b.3 Ingreso de participante
| Funcionalidad | Implementación |
|---|---|
| Validar el código de sesión | `joinSession()` L2131 |
| Ver roles libres/ocupados | `renderParticipantRoles()` L2179 |
| Elegir rol | `selectParticipantRole()` L2196 |
| Entrar (reserva el rol) | `enterAsParticipant()` L2202; si la BD devuelve `23505` muestra «ROL OCUPADO» (L2218) |

La lista de roles ocupados se calcula una sola vez al validar el código (L2159–L2164) y no se actualiza mientras el participante elige.

### b.4 Marco común de la app
| Funcionalidad | Implementación |
|---|---|
| Topbar: escenario, rol, etiquetas, nivel de alerta | `enterApp()` L2241 |
| Reloj simulado T+hh:mm:ss | `startClock()` L3600, `stopClock()` L3614 |
| Cambiar de pestaña | `switchTab()` L3589 |
| Avatar del participante (subir imagen ≤ 2 MB) | `setupParticipantAvatar()` L3905, `uploadAvatar()` L3879 |
| Notificaciones toast | `showToast()` L3638 |
| Salir | `exitToLanding()` L1955 → `backToLanding()` → `cleanup()` L1959 |

### b.5 SITREP (pestaña «SITUACIÓN»)
| Funcionalidad | Implementación |
|---|---|
| Indicadores del escenario (4 tarjetas) | `renderStatsHTML()` L2558 (sin filtro por rol) |
| Incidentes activos filtrados por rol | `renderSitrepView()` L2499, `canSeeIncident()` L2484 |
| Centrar el mapa en un incidente | `focusIncident()` L2598 |
| Mapa Leaflet con marcadores | `initMap()` L2574, `addMapMarkers()` L2583 |
| Línea de tiempo precargada filtrada | `canSeeTimeline()` L2490 |
| Eventos de línea de tiempo inyectados (históricos y en vivo) | L2537–L2555 y `handleIncomingEvent()` L2381–L2398 |

### b.6 Comunicaciones
| Funcionalidad | Implementación |
|---|---|
| Lista de conversaciones: 3 grupos (WhatsApp, X/Twitter, SMS) y privadas | `buildConversationList()` L2611, `renderConvSidebar()` L2795, `renderConvItem()` L2831 |
| Privadas del participante: con el Director y con cada otro rol | L2641–L2659 |
| Privadas del Director con cada rol y **monitoreo de solo lectura de cada par de roles** (hasta 45 pares con 10 roles) | L2620–L2640 |
| Abrir una conversación | `openConversation()` L2843, `getConvSubtitle()` L2895 |
| Obtener los mensajes de una conversación (precargados + en vivo) | `getMessagesForConv()` L2665, `eventToConvMsg()` L2754 |
| Pintar los mensajes | `renderConvMessages()` L2908, `renderAttachment()` L2972 |
| Enviar texto | `convSendMessage()` L3000 → `sendToConversation()` L3009 |
| Adjuntar archivo (≤ 20 MB) | `convAttachFile()` L3056 |
| Grabar nota de voz | `toggleRecording()` L3093, `startRecording()` L3102, `stopRecording()` L3131, `cancelRecording()` L3137, `cleanupRecording()` L3145, `updateRecordingTimer()` L3163, `uploadRecordedAudio()` L3172 |
| Foto con la cámara | `openCamera()` L3195, `capturePhoto()` L3206, `closeCamera()` L3233 |
| Recibir mensajes en tiempo real y avisar con toast | `handleIncomingEvent()` L2351 |

### b.7 Documentos
| Funcionalidad | Implementación |
|---|---|
| Documentos del escenario agrupados en SECRETO / CONFIDENCIAL / PÚBLICO y filtrados por etiquetas | `renderDocsView()` L3346, `showDoc()` L3398 |
| Archivos subidos por el Director, filtrados por etiquetas | `renderFilesList()` L3838, `canSeeFile()` L3780 |
| Visor de archivos: imagen, video, audio, PDF, Office (vía Microsoft), texto, descarga | `openFileViewer()` L3784, `closeFileViewer()` L3820 |
| Actualización en vivo de archivos | `handleIncomingFile()` L3861, `handleFileDeleted()` L3873 |

### b.8 Panel Director («CONTROL DIRECTOR»)
| Funcionalidad | Implementación |
|---|---|
| Inyectar evento (canal: línea de tiempo, WhatsApp, Twitter o SMS; urgencia; etiquetas destino; remitente; título; mensaje) | `renderDirectorView()` L3417, `injectEvent()` L3544 |
| Subir archivos (clic o arrastrar, varios a la vez). Las etiquetas se toman del formulario de inyección | `handleFileSelect()` L3759, `setupDropZone()` L3765, `uploadFile()` L3694 |
| Eliminar archivos | `deleteFile()` L3825 |
| Log de inyecciones (registra todos los mensajes que llegan) | `handleIncomingEvent()` L2416–L2436 |
| Participantes conectados con avatar | `handleParticipantChange()` L2439, `updateDirectorRoleStatus()` L2450 |
| Mostrar el código de sesión | L3525–L3531 |
| Terminar y reiniciar | `confirmReset()` L3577 |

### b.9 Funciones definidas pero sin uso
- `sendChatMessage()` L3244: busca `wa-input`, `tw-input` y `sms-input`, que ya no existen en el DOM.
- `attachAndSend()` L3277: nadie la llama. Es el único código que insertaba en `files` desde un chat.
- `safeBody()` L3629: nadie la llama.
- CSS heredado de v4 (`.comm-panel`, `.wa-msg`, `.tweet`, `.sms-msg`, `.attach-btn`, etc., L544–L685): ningún elemento lo usa.

---

## c) Modelo de datos

Los tipos de columna, claves primarias, claves foráneas, índices, valores por defecto y el comportamiento `ON DELETE` son **NO VERIFICABLES DESDE EL CÓDIGO**. Solo se documenta lo que el código lee y escribe.

### c.1 Tabla `sessions`
| Operación | Línea | Columnas |
|---|---|---|
| INSERT | L2094 | `id` (código `AAA-1234`), `scenario_id`, `roles` (array de `{name, tags}`), `status: 'active'` |
| SELECT `*` `.eq('id', code).single()` | L2140 | Lee `id`, `scenario_id`, `roles` |
| DELETE `.eq('id', code)` | L3580 | — |

- `status` se escribe pero **no se lee nunca**. No hay caducidad ni cierre: una sesión solo desaparece si el Director pulsa «Terminar y reiniciar».
- Si `scenario_id` no existe en `SCENARIOS`, `session.scenario` queda `undefined` y el código falla (L2155, L2171).
- Si `DELETE` en `sessions` borra en cascada `events`, `participants` y `files`: **NO VERIFICABLE DESDE EL CÓDIGO**.

### c.2 Tabla `events`
| Operación | Línea | Columnas |
|---|---|---|
| SELECT `*` `.eq('session_id')` ordenado por `created_at` | L2296 | Lee `id`, `channel`, `level`, `sender`, `sender_role_idx`, `recipient_role_idx`, `title`, `body`, `target_tags`, `metadata`, `created_at` |
| INSERT (inyección del Director) | L3555 | `session_id`, `channel`, `level`, `sender`, `title`, `body`, `target_tags` |
| INSERT (mensaje de conversación) | L3048 | Lo anterior más `sender_role_idx`, `recipient_role_idx` y `metadata` (`{attachment:{filename,url,size,type}}`) |
| INSERT (heredado, sin uso) | L3258, L3325 | — |

Valores de `channel`: `timeline`, `whatsapp`, `twitter`, `sms`, `private`. Valores de `level`: `red`, `amber`, `blue`, `green`. `sender_role_idx` y `recipient_role_idx`: índice del rol, `-1` para el Director, `null` para grupo. No hay UPDATE ni DELETE de eventos.

### c.3 Tabla `participants`
| Operación | Línea | Columnas |
|---|---|---|
| SELECT `role_index` `.eq('session_id')` | L2159 | `role_index` |
| INSERT `.select().single()` | L2210 | `session_id`, `role_index`, `role_name`, `tags`. Devuelve la fila; usa `id` |
| UPDATE `last_seen` (cada 15 s) | L2308 | `last_seen` |
| UPDATE `avatar_url` | L3896 | `avatar_url` |
| SELECT `*` `.eq('session_id')` (Director) | L2442 | Lee `role_index`, `avatar_url` |
| DELETE `.eq('id')` | L1967 | — |

- El manejo del código `23505` (violación de unicidad, L2218) sugiere un índice único sobre (`session_id`, `role_index`). Que ese índice exista es **NO VERIFICABLE DESDE EL CÓDIGO**. Sin él, dos personas pueden tomar el mismo rol.
- `last_seen` se escribe pero **nadie lo lee**: no se detecta a los participantes inactivos.

### c.4 Tabla `files`
| Operación | Línea | Columnas |
|---|---|---|
| SELECT `*` `.eq('session_id')` ordenado por `created_at` desc | L3685 | Lee `id`, `filename`, `file_path`, `file_url`, `file_size`, `uploaded_by`, `target_tags` |
| INSERT | L3734 | `session_id`, `filename`, `file_path`, `file_url`, `mime_type`, `file_size`, `uploaded_by`, `target_tags`, `classification: 'PÚBLICO'` |
| DELETE `.eq('id')` | L3831 | — |
| INSERT (heredado, sin uso) | L3308 | — |

`classification` siempre se guarda como `'PÚBLICO'` y no se lee. `mime_type` tampoco se lee: el tipo se deduce de la extensión del nombre (`getFileCategory` L3666).

### c.5 Bucket de Storage `crisis-files`
| Operación | Línea | Ruta |
|---|---|---|
| `upload` (archivos del Director) | L3719 | `{codigo}/{timestamp}_{nombre}` |
| `upload` (adjunto en conversación) | L3069 | `{codigo}/conv/{timestamp}_{nombre}` |
| `upload` (audio) | L3176 | `{codigo}/audio/{timestamp}_voice.webm` |
| `upload` (foto) | L3218 | `{codigo}/photos/{timestamp}_photo.jpg` |
| `upload` con `upsert: true` (avatar) | L3893 | `{codigo}/avatars/{timestamp}_{participantId}.{ext}` |
| `getPublicUrl` | L3071, L3178, L3220, L3729, L3895 | — |
| `remove` | L3830 | `file_path` |

- Todas las URLs son públicas y permanentes (`getPublicUrl`), también las de audios y fotos enviados en conversaciones **privadas**.
- Los adjuntos, audios y fotos de conversaciones no se registran en `files`, así que el Director no puede verlos ni borrarlos desde la interfaz, y `confirmReset` no los elimina.
- Las políticas del bucket (quién puede listar, subir o borrar; límite de tamaño; tipos MIME permitidos) son **NO VERIFICABLES DESDE EL CÓDIGO**. El límite de 20 MB solo se comprueba en el cliente (L3059, L3696).

### c.6 Canales Realtime
Un canal por cliente: `session-{codigo}` (`subscribeToRealtime()` L2317–L2349), con `postgres_changes` filtrados por `session_id=eq.{codigo}`:

| Tabla | Evento | Manejador |
|---|---|---|
| `events` | INSERT | `handleIncomingEvent` L2351 |
| `files` | INSERT | `handleIncomingFile` L3861 |
| `files` | DELETE | `handleFileDeleted` L3873 |
| `participants` | INSERT / DELETE / UPDATE | `handleParticipantChange` L2439 (solo actúa en el Director) |

- No hay suscripción a `sessions`: los participantes no se enteran si el Director termina la sesión.
- No se usan Broadcast ni Presence.
- Que las tablas estén en la publicación `supabase_realtime`, y su `REPLICA IDENTITY`, es **NO VERIFICABLE DESDE EL CÓDIGO**. La documentación de Supabase indica que los eventos `DELETE` de Postgres Changes no se pueden filtrar por columna y que `old` solo trae la clave primaria salvo con `REPLICA IDENTITY FULL`. Por eso las suscripciones DELETE con `filter` (L2329, L2337) podrían no recibir nada. Cómo se comportan en este proyecto es **NO VERIFICABLE DESDE EL CÓDIGO**.

### c.7 Políticas RLS
**NO VERIFICABLE DESDE EL CÓDIGO.** Solo se puede afirmar esto: como no hay autenticación, para que la aplicación funcione como está escrita, el rol anónimo necesita permiso para SELECT/INSERT/DELETE en `sessions`, SELECT/INSERT en `events`, SELECT/INSERT/UPDATE/DELETE en `participants`, SELECT/INSERT/DELETE en `files`, y upload/remove en el bucket. Si las políticas limitan esas operaciones con condiciones adicionales no se puede saber desde aquí.

---

## d) Escenarios y roles

### d.1 Estructura de un objeto en `SCENARIOS` (L1777)
```
{
  id, icon, title, desc,
  primaryTags: [TAG…],            // solo se muestra en la tarjeta (L1997)
  alertLevel, startDate,           // texto de la topbar
  startSeconds,                    // valor inicial del reloj T+
  mapCenter: [lat,lng], mapZoom,
  stats:     [{label, val, color}],                                 // sin filtro
  incidents: [{lat, lng, type: critical|warning|info, tagType, title, meta, label}],
  timeline:  [{time, dot, tagType, title, desc}],
  twitter:   [{name, handle, avatar, avatarBg, text, time, urgent?, requiresTags}],
  whatsapp:  [{type?:'system', sender, text, time, requiresTags}],
  sms:       [{from, text, time, requiresTags}],
  documents: [{id, class: SECRETO|CONFIDENCIAL|PÚBLICO, icon, title, meta[], requiresTags, body(HTML)}]
}
```

### d.2 Escenarios existentes

| | **Vacancia presidencial** (`vacancia`, L1778–L1839) | **Las Bambas** (`lasbambas`, L1840–L1903) |
|---|---|---|
| Alerta / fecha | ALERTA POLÍTICA · 12 MAR 2026 14:00 | ALERTA SOCIAL · 08 ABR 2026 06:30 |
| Reloj inicial | T+02:00:00 (7.200 s) | T+04:00:00 (14.400 s) |
| Etiquetas principales | POLITICA, COMUNICACION, INTELIGENCIA, JUDICIAL | RIESGO, INTERIOR, REGIONAL, ECONOMICO |
| Indicadores | 4 | 4 |
| Incidentes | 5 (2 POLITICA, 2 INTERIOR, 1 COMUNICACION) | 5 (3 INTERIOR, 1 RIESGO, 1 REGIONAL) |
| Línea de tiempo | 6 eventos | 7 eventos |
| Tweets | 5 (4 públicos, 1 DIPLOMATICO) | 6 (5 públicos, 1 DIPLOMATICO) |
| WhatsApp | 1 cabecera de sistema y 6 mensajes | 1 cabecera de sistema y 6 mensajes |
| SMS | 6 | 6 |
| Documentos | 5: Análisis Político (SECRETO), Movilizaciones (SECRETO), Protocolo Sucesión (CONF.), Impacto Económico (CONF.), Borrador Comunicado (PÚBLICO) | 5: Reporte Operacional (SECRETO), Acuerdos 2014 (CONF.), Impacto Económico (CONF.), Lineamientos Diálogo (PÚBLICO), Comunicación Embajada China (SECRETO) |

Todo el contenido está escrito dentro de `index.html`. No hay forma de crear o editar escenarios sin tocar el código.

### d.3 Las 11 `ACCESS_TAGS` (L1760–L1772)
`POLITICA` (Decisión Política), `MILITAR` (Militar / FFAA), `INTERIOR` (Seguridad Interna), `SALUD` (Salud Pública), `RIESGO` (Gestión de Riesgo), `INTELIGENCIA`, `COMUNICACION` (Comunicación Pública), `REGIONAL` (Nivel Regional), `ECONOMICO` (Económico-Financiero), `DIPLOMATICO` (Diplomático / RREE), `JUDICIAL` (Judicial / Fiscal).

La etiqueta `SALUD` no se usa en ningún contenido de los dos escenarios. Solo sirve para asignarla a roles.

Plantillas de roles (L2011–L2036): la **mínima** tiene 4 roles (Presidente, Primer Ministro, Ministro de Defensa, Ministro del Interior) y la **completa** tiene 10 (añade Salud, Economía, Canciller, Vocero, Jefe INDECI y Gobernador Regional).

### d.4 Cómo se aplica el filtrado

Todo el filtrado ocurre **en el navegador**. El Director siempre ve todo.

| Contenido | Regla | Función |
|---|---|---|
| Incidentes (lista y mapa) | Una sola etiqueta `tagType`; visible si el rol la tiene; sin `tagType` es público | `canSeeIncident` L2484 |
| Línea de tiempo precargada | Igual que los incidentes | `canSeeTimeline` L2490 (idéntica a la anterior) |
| Línea de tiempo inyectada | Lógica OR sobre `target_tags`; vacío = público | `canSeeByTags` L2476 |
| Tweets, WhatsApp y SMS precargados | Lógica OR sobre `requiresTags` | `canSeeByTags` en `getMessagesForConv` L2672/L2683/L2694 |
| Mensajes inyectados en grupos | Lógica OR sobre `target_tags` y `recipient_role_idx == null` | L2675–L2698 |
| Mensajes de participantes en grupos | Siempre `target_tags: []`, así que **los ve todo el mundo** | L3045 |
| Conversaciones privadas | Por `sender_role_idx` y `recipient_role_idx`; el Director ve todas | L2703–L2749, L2358–L2362 |
| Documentos del escenario | Lógica OR sobre `requiresTags` | L3348 |
| Archivos subidos | Lógica OR sobre `target_tags` | `canSeeFile` L3780 |
| Indicadores (`stats`) | Sin filtro | L2505 |

Las etiquetas del participante vienen de `sessions.roles[idx].tags` y se guardan en memoria (`session.currentParticipant.tags`, L2199). El filtrado solo decide **qué se pinta**. Los datos ya están en el navegador (ver hallazgos C1 y C2).

---

## e) Flujo de una sesión de principio a fin

1. **Creación.** El Director elige escenario y roles y pulsa «Iniciar». `launchSession` genera un código aleatorio con `Math.random` (L2118) e inserta en `sessions` (L2094). Si el código ya existe, la inserción falla (asumiendo que `id` es clave primaria, **NO VERIFICABLE**) y no se reintenta. A los 1,5 s entra a la app como `DIRECTOR` con índice `-1`. **El Director no se registra en `participants`** y su identidad solo existe en la memoria de esa pestaña.
2. **Ingreso.** El participante escribe el código. `joinSession` lee la sesión y los roles ocupados. `enterAsParticipant` inserta la fila en `participants`.
3. **Entrada a la app** (`enterApp` L2241). Pinta la topbar, arranca el reloj local desde `startSeconds`, carga **todos** los eventos y archivos de la sesión, pinta las 4 vistas, crea el mapa, se suscribe a Realtime y, si es participante, envía un heartbeat cada 15 s.
4. **Inyección de eventos.** El Director completa el formulario. `injectEvent` inserta en `events`. Cada cliente recibe el INSERT por Realtime (`handleIncomingEvent`), decide si le corresponde, lo añade a la línea de tiempo o a la conversación activa, y muestra un toast. El Director lo registra en el log.
5. **Mensajería.** `sendToConversation` inserta en `events` con `channel` y `recipient_role_idx` según la conversación activa. El remitente también ve su propio mensaje porque le llega de vuelta por Realtime; no hay pintado optimista.
6. **Adjuntos, audio y cámara.** Se sube el archivo a `crisis-files`, se obtiene la URL pública y se envía un evento con `metadata.attachment`. El audio usa `MediaRecorder`. La foto se toma con `getUserMedia({video:{facingMode:'user'}})`, se dibuja en un canvas y se exporta como JPEG al 85 %. Los archivos del panel Director van a `files` y aparecen en Documentos.
7. **Reinicio.** `confirmReset` pide confirmación, borra la fila de `sessions` (ignorando errores), ejecuta `cleanup()` y recarga la página. No borra objetos de Storage. Los participantes no reciben aviso y siguen en una sesión que ya no existe.
8. **Salida.** «Salir» ejecuta `backToLanding` → `cleanup` (quita el canal, borra la fila en `participants`, resetea el estado). Si el que sale es el Director, la sesión sigue activa en la BD pero **ya nadie puede volver a entrar como su Director**. Al cerrar la pestaña, el `sendBeacon` de L1972–L1980 no libera el rol (ver A3).

---

## f) Riesgos y brechas

### Crítico

**C1. No hay control de acceso real. Cualquiera puede actuar como cualquier rol o como el Director.**
- No hay autenticación (a.3). Cualquier persona que tenga la clave publicable, que va en el propio HTML, puede usar directamente la API REST de Supabase para: insertar eventos con `sender:'DIRECTOR'` y `sender_role_idx:-1`, es decir, hacerse pasar por el Director o por cualquier rol; borrar sesiones (L3580); borrar archivos y objetos del bucket (L3830–L3831); y borrar o modificar filas de `participants`. Para que la aplicación funcione, RLS tiene que permitir estas operaciones al rol anónimo (c.7). Si existe alguna restricción adicional es **NO VERIFICABLE DESDE EL CÓDIGO**.
- Desde la interfaz, quien tenga el código puede tomar cualquier rol libre. No hay contraseña de rol ni aprobación del Director.
- El «modo Director» es solo una bandera en memoria (`session.isDirector`, L2106). Cualquiera puede crear sesiones sin límite, y no hay forma de que el Director legítimo recupere el control de su sesión.
- El espacio de códigos es de 24³ × 8⁴ ≈ 56,6 millones. Si RLS permite `SELECT` en `sessions` sin filtrar (**NO VERIFICABLE**), cualquiera puede listar todas las sesiones y sus roles.

**C2. Las conversaciones privadas y el contenido «SECRETO» solo son privados en la interfaz.**
- `loadInjectedEvents` (L2296) descarga **todos** los eventos de la sesión en cada cliente, incluidos los mensajes privados entre otros roles y con el Director. Realtime también envía cada INSERT a todos los suscriptores (L2320–L2323). El filtro `isVisibleToMe` (L2358) solo decide qué se pinta. Cualquier participante puede leer todo abriendo la consola del navegador o la pestaña de red.
- Los documentos SECRETO y CONFIDENCIAL de ambos escenarios, y todos los mensajes precargados con restricción, están escritos en el HTML (L1777–L1904). Basta con ver el código fuente.
- Los audios, fotos y adjuntos privados tienen URLs públicas y permanentes (c.5).
- Para un ejercicio académico esto afecta la integridad de la dinámica (asimetría de información entre roles), no a datos personales reales. Pero si alguien usa datos reales o sensibles en un ejercicio, quedan expuestos.

**C3. XSS almacenado: cualquier cliente puede ejecutar código en el navegador de todos los demás, incluido el Director.**
`showToast` (L3642) mete `title` y `msg` en `innerHTML` sin escapar. Varios de esos valores vienen de la base de datos y los controla cualquiera que pueda insertar filas:
- `handleIncomingEvent` L2411–L2413: `event.sender` y `event.body`/`title` van al toast **sin escapar**. Basta un mensaje de chat, desde la propia interfaz, con `<img src=x onerror=…>` para ejecutar código en todos los clientes conectados que tengan visibilidad del mensaje. En los grupos, eso son todos.
- `handleIncomingFile` L3868: `file.filename` sin escapar.
- `enterApp` L2288: el nombre del rol sin escapar.
- También se interpolan sin escapar: `event.level` en un atributo `class` (L2390, L2547); `event.channel` y `target_tags` en el log del Director (L2430–L2432); `att.url` dentro de `onclick="window.open('…')"` y `src` (L2978–L2988), y `metadata` lo controla quien envía; `p.avatar_url` en `src` (L2464); `file.file_url` (L3794–L3813); `f.target_tags` (L3853); y los nombres y etiquetas de rol en `renderParticipantRoles` (L2189), `renderDirectorView` (L3517–L3518) y `renderRoleBuilder` (L2046).
- Combinado con C1, un atacante externo, sin estar en la sala, puede inyectar código en la sesión si conoce el código de sesión.

### Alto

**A1. No hay persistencia ante recarga ni desconexión.** Todo el estado vive en memoria (`session`, L1909). Al recargar se vuelve al landing. El participante tiene que reingresar y su rol probablemente aparece como ocupado (A3). **El Director pierde la sesión para siempre**: no puede volver a entrar como Director y el panel de control queda inaccesible. No hay `localStorage`, `sessionStorage` ni token de reconexión.

**A2. Realtime no se recupera tras un corte de red.** El callback de `subscribe` solo trata `SUBSCRIBED` (L2345). No hay manejo de `CHANNEL_ERROR`, `TIMED_OUT` ni `CLOSED`: el indicador sigue diciendo «CONECTADO» y **los eventos emitidos durante el corte no se recuperan**, porque tras una reconexión no se vuelve a leer `events`. Esto afecta sobre todo a iPad y móviles, que suspenden las pestañas en segundo plano.

**A3. Los roles quedan «ocupados» para siempre.** El `sendBeacon` de `beforeunload` (L1975) envía un **POST vacío sin la cabecera `apikey`** a `/rest/v1/participants?id=eq.X`. Esto no es un DELETE, y además `sendBeacon` no puede enviar cabeceras personalizadas, así que no libera el rol. Como `last_seen` no se lee (c.3), ningún proceso limpia las filas. Si un participante cierra la pestaña o se le cae el navegador, no puede volver a tomar su rol, y el Director lo sigue viendo «en línea».

**A4. Errores de red silenciados y avisos de éxito falsos.** supabase-js no lanza excepciones: devuelve `{error}`. Muchas llamadas no lo revisan: `loadInjectedEvents` (L2296), `loadUploadedFiles` (L3685), `joinSession` al leer participantes (L2159), `handleParticipantChange` (L2442), `updateHeartbeat` (L2308), `deleteFile` (L3830–L3831), `confirmReset` (L3580) y `uploadAvatar` al hacer el update (L3896). Además, `sendToConversation` captura su propio error, así que `convAttachFile`, `uploadRecordedAudio` y `capturePhoto` muestran «ENVIADO» aunque la inserción del evento haya fallado (L3077, L3184, L3226). Si falla la carga inicial de eventos, el usuario ve el ejercicio vacío sin ningún aviso.

**A5. No es usable en móvil y tiene limitaciones fuertes en iPad.**
- No hay ninguna `@media`. Las rejillas tienen columnas fijas: SITREP `260px 1fr 280px` (L458), Comunicaciones `280px 1fr` (L1106), Documentos `280px 1fr` (L688), Director `1fr 1fr 320px` (L768), editor de roles `60px 200px 1fr 30px` (L206).
- `body {overflow:hidden; height:100vh}` (L41–L42): en iOS Safari, `100vh` incluye la barra del navegador, así que el área de escritura del chat puede quedar tapada, y no se puede hacer scroll de página.
- El audio se etiqueta siempre como `audio/webm` (L3113, L3176), ignorando `mediaRecorder.mimeType`. Safari en iOS graba en MP4/AAC, así que el archivo queda mal etiquetado. Si se reproduce en otros navegadores es **NO VERIFICABLE DESDE EL CÓDIGO** sin probarlo en un dispositivo.
- La cámara usa solo la frontal (`facingMode:'user'`, L3197). No hay opción para la trasera ni `input capture`.
- Arrastrar y soltar archivos no funciona con pantallas táctiles. Los adjuntos sí funcionan con el botón.

**A6. Bug de layout: la vista Comunicaciones se muestra siempre.** `#view-comms.v5-layout { display: grid !important }` (L1104–L1106) anula `.tab-view { display:none }` (L454). Desde que `renderCommsView` le asigna `v5-layout` (L2775), la vista queda visible **en todas las pestañas**, compartiendo el ancho de `.main-content` (flex, L453) con SITREP, Documentos o Director. Esto se deduce de la cascada CSS; no se probó en un navegador en este diagnóstico, así que conviene confirmarlo visualmente.

### Medio

- **M1. El reloj no está sincronizado.** Cada cliente arranca su reloj local desde `startSeconds` al entrar (L2265). Quien entra más tarde ve otra hora T+. Los eventos inyectados en la línea de tiempo muestran la **hora real** de `created_at` (L2385), no la hora simulada. `sim-date` solo aparece tras el primer tic (L3610).
- **M2. Las notas de voz se muestran como video.** `getFileCategory` clasifica `.webm` como video antes que como audio (L3669), y las notas se llaman `audio_voz.webm` (L3182). Por eso se pintan con `<video controls>` (L2985).
- **M3. Los tweets precargados muestran HTML literal.** El `text` de los tweets contiene `<span class="hashtag">…</span>` (L1809, L1812, L1872), pero `renderConvMessages` lo escapa (L2961). Tampoco se usan `avatar`, `avatarBg`, `urgent` ni `handle` en la vista v5.
- **M4. Cada mensaje nuevo repinta la conversación entera.** `handleIncomingEvent` vuelve a pintar todos los mensajes de la conversación activa con `innerHTML`, aunque el mensaje nuevo sea de otra (L2372–L2378). Esto **detiene cualquier audio o video que se esté reproduciendo** y fuerza el scroll hasta abajo. No hay contador de no leídos por conversación: el único aviso es un toast de 5 s.
- **M5. El Director recibe un toast por sus propias inyecciones.** `injectEvent` no envía `sender_role_idx`, así que llega como `null` (si la columna admite nulos; **NO VERIFICABLE**), y `null !== -1` activa el toast (L2401).
- **M6. Rendimiento con muchos eventos.** Se cargan todos los eventos sin paginar (L2296). PostgREST limita las filas por respuesta según la configuración del proyecto (**NO VERIFICABLE**); si se alcanza ese límite, el historial se corta sin aviso. Además, cada evento hace varias pasadas lineales sobre `injectedEvents` y reconstruye la lista de conversaciones: con 10 roles, el Director tiene 10 privadas más 45 de monitoreo. Y cada heartbeat (10 participantes × cada 15 s) dispara un UPDATE por Realtime que hace que el Director vuelva a pedir la tabla `participants` completa (L2439).
- **M7. Los participantes no se enteran del cierre de sesión.** Ver e.7. Siguen escribiendo en una sesión que ya no existe, y el resultado depende de las FK (**NO VERIFICABLE**).
- **M8. Los archivos subidos por participantes no llegan a Documentos.** Solo `uploadFile` (Director) inserta en `files`. Los adjuntos de conversación no quedan registrados, no se pueden gestionar y `confirmReset` no los borra. El Storage acumula objetos huérfanos.
- **M9. Filtrado incoherente.** Incidentes y línea de tiempo usan una sola etiqueta (`tagType`, coincidencia exacta). El resto usa `requiresTags` con lógica OR. Los indicadores nunca se filtran. Los mensajes de participantes en grupos siempre son públicos, aunque el grupo WhatsApp precargado esté restringido (la cabecera del grupo exige ciertas etiquetas, L1816).
- **M10. Las etiquetas de los archivos del Director se toman del formulario de inyección** (L3702). El formulario se limpia tras cada inyección, así que es fácil subir un archivo sin querer como «PÚBLICO».
- **M11. Dependencias sin fijar y sin SRI.** `supabase-js@2` usa la última 2.x disponible (a.2). Un cambio en la librería puede romper la app sin que cambie el código. Tampoco hay `integrity` en los scripts de terceros.
- **M12. Terceros.** Al abrir un archivo Office, su URL pública se envía a Microsoft (L3804). Las teselas se piden a los servidores públicos de OSM (L2578).

### Bajo

- **B1. Código muerto:** `sendChatMessage`, `attachAndSend` y `safeBody` (b.9), unas 140 líneas de CSS heredado (L544–L685), las variables `ext` sin usar (L3292, L3715, L3891) y la variable `btn` sin usar en `toggleRecording` (L3094).
- **B2. Código duplicado:**
  - El HTML de un ítem de línea de tiempo está copiado en L2386–L2395 y L2543–L2552.
  - El formateo de hora está escrito en línea tres veces (L2385, L2421, L2542), aunque existe `formatTime` (L2768).
  - El cálculo de iniciales aparece 5 veces (L2466, L2626, L2654, L3511, L3910).
  - La secuencia «subir a Storage + obtener URL pública» se repite en 6 funciones.
  - Las tres ramas de grupo de `getMessagesForConv` son casi iguales.
  - `canSeeIncident` y `canSeeTimeline` son idénticas.
  - El objeto inicial de `session` se define dos veces (L1909 y L1951).
- **B3. Funciones demasiado largas:** `renderDirectorView` (126 líneas, L3417–L3542, mezcla HTML y lógica), `getMessagesForConv` (88), `handleIncomingEvent` (87) y `openConversation` (51).
- **B4. Un solo archivo de 3.925 líneas**, sin módulos, sin pruebas y sin lint. Las funciones son globales y se llaman desde `onclick` en línea.
- **B5. La lista de roles ocupados no se actualiza** en la pantalla de ingreso (b.3).
- **B6. `_redirects` es de Netlify.** Si Render lo aplica es NO VERIFICABLE (a.2). Como la app no usa rutas, el efecto es mínimo.
- **B7. Accesibilidad:** hay elementos clicables que son `div` (tarjetas de escenario, roles, conversaciones), sin roles ARIA ni navegación con teclado. El texto usa tamaños de 9–11 px.

---

## g) Funcionalidades ausentes relevantes para un ejercicio de crisis

| Funcionalidad | Estado en v5.0 | Notas para el diseño |
|---|---|---|
| **Cronograma de inyecciones programadas** (MSEL) | **No existe.** Toda inyección es manual e inmediata (`injectEvent`). | Hace falta modelar inyecciones con hora simulada o real de disparo, poder activarlas y desactivarlas, y decidir quién las dispara. Si las dispara un cliente, el Director tiene que tener la pestaña abierta; si las dispara el servidor, se necesita Edge Function, cron o pg_cron. |
| **Control de fases y reloj del ejercicio** (iniciar, pausar, acelerar, saltar, fases) | **No existe.** El reloj es local e independiente en cada cliente (M1). | Requiere un estado de reloj compartido en `sessions` (inicio, factor de velocidad, pausa) y que cada cliente calcule la hora simulada a partir de él. |
| **Informe final / after-action review exportable** | **No existe.** No hay exportación ni vista de cronología consolidada. Los datos de `events` y `files` sí permiten reconstruirlo. | Timeline consolidado, decisiones por rol, tiempos de respuesta y exportación a PDF o DOCX. Hay que resolver C2 antes de guardar evaluaciones. |
| **Editor de escenarios en la interfaz** | **No existe.** Los escenarios están escritos en `SCENARIOS` (L1777). | Hace falta una tabla `scenarios` (o JSON en Storage) y un editor de incidentes, línea de tiempo, mensajes y documentos con etiquetas. Mover los escenarios a la BD también ayuda a resolver parte de C2, siempre que se filtre en el servidor. |
| **Videoconferencia (Jitsi)** | **No existe.** Solo hay notas de voz y fotos. | Se puede incrustar Jitsi Meet (`external_api.js` o iframe) con una sala por sesión y/o por conversación. Hay que evaluar la privacidad de la sala (nombre no adivinable, JWT si se usa un servidor propio) y el rendimiento en iPad. |
| **Generación de escenarios con IA** | **No existe.** | Requiere un backend que guarde la clave del proveedor (nunca en el cliente) y que valide la salida contra el esquema de d.1. Encaja naturalmente con el editor de escenarios. |

---

## Resumen de verificabilidad

Quedan **NO VERIFICABLES DESDE EL CÓDIGO** y deberían revisarse en el panel de Supabase y de Render:
1. Políticas RLS de `sessions`, `events`, `participants` y `files`.
2. Políticas del bucket `crisis-files`: listado, subida, borrado, tamaño y MIME.
3. Esquema: tipos, PK, FK con `ON DELETE CASCADE`, índice único de (`session_id`, `role_index`) y columnas que admiten nulos.
4. Publicación Realtime y `REPLICA IDENTITY` de las tablas.
5. Límite de filas de PostgREST (`max_rows`).
6. Configuración del sitio en Render: reescrituras, cabeceras de seguridad y CSP.
7. Comportamiento real de audio y cámara en iOS/iPadOS: requiere prueba en dispositivo.
