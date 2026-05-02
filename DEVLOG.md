# DEVLOG — Tesoreros App

## [2026-05-02] — v2.88→v2.94: cuotas desktop Alt G + fixes panel

**Resumen:** Rediseño completo del panel de cuotas desktop según mockup Alt G: sidebar con tinte slate (`var(--bg)`), items como cards con sombra al seleccionar, tabs como pills con fill azul, panel derecho con banner navy (título + KPIs inline + barra de progreso) seguido de área de formulario blanca. Múltiples fixes de estabilidad y dark mode.

**Archivos:** `index.html`, `backbone-mockups/src/projects/tesoreros/CuotasDesktopAltG.tsx`, `backbone-mockups/src/projects/tesoreros/CuotasDesktopAltH.tsx`, `backbone-mockups/src/hub/data.ts`, `backbone-mockups/src/router.tsx`

**Decisiones:**
- `+ Nueva` movido al header de página (TAB_META action) — tabs con más espacio para respirar.
- Panel estable entre tabs: `#app` cambió de `min-height:100vh` a `height:100vh;overflow:hidden` para que `height:100%` en `.cuotas-md` resuelva a altura definitiva. Sin esto, el panel crecía con el contenido del sidebar.
- Banner alineado entre cuota y actividad: se renderiza un `<span visibility:hidden>` placeholder en cuotas para reservar el mismo espacio que el badge "Cerrada/Abierta" de actividades.
- Sidebar/footer usaban `#f1f5f9` hardcodeado → reemplazado por `var(--bg)` para dark mode.
- `overflow-y:scroll` (no `auto`) en el área de formulario para que el scrollbar siempre reserve espacio y no cause layout shift.

**Pendientes:**
- [ ] Verificar preview de WhatsApp con logo real subido desde superadmin
- [ ] Editar actividad existente (nombre, precios, fecha límite)
- [ ] Link apoderado generado desde la app por el tesorero
- [ ] Exportar estado de pagos (Excel/imagen)
- [ ] Recordatorios de deuda (texto pre-armado para WhatsApp)
- [ ] Comprobante de pago individual

---

## [2026-04-30] — v2.84→v2.87: panel cuotas desktop + panel actividades tramos rediseñados

**Resumen:** Registro del mockup Alt E en el hub (data.ts + router.tsx). Panel de detalle de cuota rediseñado para coincidir exactamente con Alt E: fondo blanco, label "EDITAR", campos a la izquierda (max-width 560px). Panel de actividades por tramos: reemplazada tabla con emojis desalineados por cards con botones +/− circulares (mismo estilo apoderado). Se agregó buscador de familia y toggle colapsar/expandir por card.

**Archivos:** `index.html`, `backbone-mockups/src/hub/data.ts`, `backbone-mockups/src/router.tsx`

**Decisiones:**
- Form area con fondo `var(--surface)` (blanco) en lugar de `var(--surface2)` — menos peso visual, más limpio
- Cards actividades usan mismos IDs DOM (`actseg-{tramo}-{sid}`, `acttotal-{sid}`) — funciones `actAdjTramo` / `actUpdateHeader` sin modificar
- Familias con al menos un tramo > 0 → expandidas por defecto; todas en 0 → colapsadas
- Migración Firebase→Supabase: postergada indefinidamente (complejidad alta sin beneficio urgente)

**Pendientes:**
- [ ] Verificar preview de WhatsApp con logo real subido desde superadmin
- [ ] Editar actividad existente (nombre, precios, fecha límite)
- [ ] Link apoderado generado desde la app por el tesorero
- [ ] Exportar estado de pagos (Excel/imagen)
- [ ] Recordatorios de deuda (texto pre-armado para WhatsApp)
- [ ] Comprobante de pago individual

---

## [2026-04-30] — v2.61→v2.62: Open Graph + logo colegio en superadmin

**Resumen:** Links de WhatsApp ahora muestran preview con título, descripción e imagen. Se agregó upload real de logo por colegio en el superadmin (Canvas resize → base64 → Firebase). El logo se muestra en el header de la vista apoderado y alimenta el endpoint dinámico `/api/og.py` que genera la imagen OG con el logo del colegio.

**Archivos:** `index.html`, `og-image.png` (nuevo), `api/og.py` (nuevo), `api/requirements.txt` (nuevo), `CLAUDE.md`

**Decisiones:**
- Logo como base64 en Firebase RTDB (`/plataforma/colegios/{cid}/logoBase64`) — sin servicio externo, funciona con auth ya existente.
- Endpoint Python en `/api/og.py` con Pillow — genera PNG 1200×630 con logo del colegio al vuelo; `og:image` en HTML apunta a `/api/og?colegio=sg`.
- Resized a máx 400px client-side con Canvas antes de guardar — evita almacenar imágenes enormes en Firebase.
- `og:image` estático apunta a `sg` hardcodeado por ahora — si se suman colegios se puede agregar Edge Middleware para hacerlo dinámico por URL.

**Pendientes:**
- [ ] Verificar preview de WhatsApp con logo real subido desde superadmin
- [ ] Editar actividad existente (nombre, precios, fecha límite)
- [ ] Link apoderado generado desde la app por el tesorero
- [ ] Exportar estado de pagos (Excel/imagen)
- [ ] Recordatorios de deuda (texto pre-armado para WhatsApp)
- [ ] Comprobante de pago individual

---

## [2026-04-29] — v2.42→v2.46: modal edición variable, inscripción online, refactor cuotas/actividades

**Resumen:** Sesión larga. Fix modal edición cuotas variables (muestra wizard completo con parámetros precargados). Modales con versión desktop (max-width, fixed header/footer, scroll body). Fix Bingo $0 en ficha alumno y --subtle invisible en light mode. Feature inscripción online: link compartible para actividades, la gente se auto-inscribe vía URL pública hasta fecha límite. Refactor conceptual: cuotas siempre obligatorias (igual/excepciones), actividades siempre opcionales — modo "por tramos de familia" (adulto/adolescente/niño) migrado de cuotas a actividades.

**Archivos:** `index.html`

**Decisiones:**
- Cuotas = obligatorias, sin link de inscripción. Actividades = opcionales, siempre con link disponible.
- Modo "por tramos" en actividades: `modoPrecios:"tramos"`, `compromisos:{sid:{a,adol,n}}`, precios separados por tramo. Actividades "por unidades" siguen con `compromisos:{sid:N}`.
- Cuotas perperson existentes en Firebase siguen funcionando via `amounts[sid]` — compatibilidad backwards preservada sin migración.
- Link inscripción pública: `?inscripcion=QID&colegio=sg&curso=4B` — no requiere login, escribe directo a Firebase. Cierra automáticamente al pasar `fechaLimiteInscripcion`.

**Pendientes:**
- [ ] Editar actividad existente (fecha, precio, fecha límite)
- [ ] Link apoderado generado desde la app (sin pasar por superadmin)
- [ ] Exportar estado de pagos (Excel/imagen)
- [ ] Recordatorios de deuda (texto pre-armado para WhatsApp)
- [ ] Comprobante de pago individual

---

## [2026-04-23] — v2.32: fix vista apoderados en Pagos

**Resumen:** El CSS `.apoderado-mode [onclick*="requireAdmin"]` ocultaba las filas de alumnos en Pagos (`.s-row`) porque también son botones con requireAdmin. Fix: agregar `:not(.s-row)` al selector para excluirlas. Apoderados ahora ven quién pagó y quién no.

**Archivos:** `index.html`

**Decisiones:** Mantener el CSS genérico pero excluir `.s-row` explícitamente — más simple que refactorizar todos los botones admin.

**Pendientes:**
- [ ] Link apoderado generado desde la app (sin pasar por superadmin)
- [ ] Distribución más elegante del link apoderado (QR, WhatsApp, PIN directo)
- [ ] Exportar estado de pagos (Excel/imagen)
- [ ] Recordatorios de deuda (texto pre-armado para WhatsApp)
- [ ] Comprobante de pago individual

---

## [2026-04-22] — v2.17→v2.31: ingresos proyectados, modo a-la-fecha cross-cutting, fixes UX

**Resumen:** Sesión larga de mejoras y fixes. Se agregaron ingresos proyectados como entidad nueva (pestaña Gastos). El toggle "A la fecha / Proyectado" se extendió a Reportes y Gastos de forma cross-cutting — gastos futuros se ocultan globalmente en modo "a la fecha". Fixes de consistencia entre Resumen/Reportes (colores, labels, cálculo de porRecaudar incluyendo actividades). Fix UX: link apoderado reabre sheet al guardar, apoderados ven solo lo que corresponde, sort por fecha en gastos, PxQ visible en footer de cuotas.

**Archivos:** `index.html`

**Decisiones:**
- Ingresos proyectados: entidad simple `{ id, descripcion, monto, fechaEstimada }` — se elimina cuando se crea la cuota real, sin estado "realizado"
- Modo a-la-fecha/proyectado: mismo localStorage key `resumenMode` compartido entre Resumen, Gastos y Reportes
- Ocultar botones admin para apoderados: CSS `.apoderado-mode [onclick*="requireAdmin"]{ display:none }` — cero cambios en render functions
- Actividades colectivas incluidas en `porRecaudar` — tenían montos comprometidos calculables pero estaban excluidas
- Reportes: KPI labels y colores alineados con Resumen (balance positivo = azul #5b82f0)

**Pendientes:**
- [ ] Link apoderado más elegante (QR, WhatsApp, PIN directo) — postergado
- [ ] Link apoderado generado desde la app por el tesorero (sin pasar por superadmin)
- [ ] Exportar estado de pagos (Excel/imagen)
- [ ] Recordatorios de deuda (texto pre-armado para WhatsApp)
- [ ] Comprobante de pago individual

---

## [2026-04-19 00:00] — v2.16: asistente IA más robusto e interpretativo

**Resumen:** Mejoras al asistente ✨ Entrada rápida. Parsing de JSON más robusto (extrae array con regex aunque el modelo agregue texto extra o markdown). Feedback explícito cuando un alumno ya estaba pagado (antes se omitía silenciosamente). Muestra monto ajustado si queda saldo parcial. Modelo actualizado a `claude-sonnet-4-6`. Prompt mejorado para matchear por apellido, apodo parcial o "todos".

**Archivos:** `index.html`

**Decisiones:** Usar regex `/\[[\s\S]*\]/` para extraer JSON en vez de parsear el texto raw completo — evita fallos cuando el modelo envuelve la respuesta en texto extra. Solo pasar alumnos activos al modelo. Modelo anterior `claude-sonnet-4-20250514` reemplazado por `claude-sonnet-4-6`.

**Pendientes:**
- [ ] Link apoderado generado desde la app (tesorero, sin pasar por superadmin)
- [ ] Exportar estado de pagos (Excel/imagen)
- [ ] Recordatorios de deuda (texto pre-armado para WhatsApp)
- [ ] Comprobante de pago individual

---

## [2026-04-15 12:30] — v2.13/v2.14/v2.15: fixes login + toggle proyectado + versión visible

**Resumen:** 3 fixes de login (form password managers, remove trim(), emoji favicon). Toggle "A la fecha / Proyectado" en Resumen con KPIs consistentes — modo "a la fecha" muestra gastos reales (≤ hoy) vs ingresos reales, modo "proyectado" suma por recaudar de cuotas activas y todos los gastos incluyendo futuros, con detalle desglosado. Versión app visible en sidebar desktop y header móvil.

**Archivos:** `index.html`

**Decisiones:** Toggle persiste en localStorage. Actividades colectivas excluidas del cálculo de "por recaudar" (solo cuotas normales). Modo proyectado reutiliza la lógica del antiguo "Flujo proyectado" como sección de detalle. Favicon inline SVG sin archivo externo.

**Pendientes:**
- [ ] Bug form passwords (resuelto ✓)
- [ ] Link apoderado generado desde la app (tesorero, sin pasar por superadmin)
- [ ] Exportar estado de pagos (Excel/imagen)
- [ ] Recordatorios de deuda (texto pre-armado para WhatsApp)
- [ ] Comprobante de pago individual

---

## [2026-04-14 23:55] — v2.11/v2.12: dark mode + fix paths Firebase + migración datos legacy

**Resumen:** Implementación completa del toggle claro/oscuro (v2.11). Merge de v2 a main + deploy. Diagnóstico y corrección de bug crítico: `_initSession` usaba subpath `/2026/` en Firebase cuando la sesión tenía `temporada`, causando que datos (cuotas, pagos, gastos) se guardaran en path equivocado. Migración completa de datos legacy `/cursos/4B/` y subpath `/datos/sg/4B/2026/` al path correcto `/datos/sg/4B/`. Limpieza del nodo `/2026/` en Firebase.

**Archivos:** `index.html`, `CLAUDE.md`

**Decisiones:** `_initSession` ahora ignora `ses.temporada` — siempre usa `/datos/{colegio}/{curso}` sin subpath de año. El sistema de temporadas en Firebase fue abandonado en favor de path plano. Backup GitHub Actions ya funcionaba correctamente; el problema era que los datos iban al path equivocado.

**Pendientes:**
- [ ] El nodo legacy `/cursos/4B/` sigue existiendo en Firebase (no se borró — tiene datos históricos de v1)
- [ ] Investigar/arreglar bug form passwords (pendiente de sesión anterior)
- [ ] Brainstorm funcionalidades faltantes (pendiente de sesión anterior)

---

## [2026-04-13 23:15] — QA review + brainstorm funcionalidades faltantes

**Resumen:** Sesión de QA por revisión de código (Playwright no disponible). Se identificaron 3 bugs y un gap UX clave. Se inició brainstorm de funcionalidades faltantes, quedó pendiente continuar en próxima sesión.

**Archivos:** ninguno modificado (solo análisis)

**Decisiones:** Próxima sesión: fix bug form passwords + brainstorm completo de features faltantes.

**Pendientes:**
- [ ] Fix bug: wrappear password fields en `<form>` (gestores de contraseña no funcionan)
- [ ] Fix minor: agregar favicon.ico
- [ ] Fix minor: no hacer `.trim()` al password en doLoginTesorero
- [ ] UX gap: permitir que el tesorero genere/comparta el link de apoderado (hoy solo superadmin puede)
- [ ] Continuar brainstorm de funcionalidades faltantes con superpowers
- [ ] Validar flujos manualmente: login tesorero → mis cursos → app, superadmin, apoderado link

---

## [2026-04-13 22:47] — Revisión de estado y generación de devlog

**Resumen:** Sesión de diagnóstico. Se revisó el estado del proyecto (v2.9.1), se constató que el archivo "Funcionalidad faltante" es un residuo de diseño de la feature de temporadas (ya implementada). Se creó este DEVLOG por primera vez.

**Archivos:** `DEVLOG.md` (creado)

**Decisiones:** Ninguna técnica — sesión administrativa.

**Pendientes:**
- [x] Eliminar archivo "Funcionalidad faltante" (residuo de sesión anterior)
- [x] Revisar bug backup cron 401 desde Firebase — confirmado OK, corre diario sin fallas

---

## [2026-04-04] — fix v2.9.1: fecha gastos en formato DD/MM/YYYY

**Resumen:** Corrección de visualización de fecha en la pestaña Gastos. Las fechas se mostraban en formato ISO; ahora se muestran en DD/MM/YYYY.

**Archivos:** `index.html`

**Decisiones:** Formato local chileno DD/MM/YYYY consistente con el resto de la app.

**Pendientes:**
- [ ] Backup cron retorna 401 desde Firebase (pendiente investigar nueva URL /datos/)

---

## [2026-04-04] — feat v2.9: editar gastos

**Resumen:** Se agregó botón ✏️ en cada gasto para editar con sheet prellenado. Se agregó categoría "Regalo" a la lista de categorías de gastos.

**Archivos:** `index.html`

**Decisiones:** Mismo patrón UX que edición de otros elementos (sheet deslizable desde abajo).

---

## [2026-04-03] — feat v2.8: administración completa de temporadas

**Resumen:** Lógica de administración completa desde superadmin: cerrar temporada, resetear datos, eliminar temporada, crear nueva temporada con wizard de 3 pasos.

**Archivos:** `index.html`

**Decisiones:** Wizard de 3 pasos para nueva temporada (año → copiar alumnos → confirmar). Datos históricos se conservan, nunca se eliminan automáticamente.

---

## [2026-04-03] — fix v2.7: mejoras UX

**Resumen:** 4 mejoras de experiencia de usuario en distintas partes de la app.

**Archivos:** `index.html`

---

## [2026-04-02] — feat v2.6: cuentas de tesorero con Firebase Auth

**Resumen:** Sistema de cuentas de tesorero usando Firebase Auth (email/password). Los delegados tienen cuenta propia en vez de solo PIN compartido.

**Archivos:** `index.html`

---

## [2026-04-01] — feat v2.5: sistema de temporadas/años

**Resumen:** Implementación completa del sistema de temporadas. Login ahora tiene 4 campos: Colegio + Curso + Año + PIN. Firebase estructura /datos/{cid}/{curid}/{año}/. Superadmin con CRUD de temporadas y herramienta de migración de datos legacy.

**Archivos:** `index.html`

**Decisiones:** El año va en el login (no en header) para permitir acceso histórico explícito. Compatibilidad legacy: si el curso no tiene temporadas, cae al path /datos/{cid}/{curid}/.

---

## [Historial anterior — v1.x a v2.4]

La app comenzó como proyecto SG-only (GitHub Pages, v1.x) y migró a SaaS multi-colegio (Vercel, v2.x). Ver `git log --oneline` para detalle completo de commits.

---
