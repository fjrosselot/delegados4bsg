# Diseño: Actividad Colectiva

**Fecha:** 2026-04-14  
**Versión app:** v2.9.1+  
**Estado:** Aprobado

---

## Descripción

Nuevo tipo de cuota "Actividad colectiva" para gestionar eventos donde el curso tiene una meta mínima de unidades (ej: Bingo Solidario — 15 talonarios × $10.000). Las familias se comprometen voluntariamente con X unidades. Si los compromisos no cubren el mínimo, la app propone cubrir el faltante con el saldo del curso.

---

## Estructura de Datos

Se agrega al array `quotas` existente en Firebase con `tipo: "actividad"`:

```js
{
  id: "act_...",
  tipo: "actividad",
  nombre: "Bingo Solidario",
  precioUnidad: 10000,
  metaMinima: 15,
  estado: "abierta",        // "abierta" | "cerrada"
  compromisos: {            // keyed por studentId, valor = nº unidades
    "sid_juan": 2,
    "sid_maria": 1
  },
  gastoId: null,            // ID del gasto generado al cerrar (si hubo gap)
  createdAt: "2026-04-14"
}
```

Los pagos individuales se registran en `payments` con `quotaId` apuntando a la actividad — reutilizando la infraestructura de pagos existente sin cambios.

---

## Flujo de Creación (Wizard)

Se agrega "Actividad colectiva" como nuevo tipo en el wizard existente de cuotas.

Pasos:
1. **Tipo** — seleccionar "Actividad colectiva"
2. **Datos básicos** — nombre, precio por unidad, meta mínima (nº unidades)
3. **Confirmar** — resumen antes de guardar

Al guardar: actividad queda `abierta` con `compromisos: {}` vacío.

---

## Panel de Gestión (Actividad Abierta)

Al hacer clic en una actividad desde la tab Cuotas se abre un panel (sheet) con:

### Encabezado
- Nombre + precio unitario + meta mínima
- Barra de progreso: `X unidades comprometidas / Y mínimo`
- Total recaudado estimado: `comprometidas × precioUnidad`

### Lista de compromisos
| Campo | Descripción |
|-------|-------------|
| Alumno | Nombre del estudiante |
| Comprometido | Número editable de unidades (0 = no participa) |
| Monto | Calculado: `unidades × precioUnidad` |
| Pagado | Estado del pago (igual que cuotas normales) |

### Acciones
- **Guardar cambios** — actualiza `compromisos` en Firebase
- **Cerrar actividad** — solo rol `delegado`

---

## Cierre de Actividad y Gap

Al cerrar, si `sum(compromisos) < metaMinima`:

> "Faltan X unidades para el mínimo. El curso cubriría $Y del saldo. ¿Registrar como gasto?"

- **Confirmar** → crea gasto: `"Actividad: [nombre] — aporte curso"` por el monto del gap, guarda `gastoId` en la actividad
- **Cancelar** → cierra sin gasto (el delegado puede crearlo manualmente después)

Si `sum(compromisos) >= metaMinima`: cierra directamente sin propuesta de gasto.

---

## Integración con Sistemas Existentes

| Sistema | Cambio |
|---------|--------|
| Wizard cuotas | +1 tipo "Actividad colectiva" con 2 campos extra |
| Tab Cuotas | Actividades aparecen con badge diferenciador; clic abre panel dedicado |
| Pagos | Sin cambios — pagos individuales usan `quotaId` normal |
| Gastos | Sin cambios — gasto del gap se crea igual que cualquier gasto manual |
| Firebase | Sin cambios de esquema — `tipo: "actividad"` es suficiente |

---

## Estados de la Actividad

```
abierta → (cerrar) → cerrada
```

- **Abierta:** compromisos editables en cualquier momento
- **Cerrada:** solo lectura; si hubo gap muestra link al gasto generado

---

## Fuera de Alcance

- Notificaciones a familias
- Actividades opcionales en cuotas regulares
- Mover asados/paseos existentes a este formato
