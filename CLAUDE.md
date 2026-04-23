# Tesoreros App — Contexto del Proyecto (v2.31)

## Descripción
Plataforma SaaS multi-colegio para comités de delegados/tesoreros. HTML/JS vanilla (sin frameworks), Firebase Realtime Database para persistencia, Vercel para hosting. Multi-tenant: login con colegio + curso + PIN.

## Ramas
- **`main`** — única rama. Todo el desarrollo va directo aquí, igual que los otros proyectos Backbone IA.

## URLs y Repositorios
- **App producción:** https://tesoreros-app.vercel.app (desde main)
- **Repo principal:** https://github.com/fjrosselot/tesoreros-sg
- **Repo backups:** https://github.com/fjrosselot/tesoreros-sg-backups
- **Proxy Gemini:** https://claude-proxy-vert.vercel.app/api/proxy

## Estructura de Archivos
```
tesoreros-sg/
├── index.html                    ← TODO el código (HTML + CSS + JS en un solo archivo)
├── vercel.json                   ← config deploy estático Vercel
└── .github/workflows/backup.yml  ← backup diario a GitHub (6 AM Chile)
```
Single file — no hay build process, no hay carpetas `src/`.

## Firebase
- **Proyecto:** bsg-7772d
- **URL base:** https://bsg-7772d-default-rtdb.firebaseio.com

### Estructura v2 (DEFINITIVA — no usar subpaths de año)
```
/plataforma/colegios/{colegioId}/
  - nombre, activo
  - cursos/{cursoId}/
    - nombre, pinAdmin, pinLectura, activo
    - tesorero: { nombre, email }

/datos/{colegioId}/{cursoId}/         ← PATH CANÓNICO — nunca /datos/cid/curid/año/
  - students, quotas, payments, expenses, log, saldoInicial

/sessions/{token}/
  - colegioId, cursoId, ts

/usuarios/{uid}/
  - cursos: lista de cursos del tesorero (Firebase Auth)

/cursos/4B/                           ← PATH LEGACY v1 — NO TOCAR, datos históricos
```

### IMPORTANTE — _initSession
`_initSession(ses)` siempre usa `/datos/{colegioId}/{cursoId}` — ignora `ses.temporada`.
El sistema de temporadas/subpaths fue abandonado. Nunca restaurar la rama `if(ses.temporada)`.

### Reglas Firebase
- `/plataforma` → `.read: true, .write: false`
- `/datos/$colegio/$curso` → `.read: true, .write: true`
- `/sessions` → `.read: false, .write: true`
- `/cursos` → mantener (datos legacy v1)

### Colegios Configurados
| ID | Nombre | Cursos |
|----|--------|--------|
| `sg` | Saint George's College | 4B, 3D |

### Cursos SG
| Curso | pinAdmin | pinLectura | Tesorero |
|-------|----------|------------|----------|
| 4B | 4B4B | 4BVer | Francisco Rosselot |
| 3D | 3D3D | 3DVer | [pendiente] |

## Login v2
- **Firebase Auth** para tesoreros (email + password) → pantalla "Mis cursos" → selección de curso
- **PIN apoderado** para acceso lectura (URL con UUID o PIN directo)
- Error genérico: "Acceso no válido"
- **5 intentos fallidos** → bloqueo 15 min en sessionStorage
- **Roles:** `delegado` (edición completa) / `apoderado` (solo lectura)
- **Sesión:** sessionStorage `sesionv2` = `{ _type, uid, idToken, nombre, colegioId, cursoId, rol, token, meta, colegioNombre, cursoNombre }`
- **Superadmin PIN:** `SG2025Admin#` (hardcodeado, no mostrar en UI)

## Recuperación de PIN (EmailJS)
- `SERVICE_ID: service_r7c237j` / `TEMPLATE_ID: template_u38q8sr` / `PUBLIC_KEY: gioDkEy7GGvLn5ghK`
- Variables template: `{{tesorero_nombre}}`, `{{colegio}}`, `{{curso}}`, `{{pin}}`

## Panel Superadmin
- Acceso: PIN `SG2025Admin#`
- CRUD colegios/cursos, importador alumnos (texto y Excel/CSV con SheetJS)

## Backup
- **GitHub Actions** `.github/workflows/backup.yml` — corre 6 AM Chile diariamente
- Lee `/datos/sg/4B.json` y `/datos/sg/3D.json`, commitea a `fjrosselot/tesoreros-sg-backups`
- Para forzar backup manual: `gh workflow run backup.yml --repo fjrosselot/tesoreros-sg`

## Variables de Entorno (Vercel — proyecto tesoreros-app)
- `FIREBASE_SECRET` = admin secret Firebase (para escrituras autenticadas)
- `FIREBASE_URL` = https://bsg-7772d-default-rtdb.firebaseio.com
- `GITHUB_TOKEN_BACKUPS` = PAT para repo de backups

## Arquitectura JS (v2)
- **Sesión:** `_SESSION = { colegioId, cursoId, rol, token, meta, colegioNombre, cursoNombre }`
- **Firebase URL:** `_FB_URL = _FB_BASE + "/datos/" + colegioId + "/" + cursoId` (siempre)
- **Firebase base:** `_FB_BASE = "https://bsg-7772d-default-rtdb.firebaseio.com"`
- **Roles:** `isAdmin = _SESSION.rol === 'delegado'`
- **Estado global:** `state` con `{students, quotas, payments, expenses, log, saldoInicial}`
- **Render:** `render()` → `getContent()` → `renderResumen/Cuotas/Pagos/Gastos/Alumnos/Pendientes/Reportes/Log()`
- **Firebase:** `window._fbSave(state)` / `window._fbStartPolling(callback)`
- **Versión visible:** `APP_VERSION = "v2.31"`

## Pestañas (TAB_META)
`resumen` → `cuotas` → `pagos` → `gastos` → `alumnos` → `pendientes` → `reportes` → `log`

## Features Implementados
- Login Firebase Auth (tesoreros) + PIN apoderado
- Roles: delegado / apoderado
- Bloqueo 5 intentos → 15 min
- Panel superadmin CRUD colegios/cursos
- Importador alumnos: texto y Excel/CSV (SheetJS)
- Cuotas: activas, borradores, wizard (igual/múltiplo/base+excepciones)
- Actividades colectivas: meta mínima, compromisos opt-in, gap semi-automático
- Pagos: filtro, búsqueda CSS, modo lote
- Pendientes: grilla desktop / cards móvil
- Alumnos: género inferido, pausar/reactivar, filtro género
- Reportes: gráfico Canvas + tooltip interactivo + toggle A la fecha/Proyectado
- Entrada rápida IA (✨) via Claude (solo delegados) — matchea por apellido/apodo, muestra ya-pagados, parsing robusto
- Backup manual JSON + cron diario a GitHub Actions
- Saldo inicial del curso
- Imagen compartir WhatsApp (Canvas API)
- Datepicker custom en español
- Toggle light/dark mode (slate blue dark / clean light)
- Toggle "A la fecha / Proyectado" cross-cutting — Resumen, Gastos, Reportes sincronizados via localStorage
- Ingresos proyectados: entidad `{ id, descripcion, monto, fechaEstimada }` en pestaña Gastos, incluidos en modo Proyectado
- Gastos futuros ocultos en modo "A la fecha" en todas las vistas
- Apoderados: botones admin ocultos via CSS `.apoderado-mode [onclick*="requireAdmin"]`
- Sort de gastos por fecha/descripción/monto con indicador de dirección
- Footer de cuota muestra recaudado + por recaudar (PxQ) en azul cuando hay pendientes
- Actividades colectivas incluidas en cálculo de `porRecaudar`
- `quotaAmountLabel(q)`: muestra precio/unidad o "Actividad colectiva" en vez de $0
- Versión visible en sidebar desktop y header móvil

## Modo Lote (Pagos)
- `loteSelected` = Set en memoria con student IDs
- `loteToggle(sid, row)` actualiza Set Y checkbox DOM directamente
- `saveLote(qid)` usa `loteSelected` (NO el DOM)

## Convenciones de Código
- Nunca usar template literals anidados — usar concatenación con `+`
- Para strings con comillas mixtas usar concatenación
- Incrementar `APP_VERSION` en cada commit
- Archivo ~3500+ líneas — usar grep para encontrar funciones

## Datepicker Custom
- `dateField(label,id,value)` → `calToggle(id)` → `calRender(id)` → `calNav(id,delta)` → `calSelect(id,dateStr)`
- Estado: `window._calState[id] = {year, month}` (month 0-indexed)

## Git Workflow
```bash
# Todo directo en main
git add index.html && git commit -m "feat/fix vX.Y: descripción"
git push origin main
vercel --prod --yes
```

## Bugs Conocidos / Pendientes
- Buscador móvil usa CSS show/hide (no re-renderiza contador)
- IA limitada por cuota Gemini free tier
- Las reglas de Firebase no validan el token (requeriría Firebase Auth real)

## Features Próximas (backlog)
- Link apoderado generado desde la app por el tesorero (sin pasar por superadmin)
- Distribución más elegante del link apoderado (QR, WhatsApp, PIN directo)
- Exportar estado de pagos (Excel/imagen compartible)
- Recordatorios de deuda (texto pre-armado para WhatsApp)
- Comprobante de pago individual por alumno
