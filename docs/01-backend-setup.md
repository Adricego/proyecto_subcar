# Proyecto SUBCAR – Documentación de decisiones y setup inicial

## 1. Contexto general

**Objetivo del proyecto:**  
Desarrollar un MVP funcional de una plataforma de arriendo de autos por suscripción.

**Alcance actual del MVP (versión 1):**

- Gestión de catálogo de autos.
- Administración de clientes y suscripciones.
- Definición de planes de suscripción.
- Simulación de pagos (sin integración real con pasarelas).
- Gestión básica de usuarios con roles.

---

## 2. Decisiones de dominio y roles

### 2.1. Roles de usuario

Se decidió **reducir la complejidad inicial** y trabajar con 3 roles:

- `ADMIN`
  - Administra usuarios, autos, planes.
  - Crea y gestiona suscripciones.
  - Gestiona pagos simulados y estados de pago.
- `SALES` (vendedor)
  - Crea clientes (users con rol CUSTOMER + perfil Customer).
  - Crea suscripciones para esos clientes.
  - Puede ver información de vehículos y pagos relacionados a sus suscripciones.
- `CUSTOMER`
  - Ve sus suscripciones, plan contratado, auto asignado (si existe).
  - Ve su historial de pagos simulados.
  - Puede editar datos básicos de contacto.

> Se dejó fuera por ahora el rol OPERATIONS/FLEET_MANAGER y un módulo de CRM/leads para **no inflar el MVP**.

### 2.2. Entidades principales del dominio (modelo lógico)

- **User**
  - Autenticación y rol (`ADMIN`, `SALES`, `CUSTOMER`).
  - Campos: `id`, `name`, `email`, `password` (hash), `role`, `isActive`, timestamps.

- **Customer**
  - Perfil de cliente vinculado a un `User` con rol CUSTOMER.
  - Campos: `id`, `userId`, `documentNumber`, `phone`, `address`, `city`, `country`, `dateOfBirth`, timestamps.
  - Relación 1–1 con `User`.

- **SubscriptionPlan**
  - Planes de suscripción (Básico, Plus, Premium, etc.).
  - Campos: `id`, `name`, `description`, `monthlyPrice`, `maxKmPerMonth`, `extraKmPrice`, `minCommitmentMonths`, `isActive`, timestamps.

- **Vehicle**
  - Autos disponibles para asignar a suscripciones.
  - Campos: `id`, `plate` (única), `brand`, `model`, `year`, `vin`, `color`, `currentMileage`, `status`, `location`, timestamps.
  - Estado del vehículo (`VehicleStatus`): `AVAILABLE`, `ASSIGNED`, `MAINTENANCE`, `RETIRED`.

- **Subscription**
  - Contrato de suscripción entre un cliente, un plan y opcionalmente un auto.
  - Campos: `id`, `customerId`, `planId`, `vehicleId?`, `status`, `startDate`, `endDate?`, `billingDay`, `paymentStatus`, `notes`, timestamps.
  - Estado de suscripción (`SubscriptionStatus`): `PENDING`, `ACTIVE`, `PAUSED`, `CANCELLED`, `FINISHED`.
  - Resumen de estado de pago (`PaymentSummaryStatus`): `OK`, `LATE`, `IN_ARREARS`.

- **Payment** (pagos simulados)
  - Registro de pagos mensuales simulados.
  - Campos: `id`, `subscriptionId`, `amount`, `currency`, `billingPeriodStart`, `billingPeriodEnd`, `status`, `simulatedProvider`, `simulatedTransactionId`, timestamps.
  - Estado de pago (`PaymentStatus`): `PENDING`, `SUCCESS`, `FAILED`.
  - **No hay integración real con pasarela**, solo simulación internamente.

---

## 3. Decisiones de stack tecnológico (backend)

Se evaluaron alternativas y se decidió:

- **Backend:**
  - **Node.js + Express + TypeScript**
  - Motivo: simplicidad, buena curva de aprendizaje, ecosistema maduro, tipado end-to-end junto al frontend.

- **ORM / BD:**
  - **Prisma** como ORM.
  - **PostgreSQL** como base de datos relacional.
  - Motivo: soporte robusto de relaciones y enums, integración fuerte con TypeScript, migraciones claras.

- **Frontend (planificado):**
  - **React + Vite + TypeScript** (aún no implementado en esta etapa).
  - Motivo: SPA rápida para paneles (admin, sales, customer), sin complejidad extra de SSR en el MVP.

- **Auth (plan):**
  - JWT + sistema de roles basado en el campo `role` de `User`.
  - Más adelante se podrían sofisticar permisos en base a políticas más finas.

---

## 4. Estructura de proyecto y Git

### 4.1. Estructura actual del repo

En la raíz:

- `backend/` → proyecto del API (Node + TS + Express + Prisma).
- `README.md` → descripción general del proyecto.
- `.gitignore` → configuración para ignorar `node_modules`, `dist`, `.env`, etc.
- (Futuro) `frontend/` → SPA React + Vite.

### 4.2. Git y GitHub

- Se creó un repositorio Git en la carpeta raíz `proyecto_subcar`.
- Se añadió `.gitignore` en la raíz para:
  - Ignorar `node_modules`, `dist`, `.env` y archivos de editor/sistema.
  - Se decidió **eliminar el `.gitignore` dentro de `backend/`** para centralizar reglas en la raíz.
- Se creó un repositorio remoto en GitHub y se configuró:
  - `origin` apuntando al repo.
  - Rama principal: `main`.
- Se realizó un primer commit con:
  - Configuración inicial del backend.
  - Setup de Prisma.
  - README y `.gitignore`.

---

## 5. Setup del backend (Node + Prisma)

### 5.1. Creación del backend

Dentro de la carpeta raíz:

```bash
mkdir backend
cd backend
npm init -y
