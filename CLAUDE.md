# CLAUDE.md — Contexto rápido de Lollan's

> Leer esto **antes** que cualquier otro doc. Solo abrir los documentos largos
> si la tarea lo exige (ver "Dónde buscar").
> Historial de cambios → [`docs/BITACORA.md`](docs/BITACORA.md).

## Proyecto en 5 líneas

- Web app interna para la fábrica de calzado de hogar Lollan's (Caravaca de la Cruz + Alhama de Murcia).
- Sustituye Monday.com + Make + Query Pro (parcial) + Dropbox + Trello + Excel.
- **Móvil primero**: operarios de pie, manos ocupadas → botones grandes, 1 acción por pantalla, offline.
- Entidad central: **`Nota` de fabricación** (nace en Query Pro, recorre todas las etapas).
- Estado: **Fase 0 — análisis**. Aún no hay código.

## Estructura del repo

| Ruta | Qué es |
|---|---|
| `CLAUDE.md` | Este resumen (contexto mínimo) |
| `docs/BITACORA.md` | Registro de cambios y decisiones |
| `docs/analisis/01-radiografia-operativa.md` | Análisis completo (478 líneas) |
| `docs/presentaciones/radiografia-operativa.html` | Mismo análisis en presentación |
| `README.md` | Presentación pública del repo |

## Dónde buscar (en `01-radiografia-operativa.md`)

| Necesito… | Sección |
|---|---|
| Flujo de 14 etapas | §2 |
| Tableros Monday y estados | §3 |
| Reglas de negocio (14 automatizaciones Make) | §4 |
| 15 procesos maestros y responsables | §5 |
| Personas, roles, formas de cobro | §6 |
| KPIs y fórmulas | §7 |
| Capacidad y activos | §8 |
| Problemas → funcionalidad | §9 |
| 13 módulos y requisitos de planta | §10 |
| Pantallas por rol | §11 |
| Plan por fases | §12 |
| Decisiones pendientes | §13 |

## Flujo industrial

```
Pedido → Nota → Corte (tejido + plantas, separados) → Reparto → Aparado externo
  → Control previo → Inyección PVC (Caravaca) | Vulcanizado (Alhama)
  → Vías → Recuento → Embalaje → Almacén → Expedición
```

- Enrutado: `Tipo Cliente = Lollans → Inyección` · `Saez → Vulcanizado` → en la app será campo `Ruta` en el modelo, versionado.
- **Bucle de coste**: falta detectada en vías/embalaje vuelve a Corte. Detectar antes = máximo valor.
- Puntos ciegos: reparto, aparado externo, traslado a Alhama.

## Reglas de dominio (no reinventar)

- Estados: `En proceso` · `Pausa` · `Hecho` · `Falta`
- Prioridad: `Urgente` · `Alta` · `Media` · `Baja`
- Corte = 2 tareas independientes: `Corte Tejido` y `Corte Plantas`.
- Marcar `Falta` ⇒ **comentario obligatorio** (qué y cuánto, con foto).
- La nota **se mueve** entre fases por cambio de estado; no se copia.
- Notas > 1.200 pares se dividen.
- Cobro: nómina (corte, reparto) · horas (goma, inyección) · destajo (aparado por modelo, vías por cliente). Cupón = persona + orden + operación + cantidad aceptada + fecha + tarifa.

## Principios técnicos

- Movimientos de inventario/producción: **append-only**, nunca sobrescribir.
- Identificadores estables (no depender de nombres de columna).
- Cada dato crítico: fuente, responsable, fecha, estado de validación.
- Permisos mínimos por rol + auditoría.
- Capacidad = `min(demanda, materiales, corte, aparado, suela, vías, embalaje, liquidez)`. Nunca usar los 240 pares/h teóricos como compromiso.
- Query Pro sigue como ERP administrativo; Notion como capa de conocimiento.

## Personas clave

- **Oficina**: Cati (pedidos, notas, compras) · Santos (contabilidad, Query) · Marina (vías, embalaje, etiquetas) · Jonathan (comercial, Francia) · Antonio hijo (mejora continua) · Jaime (dirección).
- **Fábrica**: Jesús (reparto/aparadoras) · Juan Carlos (cuñas, plantas) · Rubén y Porretas (corte) · Jorge (moldes, mantenimiento) · Coronel (inyección) · Manolo y Jordi (embalaje, almacén).
- Sin dueño: Transferencia Caravaca–Alhama y Vulcanización.

## Plan

- **F0** cerrar inventario (blueprints Make, API Monday) ← *actual*
- **F1** piloto: Nota extremo a extremo (corte, faltas, vista operario + oficina)
- **F2** aparado externo + transferencias Alhama
- **F3** KPIs reales, partes diarios, destajo
- **F4** costes, márgenes, capacidad
- **F5** integración Query Pro, inventario por lote, CRM

## Bloqueantes de F1 (§13)

1. ¿Sustituye o convive con Monday?
2. ¿Entrada de la nota: PDF/Dropbox o Query directo?
3. ¿Quién firma salida/recepción Caravaca–Alhama?
4. ¿Las aparadoras usan la app o solo Jesús?

## Flujo de trabajo (obligatorio)

1. Leer este archivo + últimas entradas de `docs/BITACORA.md`.
2. Abrir solo la sección del análisis que haga falta.
3. Al terminar un cambio relevante → **añadir entrada arriba en `docs/BITACORA.md`**.
4. Si cambia algo de este resumen (fase, decisión, estructura, regla) → **actualizar `CLAUDE.md`** en el mismo commit.
5. Rama de trabajo, commits en español, PR en borrador.

**Qué cuenta como relevante**: decisiones tomadas, cambio de fase, nuevos módulos/archivos, cambios de modelo de datos o reglas, dependencias añadidas. No registrar typos ni formato.
