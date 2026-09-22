 📦 Sistema de Control de Inventario para Mipymes

Sistema web completo para la gestión de inventario empresarial con soporte multi-tenant, jerarquía organizacional (Empresa → Sucursal → Área), movimientos trazables y reportes exportables.

**Stack:** Laravel 12 · Breeze · Blade + Tailwind CSS · Spatie Permission · Maatwebsite Excel · DomPDF

---

## 🚀 Instalación

### Requisitos Previos
- PHP 8.2+ con extensiones: `pdo_mysql`, `gd`, `zip`, `mbstring`, `xml`
- Composer 2.x
- Node.js 18+ y npm
- MySQL 8.0+ o MariaDB 10.6+

### Pasos de Instalación

```bash
# 1. Clonar el repositorio
git clone https://github.com/joseortiz20011/SistemaInventario_JoseOrtiz.git
cd SistemaInventario_JoseOrtiz

# 2. Instalar dependencias PHP
composer install

# 3. Instalar dependencias JavaScript
npm install

# 4. Configurar entorno
cp .env.example .env
php artisan key:generate
```

### Configuración de Base de Datos

Editar el archivo `.env`:
```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=sistema_inventario
DB_USERNAME=root
DB_PASSWORD=tucontraseña
```

```bash
# 5. Crear la base de datos y ejecutar migraciones con datos demo
php artisan migrate:fresh --seed

# 6. Compilar assets de frontend
npm run dev
# o para producción:
npm run build

# 7. Levantar el servidor de desarrollo
php artisan serve
```

Acceder en: **http://localhost:8000**

---

## 👤 Credenciales de Demo

| Rol                | Email                           | Contraseña | Empresa               |
|--------------------|---------------------------------|------------|-----------------------|
| Super Administrador | `superadmin@inventario.com`    | `password` | Acceso total          |
| Admin de Empresa   | `admin@lacentral.com`          | `password` | Distribuidora La Central |
| Encargado de Área  | `encargado1@lacentral.com`     | `password` | Distribuidora La Central |
| Encargado de Área  | `encargado2@lacentral.com`     | `password` | Distribuidora La Central |
| Solo Lectura       | `lector@lacentral.com`         | `password` | Distribuidora La Central |

---

## 🏗️ Estructura del Módulos

```
Empresas
├── Sucursal Central
│   ├── Bodega Principal
│   ├── Recepción y Despacho
│   └── Sala de Exhibición
└── Sucursal Norte
    ├── Bodega Norte
    └── Oficina Norte
```

---

## ⚙️ Roles y Permisos

| Permiso            | Super Admin | Admin Empresa | Encargado Área | Consulta |
|--------------------|:-----------:|:-------------:|:--------------:|:--------:|
| Gestión Empresas   | ✅          | ⚠️ (solo suya) | ❌            | ❌       |
| Gestión Sucursales | ✅          | ✅            | ❌             | ❌       |
| Gestión Áreas      | ✅          | ✅            | 👁️ ver solo    | ❌       |
| Catálogo Ítems     | ✅          | ✅            | 👁️ ver solo    | 👁️      |
| Entradas/Salidas   | ✅          | ✅            | ✅             | ❌       |
| Traslados          | ✅          | ✅            | ✅             | ❌       |
| Ajustes de Stock   | ✅          | ✅            | ❌             | ❌       |
| Dashboard          | ✅          | ✅            | ✅             | ✅       |
| Reportes (ver)     | ✅          | ✅            | ✅             | ✅       |
| Exportar Excel/PDF | ✅          | ✅            | ❌             | ✅       |

---

## 📊 Funcionalidades Principales

### Dashboard Ejecutivo
- KPIs en tiempo real (ítems, stock total, valorización)
- Semáforo de alertas de stock mínimo y agotados
- Gráficos interactivos con Chart.js (por sucursal y categoría)
- Tabla de últimos 10 movimientos
- Accesos rápidos a operaciones frecuentes

### Módulos Incluidos
1. **Estructura Organizacional:** Empresas → Sucursales → Áreas (con encargado asignado)
2. **Catálogo Maestro:** Categorías, Unidades de Medida, Proveedores, Ítems (con SKU, stock mínimo, costo)
3. **Movimientos de Inventario:**
   - **Entrada:** Ingreso de mercancía a un área
   - **Salida:** Retiro de stock con validación de existencias
   - **Traslado:** Mover stock entre áreas (con validación de origen)
   - **Ajuste:** Corrección de stock con justificación obligatoria
4. **Bitácora:** Historial inmutable de todos los movimientos
5. **Reportes:**
   - Inventario actual por empresa/sucursal/área/categoría
   - Historial de movimientos con filtros de fecha, tipo, ítem y usuario
   - Exportación a **Excel (.xlsx)** y **PDF (.pdf)**

### Seguridad
- Autenticación con Laravel Breeze
- Sistema de roles y permisos granular (Spatie Laravel Permission)
- Aislamiento multi-tenant: cada usuario solo accede a datos de su empresa
- Soft Deletes en todos los modelos principales
- Validación de stock suficiente antes de registrar salidas/traslados
- Bitácora inmutable (sin opción de editar ni eliminar movimientos)

---

## 🔧 Comandos Útiles

```bash
# Regenerar datos demo
php artisan migrate:fresh --seed

# Ver rutas registradas
php artisan route:list

# Limpiar caché
php artisan cache:clear
php artisan config:clear
php artisan view:clear

# Compilar assets (modo watch)
npm run dev
```

---

 📁 Estructura de Archivos Clave

```
app/
├── Http/Controllers/
│   ├── DashboardController.php
│   ├── ReporteController.php
│   ├── EmpresaController.php
│   ├── SucursalController.php
│   ├── AreaController.php
│   ├── ItemController.php
│   ├── MovimientoInventarioController.php
│   └── ...
├── Models/           # Eloquent models con relaciones
├── Services/         # InventarioService (lógica transaccional)
├── Exports/          # InventarioExport, MovimientosExport
└── Policies/         # ItemPolicy, EmpresaPolicy

database/
├── migrations/       # Todas las migraciones ordenadas
└── seeders/
    ├── RolesAndPermissionsSeeder.php
    ├── UnidadesMedidaSeeder.php
    └── DemoDataSeeder.php

resources/views/
├── dashboard.blade.php
├── layouts/          # app.blade.php, navigation.blade.php
├── reportes/         # inventario.blade.php, movimientos.blade.php, pdf/
├── empresas/
├── sucursales/
├── areas/
├── items/
├── movimientos/
└── ...
```

---

## 📄 Licencia

Proyecto académico — Todos los derechos reservados.
