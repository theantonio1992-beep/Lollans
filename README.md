# Lollan's — Sistema Operativo de Fábrica

Web app interna para gestionar la fábrica de calzado de hogar de Lollan's
(Caravaca de la Cruz y Alhama de Murcia), sustituyendo el montaje actual de
Monday.com + Make + Query Pro + Dropbox + Trello + Excel.

**Uso principal: móvil.** La mayoría de las personas que capturan datos trabajan
de pie y con las manos ocupadas.

## Estado

Fase 0 — análisis. Todavía no hay código de aplicación.

## Documentación

| Documento | Contenido |
|---|---|
| [`CLAUDE.md`](CLAUDE.md) | Contexto rápido del proyecto (leer primero) |
| [`docs/BITACORA.md`](docs/BITACORA.md) | Registro de cambios y decisiones |
| [`docs/analisis/01-radiografia-operativa.md`](docs/analisis/01-radiografia-operativa.md) | Análisis completo: flujos, tableros, automatizaciones, datos y plan |
| [`docs/presentaciones/radiografia-operativa.html`](docs/presentaciones/radiografia-operativa.html) | La misma información como presentación navegable |

## Las dos rutas industriales

    Pedido → Nota de fabricación → Corte → Reparto → Aparado externo
                                                          │
                                          ┌───────────────┴───────────────┐
                            Caravaca · Inyección PVC        Alhama · Vulcanizado caucho
                                          └───────────────┬───────────────┘
                                                          │
                                        Vías → Embalaje → Almacén → Expedición

El enrutado depende del cliente: `Lollans → Inyección`, `Saez → Vulcanizado`.

## Fuentes

Toda la información procede de la documentación de Notion de Lollan's:
el Sistema Operativo Empresarial (secciones 00–11 y sus seis bases maestras),
el Manual Operativo de Monday.com, la guía del equipo de corte, la página
Organización con Monday, la Documentación general por secciones y el Flujo de
Producción de la fábrica.
