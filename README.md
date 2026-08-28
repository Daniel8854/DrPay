# Especificaciones del Sistema — Proyecto tipo "PagoVentas"
*Documento vivo — se actualiza a medida que se revisan más módulos. Uso: referencia para desarrollo y para dimensionar infraestructura/hosting.*

---

## 📌 CÓMO USAR ESTE DOCUMENTO EN UN CHAT NUEVO DE CLAUDE CODE

1. Copia y pega **todo este archivo** como primer mensaje del chat (o cárgalo como archivo si el proyecto ya está inicializado con Claude Code — mejor aún, guárdalo dentro del repo como `ESPECIFICACIONES.md` para que quede siempre disponible).
2. Dile a Claude Code en qué fase vas a trabajar (ver sección 9 — Hoja de ruta), por ejemplo: *"Vamos a trabajar en la Fase 1: diseño del esquema de base de datos"*.
3. Pide que te explique las decisiones técnicas, no solo que genere código — así aprendes mientras avanzas.
4. Cuando termines una sesión de trabajo, pide un resumen de lo avanzado y **actualiza este documento** (marca qué se implementó, qué decisiones de diseño se tomaron, qué quedó pendiente) antes de cerrar el chat. Así el siguiente chat arranca con contexto real, no solo con el diseño original.
5. Si tomas una decisión técnica importante (ej. "el interés se calcula así: ...", "elegí PostgreSQL porque..."), anótala en la sección 10 (Decisiones de diseño y avance real) para que no se pierda.

---

## 📊 ESTADO GENERAL DEL DOCUMENTO

| # | Módulo | Estado documentación | Estado desarrollo |
|---|--------|----------------------|--------------------|
| 1 | Visión general del sistema | ✅ Completo | ⏳ No iniciado |
| 2 | Caja (Gastos, Inversión/Retiro, Cierre/Reapertura, Vales) | ✅ Completo | ⏳ No iniciado |
| 3 | Reportes (Cuadre Recaudador, Resumen Ruta, Visitas en Blanco, Informes del Día, Clientes) | 🔄 Parcial — faltan: Cuadre Caja, Saldo Caja Ruta, Informe Utilidad, Informe Cuadre de Socios, Informes Geográficos, Total de Rutas | ⏳ No iniciado |
| 4 | Mapas | ❌ Fuera de alcance (decisión del cliente) | N/A |
| 5 | Configuración (Gestionar/Vincular Unidades, Ajustes, Usuarios) | 🔄 Parcial — faltan: Días no Laborables, SMTP, Auditorías, Auditorías Accesos, submenú Usuarios completo | ⏳ No iniciado |
| 6 | App Móvil (Login, Pantalla Inicial, Lista Clientes) | 🔄 Parcial — faltan: Menú general, Clientes nuevos/disponibles móvil, Ajustes móvil, Cuotas Pendientes/Todas las Cuotas | ⏳ No iniciado |
| 7 | Implicaciones técnicas | ✅ Completo (se ampliará si surge algo nuevo) | — |
| 8 | Infraestructura/hosting | ✅ Completo (sin mapas) | — |
| 9 | Hoja de ruta de desarrollo | ✅ Completo | — |
| 10 | Decisiones de diseño y avance real | 🆕 Se llena a medida que se desarrolla | — |

---

## 1. Visión general del sistema

- Sistema de control de préstamos/ventas a crédito con **web administrativa** + **app móvil**.
- Estructura jerárquica geográfica: **País → Departamento/Estado → Ciudad → Ruta/Unidad**.
- Cada **Ruta/Unidad** tiene su propia caja, cobrador(es) y cartera de clientes.
- Existen **Socios** (inversionistas) asociados a rutas, además de rutas normales.
- Modelo de negocio propio (el cliente final que compra el sistema): **superadmin → empresa cliente → trabajadores/cobradores (1 acceso móvil c/u)**.
- La **app móvil funciona offline** y sincroniza después con el servidor (se ve por los campos "Digitado" vs. "Sincronizado" en varias pantallas). Esto implica necesidad de:
  - API de sincronización robusta (no CRUD simple)
  - Manejo de conflictos de sincronización
  - Cola de sincronización en backend
- La app **no se publicará en tiendas** — se distribuye por descarga directa (APK) desde la propia web (botón "Descargar App").
- Login web: email + password.
- Operaciones sensibles (dinero) requieren **doble confirmación con contraseña**.

---

## 2. Módulo: CAJA ✅ (completo)

Menú: Gastos | Inversión y Retiro | Cierre de Unidad | Reapertura de Unidad | Vales

### 2.1 Gastos
- Registro de gastos por ruta, con tipo/categoría de gasto (catálogo configurable).
- Filtros: rango de fechas, ruta (obligatoria seleccionar), texto, tipo de gasto.
- Exportación a Excel.
- CRUD: crear y eliminar (no se ve edición directa).

### 2.2 Inversión y Retiro
- Control de capital de la caja: aportes de socios, ingresos, retiros.
- Filtros: rango de fecha, ruta obligatoria, tipo de movimiento, filtrar solo ingresos/retiros/todo.
- Exportar a Excel, opción "Traslado de Efectivo".
- Eliminar movimiento requiere **contraseña del usuario web**.
- Crear movimiento requiere: fecha, valor, tipo de movimiento (catálogo), motivo (opcional), **contraseña de confirmación**.

### 2.3 Cierre de Unidad
- Bloquea operaciones desde el móvil para una ruta en una fecha específica.
- Selección múltiple de rutas (árbol tipo checkbox) para cierre masivo.
- Estado: Pendiente / Sincronizado.
- Se puede revertir eliminando la solicitud de cierre.

### 2.4 Reapertura de Unidad
- Revierte un cierre. Requiere: unidad (o ID directo), fecha, motivo opcional.
- Se puede volver a cerrar una unidad reaperturada.

### 2.5 Vales
- Préstamos internos SIN interés, por ruta.
- Estados: Activos, Cancelados, Eliminados.
- Permite abonos parciales (con fecha, valor, comentario).
- Eliminar vale solo si no tiene abonos, requiere contraseña.
- Eliminar abono individual.
- Exportar listado a Excel.

---

## 3. Módulo: REPORTES 🔄 (en progreso)

Submenú: Cuadre Recaudador | Cuadre Caja | Resumen Ruta | Saldo Caja Ruta | Informe Utilidad | Informe Cuadre de Socios | Visitas en Blanco | Informes del Día (submenú) | Clientes (submenú) | Informes Geográficos | Total de Rutas

### 3.1 Cuadre Recaudador ✅
- Reporte financiero completo por ruta y rango de fechas. Incluye:
  - Info general: ID ruta, rango fechas, días laborados, valor apertura
  - Movimientos: caja inicial, aportes capital, recaudos, retiros, gastos, ventas, vales, abonos a vales, saldo final
  - Promedio cobro diario, clientes activos (hoy / rango), promedio venta por cliente
  - Cartera total (con y sin ajuste/interés)
  - % del negocio en gastos, % en cobro, % de utilidad sobre cartera
  - Relación detallada de gastos (tipo, comentario, valor)
- Exportación: Excel (todo), Excel (individual), **PDF "Liquidación Cobrador"** (multi-hoja si son varias rutas)
- Opción "Agrupar por cliente"
- **Es un motor de cálculo financiero**, no un reporte estático — recalcula cartera, interés, utilidad.

### 3.2 Resumen Ruta ✅
- Resumen semanal por ruta con totales acumulados por semana y total general.
- Multi-selección de rutas (checkbox árbol).
- Exportable a Excel.

### 3.3 Visitas en Blanco ✅
- Registro de visitas sin venta/cobro (control de gestión del cobrador), viene del móvil.
- Filtros: ruta, rango fecha, nombre/identificación/alias del cliente.
- Exportable a Excel.

### 3.4 Informes del Día 🔄 (submenú)
Contiene: Ventas | Pagos Realizados | **Consolidado Mora** | **Detallado Mora** | Notas Débito-Crédito | Resumen Diario
- ⚠️ Confirma que el sistema calcula **mora** (cartera vencida) automáticamente — lógica financiera adicional (días de atraso, posible interés moratorio).

#### 3.4.3 Pagos Realizados ✅
- Muestra pagos (abonos) realizados a préstamos en un rango de fechas.
- Selección de ruta(s) vía árbol (una, varias o todas), rango de fechas, filtro por Nombre/Identificación/Código Cliente/Id Préstamo.
- Listado: fecha, fecha móvil, modo, ID cliente, cliente, alias, barrio, teléfono, valor préstamo, % interés, total préstamo, valor abono, saldo, ruta.
- Acciones por abono:
  - **x (Eliminar):** solo si el cliente no tiene préstamos activos. Requiere contraseña del usuario web.
  - **m (Modificar):** modifica el valor del abono (no puede ser mayor al valor del préstamo ni al saldo) + comentario.
- Botón **Recibo** (genera/imprime recibo de pago — implica plantilla de recibo imprimible).
- Exportar a Excel, refrescar listado.

#### 3.4.4 Consolidado Mora ✅
- Muestra préstamos con cuotas atrasadas y días en mora, según la modalidad de cada préstamo (diaria/semanal/quincenal/mensual).
- Selección ruta(s), rango de fechas.
- **Tipo de informe:** por Fecha de Vencimiento o por Fecha de Pago. Opción "Agrupar por cliente".
- Listado: fecha venta, nombre cliente, valor venta, valor cuota, cuotas (valor a pagar), cuotas pagadas (cantidad|valor), cuotas pendientes (cantidad|valor), **cuotas atrasadas (cantidad|valor)**, **días vencidos|valor**, fecha último abono, saldo.
- **Resaltado visual en rojo** para préstamos con días de atraso — indicador de mora a simple vista.
- Exportar a Excel.
- ⚠️ Esto confirma **cálculo automático de mora por cuota**, no solo a nivel de préstamo — el sistema debe llevar el estado de cada cuota individual (pagada/pendiente/atrasada) según la modalidad de pago.

#### 3.4.5 Detallado Mora ✅
- Muestra los días en mora **por cada cuota individual** del préstamo (nivel más granular que Consolidado Mora).
- Mismos filtros: árbol de rutas, rango de fechas.

#### 3.4.6 Notas Débito-Crédito ✅
- Muestra el historial de Notas Crédito y Notas Débito aplicadas a ventas (creadas desde el módulo Ventas, ver 3.4.1).
- Selección ruta(s), rango de fechas, tabs "Notas Crédito" / "Notas Débito".
- Listado: tipo (C/D), fecha, venta (ID), cliente, vendedor, valor, comentario, estado.
- Eliminar nota (botón X). **Nota importante:** al eliminar una nota crédito, pasa a la pantalla de notas débito y viceversa (parece ser un mecanismo de reversión/auditoría, no un borrado simple — revisar bien esta lógica al desarrollar).

#### 3.4.7 Resumen Diario ✅
- Informe de "Cobro Diario" por ruta: visitas programadas vs. realizadas, total a cobrar vs. total cobrado, % de cobro.
- Selección ruta(s), rango de fechas.
- Listado por ruta y fecha: visitas programadas, visitas realizadas, total a cobrar, total cobrado, % cobro. Con totales generales.
- Exportar a Excel, refrescar listado.
- ⚠️ Implica que el sistema lleva una **programación de visitas esperadas por día** (ruta/cobrador tiene una cantidad de visitas "programadas" vs. las que efectivamente se hicieron) — funcionalidad de planificación/productividad del cobrador.
- Gestiona todas las ventas (préstamos) de una ruta en un rango de fechas.
- Selección de ruta vía árbol (país/depto/ciudad/ruta), rango de fechas, checkbox "Solo activos".
- Filtros: rango de valor (min/max), rango de % de ajuste/interés (min/max), y filtro por Nombre / Identificación / Código Cliente / Id Venta.
- Listado muestra: fecha, fecha móvil, modo, identificación, nombre, alias, barrio, teléfono, valor venta, % ajuste, total venta, abonos, saldo, método (Diaria/Manual/Quincenal/Semanal), valor cuota, inicio, termina, días, ruta.
- Exportar a Excel, refrescar listado.
- **Operaciones por venta (columna "Operaciones", 4 botones):**
  - **A (Abono):** registra abono a la venta desde la web (fecha, valor —no puede superar saldo—, comentario). Actualiza saldo y abonos automáticamente.
  - **X (Eliminar):** solo si la venta no tiene abonos. Requiere contraseña del usuario web.
  - **M (Modificar préstamo):** solo si la venta no tiene movimientos/abonos. Permite modificar:
    - Valor neto de la venta
    - Porcentaje de ajuste (interés) — **requiere permiso habilitado**
    - Plazo venta (cuotas)
    - Modalidad de pago (diaria/mensual/quincenal/semanal) — **requiere permiso habilitado**
  - **C / D (Nota Crédito / Nota Débito):** ajustan la venta sin ser un abono real. Fecha, valor, comentario, y "Modo":
    - Disminuir valor saldo
    - Disminuir valor cuota (modifica cuotas pendientes)
    - Disminuir días (modifica cantidad de cuotas)
    - Ajuste (recalcula cuotas y saldo)
    - Nota crédito disminuye valores, nota débito los aumenta (mismo formulario para ambas).

#### 3.4.2 Sistema de Roles y Permisos ✅ (descubierto vía módulo Ventas — es transversal a todo el sistema)
- Ubicación: **Configuración → Usuarios → Roles**
- Permisos granulares por módulo, organizados en checkboxes tipo matriz (ej. columnas: Usuarios, Vales, Ventas...).
- Ejemplo de permisos en sección "Ventas": Adicionar Notas, Eliminar Notas, Eliminar Venta, Informe Notas, Informe Ventas Activas, Informe Ventas Finalizadas, Informe Ventas Nuevas, **Modificar modalidad pago**, Modificar Venta.
- Ejemplo permisos "Usuarios": Adicionar, Definir Permisos Rutas, Eliminar, Modificar, Ver.
- Ejemplo permisos "Vales": Adicionar Abono Vale, Adicionar Vale, Eliminar Abono Vale, Eliminar Vale, Ver Vales.
- **Roles configurables**, no fijos — se pueden crear/editar roles y marcar qué puede hacer cada uno, por módulo y acción específica.
- A nivel de usuario individual (Configuración → Usuarios → Modificar Datos) hay también flags específicos:
  - Habilitar manejo fecha de movimiento
  - Permitir consulta multiempresa
  - Permitir modificar porcentaje
  - Estado (Activo/Inactivo)
  - Rol (dropdown, ej. Administrador)
  - Es socio / Es dependiente de socio
  - Asignar/cambiar contraseña
- ⚠️ **Implicación técnica importante:** esto es un sistema de **permisos RBAC (Role-Based Access Control) granular** — no roles fijos tipo "admin/usuario". Cada acción de cada módulo (crear, editar, eliminar, ver, exportar, informes específicos) es un permiso independiente asignable por rol. Esto es una funcionalidad grande de desarrollar bien (tabla de permisos, matriz de roles, validación en frontend Y backend de cada permiso).
- ⚠️ **"Permitir consulta multiempresa"** sugiere que ya el sistema original contempla usuarios que pueden ver más de una empresa/tenant — relevante para el diseño multi-tenant que se busca replicar.

### 3.6 Módulo: Clientes 🔄 (submenú)
Contiene: Listar Clientes | Clientes Nuevos | Clientes Activos | Clientes Inactivos | Clientes Disponibles

#### 3.6.1 Listar Clientes ✅ (completo)
- Lista clientes según ruta(s) seleccionada(s) en árbol; permite crear cliente nuevo y préstamo asociado desde la misma pantalla.
- **Estado (dropdown):** Activos, Disponibles, Eliminados, Todos.
- **Filtro avanzado** (checkbox "Aplicar filtro" + tipo de filtro + rango de fecha), con 3 tipos:
  - **A. Cupo aumentado:** lista clientes con préstamo creado en el rango de fechas, comparando el valor bruto del penúltimo vs. último préstamo — si el último es mayor, aparece en la lista. (Detecta clientes a quienes se les subió el cupo de crédito.)
  - **B. Clientes Renovados:** clientes con préstamo en el rango de fechas que tienen más de un préstamo en todo su histórico.
  - **C. Clientes NO Renovados:** clientes con préstamo en el rango de fechas que solo tienen un préstamo en su histórico (nunca han renovado).
  - ⚠️ Esta lógica de comparación entre préstamos históricos (penúltimo vs. último, conteo de préstamos totales) es una regla de negocio específica de scoring/gestión de cartera — hay que replicarla con precisión.
- Buscador de cliente por nombre, exportar a Excel, refrescar listado, botón "+Crear Nuevo".

**Crear Nuevo Cliente** (formulario con 2 secciones):
- *Datos personales:* fecha, identificación, nombre, alias, dirección, barrio, teléfono, correo, género.
- *Datos del préstamo asociado* (se crea el cliente Y su primer préstamo en un solo paso):
  - **Ubicar antes de:** posición del cliente en la lista de la ruta (orden de visita del cobrador — importante para las rutas físicas)
  - **Modalidad:** diaria, semanal, quincenal, mensual
  - **% Interés:** lista de porcentajes predefinidos (catálogo configurable)
  - **Valor sin interés:** monto del préstamo
  - **Valor total:** calculado automáticamente (capital + interés)
  - **Días:** número de cuotas/días de plazo
  - **Cuota:** calculada automáticamente (valor total ÷ días, con lógica propia)
  - ⚠️ Este es el **motor de cálculo de préstamos** — fórmulas de interés y cuotas que deben quedar exactamente iguales al sistema original.

**Listado de clientes — acciones por cliente:**
- **m (Modificar datos):** edita datos personales (no financieros) del cliente.
- **h (Ver historial):** muestra todos los préstamos del cliente (histórico) con sus abonos y saldos por préstamo, más notas asociadas.
- **x (Eliminar cliente):** requiere contraseña del usuario web.
- **t (Trasladar cliente):** mueve al cliente de una ruta/unidad a otra. Requiere: código unidad destino (vía árbol o código directo), motivo. Al procesar, el cliente queda reasignado a la nueva ruta.
  - ⚠️ Implica mantener trazabilidad histórica de a qué ruta perteneció un cliente en cada momento (auditoría de traslados).

#### 3.6.2 Clientes Nuevos ✅
- Ve clientes que fueron nuevos en el rango de fecha seleccionado, según la unidad seleccionada.
- Filtros: rango de fechas, dropdown "Unidad" (mostrar todas o una ruta específica), buscador de cliente.
- Listado: código, nombre, dirección, barrio, valor, saldo, método (modalidad), inicia, termina, cobro (nombre de ruta/control).
- Checkbox de selección por fila (posible acción masiva, a confirmar más adelante), exportar a Excel, refrescar listado.

#### 3.6.3 Clientes Activos ✅
- Lista clientes con préstamos sin cancelar (activos).
- Árbol de rutas, buscador de cliente, exportar a Excel, refrescar listado.
- Acciones por cliente: **m** (modificar datos personales), **h** (ver historial: preéstamos, abonos, saldos, notas), **x** (eliminar cliente, requiere contraseña).

#### 3.6.4 Clientes Inactivos ✅
- Lista clientes marcados como inactivos.
- Árbol de rutas, rango de fechas (de inactivación), buscador, exportar a Excel, refrescar listado.
- Listado muestra: nombre, dirección, barrio, teléfono, **fecha inactivación**, saldo, **último abono**, ruta.
- Acción: **a (Activar cliente)** — requiere contraseña del usuario web; al activarlo el cliente pasa a la lista de Clientes Activos.
- ⚠️ Implica un **estado explícito de inactivación** con fecha y motivo (posiblemente automático tras X días sin movimiento, o manual) — hay que definir bien el trigger de inactivación al desarrollar.

#### 3.6.5 Clientes Disponibles ✅
- Clientes que **ya tienen sus préstamos cancelados totalmente** — disponibles para renovar o generar préstamo nuevo.
- Filtro especial: checkbox "Obtener lista de clientes buenos" + campo **"Días de tolerancia"** — genera un informe de clientes "buenos" (pagadores) que estuvieron dentro del rango de tolerancia ingresado (probablemente relacionado a mora/atraso histórico bajo).
- Árbol de rutas, buscador, exportar a Excel, refrescar listado.
- Acciones por cliente:
  - **r (Renovar crédito):** abre la misma pantalla de "Crear cliente nuevo" pero pre-cargada con los datos personales del cliente disponible — solo se ingresan los datos del nuevo préstamo (modalidad, % interés, valor, días, cuota).
  - **m (Modificar datos), h (Ver historial), x (Eliminar cliente)** — igual que en los demás listados.
- ⚠️ El concepto de "clientes buenos" con "días de tolerancia" es otra regla de negocio de scoring de cartera — parece usarse para identificar a quién ofrecer renovación de forma prioritaria.

**✅ Módulo Clientes — COMPLETO** (Listar, Nuevos, Activos, Inactivos, Disponibles)

---

## 4. Módulo: MAPAS ❌ (FUERA DE ALCANCE — decisión del cliente, no se desarrollará por ahora)
*Se deja documentado por si se retoma en una fase futura.*

Submenú: Rutas | Pagos

### 4.1 Rutas
- Muestra ubicación GPS de las rutas/unidades sobre un mapa (Google Maps, vista Mapa/Satélite).
- Panel lateral con listado de rutas (nombre, fecha/hora del último registro de ubicación); clic en una muestra su posición en el mapa.
- Opción "Ver Recorrido" por ruta — implica que se guarda un **historial de posiciones GPS**, no solo la última ubicación (tracking tipo polyline del recorrido del cobrador).

### 4.2 Pagos
- Muestra ubicación de dónde se realizaron los pagos (geolocalización de cada transacción de cobro desde el móvil).

⚠️ **Implicación técnica importante:** el sistema requiere **captura de GPS desde la app móvil** en cada movimiento relevante (posición del cobrador + ubicación de cada pago), almacenamiento de tracks/recorridos, e integración con Google Maps API (o alternativa) en la web para visualización. Esto es costo adicional real: la API de Google Maps tiene cuotas de uso pagas más allá de cierto volumen.

---

## 5. Módulo: CONFIGURACIÓN 🔄 (en progreso)

Submenú: Gestionar Unidades | Unidades Vinculadas | Usuarios (submenú) | Ajustes (submenú)

### 5.1 Gestionar Unidades ✅ (completo)
- Crea y modifica unidades/rutas (las que tienen funcionalidad en el móvil).
- Filtro Estado (Activas/Inactivas), buscador por nombre/usuario, botón "+Crear Nuevo", refrescar listado.

**Crear/Modificar Unidad — formulario con 2 secciones:**

*Sección 1 — Datos de la unidad:*
- Nombre Unidad, Encargado, Revisor (dropdown, puede ser "Sin revisor"), Estado (Activo/Inactivo)
- **Permisos de unidad** (dropdown, no confundir con el RBAC de usuarios — esto es a nivel de unidad/ruta):
  - **Cobranza:** permite todas las operaciones
  - **Caja bloqueada:** no permite movimientos directos de caja
  - **Eliminar pagos bloqueado:** bloquea eliminación de pagos para esa unidad
  - **Caja y eliminar bloqueado:** combina las dos anteriores
- Usuario Móvil (único — es el login para la app), Número telefónico, Correo electrónico
- Identificación única de la unidad (autogenerada, solo visible en modificar), fecha/hora de creación (auditoría)

*Sección 2 — Datos de venta/operación de la unidad:*
- **Caja inicial:** capital con el que arranca la unidad
- **Límite Venta / Límite Gasto:** topes que la unidad no puede superar
- **"Los préstamos deben ser aprobados":** checkbox — si se marca, los préstamos creados por esa unidad requieren aprobación previa (ver "Autorizar Ventas" visto en headers anteriores)
- **Tiempo para cierre de sesión:** timeout de sesión automático en el móvil
- **Cuotas máximas para refinanciación:** número de cuotas pendientes máximo permitido para poder refinanciar un préstamo (saldar la totalidad solo si cuotas pendientes ≤ este número; default 100)
- **Interés exclusivo:** función aparte (mencionada pero no detallada aún — pendiente)
- **Visualización de Caja:** Global / Diaria / Semanal / No mostrar
- **Acceder en modo incógnito** (checkbox)
- **Permitir retiros con caja negativa** (checkbox)
- Ubicación: País / Departamento / Ciudad (dropdowns dependientes)
- Asignar/repetir contraseña de la unidad (login móvil)

**Listado de unidades — acciones por unidad:**
- **Importar:** carga masiva desde Excel (.xls). Descarga plantillas separadas para "Préstamos Nuevos" y "Saldos", más una guía en PDF. Sube archivo y valida, muestra éxito o errores.
  - ⚠️ Funcionalidad de **importación masiva con plantilla y validación** — se usará seguramente para migrar datos de clientes que ya tengan en Excel/otro sistema.
- **Modificar:** igual formulario que crear.
- **Trasladar:** traslada TODOS los clientes de una ruta origen a una ruta destino (distinto del traslado individual visto en Clientes). Requiere ruta destino (árbol o código) y motivo.
- **Permisos:** permisos especiales adicionales por unidad (más granulares que el dropdown Cobranza/Caja bloqueada), checkboxes: Modificar Venta, Eliminar Venta, Modificar Pago, Eliminar Pago, Modificar Gasto, Eliminar Gasto, Modificar Retiro, Eliminar Retiro, Modificar Ingreso, Eliminar Ingreso, Gestionar Gastos, Modificar Cliente, Visualizar Cierre.
  - ⚠️ Esto es un **segundo nivel de permisos**, a nivel de unidad/ruta (no de usuario/rol) — el sistema termina teniendo permisos por usuario (RBAC visto en 3.4.2) Y permisos por unidad. Hay que diseñar bien cómo interactúan ambos niveles.
- **Gastos:** define **límite de gasto mensual por categoría** para esa unidad (ej: Mantenimiento $4.00, Sueldo Cobrador, etc.). El límite se resetea a blanco cada día 1 del mes (no es por rango de fecha, es por mes calendario).
- **Inactivar / Activar:** cambia estado de la unidad, con mensaje de confirmación.

### 5.2 Unidades Vinculadas ✅ (completo)
- Gestiona la vinculación/desvinculación entre una unidad/ruta y un **dispositivo móvil físico**.
- Botón "Vincular Unidad": selecciona ruta (árbol o código), ingresa **IMEI** del equipo y teléfono, guardar.
- Listado muestra: Unidad, IMEI, Teléfono, Estado (Vinculada/Desvinculada), **VersionApp** (versión de la app instalada en ese dispositivo), Acción (v = vincular, d = desvincular, con confirmación modal).
- ⚠️ **Implicación técnica clave:** el sistema controla a nivel de **IMEI específico** qué dispositivo puede operar con qué unidad — esto es una capa de seguridad antifraude/anti-clonación de acceso (evita que un usuario móvil se loguee desde cualquier dispositivo). También permite ver qué versión de app tiene cada dispositivo, útil para forzar actualizaciones.
- Esto también valida algo importante: **una unidad se vincula a UN dispositivo por vez**, reforzando el diseño de "1 acceso móvil = 1 persona/dispositivo" que buscas para tu modelo de negocio.

### 5.3 Ajustes 🔄 (submenú)
Contiene: Configuración | Días no Laborables | Reconstruir Saldos | Configuración SMTP | Auditorías | Auditorías Accesos

#### 5.3.1 Configuración ✅ (completo)
Sub-secciones dentro de "Configuración" (dentro de Ajustes):

1. **Encabezado de Impresión:** datos que aparecen en el recibo de pago impreso (nombre empresa, NIT, dirección, teléfono, moneda — símbolo $ configurable). Relacionado con el botón "Recibo" visto en Pagos Realizados.
2. **Porcentaje de Ajuste:** catálogo de hasta 10 porcentajes de interés predefinidos, usados al crear/modificar préstamos (tanto web como móvil). Editable, algunos valores pueden quedar vacíos.
3. **Ajustes Empresa:** (mencionado, pendiente ver detalle específico — no se alcanzó a capturar el contenido completo del formulario).
4. **Gestión de Gastos:** catálogo de tipos/categorías de gasto (ej. Mantenimiento, Sueldo Cobrador, etc.) — crear, modificar (M), eliminar (X), activar/desactivar por checkbox Estado. Es el catálogo maestro que alimenta el dropdown "Tipo de Gasto" visto en Caja → Gastos.
5. **Gestión de Movimientos de Caja:** catálogo de tipos de movimiento (para Inversión y Retiro) — descripción, tipo (Retiro o Ingreso), activo/inactivo, modificar/eliminar. Alimenta el dropdown "Tipo de Movimiento" visto en Caja → Inversión y Retiro.

⚠️ Todos estos catálogos (tipos de gasto, tipos de movimiento, porcentajes de interés) son **configurables por el cliente final (empresa)**, no fijos en el sistema — hay que diseñarlos como tablas maestras editables por tenant, no como enums fijos en código.

#### 5.3.2 Reconstruir Saldos ✅ (completo)
- Recalcula/reconstruye los saldos de las rutas seleccionadas en un rango de fechas.
- Selección de ruta(s) vía árbol, rango de fechas (inicial/final).
- Modo **Prueba** o **Definitivo** — Prueba simula sin guardar cambios, Definitivo aplica los cambios de forma permanente.
- ⚠️ **Herramienta crítica de mantenimiento/corrección de datos** — para cuando hay descuadres de caja (por errores de sincronización, ediciones manuales, etc.). Es señal de que el sistema original ya tuvo problemas de consistencia de datos en algún punto — hay que diseñar el motor de cálculo de saldos de forma que sea:
  1. Determinístico (mismo input = mismo output siempre)
  2. Re-ejecutable sin duplicar datos
  3. Con modo "dry-run" (prueba) desde el diseño inicial, no como parche después

#### 5.3.3 Pendientes por documentar (Ajustes)
- Días no Laborables (ya se mencionó el nombre en el menú, falta el detalle — probablemente afecta el cálculo de mora/vencimientos)
- Configuración SMTP (correo saliente del sistema — notificaciones, recuperación de contraseña, etc.)
- Auditorías
- Auditorías Accesos

## 6. Módulo: APP MÓVIL 🔄 (en progreso)

⚠️ **Nota general:** esta es la app que usan los cobradores/trabajadores en campo — el "corazón operativo" del sistema para el modelo de negocio (1 acceso móvil = 1 trabajador/ruta). Debe funcionar offline con sincronización posterior.

### 6.1 Inicio de sesión ✅
- Login con **Usuario Móvil** (creado desde la web en Gestionar Unidades) + Contraseña.
- Pantalla simple, sin recuperación de contraseña visible en esta captura (a confirmar).

### 6.2 Pantalla Inicial ✅ (completo — pantalla principal/operativa de la app)

**Barra superior (5 accesos directos):**
1. **Menú:** accede al menú general de la app (aún no detallado — sección aparte).
2. **Aprob. (Aprobación/rechazo de préstamos):** cola de preéstamos creados que requieren autorización (relacionado con el checkbox "Los préstamos deben ser aprobados" visto en Gestionar Unidades). 3 pestañas:
   - **Pendientes:** préstamos por aprobar/rechazar desde la web.
   - **Aprobados:** préstamos ya aprobados desde la web — al tocar uno sale confirmación "Grabar Préstamo" (Sí/No) para registrarlo definitivamente en el móvil.
   - **Rechazados:** préstamos que la web rechazó.
   - Incluye buscador de cliente.
   - ⚠️ **Flujo de aprobación asíncrono:** el cobrador solicita un préstamo desde el móvil → si la unidad tiene "aprobación requerida" activada, queda pendiente → el admin web lo aprueba/rechaza → el móvil sincroniza y el cobrador confirma el registro final. Esto es un **estado adicional en la máquina de estados de un préstamo** (Pendiente → Aprobado/Rechazado → Registrado), hay que modelarlo bien en la base de datos.
3. **Nuevo:** crear cliente/préstamo nuevo desde el móvil (misma lógica vista en Clientes Nuevos y Disponibles web).
4. **Lista:** lista de clientes con préstamos activos.
5. **Más:** menú contextual sobre el cliente actualmente en pantalla — Llamar (abre app de teléfono), Marcar Visita en Blanco (con motivo seleccionable de lista + comentario opcional; si se marca "Comentario" el cliente se saca de la lista de pendientes del día), Pagos Realizados, Cuotas Pendientes, Ver Todas las Cuotas.

**Zona de cobro (pantalla principal, un cliente a la vez):**
6. **Micrófono (búsqueda por voz):** busca un préstamo/cliente diciendo su nombre en voz alta. ⚠️ Requiere integración de reconocimiento de voz (Speech-to-Text) nativo del dispositivo o vía servicio (Google Speech API, etc.) — otro costo/dependencia técnica a considerar.
7. **Barra de progreso:** muestra el total de clientes por cobrar vs. cobrados (contador y %), y saldo recaudado vs. saldo a recaudar del día — se actualiza en tiempo real (offline, local) a medida que se hacen pagos.
8. **Tarjeta de cliente:** datos básicos (ID, nombre completo, direcciones 1 y 2, teléfonos 1 y 2, rango de fechas del préstamo).
9. **Datos del préstamo (navegable con flechas ← →, por si el cliente tiene más de un préstamo activo):** valor préstamo, saldo, **cuotas vencidas, saldo vencido, días de atraso** (cálculo de mora en tiempo real en el dispositivo), valor de cuota según modalidad.
10. **Botón deslizable central (slider tipo "swipe to pay"):**
    - Deslizar a la **derecha:** registra el pago de la cuota correspondiente (con confirmación de valor, ajustable +/-). Si la cuota supera el saldo, no se realiza y se muestra advertencia.
    - Deslizar a la **izquierda:** marca visita en blanco (atajo directo, sin pasar por el menú "Más").
11. **Calculadora:** accede a la calculadora nativa del dispositivo.
12. **WhatsApp:** accede a la app de WhatsApp del dispositivo (probablemente para contactar al cliente).
13. **"Diferente a N cuota(s)":** permite pagar un valor distinto a la cuota exacta calculada (menor al saldo, o pago total). Abre ventana con +/- para ajustar el valor antes de aceptar.

⚠️ **Nota crítica de sincronización:** "Los pagos se van a cola si: (1) el móvil no tiene internet, (2) tiene activado el modo fuera de línea". Esto **confirma explícitamente el modelo offline-first con cola de sincronización** que se sospechaba desde el inicio — hay una configuración específica ("modo fuera de línea") que se puede activar manualmente además del caso de no tener señal. Esto es una pieza de arquitectura central a diseñar con cuidado (cola persistente local en el dispositivo — SQLite local o similar — con reintentos de sync).

### 6.3 Lista de Clientes ✅ (completo)
- Lista todos los clientes con préstamos activos, accesible desde "Lista" en pantalla inicial.
- Buscador de cliente, 3 pestañas: **Todos** (con préstamos activos aún no visitados/pagados hoy) | **Pendientes** | **Visitados**.
- Al tocar un cliente, aparece menú de acciones:
  - **Realizar Pago:** abre teclado numérico del dispositivo para ingresar valor, confirmar, muestra confirmación.
  - **Pagos Realizados:** lista de pagos hechos a ese cliente (fecha, hora, valor), con contador total y suma total en la cabecera (formato "(5): $3.050").
  - **Modificar Cliente:** edición de datos básicos únicamente — **NO permite modificar el préstamo** desde aquí (nota explícita de la documentación original).
  - **Historial Préstamos:** historial completo de préstamos del cliente (fecha préstamo, fecha último movimiento, valor neto, valor total, saldo), con contador y suma total en cabecera.
  - **Cuotas Pagadas:** (mencionado en menú, pendiente de detalle específico).

### 6.4 Pendientes por documentar (App Móvil)
- Menú (general)
- Clientes nuevos y disponibles (crear cliente/préstamo desde móvil)
- Préstamos rechazados y aprobados (proceso completo, ya se vio parcialmente en 6.2)
- Ajustes (mencionado "modo fuera de línea" en Ajustes opción 1 — pendiente ver todas las opciones)
- Cuotas Pendientes / Ver Todas las Cuotas (vistas en menú "Más" pero sin detalle capturado aún)

---

## 7. Implicaciones técnicas identificadas hasta ahora

- **Backend:** API REST con lógica de sincronización offline-first para móvil.
- **Base de datos:** relacional (PostgreSQL/MySQL), con jerarquía geográfica, cartera, mora, interés — requiere buen diseño de esquema.
- **Motor de cálculo financiero:** interés, mora, utilidad, cartera — debe recalcularse de forma consistente entre web y móvil.
- **Exportaciones:** Excel y PDF en múltiples módulos — consume CPU/RAM en cada generación, especialmente con multi-ruta.
- **Seguridad:** doble confirmación con contraseña en operaciones de dinero (eliminar, mover capital).
- **Concurrencia:** múltiples cobradores sincronizando simultáneamente desde el móvil.
- **Backups:** críticos — el sistema maneja dinero real de terceros.
- **Multi-tenant:** el sistema final debe soportar múltiples empresas clientes, cada una con sus propias rutas/usuarios, aisladas entre sí.

---

## 8. Recomendación de infraestructura/hosting (para 100 usuarios móviles simultáneos)

- **Servidor (backend + API):** VPS/Cloud 4-8 vCPU, 8-16 GB RAM, 100+ GB SSD — $40-90 USD/mes.
- **Base de datos:** PostgreSQL/MySQL, gestionada aparte si el presupuesto lo permite — $15-40 USD/mes.
- **Backups automáticos:** $5-15 USD/mes.
- **Dominio + SSL:** ~$12 USD/año (SSL gratis con Let's Encrypt).
- **Mapas/GPS:** descartado por ahora (ver sección 4) — se retira el costo de Google Maps API.
- **Total estimado:** ~$60-145 USD/mes para arrancar con 100 usuarios, escalable según uso real.
- Nota: estos números son de infraestructura únicamente, no incluyen costo de desarrollo.

---

## 9. Hoja de ruta de desarrollo (fases sugeridas)

*Contexto: proyecto desarrollado por una sola persona (estudiante de 8vo semestre de ingeniería), con apoyo de IA como copiloto de desarrollo. Se prioriza construir de lo simple/fundamental hacia lo complejo, evitando empezar por las partes más riesgosas (sync offline, motor de mora).*

### Fase 0 — Preparación (antes de escribir código)
- Definir stack tecnológico completo (backend, base de datos, frontend web, framework móvil) — esto se decide ANTES de empezar, no sobre la marcha.
- Diseñar el esquema de base de datos completo en papel/diagrama (entidades: empresas/tenant, usuarios, roles/permisos, unidades/rutas, clientes, préstamos, cuotas, pagos, gastos, movimientos de caja, vales, notas crédito/débito). Este es el paso más importante de todo el proyecto — un mal diseño de datos aquí se paga muy caro después.
- Configurar entorno de desarrollo, control de versiones (Git), y un entorno de pruebas separado del de producción.

### Fase 1 — Backend base + Web administrativa (sin móvil todavía)
- Autenticación y estructura multi-tenant básica (empresa → usuarios).
- CRUD de Rutas/Unidades (sin todos los permisos avanzados aún, versión simple).
- CRUD de Clientes + creación de préstamo (motor de cálculo de interés/cuotas — ¡ojo aquí, validar fórmulas contra las capturas!).
- Módulo Caja básico: Gastos, Inversión y Retiro (sin vales todavía).
- Reportes básicos: Cuadre Recaudador simplificado (sin todos los % y detalles al inicio).

### Fase 2 — Motor financiero completo (la parte más delicada)
- Cálculo de mora por cuota (Consolidado/Detallado Mora).
- Vales, Notas Crédito/Débito.
- Cierre/Reapertura de unidad.
- Reconstruir Saldos (para cuando algo se descuadre — construirlo pronto, no al final).

### Fase 3 — Sistema de permisos (RBAC)
- Roles y permisos por usuario (Configuración → Usuarios → Roles).
- Permisos por unidad (los vistos en Gestionar Unidades).
- Definir cómo interactúan ambos niveles.

### Fase 4 — App móvil (SIN offline todavía — versión "online-only" primero)
- Login, Pantalla Inicial, Lista de Clientes, Realizar Pago — funcionando siempre conectado a internet.
- Esto permite validar toda la lógica de negocio en el móvil sin la complejidad de sincronización aún.

### Fase 5 — Offline-first (la parte técnicamente más difícil)
- Base de datos local en el dispositivo (SQLite u otra).
- Cola de sincronización, manejo de conflictos.
- Modo "fuera de línea" manual.
- ⚠️ Esta fase se beneficia mucho de ir con calma — es donde más proyectos similares se atrasan o fallan si se apura.

### Fase 6 — Refinamiento
- Exportaciones Excel/PDF.
- Importación masiva.
- Auditorías, SMTP, Días no laborables.
- Vinculación de unidades por IMEI (seguridad).
- (Mapas — descartado por ahora, ver sección 4)

### Notas de trabajo con IA como copiloto
- Cada fase se puede desarrollar en sesiones de código dedicadas (ej. con Claude Code), trayendo siempre este documento como contexto de referencia.
- Es buena práctica pedir que se expliquen las decisiones técnicas a medida que se avanza, no solo que se genere código — así el aprendizaje se consolida y se puede sostener el proyecto sin depender 100% de la IA en el futuro.
- Antes de avanzar de fase, vale la pena probar bien la fase anterior con datos reales o de prueba.

---

## 10. Decisiones de diseño y avance real
*Esta sección se llena a medida que se desarrolla el proyecto — es la memoria viva entre sesiones de Claude Code. Actualízala al final de cada sesión de trabajo.*

### Stack tecnológico elegido
- Backend: *(pendiente de definir)*
- Base de datos: *(pendiente de definir)*
- Frontend web: *(pendiente de definir)*
- Framework móvil: *(pendiente de definir)*
- Hosting elegido: *(pendiente de definir)*

### Decisiones de diseño importantes
*(ej. "el cálculo de la cuota se hace así: valor_total / días, redondeando a 2 decimales..." — anotar aquí cualquier fórmula, convención o decisión de arquitectura que se defina durante el desarrollo, para no tener que redescubrirla o inventarla distinto en otra sesión)*

### Avance por fase
| Fase | Estado | Notas |
|------|--------|-------|
| Fase 0 — Preparación | ⏳ No iniciado | |
| Fase 1 — Backend base + Web admin | ⏳ No iniciado | |
| Fase 2 — Motor financiero | ⏳ No iniciado | |
| Fase 3 — RBAC | ⏳ No iniciado | |
| Fase 4 — App móvil online | ⏳ No iniciado | |
| Fase 5 — Offline-first | ⏳ No iniciado | |
| Fase 6 — Refinamiento | ⏳ No iniciado | |

### Pendientes / dudas abiertas
*(cosas que no quedaron claras del sistema original y que se resolvieron con una decisión propia — o que siguen sin resolver)*

---