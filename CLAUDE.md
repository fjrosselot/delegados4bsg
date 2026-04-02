# Tesoreros App — Contexto del Proyecto (v2.0)

## Descripción
Plataforma SaaS multi-colegio para comités de delegados/tesoreros. HTML/JS vanilla (sin frameworks), Firebase Realtime Database para persistencia, Vercel para hosting. Multi-tenant: login con colegio + curso + PIN.

## Ramas
- **`main`** — versión v1.x original (SG-only, GitHub Pages), NO TOCAR hasta validar v2
- **`v2`** — versión v2.0 SaaS multi-colegio (rama activa de desarrollo)

## URLs y Repositorios
- **App v2 producción:** https://tesoreros-app.vercel.app (rama v2)
- **App v1 producción:** https://fjrosselot.github.io/tesoreros-sg?curso=4B (main, no tocar)
- **Repo principal:** https://github.com/fjrosselot/tesoreros-sg
- **Repo backups:** https://github.com/fjrosselot/tesoreros-sg-backups
- **Proxy Gemini:** https://claude-proxy-vert.vercel.app/api/proxy
- **Backup cron:** https://claude-proxy-vert.vercel.app/api/backup-cron

## Estructura de Archivos
```
tesoreros-sg/
├── index.html          ← TODO el código (HTML + CSS + JS en un solo archivo)
└── vercel.json         ← config deploy estático Vercel
```
Single file — no hay build process, no hay carpetas `src/`.

## Firebase
- **Proyecto:** bsg-7772d
- **URL base:** https://bsg-7772d-default-rtdb.firebaseio.com

### Estructura v2
```
/plataforma/colegios/{colegioId}/
  - nombre                    ← string
  - activo                    ← bool
  - cursos/{cursoId}/
    - nombre, pinAdmin, pinLectura, activo
    - tesorero: { nombre, email }

/datos/{colegioId}/{cursoId}/
  - students, quotas, payments, expenses, log, saldoInicial

/sessions/{token}/
  - colegioId, cursoId, ts  (auditoría de logins)
```

### Reglas Firebase
- `/plataforma` → `.read: true, .write: false` (público solo lectura)
- `/datos/$colegio/$curso` → `.read: true, .write: true`
- `/sessions` → `.read: false, .write: true`
- `/cursos` → mantener para compatibilidad con v1 (main)

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
- **3 campos:** Colegio (texto libre) + Curso + PIN
- Sin dropdowns — privacidad total
- Error genérico: "Acceso no válido"
- **5 intentos fallidos** → bloqueo 15 min en sessionStorage
- **Roles:** `delegado` (pinAdmin, edición completa) / `apoderado` (pinLectura, solo lectura)
- **Sesión:** sessionStorage `sesionv2` = `{ colegioId, cursoId, rol, token, meta, colegioNombre, cursoNombre }`
- **Token:** generado al login, guardado en `/sessions/{token}` para auditoría
- **Superadmin PIN:** `SG2025Admin#` (hardcodeado, no mostrar en UI)

## Recuperación de PIN (EmailJS)
- Credenciales (public-safe, van en HTML):
  - `SERVICE_ID: service_r7c237j`
  - `TEMPLATE_ID: template_u38q8sr`
  - `PUBLIC_KEY: gioDkEy7GGvLn5ghK`
- Variables template: `{{tesorero_nombre}}`, `{{colegio}}`, `{{curso}}`, `{{pin}}`
- Valida que el email ingresado coincida con `tesorero.email` del curso en Firebase

## Panel Superadmin
- Acceso: PIN `SG2025Admin#` en el campo PIN del login (cualquier colegio/curso)
- CRUD de colegios: nombre, ID, activo/inactivo
- CRUD de cursos: nombre, pinAdmin, pinLectura, tesorero, activo/inactivo
- Importador de alumnos — 2 modos:
  - **Texto**: textarea con "APELLIDO APELLIDO, NOMBRE" (uno por línea)
  - **Excel/CSV**: SheetJS — detecta columna de nombres automáticamente
  - Preview antes de confirmar; no duplica alumnos existentes

## Variables de Entorno (Vercel — proyecto tesoreros-app)
- `GITHUB_TOKEN_BACKUPS` = (token PAT scope solo repo tesoreros-sg-backups — ver Vercel env)
- `FIREBASE_SECRET` = (legacy admin secret Firebase — ver Vercel env)
- `FIREBASE_URL` = https://bsg-7772d-default-rtdb.firebaseio.com

## Variables de Entorno (Vercel — proyecto claude-proxy, NO TOCAR)
- `GEMINI_API_KEY`, `OPENROUTER_API_KEY`, `GITHUB_TOKEN`, `FIREBASE_URL`, `FIREBASE_SECRET`

## Arquitectura JS (v2)
- **Sesión:** `_SESSION = { colegioId, cursoId, rol, token, meta, colegioNombre, cursoNombre }`
- **Firebase URL:** `_FB_URL` — inicializada por `_initSession(ses)` tras login exitoso
- **Firebase base:** `_FB_BASE = "https://bsg-7772d-default-rtdb.firebaseio.com"`
- **Roles:** `isAdmin = _SESSION.rol === 'delegado'`
- **Helpers:** `getColegioNombre()`, `getCursoNombre()`, `getCursoId()`
- **Estado global:** `state` objeto con `{students, quotas, payments, expenses, log, saldoInicial}`
- **Render:** `render()` → `getContent()` → `renderResumen/Cuotas/Pagos/Gastos/Alumnos/Pendientes/Reportes/Log()`
- **Firebase:** `window._fbSave(state)` / `window._fbStartPolling(callback)`
- **Versión visible:** `APP_VERSION = "v2.0"`

## Pestañas (TAB_META)
`resumen` → `cuotas` → `pagos` → `gastos` → `alumnos` → `pendientes` → `reportes` → `log`

## Features Implementados
- Login multi-tenant: colegio + curso + PIN
- Roles: delegado (admin) / apoderado (lectura)
- Bloqueo 5 intentos → 15 min
- Recuperar PIN via EmailJS
- Panel superadmin con CRUD colegios/cursos
- Importador alumnos: texto y Excel/CSV (SheetJS)
- Token de sesión en Firebase (/sessions)
- Cuotas: activas, borradores, wizard (igual/múltiplo por personas/base+excepciones)
- Wizard múltiplo: tarifa única o segmentada por edad (Adulto/Adolescente/Niño)
- Pagos: filtro, búsqueda CSS, modo lote (Set en memoria)
- Pendientes: grilla desktop / cards móvil
- Alumnos: género inferido, pausar/reactivar, filtro por género
- Reportes: gráfico Canvas + tooltip interactivo
- Entrada rápida IA (botón ✨ solo delegados) via Gemini
- Backup manual JSON + cron diario a GitHub
- Saldo inicial del curso
- Imagen compartir WhatsApp (Canvas API)
- Datepicker custom en español

## Modo Lote (Pagos)
- `loteSelected` = Set en memoria con student IDs seleccionados
- `loteToggle(sid, row)` actualiza Set Y checkbox DOM directamente
- `saveLote(qid)` usa `loteSelected` (NO el DOM) para determinar seleccionados

## Convenciones de Código
- Balance de llaves/paréntesis: validar con script Python (ver historial)
- Nunca usar template literals anidados — usar concatenación con `+`
- Para strings con comillas mixtas usar concatenación: `'texto' + variable + 'texto'`
- Incrementar `APP_VERSION` en cada commit
- Archivo ~3000+ líneas — usar grep para encontrar funciones

## Datepicker Custom
- Funciones: `dateField(label,id,value)` → `calToggle(id)` → `calRender(id)` → `calNav(id,delta)` → `calSelect(id,dateStr)`
- Estado navegación: `window._calState[id] = {year, month}` (month = 0-indexed)
- **Gotcha:** `calRender` reemplaza `pop.innerHTML` completo — el handler de "click fuera" usa `document.body.contains(e.target)`

## Git Workflow (v2)
```bash
# Trabajar siempre en rama v2
git checkout v2
git add index.html
git commit -m "feat/fix vX.Y: descripción"
git push origin v2
# Vercel auto-deploya desde rama v2 → tesoreros-app.vercel.app
# main NO se toca hasta validación completa de v2
```

## Superadmin — Funciones JS
| Función | Descripción |
|---------|-------------|
| `openSuperAdmin()` | Carga y renderiza el panel SA desde Firebase |
| `renderSuperAdmin(panel, data)` | Dibuja la UI con colegios y cursos |
| `saOpenAddColegio()` / `saSaveColegio()` | Crear colegio |
| `saEditColegio(cid, json)` / `saUpdateColegio(cid)` | Editar colegio |
| `saAddCurso(cid, nom)` / `saSaveCurso(cid)` | Crear curso |
| `saEditCurso(cid, curid, json)` / `saUpdateCurso(cid, curid)` | Editar curso |
| `saImportAlumnos(cid, curid)` | Abrir importador para un curso |
| `impSetModo(modo)` | Cambiar entre modo texto/excel |
| `impPreviewTexto(txt)` | Parsear textarea y mostrar preview |
| `impHandleFile(file)` | Parsear Excel/CSV con SheetJS |
| `impDetectNombresXLSX(rows)` | Detectar columna de nombres |
| `impConfirm(cid, curid)` | Confirmar importación a Firebase |

## Funciones Clave
| Función | Descripción |
|---------|-------------|
| `doLogin()` | Login async multi-tenant — 3 campos + bloqueo |
| `loginFail(btn)` | Registrar intento fallido + bloqueo |
| `_initSession(ses)` | Establecer _SESSION y _FB_URL |
| `doLogout()` | Cerrar sesión — limpia sessionStorage, state, interval |
| `requireAdmin(fn)` | Ejecuta fn solo si rol===delegado |
| `openRecuperarPIN()` | Mostrar modal recuperar PIN |
| `enviarPIN()` | Enviar PIN via EmailJS |
| `startApp(admin)` | Arrancar app + actualizar header dinámico |
| `render()` | Re-renderiza contenido actual |
| `saveData()` | Guarda state en Firebase (con token de auditoría) |
| `mergeState(val)` | Deserializa datos de Firebase |

## Bugs Conocidos / Pendientes
- Backup cron retorna 401 desde Firebase (pendiente investigar nueva URL /datos/)
- Buscador móvil usa CSS show/hide (no re-renderiza contador)
- IA limitada por cuota Gemini free tier
- Las reglas de Firebase no validan el token (requeriría Firebase Auth real)
