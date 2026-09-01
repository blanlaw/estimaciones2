# 📋 Estimación Técnica de Desarrollo
## Portal File Proveedores — Bitrix24

> **Proyecto:** Flujo de Registro, Gestión de Facturación y Pago a Proveedores  
> **Plataforma:** Bitrix24 (CRM + Extranet + Automatizaciones + API REST)  
> **Fecha de estimación:** 01 de septiembre de 2026  
> **Estimado por:** Área de Desarrollo  
> **Versión del documento fuente:** Proveedor v1.0

---

## 1. Resumen Ejecutivo

El proyecto consiste en implementar un flujo centralizado en **Bitrix24** que permita:

1. Dar de alta proveedores desde usuarios internos (Logística, Proyectos, Finanzas, Operaciones).
2. Asignar Órdenes de Servicio / Compra (OS/OC).
3. Proveer acceso extranet al proveedor para completar su perfil y subir comprobantes fiscales (XML/PDF).
4. Validar la documentación a través de tres áreas: Finanzas → Contabilidad → Tesorería.
5. Liquidar el pago y notificar al proveedor con la constancia correspondiente.

---

## 2. Alcance del Desarrollo

### 2.1 Módulos Identificados

| # | Módulo | Descripción |
|---|--------|-------------|
| M1 | Estructura de Datos / Campos Personalizados | Perfil del Proveedor + Entidad de Facturación |
| M2 | Pipeline / Flujo de Estados (SPA) | 8 estados + transiciones jerárquicas |
| M3 | Matriz de Roles y Permisos | 5 roles: Solicitante, Proveedor Extranet, Finanzas, Contabilidad, Tesorería |
| M4 | Portal Extranet del Proveedor | Pantalla de login, perfil, OS/OC, carga de documentos |
| M5 | Automatizaciones y Disparadores | Auto-registro, credenciales, notificaciones, cambios de estado |
| M6 | Módulo de Carga Masiva de Pagos | Script/API REST para procesamiento batch de pagos |
| M7 | Notificaciones por correo | Bienvenida, observaciones, pagos, constancias |
| M8 | UI/UX del Portal (según manual de usuario) | Login, perfil, historial OS, carga de documentos, estados visuales |

---

## 3. Desglose Técnico por Módulo

### 🗄️ M1 — Estructura de Datos y Campos Personalizados

**Entidades a crear/configurar en Bitrix24 CRM:**

**Perfil del Proveedor (Company CRM):**
- RUC (Texto único, obligatorio)
- Razón Social (Texto)
- Correo de Notificaciones (Email, obligatorio)
- Datos Representante Legal: Nombre, Teléfono, Correo
- Datos Contacto Cobranzas: Nombre, Teléfono, Correo
- Información Bancaria: Banco Principal, Banco Secundario, N° Cuenta Corriente / CCI, N° Cuenta Banco Nación - Detracciones

**Entidad de Flujo de Facturación (SPA / Smart Process Automation):**
- Proveedor (Vínculo a Compañía CRM)
- N° OS/OC (Texto o Vínculo a Negocio/Cotización)
- Solicitante Interno (Usuario Bitrix24)
- Adjuntos: XML Factura, PDF Comprobante, Guía de Remisión, Guía de Transporte
- Monto Total y Moneda, Fechas de Emisión/Vencimiento/Pago
- Constancias de Pago (múltiples adjuntos)
- Motivo de Observación / Rechazo

| Tarea | Horas |
|-------|-------|
| Diseño y creación de campos personalizados en Compañía CRM | 3 h |
| Creación de SPA (Smart Process Automation) con todos sus campos | 5 h |
| Configuración de validaciones y campos obligatorios | 2 h |
| Pruebas de estructura de datos | 2 h |
| **Subtotal M1** | **12 h** |

---

### 🔄 M2 — Pipeline / Flujo de Estados

**Estados del pipeline:**

```
[REGISTRADO] ➔ [REVISADO] ➔ [APROBADO] ➔ [PROGRAMADO] ➔ [PAGADO / PAGO PARCIAL]
      │               │            │
      ├─► OBSERVADO    ├─► OBSERVADO └─► OBSERVADO
      └─► RECHAZADO    └─► RECHAZADO └─► RECHAZADO
```

**Transiciones de estado por rol:**
- Usuario Solicitante → REGISTRADO (al cargar OS/OC)
- Finanzas → REVISADO / OBSERVADO / RECHAZADO
- Contabilidad → APROBADO / OBSERVADO / RECHAZADO
- Tesorería → PROGRAMADO → PAGADO / PAGO PARCIAL

| Tarea | Horas |
|-------|-------|
| Diseño del pipeline en SPA (8 estados + etapas) | 4 h |
| Configuración de transiciones permitidas por rol | 5 h |
| Lógica condicional para OBSERVADO y RECHAZADO | 3 h |
| Pruebas de flujo completo | 3 h |
| **Subtotal M2** | **15 h** |

---

### 🔐 M3 — Matriz de Roles y Permisos

**Roles a configurar:**

| Rol | Área | Acciones |
|-----|------|----------|
| Usuario Solicitante | Logística, Proyectos, Finanzas, Marketing, Post Venta, Legal | Crear proveedor, crear OS/OC, consultar estado |
| Proveedor (Extranet) | Externa | Completar perfil, ver OS/OC, subir facturas/documentos, ver estado de pago |
| Usuario Finanzas | Finanzas / Control Operativo | Validar OS/OC vs Factura, observar o rechazar |
| Usuario Contabilidad | Contabilidad / Tributación | Validar XML/Detracción, aprobar para programación |
| Usuario Tesorería | Tesorería | Asignar fecha pago, cargar constancias, marcar pagado |

| Tarea | Horas |
|-------|-------|
| Configuración de grupos y perfiles de acceso en Bitrix24 | 3 h |
| Configuración de permisos por entidad CRM y SPA | 5 h |
| Configuración de acceso Extranet para proveedores | 4 h |
| Restricciones de visibilidad de campos según rol | 3 h |
| Pruebas de permisos por cada rol | 4 h |
| **Subtotal M3** | **19 h** |

---

### 🌐 M4 — Portal Extranet del Proveedor

**Pantallas y secciones identificadas (basado en el manual de usuario v1.0):**

1. **Pantalla de Login** — URL: `portaldeproveedores.movilatcargo.com`
2. **Aceptación de Términos y Condiciones** — Modal obligatorio al primer ingreso
3. **Completar Perfil** — Aviso y flujo guiado de datos incompletos
4. **Mi Perfil** con sub-secciones:
   - Información de usuario (foto, nombres, teléfono, correo)
   - Contacto y Representantes
   - Catálogo (documentos de productos/servicios)
   - Instalaciones (sedes/sucursales)
   - Cuentas bancarias
   - Homologación (documentación evaluada por Movilat Cargo)
5. **Ver Historial / Órdenes de Servicio** — Lista y detalle de OS con estados visuales (verde/rojo)
6. **Detalle de OS** con:
   - Información resumida de la OS
   - Ítems de la OS
   - Historial de aprobaciones (línea de tiempo)
   - Formulario de carga de documentos (por tipo de OS)
7. **Carga de documentos según tipo de OS:**

| Tipo de OS | Documentos requeridos |
|------------|----------------------|
| Transporte | XML Factura, PDF Factura, Guía de Remisión, Guía de Transporte, Liquidación |
| Estiba / Devolución | XML Factura, PDF Factura, Liquidación |
| Desestiba | XML Factura, PDF Factura, Liquidación |
| Servicios Profesionales | XML Factura, PDF Factura, Suspensión de Cuarta (condicional) |
| Gastos Adicionales de Transporte | XML Factura, PDF Factura, Liquidación |

**Estados visuales de la OS:**
- Generado (Verde), Aprobado (Verde), Observado (Rojo), Cancelado (Rojo)

**Estados visuales de la Factura:**
- Enviado (Verde), Registrado (Amarillo), Observado (Rojo), Rechazado (Rojo)

| Tarea | Horas |
|-------|-------|
| Diseño y configuración del portal Extranet en Bitrix24 | 6 h |
| Configuración de la sección Mi Perfil (6 sub-secciones) | 8 h |
| Desarrollo del módulo Historial / Órdenes de Servicio | 5 h |
| Pantalla de detalle de OS (4 secciones + línea de tiempo) | 6 h |
| Formulario de carga de documentos condicional por tipo de OS | 8 h |
| Modal de Términos y Condiciones (primer ingreso) | 2 h |
| Indicadores visuales de estado (colores, etiquetas) | 3 h |
| Diseño responsive y UX del portal | 5 h |
| Pruebas del portal como usuario proveedor | 5 h |
| **Subtotal M4** | **48 h** |

---

### ⚡ M5 — Automatizaciones y Disparadores

**Automatizaciones identificadas:**

**A. Auto-registro y Envío de Credenciales**
- Disparador: Creación de proveedor (RUC + Email)
- Acción: Generar usuario Extranet + enviar correo con credenciales temporales

**B. Notificación de Observación**
- Disparador: Estado cambia a OBSERVADO
- Acción: Email/portal al Proveedor y Solicitante con `{Motivo_de_Observación}`

**C. Notificación de Pago**
- Disparador: Estado cambia a PAGADO o PAGO PARCIAL
- Acción: Email al Proveedor con invitación a descargar Constancia de Pago

**D. Notificaciones internas de cambio de estado**
- Notificaciones a los usuarios responsables de cada etapa

| Tarea | Horas |
|-------|-------|
| Automatización: auto-registro + generación de usuario Extranet | 5 h |
| Automatización: envío de credenciales (template de bienvenida) | 3 h |
| Automatización: notificación de OBSERVADO con motivo dinámico | 3 h |
| Automatización: notificación de PAGADO / PAGO PARCIAL | 2 h |
| Configuración de plantillas de correo (HTML + variables dinámicas) | 5 h |
| Automatización: notificaciones internas por cambio de estado | 3 h |
| Pruebas de todas las automatizaciones en escenarios reales | 4 h |
| **Subtotal M5** | **25 h** |

---

### 📦 M6 — Módulo de Carga Masiva de Pagos (Tesorería)

**Características:**
- Importación vía **API REST**, **CSV** o **Excel**
- Criterio de cruce: RUC + N° Factura / OS
- Resultado: Actualización masiva de estados a PAGADO / PAGO PARCIAL
- Adjunción automática de PDFs de constancias de pago

| Tarea | Horas |
|-------|-------|
| Diseño del esquema de importación (mapeo de columnas, validaciones) | 4 h |
| Desarrollo del script de carga masiva (API REST / parsing CSV/Excel) | 10 h |
| Lógica de cruce por RUC + N° Factura / OS | 4 h |
| Actualización masiva de estados en Bitrix24 vía API | 5 h |
| Adjunción automática de PDFs de constancias | 4 h |
| Manejo de errores, log de resultados y rollback | 4 h |
| Interfaz de carga (UI simple para Tesorería) | 4 h |
| Pruebas con datos reales de Tesorería | 5 h |
| **Subtotal M6** | **40 h** |

---

### 📧 M7 — Plantillas y Notificaciones por Correo

| Correo | Destinatario | Disparador |
|--------|-------------|------------|
| Bienvenida + Credenciales | Proveedor | Alta de proveedor |
| Términos y Condiciones | Proveedor | Primer ingreso |
| Orden de Servicio generada | Proveedor / Solicitante | Creación de OS |
| Observación con motivo | Proveedor + Solicitante | Cambio a OBSERVADO |
| Aprobación de OS | Proveedor | Cambio a APROBADO |
| Pago liquidado + Constancia | Proveedor | Cambio a PAGADO/PAGO PARCIAL |
| Rechazo con motivo | Proveedor + Solicitante | Cambio a RECHAZADO |

| Tarea | Horas |
|-------|-------|
| Diseño de plantillas HTML responsive (7 correos) | 8 h |
| Integración de variables dinámicas en cada plantilla | 3 h |
| Configuración de envío en Bitrix24 (SMTP / servicio de email) | 2 h |
| Pruebas de entrega y renderizado de correos | 3 h |
| **Subtotal M7** | **16 h** |

---

### 🎨 M8 — Configuración General, Integraciones, QA y Despliegue

| Tarea | Horas |
|-------|-------|
| Configuración del dominio / subdominio del portal externo | 2 h |
| Configuración SSL y acceso seguro | 1 h |
| Integración con sistema de validación de XML SUNAT (si aplica) | 6 h |
| Documentación técnica del sistema desarrollado | 6 h |
| Capacitación a usuarios internos (Finanzas, Contabilidad, Tesorería) | 4 h |
| Capacitación a usuarios solicitantes | 2 h |
| QA integral (flujo completo de inicio a fin) | 8 h |
| Corrección de bugs post-QA | 5 h |
| Despliegue a producción y validación final | 4 h |
| **Subtotal M8** | **38 h** |

---

## 4. Resumen de Horas por Módulo

| Módulo | Descripción | Horas Estimadas |
|--------|-------------|-----------------|
| M1 | Estructura de Datos / Campos Personalizados | 12 h |
| M2 | Pipeline / Flujo de Estados | 15 h |
| M3 | Matriz de Roles y Permisos | 19 h |
| M4 | Portal Extranet del Proveedor | 48 h |
| M5 | Automatizaciones y Disparadores | 25 h |
| M6 | Módulo de Carga Masiva de Pagos | 40 h |
| M7 | Plantillas y Notificaciones por Correo | 16 h |
| M8 | Configuración, Integraciones, QA y Despliegue | 38 h |
| | **TOTAL BASE** | **213 h** |
| | **+ Buffer 15% (imprevistos / ajustes)** | **+32 h** |
| | **TOTAL RECOMENDADO** | **245 h** |

---

## 5. Estimación de Tiempo — Lunes a Viernes, 4 h/día

> **Base de cálculo:** 4 horas laborales diarias, de lunes a viernes.  
> **Total horas base:** 213 h → **54 días hábiles** (~10 semanas y 4 días)  
> **Total con buffer 15%:** 245 h → **62 días hábiles** (~12 semanas y 2 días)  
> **Fecha de inicio propuesta:** 01 de septiembre de 2026  
> **Fecha de entrega base:** 12 de noviembre de 2026  
> **Fecha de entrega estimada (con buffer):** 25 de noviembre de 2026

---

## 5.1 Cronograma Detallado — Semana a Semana

> Cada semana aporta **20 horas** (5 días × 4 h/día)

### 📅 SEMANA 1 — Del 01 al 05 de septiembre (20 h)
> **Objetivo:** Kickoff + Estructura de datos completa (M1) + Pipeline inicio (M2)

| Día | Fecha | Actividad | Módulo | Horas |
|-----|-------|-----------|--------|-------|
| Lunes | 01/09 | Kickoff + Diseño de campos personalizados en Compañía CRM | M1 | 4 h |
| Martes | 02/09 | Creación de la SPA con todos sus campos | M1 | 4 h |
| Miércoles | 03/09 | Configuración de validaciones y campos obligatorios + Pruebas M1 | M1 | 4 h |
| Jueves | 04/09 | Diseño del pipeline en SPA (8 estados + etapas) | M2 | 4 h |
| Viernes | 05/09 | Configuración de transiciones permitidas por rol (inicio) | M2 | 4 h |

**Horas semana 1:** 20 h | **Acumulado:** 20 h / 213 h

**✅ Entregable S1:** M1 completo (12 h) + M2 en progreso (8 h de 15 h)

---

### 📅 SEMANA 2 — Del 08 al 12 de septiembre (20 h)
> **Objetivo:** Completar Pipeline (M2) + Arrancar Roles y Permisos (M3)

| Día | Fecha | Actividad | Módulo | Horas |
|-----|-------|-----------|--------|-------|
| Lunes | 08/09 | Configuración de transiciones por rol (fin) + Lógica OBSERVADO/RECHAZADO | M2 | 4 h |
| Martes | 09/09 | Pruebas de flujo completo del pipeline | M2 | 3 h |
| Martes | 09/09 | Configuración de grupos y perfiles de acceso en Bitrix24 | M3 | 1 h |
| Miércoles | 10/09 | Configuración de grupos y perfiles + permisos por entidad CRM y SPA | M3 | 4 h |
| Jueves | 11/09 | Configuración de acceso Extranet para proveedores | M3 | 4 h |
| Viernes | 12/09 | Restricciones de visibilidad de campos según rol | M3 | 4 h |

**Horas semana 2:** 20 h | **Acumulado:** 40 h / 213 h

**✅ Entregable S2:** M2 completo (15 h) + M3 en progreso (14 h de 19 h)

---

### 📅 SEMANA 3 — Del 15 al 19 de septiembre (20 h)
> **Objetivo:** Completar Roles/Permisos (M3) + Iniciar Portal Extranet (M4)

| Día | Fecha | Actividad | Módulo | Horas |
|-----|-------|-----------|--------|-------|
| Lunes | 15/09 | Pruebas de permisos por cada rol (5 roles) | M3 | 4 h |
| Martes | 16/09 | Diseño y configuración del portal Extranet en Bitrix24 | M4 | 4 h |
| Miércoles | 17/09 | Mi Perfil — Información de usuario + Contacto y Representantes | M4 | 4 h |
| Jueves | 18/09 | Mi Perfil — Catálogo + Instalaciones | M4 | 4 h |
| Viernes | 19/09 | Mi Perfil — Cuentas bancarias + Homologación | M4 | 4 h |

**Horas semana 3:** 20 h | **Acumulado:** 60 h / 213 h

**✅ Entregable S3:** M3 completo (19 h) + M4 en progreso — Perfil completo (16 h de 48 h)

---

### 📅 SEMANA 4 — Del 22 al 26 de septiembre (20 h)
> **Objetivo:** Continuar Portal Extranet (M4) — Historial OS + Detalle OS

| Día | Fecha | Actividad | Módulo | Horas |
|-----|-------|-----------|--------|-------|
| Lunes | 22/09 | Modal de Términos y Condiciones (primer ingreso obligatorio) | M4 | 4 h |
| Martes | 23/09 | Módulo Historial / Órdenes de Servicio (lista + estados visuales) | M4 | 4 h |
| Miércoles | 24/09 | Pantalla detalle OS — Info resumen + Ítems | M4 | 4 h |
| Jueves | 25/09 | Pantalla detalle OS — Historial de aprobaciones (línea de tiempo) | M4 | 4 h |
| Viernes | 26/09 | Formulario carga docs — Tipo Transporte (5 documentos) | M4 | 4 h |

**Horas semana 4:** 20 h | **Acumulado:** 80 h / 213 h

**✅ Entregable S4:** M4 en progreso — Historial OS + Detalle OS funcional (36 h de 48 h)

---

### 📅 SEMANA 5 — Del 29 de septiembre al 03 de octubre (20 h)
> **Objetivo:** Completar Portal Extranet (M4) + Iniciar Automatizaciones (M5)

| Día | Fecha | Actividad | Módulo | Horas |
|-----|-------|-----------|--------|-------|
| Lunes | 29/09 | Formulario carga docs — Estiba/Devolución + Desestiba | M4 | 4 h |
| Martes | 30/09 | Formulario carga docs — Servicios Profesionales (condicional) + Gastos Adicionales | M4 | 4 h |
| Miércoles | 01/10 | Indicadores visuales de estado + Diseño responsive y UX | M4 | 4 h |
| Jueves | 02/10 | Pruebas del portal como usuario proveedor | M4 | 2 h |
| Jueves | 02/10 | Automatización: auto-registro + generación usuario Extranet | M5 | 2 h |
| Viernes | 03/10 | Automatización: auto-registro + generación usuario Extranet (fin) | M5 | 4 h |

**Horas semana 5:** 20 h | **Acumulado:** 100 h / 213 h

**✅ Entregable S5:** M4 completo (48 h) + M5 en progreso (6 h de 25 h)

---

### 📅 SEMANA 6 — Del 06 al 10 de octubre (20 h)
> **Objetivo:** Continuar Automatizaciones (M5)

| Día | Fecha | Actividad | Módulo | Horas |
|-----|-------|-----------|--------|-------|
| Lunes | 06/10 | Automatización: envío de credenciales (template de bienvenida) | M5 | 4 h |
| Martes | 07/10 | Automatización: notificación OBSERVADO con motivo dinámico | M5 | 4 h |
| Miércoles | 08/10 | Automatización: notificación PAGADO / PAGO PARCIAL | M5 | 4 h |
| Jueves | 09/10 | Configuración de plantillas correo HTML + variables dinámicas | M5 | 4 h |
| Viernes | 10/10 | Automatización: notificaciones internas por cambio de estado | M5 | 4 h |

**Horas semana 6:** 20 h | **Acumulado:** 120 h / 213 h

**✅ Entregable S6:** M5 en progreso (26 h de 25 h → prácticamente completo, queda pruebas)

---

### 📅 SEMANA 7 — Del 13 al 17 de octubre (20 h)
> **Objetivo:** Cerrar Automatizaciones (M5) + Plantillas de correo completas (M7)

| Día | Fecha | Actividad | Módulo | Horas |
|-----|-------|-----------|--------|-------|
| Lunes | 13/10 | Pruebas de todas las automatizaciones (escenarios reales) | M5 | 4 h |
| Martes | 14/10 | Plantillas HTML — Bienvenida + Términos y Condiciones | M7 | 4 h |
| Miércoles | 15/10 | Plantillas HTML — OS generada + Observación con motivo | M7 | 4 h |
| Jueves | 16/10 | Plantillas HTML — Aprobación + Pago liquidado + Rechazo | M7 | 4 h |
| Viernes | 17/10 | Integración de variables dinámicas + Configuración SMTP + Pruebas | M7 | 4 h |

**Horas semana 7:** 20 h | **Acumulado:** 140 h / 213 h

**✅ Entregable S7:** M5 completo (25 h) + M7 completo (16 h)

---

### 📅 SEMANA 8 — Del 20 al 24 de octubre (20 h)
> **Objetivo:** Iniciar y avanzar Módulo de Carga Masiva (M6)

| Día | Fecha | Actividad | Módulo | Horas |
|-----|-------|-----------|--------|-------|
| Lunes | 20/10 | Diseño del esquema de importación (mapeo de columnas, validaciones) | M6 | 4 h |
| Martes | 21/10 | Desarrollo del script de carga masiva — Parsing CSV/Excel | M6 | 4 h |
| Miércoles | 22/10 | Desarrollo del script — continuación + integración API REST Bitrix24 | M6 | 4 h |
| Jueves | 23/10 | Lógica de cruce por RUC + N° Factura / OS | M6 | 4 h |
| Viernes | 24/10 | Actualización masiva de estados en Bitrix24 vía API | M6 | 4 h |

**Horas semana 8:** 20 h | **Acumulado:** 160 h / 213 h

**✅ Entregable S8:** M6 en progreso (20 h de 40 h) — Script base + API funcional

---

### 📅 SEMANA 9 — Del 27 al 31 de octubre (20 h)
> **Objetivo:** Completar Módulo de Carga Masiva (M6) + Inicio configuración general (M8)

| Día | Fecha | Actividad | Módulo | Horas |
|-----|-------|-----------|--------|-------|
| Lunes | 27/10 | Adjunción automática de PDFs de constancias de pago | M6 | 4 h |
| Martes | 28/10 | Manejo de errores, log de resultados y rollback | M6 | 4 h |
| Miércoles | 29/10 | Interfaz de carga (UI simple para Tesorería) | M6 | 4 h |
| Jueves | 30/10 | Pruebas con datos reales de Tesorería | M6 | 4 h |
| Viernes | 31/10 | Configuración del dominio/subdominio + SSL + acceso seguro | M8 | 4 h |

**Horas semana 9:** 20 h | **Acumulado:** 180 h / 213 h

**✅ Entregable S9:** M6 completo (40 h) + M8 inicio (3 h de 38 h)

---

### 📅 SEMANA 10 — Del 03 al 07 de noviembre (20 h)
> **Objetivo:** QA + Documentación + Capacitación (M8)

| Día | Fecha | Actividad | Módulo | Horas |
|-----|-------|-----------|--------|-------|
| Lunes | 03/11 | Integración con validación de XML SUNAT (si aplica) | M8 | 4 h |
| Martes | 04/11 | Documentación técnica del sistema (inicio) | M8 | 4 h |
| Miércoles | 05/11 | Documentación técnica del sistema (fin) | M8 | 4 h |
| Jueves | 06/11 | Capacitación — Usuarios internos (Finanzas, Contabilidad, Tesorería) | M8 | 4 h |
| Viernes | 07/11 | Capacitación — Usuarios solicitantes + QA integral (inicio) | M8 | 4 h |

**Horas semana 10:** 20 h | **Acumulado:** 200 h / 213 h

**✅ Entregable S10:** M8 en progreso — Documentación + Capacitación completas (23 h de 38 h)

---

### 📅 SEMANA 11 — Del 10 al 12 de noviembre (13 h) — CIERRE BASE
> **Objetivo:** Finalizar QA + Correcciones + Despliegue a producción

| Día | Fecha | Actividad | Módulo | Horas |
|-----|-------|-----------|--------|-------|
| Lunes | 10/11 | QA integral — Flujo completo: Solicitante → Proveedor → Finanzas | M8 | 4 h |
| Martes | 11/11 | QA integral — Flujo: Contabilidad → Tesorería → Pago + Corrección bugs | M8 | 4 h |
| Miércoles | 12/11 | Despliegue a producción y validación final | M8 | 4 h |
| Miércoles | 12/11 | *(1 h sobrante — inicio buffer)* | Buffer | 1 h |

**Horas semana 11:** 13 h base + 1 h buffer | **Acumulado base:** 213 h ✅

**🏁 CIERRE DEL PROYECTO BASE — 12 de noviembre de 2026**

---

### 📅 SEMANAS 11–13 — Buffer (14 nov al 25 nov, 32 h)
> **Objetivo:** Absorber imprevistos, ajustes por feedback, pruebas de regresión post-lanzamiento

| Período | Actividad | Horas |
|---------|-----------|-------|
| 13 nov (jue–vie) | Buffer — Ajustes por feedback inmediato post-lanzamiento | 7 h |
| 17–21 nov (S12) | Buffer — Correcciones menores, optimizaciones, ajustes UX | 20 h |
| 24–25 nov (S13 lun–mar) | Buffer — Validación final y cierre formal del proyecto | 5 h |

**Horas buffer total:** 32 h | **Total con buffer:** 245 h ✅

**🏁 CIERRE FORMAL DEL PROYECTO — 25 de noviembre de 2026**

---

## 5.2 Resumen del Cronograma (4 h/día)

| Semana | Fechas | Módulos | Horas/sem | Acumulado |
|--------|--------|---------|:---------:|-----------|
| S1 | 01 – 05 sep | M1 (completo) + M2 (inicio) | 20 h | 20 h |
| S2 | 08 – 12 sep | M2 (completo) + M3 (inicio) | 20 h | 40 h |
| S3 | 15 – 19 sep | M3 (completo) + M4 (inicio) | 20 h | 60 h |
| S4 | 22 – 26 sep | M4 (continúa) | 20 h | 80 h |
| S5 | 29 sep – 03 oct | M4 (completo) + M5 (inicio) | 20 h | 100 h |
| S6 | 06 – 10 oct | M5 (continúa) | 20 h | 120 h |
| S7 | 13 – 17 oct | M5 (completo) + M7 (completo) | 20 h | 140 h |
| S8 | 20 – 24 oct | M6 (inicio/mitad) | 20 h | 160 h |
| S9 | 27 – 31 oct | M6 (completo) + M8 (inicio) | 20 h | 180 h |
| S10 | 03 – 07 nov | M8 (continúa) | 20 h | 200 h |
| S11 | 10 – 12 nov | M8 (completo) + Despliegue | 13 h | **213 h** ✅ |
| S12–S13 | 13 – 25 nov | 🛡️ Buffer — Ajustes post-lanzamiento | 32 h | **245 h** ✅ |

```
SEP 01 ─────────────────────────────────────────────── NOV 25
│ S1 │ S2 │ S3 │ S4 │ S5 │ S6 │ S7 │ S8 │ S9 │ S10 │ S11 🏁 │ Buffer │
  M1+M2 M2+M3 M3+M4  M4  M4+M5  M5  M5+M7  M6  M6+M8  M8  Despliegue  32h
```

> 🗓️ **Fecha de inicio:** 01 de septiembre de 2026  
> 🏁 **Fecha de cierre base (sin buffer):** 12 de noviembre de 2026  
> 🏁 **Fecha de cierre (con buffer 15%):** 25 de noviembre de 2026  
> 📆 **Duración total:** 11 semanas base (54 días hábiles) / ~13 semanas con buffer (~62 días hábiles)

---

## 6. Supuestos y Consideraciones

1. **Plataforma:** Todo el desarrollo se realiza sobre **Bitrix24** (On-Cloud o On-Premise).
2. **Acceso Extranet:** El plan de Bitrix24 contratado debe incluir acceso a usuarios Extranet.
3. **Dominio:** El subdominio `portaldeproveedores.movilatcargo.com` debe estar disponible y configurado.
4. **SUNAT / XML:** Si se requiere validación automática de XML contra SUNAT, el tiempo puede aumentar en **+6 a +10 horas adicionales** dependiendo del tipo de integración.
5. **Diseño:** El portal extranet seguirá el branding de Movilat Cargo. Si se requiere diseño personalizado adicional, se estima **+8 a +12 horas de diseño UI**.
6. **Carga Masiva:** El módulo M6 asume que los archivos de Tesorería vienen en formato CSV/Excel con estructura estandarizada acordada previamente.
7. **Correos:** Se asume que Bitrix24 tiene configurado un servidor SMTP funcional.
8. **Capacitación:** Se contempla una sesión por área usuaria (no incluye documentación de usuario final extendida).
9. **No incluye:** Desarrollo de app móvil, integraciones con ERP externo, ni implementación de firma electrónica.

---

## 7. Riesgos y Mitigaciones

| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|-------------|---------|------------|
| Cambios en el alcance durante el desarrollo | Alta | Alto | Congelar alcance antes de iniciar. Gestionar cambios con control de versiones. |
| Dificultades con la API de Bitrix24 para carga masiva | Media | Alto | Prototipo temprano del M6 en las primeras 2 semanas. |
| Validación de XML SUNAT fuera del alcance inicial | Media | Medio | Definir si aplica antes de iniciar. Estimación separada si se necesita. |
| Retrasos en la entrega de información por parte del cliente | Media | Alto | Cronograma de entregables acordado con el cliente desde el inicio. |
| Limitaciones del plan Bitrix24 contratado | Baja | Alto | Verificar límites de usuarios Extranet y automatizaciones antes de iniciar. |

---

## 8. Hitos del Proyecto

| Hito | Entregable | Fecha estimada |
|------|-----------|----------------|
| H1 | 🔧 Estructura de datos (M1) + Pipeline (M2) completados | 09 sep 2026 |
| H2 | 🔐 Roles y permisos completos (M3) | 15 sep 2026 |
| H3 | 🌐 Portal Extranet funcional completo (M4) | 02 oct 2026 |
| H4 | ⚡ Automatizaciones completas (M5) | 13 oct 2026 |
| H5 | 📧 Plantillas de correo activas (M7) | 17 oct 2026 |
| H6 | 📦 Módulo de carga masiva funcional (M6) | 30 oct 2026 |
| H7 | ✅ QA integral + correcciones + despliegue (M8) | 12 nov 2026 |
| H8 | 🏁 Cierre formal del proyecto (con buffer 15%) | 25 nov 2026 |

---

## 9. Próximos Pasos

- [ ] Confirmar plataforma Bitrix24 y plan contratado (verificar límites de Extranet y automatizaciones)
- [ ] Validar si se requiere integración con SUNAT para validación de XML
- [ ] Confirmar diseño visual / branding del portal extranet
- [ ] Definir si el módulo de carga masiva es CSV, Excel o ambos
- [ ] Agendar reunión de kickoff con todas las áreas involucradas (Logística, Finanzas, Contabilidad, Tesorería)
- [ ] Congelar alcance y firmar aprobación antes de iniciar desarrollo

---

## 10. Cuadros de Estimación Consolidados

### 📊 Cuadro A — Detalle Completo de Tareas por Módulo

| Módulo | Tarea | Horas |
|--------|-------|------:|
| **M1 — Estructura de Datos** | Diseño y creación de campos personalizados en Compañía CRM | 3 h |
| **M1 — Estructura de Datos** | Creación de SPA (Smart Process Automation) con todos sus campos | 5 h |
| **M1 — Estructura de Datos** | Configuración de validaciones y campos obligatorios | 2 h |
| **M1 — Estructura de Datos** | Pruebas de estructura de datos | 2 h |
| **M1 SUBTOTAL** | | **12 h** |
| | | |
| **M2 — Pipeline / Estados** | Diseño del pipeline en SPA (8 estados + etapas) | 4 h |
| **M2 — Pipeline / Estados** | Configuración de transiciones permitidas por rol | 5 h |
| **M2 — Pipeline / Estados** | Lógica condicional para OBSERVADO y RECHAZADO | 3 h |
| **M2 — Pipeline / Estados** | Pruebas de flujo completo | 3 h |
| **M2 SUBTOTAL** | | **15 h** |
| | | |
| **M3 — Roles y Permisos** | Configuración de grupos y perfiles de acceso en Bitrix24 | 3 h |
| **M3 — Roles y Permisos** | Configuración de permisos por entidad CRM y SPA | 5 h |
| **M3 — Roles y Permisos** | Configuración de acceso Extranet para proveedores | 4 h |
| **M3 — Roles y Permisos** | Restricciones de visibilidad de campos según rol | 3 h |
| **M3 — Roles y Permisos** | Pruebas de permisos por cada rol (5 roles) | 4 h |
| **M3 SUBTOTAL** | | **19 h** |
| | | |
| **M4 — Portal Extranet** | Diseño y configuración del portal Extranet en Bitrix24 | 6 h |
| **M4 — Portal Extranet** | Mi Perfil — Información de usuario + Contacto y Representantes | 4 h |
| **M4 — Portal Extranet** | Mi Perfil — Catálogo + Instalaciones | 2 h |
| **M4 — Portal Extranet** | Mi Perfil — Cuentas bancarias + Homologación | 2 h |
| **M4 — Portal Extranet** | Modal de Términos y Condiciones (primer ingreso obligatorio) | 2 h |
| **M4 — Portal Extranet** | Módulo Historial / Órdenes de Servicio (lista + estados visuales) | 5 h |
| **M4 — Portal Extranet** | Pantalla de detalle de OS — Info resumen + Ítems | 3 h |
| **M4 — Portal Extranet** | Pantalla de detalle de OS — Historial de aprobaciones (línea de tiempo) | 3 h |
| **M4 — Portal Extranet** | Formulario carga docs — Tipo Transporte (5 documentos) | 3 h |
| **M4 — Portal Extranet** | Formulario carga docs — Estiba / Devolución + Desestiba | 2 h |
| **M4 — Portal Extranet** | Formulario carga docs — Servicios Profesionales (condicional) + Gastos Adicionales | 3 h |
| **M4 — Portal Extranet** | Indicadores visuales de estado (verde / amarillo / rojo) | 3 h |
| **M4 — Portal Extranet** | Diseño responsive y UX del portal | 5 h |
| **M4 — Portal Extranet** | Pruebas del portal como usuario proveedor | 5 h |
| **M4 SUBTOTAL** | | **48 h** |
| | | |
| **M5 — Automatizaciones** | Auto-registro + generación de usuario Extranet | 5 h |
| **M5 — Automatizaciones** | Envío de credenciales (template de bienvenida) | 3 h |
| **M5 — Automatizaciones** | Notificación de OBSERVADO con motivo dinámico | 3 h |
| **M5 — Automatizaciones** | Notificación de PAGADO / PAGO PARCIAL | 2 h |
| **M5 — Automatizaciones** | Configuración de plantillas de correo HTML + variables dinámicas | 5 h |
| **M5 — Automatizaciones** | Notificaciones internas por cambio de estado | 3 h |
| **M5 — Automatizaciones** | Pruebas de todas las automatizaciones (escenarios reales) | 4 h |
| **M5 SUBTOTAL** | | **25 h** |
| | | |
| **M6 — Carga Masiva** | Diseño del esquema de importación (mapeo de columnas, validaciones) | 4 h |
| **M6 — Carga Masiva** | Desarrollo del script de carga masiva (API REST / parsing CSV/Excel) | 10 h |
| **M6 — Carga Masiva** | Lógica de cruce por RUC + N° Factura / OS | 4 h |
| **M6 — Carga Masiva** | Actualización masiva de estados en Bitrix24 vía API | 5 h |
| **M6 — Carga Masiva** | Adjunción automática de PDFs de constancias de pago | 4 h |
| **M6 — Carga Masiva** | Manejo de errores, log de resultados y rollback | 4 h |
| **M6 — Carga Masiva** | Interfaz de carga (UI simple para Tesorería) | 4 h |
| **M6 — Carga Masiva** | Pruebas con datos reales de Tesorería | 5 h |
| **M6 SUBTOTAL** | | **40 h** |
| | | |
| **M7 — Correos** | Plantillas HTML responsive — Bienvenida + Términos y Condiciones | 3 h |
| **M7 — Correos** | Plantillas HTML — OS generada + Observación con motivo | 3 h |
| **M7 — Correos** | Plantillas HTML — Aprobación + Pago liquidado + Rechazo | 3 h |
| **M7 — Correos** | Integración de variables dinámicas en cada plantilla (7 correos) | 3 h |
| **M7 — Correos** | Configuración SMTP en Bitrix24 + Pruebas de entrega | 4 h |
| **M7 SUBTOTAL** | | **16 h** |
| | | |
| **M8 — QA y Despliegue** | Configuración del dominio / subdominio del portal externo | 2 h |
| **M8 — QA y Despliegue** | Configuración SSL y acceso seguro | 1 h |
| **M8 — QA y Despliegue** | Integración con validación de XML SUNAT (si aplica) | 6 h |
| **M8 — QA y Despliegue** | Documentación técnica del sistema desarrollado | 6 h |
| **M8 — QA y Despliegue** | Capacitación — Usuarios internos (Finanzas, Contabilidad, Tesorería) | 4 h |
| **M8 — QA y Despliegue** | Capacitación — Usuarios solicitantes | 2 h |
| **M8 — QA y Despliegue** | QA integral — Flujo completo de inicio a fin | 8 h |
| **M8 — QA y Despliegue** | Corrección de bugs post-QA | 5 h |
| **M8 — QA y Despliegue** | Despliegue a producción y validación final | 4 h |
| **M8 SUBTOTAL** | | **38 h** |

---

### 📋 Cuadro B — Resumen Ejecutivo por Módulo

| # | Módulo | Tareas | Horas | % del Total | Semana(s) |
|---|--------|-------:|------:|:-----------:|-----------|
| M1 | Estructura de Datos / Campos Personalizados | 4 | 12 h | 5.6 % | S1 |
| M2 | Pipeline / Flujo de Estados (SPA) | 4 | 15 h | 7.0 % | S1 |
| M3 | Roles y Permisos | 5 | 19 h | 8.9 % | S1–S2 |
| M4 | Portal Extranet del Proveedor | 14 | 48 h | 22.5 % | S2–S3 |
| M5 | Automatizaciones y Disparadores | 7 | 25 h | 11.7 % | S3–S4 |
| M6 | Carga Masiva de Pagos (Tesorería) | 8 | 40 h | 18.8 % | S4–S5 |
| M7 | Plantillas y Notificaciones por Correo | 5 | 16 h | 7.5 % | S4 |
| M8 | Configuración, QA, Documentación y Despliegue | 9 | 38 h | 17.8 % | S5–S6 |
| | **TOTAL BASE** | **56 tareas** | **213 h** | **100 %** | **S1–S6** |
| | **Buffer 15% (imprevistos / ajustes)** | — | +32 h | — | S6–S7 |
| | **TOTAL RECOMENDADO** | | **245 h** | | **01 sep – 15 oct 2026** |

---

*Documento generado el 01 de septiembre de 2026*  
*Basado en: `Proveedor_v1.0.pdf` + `ESPECIFICACIÓN TÉCNICA DE DESARROLLO PORTAL FILE PROVEEDORES.docx`*
