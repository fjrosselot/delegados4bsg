# DEVLOG — Tesoreros App

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
