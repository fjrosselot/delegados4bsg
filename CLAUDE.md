# Tesoreros SG — Contexto del Proyecto

## Descripción
App de gestión de pagos para comités delegados de colegios. HTML/JS vanilla (sin frameworks), Firebase Realtime Database para persistencia, GitHub Pages para hosting. Soporta múltiples cursos via parámetro URL `?curso=4B`.

## URLs y Repositorios
- **App producción:** https://fjrosselot.github.io/tesoreros-sg?curso=4B
- **Repo principal:** https://github.com/fjrosselot/tesoreros-sg
- **Repo backups:** https://github.com/fjrosselot/tesoreros-sg-backups
- **Proxy Vercel:** https://claude-proxy-vert.vercel.app/api/proxy (Gemini via OpenRouter)
- **Backup cron Vercel:** https://claude-proxy-vert.vercel.app/api/backup-cron

## Estructura de Archivos
```
tesoreros-sg/
└── index.html          ← TODO el código (HTML + CSS + JS en un solo archivo)
```

El proyecto es un **single file** — todo vive en `index.html`. No hay build process, no hay dependencias npm, no hay carpetas `src/`.

## Firebase
- **Proyecto:** bsg-7772d
- **URL:** https://bsg-7772d-default-rtdb.firebaseio.com
- **Estructura:** `/cursos/{CURSO_ID}/{expenses,log,payments,quotas,students,saldoInicial}`
- **Reglas:** `.read: true, .write: true` por nodo de curso
- **Polling:** cada 6 segundos via REST API

## Cursos Configurados
| Curso | URL param | PIN | Alumnos |
|-------|-----------|-----|---------|
| 4°B | `?curso=4B` | 4B4B | 38 (Pancho Rosselot) |
| 3°D | `?curso=3D` | 3D3D | 38 (otra delegada) |

## Arquitectura JS
- **Estado global:** `state` objeto con `{students, quotas, payments, expenses, log, saldoInicial}`
- **Render:** `render()` → `getContent()` → `renderResumen/Cuotas/Pagos/Gastos/Alumnos/Pendientes/Reportes/Log()`
- **Firebase:** `window._fbSave(state)` / `window._fbStartPolling(callback)`
- **PIN admin:** modo público (lectura) + modo Delegados con PIN por curso
- **Versión visible:** `APP_VERSION` constante en el JS, mostrar en sidebar y header móvil

## Pestañas (TAB_META)
`resumen` → `cuotas` → `pagos` → `gastos` → `alumnos` → `pendientes` → `reportes` → `log`

## Features Implementados
- Multi-curso con parámetro URL y PIN por curso
- Cuotas: activas, borradores, wizard de creación (igual/múltiplo por personas/base+excepciones)
- Wizard múltiplo: tarifa única o segmentada por edad (Adulto/Adolescente/Niño)
- Pagos: filtro, búsqueda CSS, modo lote (selección múltiple con Set en memoria)
- Pendientes: grilla desktop (tabla filas=alumnos, columnas=cuotas) / cards móvil
- Alumnos: género inferido automáticamente, pausar/reactivar, filtro por género
- Reportes: gráfico Canvas con pills (saldo/ingresos/gastos) + tooltip interactivo
- Entrada rápida IA: botón ✨ en modo admin, interpreta lenguaje natural via Gemini
- Backup manual JSON + cron automático diario a GitHub via Vercel
- Saldo inicial del curso
- Imagen compartir WhatsApp (Canvas API, 2 columnas alfabéticas)
- Datepicker custom en español (calendario visual)

## Modo Lote (Pagos)
- `loteSelected` = Set en memoria con student IDs seleccionados
- `loteToggle(sid, row)` actualiza Set Y checkbox DOM directamente
- `saveLote(qid)` usa `loteSelected` (NO el DOM) para determinar seleccionados
- Los alumnos en renderPagos están ordenados alfabéticamente

## Convenciones de Código
- Siempre validar sintaxis JS antes de commit: `node --check index.html` NO funciona directo, extraer scripts primero
- Nunca usar template literals anidados (backtick dentro de ${} dentro de backtick) — causa bugs en móvil
- Para strings con comillas mixtas usar concatenación: `'texto' + variable + 'texto'`
- Incrementar `APP_VERSION` en cada commit para debugging de caché
- El archivo puede tener ~2200+ líneas — usar grep/search para encontrar funciones

## Variables de Entorno (Vercel - proyecto claude-proxy)
- `GEMINI_API_KEY` = key de Google AI Studio
- `OPENROUTER_API_KEY` = key de OpenRouter
- `GITHUB_TOKEN` = token de GitHub de fjrosselot
- `FIREBASE_URL` = https://bsg-7772d-default-rtdb.firebaseio.com/cursos
- `FIREBASE_SECRET` = secret de Firebase para backup cron

## Git Workflow
```bash
git add -A
git commit -m "descripción"
git push origin main
# GitHub Pages actualiza en ~1 minuto
```

## Bugs Conocidos / Pendientes
- Proxy backup cron retorna 401 desde Firebase (investigar autenticación)
- Buscador en móvil abre teclado pero no re-renderiza contador (usa CSS show/hide)
- Entrada rápida IA limitada por cuota de Gemini free tier

## Funciones Clave
| Función | Descripción |
|---------|-------------|
| `render()` | Re-renderiza contenido actual |
| `saveData()` | Guarda state en Firebase |
| `mergeState(val)` | Deserializa datos de Firebase |
| `ensureGenders()` | Infiere género de alumnos sin gender |
| `getStudentAmount(sid, q)` | Monto específico del alumno en una cuota |
| `isPaid(sid, qid)` | Verifica si alumno pagó cuota completa |
| `qStats(qid)` | Estadísticas de cuota (paid/pending/collected/expected) |
| `requireAdmin(fn)` | Ejecuta fn solo si es admin, pide PIN si no |
| `openSheet(title, body, onSave)` | Abre modal sheet |
| `showToast(msg, type)` | Muestra notificación temporal |
| `drawReportesChart()` | Redibuja gráfico en pestaña Reportes |
| `loteToggle(sid, row)` | Toggle selección en modo lote |
| `ensureGenders()` | Se llama en FB polling callback, no en startApp |
