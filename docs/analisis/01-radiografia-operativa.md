# Radiografía operativa de Lollan's

> Análisis previo al diseño de la web app. 10 de septiembre de 2026.
> Reconstruido a partir de la documentación de Notion.

## Resumen

| Cifra | Valor |
|---|---|
| Tableros de Monday encadenados | 12 |
| Automatizaciones de Make documentadas | 14 |
| Procesos maestros identificados | 15 |
| Procesos con documentación completa | **0 de 15** |
| KPIs definidos con dato real | **0 de 14** |
| Activos con dato confirmado | **0 de 8** |
| Cuentas de cliente clasificadas | **0 de 8** |

La definición existe y es buena. Lo que falta es la **captura del dato en el
punto de trabajo**. Ese es el hueco que llena la app.

---

## 1. El negocio, operativamente

Fabricante español de calzado de hogar B2B y marca privada, con dos rutas
industriales que comparten etapas iniciales y se separan tras el aparado.

**Caravaca (planta principal)** — almacén, goma, cuñas, corte, reparto,
recepción de aparado, inyección de PVC, vías, embalaje, almacén y expedición.
4 máquinas de inyección (2–3 de día, 2 de noche), 12 estaciones rotativas por
operario a 14–15 s por estación, 4 prensas de corte, +100 troqueles.

**Alhama (planta secundaria)** — vulcanización de caucho. ~20 puestos
individuales, normalmente 1–2 vulcanizadores. Ciclo aproximadamente el doble de
lento. Recibe desde Caravaca cortes aparados, planchas de goma y materiales.

**Regla de enrutado ya automatizada:** `Tipo Cliente = Lollans → Inyección`,
`= Saez → Vulcanizado`. En el sistema propio debe ser un campo `Ruta` en la
ficha del modelo, versionado y visible, no un `if` dentro de una automatización.

### El objeto central: la nota de fabricación

Nace en Query Pro desde un pedido, se imprime en PDF, se sube a Dropbox, Make la
lee y la convierte en ítem de Monday, y viaja de tablero en tablero hasta que
sale el camión. En paralelo viaja el papel físico. **La app debe tener una única
entidad `Nota`, y es la columna vertebral del producto.**

---

## 2. Flujo real de fábrica (14 etapas)

| # | Etapa | Quién / dónde | Nota |
|---|---|---|---|
| 01 | Pedido y condiciones | Jonathan | Versión de modelo y fecha comprometida |
| 02 | Alta en Query Pro y nota de fabricación | Cati | División si supera 1.200 pares |
| 03 | Pedido a proveedor | Cati | Generado desde las notas de corte |
| 04 | Recepción, identificación y ubicación | Caravaca | Albarán, lote, reserva |
| 05 | Preparación de goma y compuestos | Caravaca | Grano, triturado, reciclado |
| 06 | Preparación de cuñas | Juan Carlos | Serrín, corcho, cola al agua; ~10 tipos |
| 07 | Corte de tejido y plantas | Rubén, Porretas, Juan Carlos | **Dos cortes distintos, 1–2 días de desfase** |
| 08 | Reparto a aparadoras | Jesús · externo | ⚠️ Punto ciego |
| 09 | Aparado a domicilio | 10–15 aparadoras · externo | ⚠️ Punto ciego. Destajo por modelo |
| 10 | Control previo de pares | Por validar | Antes de entrar en ruta de suela |
| 11 | Inyección de PVC | Caravaca | Corte al molde, cuña, inyección, carro por números |
| 11b | Traslado a Alhama y vulcanización | Alhama | ⚠️ Punto ciego. Sin albarán interno |
| 12 | Vías: limpieza y acabado | Marina · 8–10 personas, día y noche | Hilos, rebabas, serret, percha, etiquetas, bolsas |
| 13 | Recuento y detección de faltas | — | 🔴 Bucle de coste: vuelve al paso 07 |
| 14 | Embalaje, paletizado, almacén y carga | Manolo, Jordi | Revisores externos de cliente en almacén |

### El bucle que se paga dos veces

Una falta detectada en vías o embalaje devuelve el par al **paso 07**: hay que
volver a cortar, repartir, aparar e inyectar. Es el único retroceso del flujo.
La documentación lo describe así: *"Los pares de faltas quitan mucha eficiencia
en la producción."*

Cualquier funcionalidad que adelante esa detección — conteo firmado en aparado,
control previo obligatorio antes de suela — vale más que cualquier dashboard.

---

## 3. Los 12 tableros de Monday

| Tablero | Área | Para qué | Quién |
|---|---|---|---|
| CRM | Comercial | Seguimiento de oportunidades | Jonathan, Antonio |
| Pedidos | Oficina | Embudo de pedidos confirmados | Cati |
| Alta de Notas | Oficina | Datos técnicos + PDF adjunto | Antonio |
| Notas de Producción | Producción | Vista global de todo lo vivo | Cati, Marina |
| Corte | Producción | Lotes en corte, con cronómetro | Operarios de corte |
| Aparado | Producción | Sub-items por paso | Jesús |
| Inyección / Vulcanizado | Producción | Fase de suela según cliente | Coronel |
| Vías | Producción | Limpieza, etiquetado, embolsado | Encargada de vías, Marina |
| Almacén | Logística | Listo para cargar | Manolo, Jordi, Marina |
| Faltas | Calidad | Centraliza cualquier incidencia | Cati, Marina |
| Pedidos Enviados | Históricos | Archivo para facturación | Marina, Cati |
| Partes diarios ×6 | Control | Rendimiento por sección y persona | Marina |

**Patrón clave:** la nota no se copia, **se mueve**, y cada movimiento lo
dispara un cambio de estado. Eso es lo que hay que replicar, no la UI de Monday.

### Estados que ya usa la gente (no hay que inventarlos)

- Corte tiene **dos columnas independientes**: `Corte Tejido` y `Corte Plantas`
  — dos trabajos, dos personas, hasta dos días de desfase.
- Estados: `En proceso` · `Pausa` · `Hecho` · `Falta`
- Prioridad: `Urgente` · `Alta` · `Media` · `Baja`
- Si se marca falta, **el comentario es obligatorio** — es lo que permite que la
  automatización actúe.

---

## 4. Las 14 automatizaciones de Make

Estas son el requisito funcional real del sistema propio. No son
"integraciones": son las reglas de negocio de la fábrica.

| # | Disparador | Acción |
|---|---|---|
| 01 | Pedido creado | Make o un GPT rellena campos del pedido desde el PDF de Query |
| 02 | Pedido creado | Se generan las tareas comunes de ese cliente |
| 03 | PDFs nuevos en Dropbox `Notas de fabricación` | Escenario **Alta de notas automáticas**: un ítem por nota, con datos extraídos del PDF |
| 04 | Estado → `En proceso` en Corte | Arranca el cronómetro |
| 05 | Corte → `Hecho` | Notificación al usuario Oficina |
| 06 | Nota de Alhama en Corte con cambrillones / pisos / tapas | Crea sub-items para cortarlo todo a la vez |
| 07 | `Tejido = Hecho` | `Fase → Aparado` y actualiza Notas de Producción |
| 08 | Nota entra en Aparado | Crea sub-items con los pasos de aparado del modelo |
| 09 | Sub-item `Coser plantas` = Hecho | La nota completa pasa a Hecho |
| 10 | Nota lista para suela | Enrutado por `Tipo Cliente` |
| 11 | Nota entra en Vías | To-do para Marina: *"Sacar etiquetas embalaje (cliente) nota #xxxx"* |
| 12 | `Estado = Faltas` + comentario | Clona el ítem al tablero Faltas y notifica a Oficina |
| 13 | Almacén → `Hecho` | Notas de Producción → `Completado` y notifica a Oficina |
| 14 | Cada día a las 21:30 | Refresca los seis partes diarios |

### Fragilidad conocida

El manual advierte: *"nunca cambies ni borres columnas, eso rompería las
automatizaciones"* y *"mantén los nombres de columnas exactos, coinciden con
Make"*. Todo depende de que nadie renombre una columna. En el sistema propio,
estas reglas van en el modelo de datos con **identificadores estables**.

### Pendiente de verificar

Este mapa procede de la documentación de Notion, que es detallada. **No se ha
podido acceder a Monday ni a Make directamente** (se abren en el navegador del
usuario, no en el entorno de esta sesión, y no hay credenciales). Para cerrar el
inventario al 100 % hace falta una de estas tres cosas:

1. Export del blueprint de cada escenario de Make (JSON)
2. Token de API de Monday de solo lectura
3. Capturas de la lista de escenarios y de las columnas de cada tablero

---

## 5. Los 15 procesos maestros

Ya inventariados en Notion con entrada, salida, responsable y KPI.

| # | Proceso | Planta | Responsable | KPI principal | Doc. |
|---|---|---|---|---|---|
| 1 | Pedido y nota de fabricación | Caravaca | Cati / Comercial | Pedidos completos y liberados | Parcial |
| 2 | Compra a proveedor | Caravaca | Cati | Entrega a tiempo | Parcial |
| 3 | Recepción y almacenamiento | Caravaca | Almacén / Cati | Exactitud de inventario | Parcial |
| 4 | Preparación de goma y reciclado | Caravaca | Producción | Kg conformes / merma | Parcial |
| 5 | Preparación de cuñas | Caravaca | Juan Carlos | Cuñas buenas / hora | Parcial |
| 6 | Corte de tejido y plantas | Caravaca | Juan Carlos, Rubén, Porretas | Aprovechamiento y faltas | Parcial |
| 7 | Reparto a aparado | Externo | Jesús | Tiempo y exactitud | Parcial |
| 8 | **Aparado externo** | Externo | Red de aparadoras | Aceptación a la primera | 🔴 **Vacía** |
| 9 | Control previo de pares | Caravaca | Responsable de ruta | Faltas antes de suela | 🔴 Por validar |
| 10 | Inyección de PVC | Caravaca | Coronel, Mohammed, Carlos, Jorge | Pares buenos/hora y OEE | Parcial |
| 11 | **Transferencia Caravaca–Alhama** | Ambas | **Por asignar** | Diferencias y tiempo de tránsito | 🔴 Por validar |
| 12 | **Vulcanización de caucho** | Alhama | **Por asignar** | Pares buenos/hora | 🔴 Por validar |
| 13 | Vías, limpieza y acabado | Caravaca | Marina, equipo de vías | Defectos por 100 pares | Parcial |
| 14 | Embalaje y almacén | Caravaca | Manolo, Jordi | Exactitud y productividad | Parcial |
| 15 | Carga y expedición | Caravaca | Almacén / Administración | OTIF y ocupación | Parcial |

**Dos procesos sin dueño.** Transferencia Caravaca–Alhama y Vulcanización
figuran con responsable "Por asignar": el 50 % de la capacidad industrial opera
sin propietario de proceso identificado. Antes de programar la pantalla de
transferencias hay que decidir quién firma la salida y quién firma la recepción.

---

## 6. Personas

Unas 30 personas al día en fábrica más 10–15 aparadoras externas. Siete perfiles
distintos; cuatro de ellos trabajan de pie con las manos ocupadas.

**Oficina** — Cati (pedidos, notas, compras, recepción, albaranes, muestras) ·
Santos (contabilidad, Query, packing lists, fichajes, facturación, Intrastat,
modelo 349) · Marina (etiquetas y embalaje, cuentas de vías y aparado, ensayos,
suministros) · Jonathan (comercial, muestras, ferias, visitas, francés) ·
Antonio hijo (mejora continua, documentación, escandallos, controlling).

**Fábrica** — Jesús (reparto y enlace con aparadoras) · Juan Carlos (cuñas,
plantas, muestras) · Rubén y Porretas (corte de tejido) · Jorge (cambios de
molde, carga, casi todo el mantenimiento) · Coronel y equipo (inyección) ·
Manolo y Jordi (revisión, embalaje, paletizado, almacén, expedición) · equipo de
vías (8–10 personas, día y noche).

### Cuatro dependencias de una sola persona

1. **Comercial y francés** — Jonathan. Francia es el primer destino del calzado
   español y el principal mercado de Lollan's.
2. **Reparto y memoria de aparado** — Jesús "lleva el control él propio".
3. **Mantenimiento de inyección** — Jorge. Sin sección de mantenimiento asignada.
4. **Requisitos de etiquetado por cliente** — Marina.

Estas cuatro dependencias son el mejor argumento de venta interna de la app: no
se trata de controlar a nadie, sino de que el conocimiento deje de vivir solo en
una cabeza.

### Tres formas de cobrar en la misma nota

- **Nómina** — corte, reparto
- **Por horas** — goma, inyección, faltas de aparado
- **Destajo** — aparado (tarifa por modelo), vías (tarifa por cliente)

Hoy se liquida con Excel, libretas de las trabajadoras y cupones que Marina
teclea en Query. Cada cupón debe vincular **persona, orden, operación, cantidad
aceptada, fecha y tarifa vigente**.

---

## 7. Datos y KPIs

Los 14 KPIs están definidos con fórmula y frecuencia. **Los 14 están en "Sin
dato".** Ocho son críticos:

| Indicador | Fórmula | Frecuencia | Área |
|---|---|---|---|
| OTIF | pedidos OTIF / entregados × 100 | Semanal | Logística |
| Rendimiento a la primera | buenos 1ª / procesados × 100 | Por turno | Calidad |
| Defectos por 100 pares | defectos / inspeccionados × 100 | Por turno | Calidad |
| OEE de inyección | disponibilidad × rendimiento × calidad | Por turno | Producción |
| Pares buenos por hora | pares aceptados / horas productivas | Por turno | Producción |
| WIP por etapa | conteo de pares por estado | Diaria | Producción |
| Valor y días de inventario | valor stock; stock / consumo diario | Semanal | Inventario |
| Margen por modelo y cliente | ingreso neto − coste completo | Mensual | Finanzas |

**Cinco de los ocho son "por turno" o "diaria".** El dato no puede introducirse
en la oficina al final del día: tiene que capturarse en el punto de trabajo, en
el momento, desde el móvil. Ese es el argumento de que la app sea móvil primero.

### Las seis bases maestras que ya existen en Notion

Catálogo maestro de producto y materiales · Procesos maestros · Activos y
capacidades · Indicadores y datos pendientes · Clientes y oportunidades ·
Riesgos, mejoras y decisiones.

Son el punto de partida del esquema de base de datos: hay que traducirlas a
tablas transaccionales, no diseñarlas desde cero.

---

## 8. Capacidad

> ⚠️ **La documentación calcula un techo teórico de 240 pares/hora por máquina y
> 9.600 pares/día. NO es capacidad comprometible.** Asume un par por estación y
> vuelta, y no descuenta cambios de molde, averías, descansos, falta de
> material, velocidad por modelo, rechazo, ni el cuello de botella de aparado,
> vías o embalaje.

**Fórmula correcta, ya escrita en Notion y que debe codificarse en la app:**

    Capacidad vendible = mínimo de (demanda confirmada, materiales disponibles,
                                    corte, aparado, inyección/vulcanización,
                                    vías/calidad, embalaje/transporte, liquidez)

    Capacidad efectiva por etapa = nominal × disponibilidad × rendimiento × calidad

### Activos y nivel de confianza

| Activo | Cantidad | Planta | Estado del dato |
|---|---|---|---|
| Máquinas de inyección | 4 | Caravaca | Por validar |
| Estaciones rotativas por operario | 12 · 240 pares/h teóricos | Caravaca | Estimado |
| Prensas de corte | 4 | Caravaca | Por validar |
| Troqueles | +100 | Caravaca | Estimado |
| Puestos de vulcanizado | ~20 | Alhama | Estimado |
| Red de aparadoras externas | 10–15 | Externo | Estimado |
| Instalación fotovoltaica | 100 kW pico | Caravaca | Por validar |
| Camión MAN (2007) | 114 m³ · 22 t ideales | Caravaca | Histórico |

Un dato deja de estar "por validar" solo cuando tiene **unidad, periodo, fuente,
propietario y contraste**. La app debe mostrar ese sello junto a cada cifra.

---

## 9. Dónde duele hoy

| Problema | Funcionalidad que lo resuelve |
|---|---|
| Faltas detectadas tarde (en vías o embalaje) | Conteo firmado en cada entrega + control previo obligatorio antes de suela |
| Notas incompletas o perdidas en el reparto | Cupón digital con firma y foto en entrega y recogida |
| Tejido con trazas que no siguen los patrones | Consumo real por nota frente a estándar, por proveedor y partida |
| Traslados a Alhama sin trazabilidad | Albarán interno con bultos identificados y estado de tránsito |
| Recuentos repetidos (vías, embalaje, nota, pedido) | Un único conteo por operación que alimenta producción, calidad y destajo |
| Mantenimiento reactivo | Registro de máquina, molde, lote, turno y operador; horas de uso por molde |
| Miles de kg de goma sin coste calculado, parte con ftalatos | Inventario con lote, valor, antigüedad y bloqueo por cumplimiento |
| Clientes que envían sus propios revisores al almacén | Liberación documentada con evidencia fotográfica por lote |

---

## 10. La web app

### 13 módulos funcionales

1. Usuarios, roles, permisos y auditoría
2. Clientes, contactos, muestras, ofertas, pedidos y reclamaciones
3. Modelos, versiones, listas de materiales, fichas técnicas, fotos, certificados
4. Proveedores, compras, albaranes y recepción
5. Inventario por artículo, lote, ubicación, reserva y movimiento
6. Órdenes, rutas, WIP y operaciones por turno
7. Aparado externo: reparto, cupones, aceptación y pago
8. Transferencias Caravaca–Alhama
9. Calidad, defectos, reparaciones y liberación
10. Activos, moldes, mantenimiento, averías y repuestos
11. Costes, márgenes, energía, nóminas/pieza y transporte
12. Expediciones, packing lists, pallets, carga y entrega
13. Documentos, fotografías, comentarios y firma

### Requisitos no negociables en planta

**Diseño para manos ocupadas** — botones grandes, poco texto, una acción
principal por pantalla; lectura por cámara o código para identificar nota, lote
y ubicación; funcionamiento sin conexión en operaciones críticas con
sincronización posterior; recuperación ante error sin perder la captura.

**Gobierno del dato desde el día uno** — los movimientos de inventario y
producción no se sobrescriben, se anexan; cada dato crítico lleva fuente,
responsable, fecha y estado de validación; permisos mínimos por rol y auditoría
completa; sin contraseñas ni datos personales innecesarios.

### Convivencia con lo existente

- **Query Pro** — sigue siendo el ERP administrativo (facturación, contabilidad,
  Intrastat, modelo 349) hasta mapear la integración.
- **Notion** — capa de conocimiento, procedimientos y decisiones.
- **App nueva** — lo transaccional: movimientos, órdenes, inventario, calidad,
  costes y usuarios.

El puente crítico a resolver pronto es la salida de notas de fabricación desde
Query, que hoy pasa por PDF impreso → carpeta de Dropbox → escenario de Make.

---

## 11. Pantallas por rol (punto de partida)

Cada persona entra y ve **una sola lista: lo suyo, hoy**.

**Operario de corte — "Mis notas de hoy"** · lista por prioridad filtrada por su
nombre; modelo, foto patrón, pares, tallas, tejido y planta como dos tareas
separadas; Empezar / Pausar / Hecho con cronómetro automático; botón Falta que
obliga a decir qué falta y cuánto, con foto.

**Jesús, reparto — "Ruta de hoy"** · mapa de entregas y recogidas por aparadora;
entrega con escaneo de nota, confirmación de pares y pasos, firma; recogida con
conteo de aceptados y rechazados y foto de incidencia; vista "quién tiene qué"
en tiempo real.

**Inyección — "Cola de máquina"** · qué nota en qué máquina y con qué molde;
registro de máquina, molde, lote, turno y operador al arrancar; contador de
pares buenos y rechazados; parada con causa en un toque.

**Marina, vías y embalaje — "Reparto de vías"** · asignar notas y ver avance;
ficha de embalaje por cliente precargada (percha, bolsa, serret, etiquetas,
caja); recuento final contra nota y pedido con faltas calculadas; liquidación de
destajo generada del conteo.

**Cati, oficina — "Pedidos y materiales"** · pedidos por fecha comprometida con
semáforo de riesgo; qué material falta para liberar cada nota y a qué proveedor
se reclamó; alta de nota con datos ya extraídos, solo revisar; bandeja de faltas
abiertas de más de 24 h.

**Almacén — "Preparación de carga"** · cargas de la semana por cliente, fecha y
hora con estado; palés, bultos y pares con avance de paletizado; camión de
Lollan's o del cliente; packing list y cita de carga.

**Jaime, dirección — "Estado de la fábrica"** · pedidos en riesgo ordenados por
importe; WIP por etapa; faltas abiertas y coste estimado; cargas y prioridades
de la semana.

**Jonathan, comercial — "Cartera y muestras"** · cuentas por facturación,
margen, potencial y riesgo; embudo muestra → oferta → pedido con motivo de
pérdida; estado de fabricación de sus pedidos; ferias y visitas.

### Ya prototipado (v2 y v3 en Notion)

Tres accesos para Jaime: **cargas de la semana** (plan mensual navegable, fotos,
pares, bultos, palés, camión propio o de cliente), **prioridades de la semana**
(pendiente → en curso → hecha) e **inventario de embalaje** (cajas, perchas,
bolsas, etiquetas, serret, film, con alerta de mínimo). Validados a 390 y 320 px
y funcionan sin conexión, pero **guardan solo en el dispositivo**: necesitan
servidor para que Antonio y Jaime vean lo mismo.

---

## 12. Plan por fases

**Fase 0 · ahora — cerrar el inventario**
Export de blueprints de Make o capturas; token de API de Monday de solo lectura
o export de columnas; confirmar qué automatizaciones son nativas de Monday;
decidir el punto de retorno desde Alhama y quién firma cada extremo.

**Fase 1 · piloto — un solo flujo extremo a extremo: la nota**
Entidad Nota con ruta, prioridad, fases y estados idénticos a los actuales;
corte con tejido y planta separados, cronómetro y falta con comentario
obligatorio; circuito de faltas completo con vuelta a la fase de origen;
pantalla móvil de operario y vista global de oficina.

**Fase 2 — los dos puntos ciegos**
Aparado externo (cupón digital con entrega, recogida, aceptación y firma) y
transferencias Caravaca–Alhama (albarán interno con bultos y estado de tránsito).

**Fase 3 — medir de verdad**
WIP por etapa, pares buenos/hora, defectos por 100 pares, rendimiento a la
primera; partes diarios y semanales sustituyendo los seis tableros de Monday;
liquidación automática de destajo.

**Fase 4 — coste, margen y capacidad**
Solo cuando los conteos sean fiables. Escandallo por modelo con consumo real
frente a estándar; margen por modelo, cliente y pedido; capacidad efectiva y
capacidad vendible con su cuello de botella.

**Fase 5 — integrar y escalar**
Puente con Query Pro sustituyendo el PDF por Dropbox; inventario con lote,
ubicación y bloqueo por cumplimiento (ftalatos, REACH); CRM, muestras y
reclamaciones.

---

## 13. Decisiones pendientes

Las cuatro primeras bloquean la Fase 1.

1. **¿La app sustituye a Monday o convive con él durante la transición?**
   Si sustituye, la Fase 1 debe cubrir todo el circuito de la nota antes de
   apagar nada. Si convive, hay que decidir cuál manda ante el mismo dato.
2. **¿De dónde entra la nota de fabricación?**
   Seguir con Query → PDF → Dropbox → extracción, o atacar Query directamente.
   Lo primero se monta en días; lo segundo es lo correcto a medio plazo.
3. **¿Quién firma la salida y la recepción entre Caravaca y Alhama?**
   Hoy figura "Por asignar". Sin un nombre no se puede diseñar la pantalla.
4. **¿Las aparadoras entran en la app, o solo Jesús?**
   Cambia por completo el módulo 7.
5. **¿Qué pasa con Trello y los Excel de cuentas?**
   Cati usa Trello; Marina lleva vías, embalaje y aparado en Excel y libretas.
6. **¿Móvil propio o dispositivo de fábrica?**
   Determina login, sesión y modo sin conexión.
7. **¿Se identifican las notas con QR o código de barras?**
   Afecta a la impresión de la nota en Query.
8. **¿Los datos de destajo son visibles para quien cobra?**
   Elimina la reconciliación con libretas, pero es decisión de gestión de
   personas, no técnica.
9. **¿Alhama entra en la Fase 1 o después?**
   Es media empresa y donde menos visibilidad hay.

---

## Fuentes

Documentación de Notion de Lollan's:

- LOLLAN'S — Sistema Operativo Empresarial (secciones 00–11)
- Bases maestras: Catálogo de producto y materiales, Procesos maestros, Activos
  y capacidades, Indicadores y datos pendientes, Clientes y oportunidades,
  Riesgos/mejoras/decisiones
- 📚 Manual Operativo de Monday.com (v. 20 jun 2025)
- 💼 Guía de uso para el equipo de corte – Monday.com
- Organización con Monday
- Documentación (resumen por secciones + tareas de Santos, Marina, Cati,
  Jonathan y Antonio hijo)
- Flujo de Producción – Fábrica Lollan's y Documentación Sección de Corte

Los datos marcados como estimados, por validar o históricos se han mantenido con
esa etiqueta deliberadamente. **Ninguna cifra de capacidad, coste o consumo de
este documento debe usarse para comprometer un pedido sin validarla en planta.**
