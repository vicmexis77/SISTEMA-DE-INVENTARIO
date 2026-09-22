# Especificaciones Técnicas — Sistema de Control de Inventario para Mipymes

## 1. Objetivo del Sistema

Desarrollar una aplicación web para que micro, pequeñas y medianas empresas (Mipymes) puedan controlar su inventario de forma centralizada, permitiendo:

- Gestionar múltiples empresas, cada una con sus propias sucursales y áreas.
- Registrar, editar, consultar y eliminar ítems de inventario.
- Trasladar inventario entre áreas/sucursales, reasignando automáticamente el responsable.
- Visualizar en un dashboard el estado general del inventario.
- Mantener un historial completo (auditoría) de todos los movimientos.

---

## 2. Stack Tecnológico

| Capa | Tecnología |
|---|---|
| Backend | Laravel 12 (PHP 8.2 o superior) |
| Base de datos | MySQL 8.0.41 |
| Autenticación | Laravel Breeze |
| Frontend | HTML5, CSS3, JavaScript para la interacción y el motor de plantillas Blade |
| Peticiones asíncronas | Fetch API / AJAX hacia endpoints de Laravel |
| Exportación de reportes | Paquete `maatwebsite/excel` (Excel) y `barryvdh/laravel-dompdf` (PDF) |
| Control de versiones | Git |

---

## 3. Alcance Funcional (Módulos)

### 3.1 Módulo de Autenticación
- Registro de usuario (con verificación de correo).
- Inicio de sesión / cierre de sesión.
- Recuperación y restablecimiento de contraseña.
- Perfil de usuario editable (nombre, correo, contraseña, foto opcional).
- Roles y permisos (ver sección 4).
- Bloqueo de cuenta tras múltiples intentos fallidos.

### 3.2 Módulo de Empresas
- CRUD de empresas (Crear, Ver, Editar, Eliminar lógicamente).
- Cada empresa tiene: nombre, RTN/identificación fiscal, dirección, teléfono, correo, logo (opcional), estado (activa/inactiva).
- Una empresa puede tener **una o más sucursales**.

### 3.3 Módulo de Sucursales
- CRUD de sucursales, asociadas a una empresa.
- Cada sucursal tiene: nombre, dirección, teléfono, estado (activa/inactiva).
- Una sucursal puede tener **múltiples áreas**.

### 3.4 Módulo de Áreas
- CRUD de áreas, asociadas a una sucursal.
- Cada área tiene: nombre (ej. "Bodega Principal", "Recepción", "Cocina"), descripción, **encargado responsable** (usuario del sistema), estado (activa/inactiva).
- Un área tiene siempre **un único encargado activo** a la vez (puede cambiar con el tiempo, pero no tener dos encargados simultáneos).

### 3.5 Módulo de Catálogo de Inventario (Ítems)
- CRUD de ítems del inventario.
- Cada ítem tiene:
  - Nombre
  - Código/SKU (único por empresa, se puede autogenerar)
  - Categoría (relación con tabla `categorias`)
  - Unidad de medida (unidad, caja, kg, litro, etc.)
  - Descripción
  - Imagen (opcional)
  - Costo unitario (opcional, para valorización de inventario)
  - Stock mínimo (para alertas)
  - Proveedor (opcional, relación con tabla `proveedores`)
  - Estado (activo/inactivo)
- El **stock real** de un ítem se calcula por área (ver sección 3.6), no es un campo fijo en el ítem. El ítem es el "catálogo maestro"; el stock vive en la relación ítem-área.

### 3.6 Módulo de Inventario por Área (Stock)
- Relación `inventario_area`: cuánta cantidad de un ítem específico existe en un área específica.
- Permite ver el inventario consolidado (todas las áreas) o filtrado por sucursal/área.
- Vista detallada por ítem: en qué áreas está, cuánta cantidad hay en cada una, y quién es el responsable de cada área.

### 3.7 Módulo de Movimientos de Inventario
Se definen **4 tipos de movimiento**, todos registrados en una bitácora (`movimientos_inventario`) que **nunca se edita ni se borra** (es el historial oficial):

1. **Entrada**: ingreso de nuevo stock a un área (ej. compra a proveedor).
2. **Salida**: descuento de stock de un área (ej. consumo, venta, merma).
3. **Traslado**: mueve una cantidad de un ítem desde un área origen hacia un área destino, **dentro de la misma empresa**. Al completarse:
   - Se descuenta del área origen.
   - Se incrementa en el área destino.
   - El responsable del inventario trasladado pasa a ser el encargado del área destino automáticamente.
4. **Ajuste**: corrección manual de cantidad (positiva o negativa), usada para conciliar inventario físico vs. sistema. Requiere un campo de "motivo" obligatorio.

Cada movimiento registra: tipo, ítem, cantidad, área origen (si aplica), área destino (si aplica), usuario que lo ejecutó, fecha/hora, motivo/observación.

**Reglas de negocio clave:**
- No se puede hacer una salida, traslado o ajuste negativo si no hay stock suficiente en el área origen (validación obligatoria).
- No se puede eliminar un ítem, área o sucursal que tenga stock activo distinto de cero (se debe trasladar o dar salida primero).
- Todo movimiento debe quedar asociado al usuario autenticado que lo realizó (trazabilidad).

### 3.8 Dashboard
Debe mostrar un resumen ejecutivo con, como mínimo:
- Total de ítems distintos en catálogo.
- Total de unidades en inventario (consolidado y por sucursal).
- Ítems con stock por debajo del mínimo (alerta visual, ej. tabla o tarjetas en rojo/amarillo).
- Últimos movimientos registrados (últimos 10, con tipo, ítem, cantidad, usuario, fecha).
- Gráfico simple de distribución de inventario por sucursal o por categoría (puede usarse Chart.js en el frontend).
- Accesos rápidos a: registrar entrada, registrar traslado, ver reportes.

### 3.9 Módulo de Reportes
- Reporte de inventario actual (filtrable por empresa, sucursal, área, categoría).
- Reporte de movimientos (filtrable por rango de fechas, tipo de movimiento, usuario, ítem).
- Exportación a Excel y PDF.

### 3.10 CRUD General
Todos los recursos (Empresas, Sucursales, Áreas, Categorías, Unidades de Medida, Proveedores, Ítems, Usuarios) deben soportar:
- Crear
- Leer / Listar (con paginación, búsqueda y filtros)
- Editar
- Eliminar (soft delete, no eliminación física de la base de datos)

---

## 4. Roles y Permisos

| Rol | Descripción | Permisos principales |
|---|---|---|
| **Super Administrador** | Administra toda la plataforma | Gestiona empresas, usuarios globales, configuración general |
| **Administrador de Empresa** | Dueño o gerente de la Mipyme | CRUD completo dentro de su empresa: sucursales, áreas, ítems, usuarios de su empresa, reportes |
| **Encargado de Área** | Responsable de un área específica | Ver y gestionar el inventario únicamente de su(s) área(s) asignada(s); puede registrar entradas, salidas y solicitar/aceptar traslados |
| **Consulta / Solo Lectura** | Usuario que solo necesita ver información | Solo puede ver dashboard y reportes, sin permisos de edición |

> Utilice el paquete `spatie/laravel-permission` para gestionar roles y permisos de forma robusta.

---

## 5. Autenticación

- Usar **Laravel Breeze** (con Blade) como base.
- Incluye de fábrica(Factories): registro, login, logout, verificación de correo, recuperación de contraseña.
- Sobre esa base, se agrega el sistema de roles (`spatie/laravel-permission`).
- Cada usuario pertenece a una empresa (excepto el Super Administrador).

---

## 6. Modelo de Datos (Entidades Principales)

```
empresas
├── id
├── nombre
├── identificacion_fiscal
├── direccion
├── telefono
├── correo
├── logo
├── estado (activo/inactivo)
├── timestamps
└── soft deletes

sucursales
├── id
├── empresa_id (FK)
├── nombre
├── direccion
├── telefono
├── estado
├── timestamps
└── soft deletes

areas
├── id
├── sucursal_id (FK)
├── nombre
├── descripcion
├── encargado_id (FK -> users.id)
├── estado
├── timestamps
└── soft deletes

categorias
├── id
├── empresa_id (FK)
├── nombre
├── timestamps
└── soft deletes

unidades_medida
├── id
├── nombre (ej. Unidad, Caja, Kg, Litro)
├── abreviatura
└── timestamps

proveedores
├── id
├── empresa_id (FK)
├── nombre
├── contacto
├── telefono
├── correo
├── timestamps
└── soft deletes

items
├── id
├── empresa_id (FK)
├── categoria_id (FK)
├── unidad_medida_id (FK)
├── proveedor_id (FK, nullable)
├── nombre
├── sku (único por empresa)
├── descripcion
├── imagen
├── costo_unitario
├── stock_minimo
├── estado
├── timestamps
└── soft deletes

inventario_area   -- tabla pivote con cantidad
├── id
├── item_id (FK)
├── area_id (FK)
├── cantidad
└── timestamps

movimientos_inventario   -- bitácora, nunca se edita/borra
├── id
├── item_id (FK)
├── tipo (entrada | salida | traslado | ajuste)
├── cantidad
├── area_origen_id (FK, nullable)
├── area_destino_id (FK, nullable)
├── usuario_id (FK -> users.id)
├── motivo / observacion
└── created_at

users
├── id
├── empresa_id (FK, nullable para Super Admin)
├── name
├── email
├── password
├── estado
├── timestamps
└── soft deletes

roles / permissions / model_has_roles   -- generadas por spatie/laravel-permission
```

### 6.1 Relaciones clave
- `Empresa` → tiene muchas `Sucursales`
- `Sucursal` → tiene muchas `Areas`
- `Area` → pertenece a un `User` (encargado)
- `Item` → pertenece a `Categoria`, `UnidadMedida`, `Proveedor` (opcional)
- `Item` ↔ `Area` → relación muchos a muchos a través de `inventario_area` (con campo `cantidad`)
- `MovimientoInventario` → pertenece a `Item`, `User`, y referencia opcional a dos `Area` (origen/destino)

---

## 7. Migraciones (orden sugerido)

1. `users` (extender la tabla base de Laravel con `empresa_id`, `estado`)
2. `empresas`
3. `sucursales`
4. `areas`
5. `categorias`
6. `unidades_medida`
7. `proveedores`
8. `items`
9. `inventario_area`
10. `movimientos_inventario`
11. Migraciones de `spatie/laravel-permission` (roles, permissions)

> Todas las tablas de negocio (excepto `movimientos_inventario` y las de catálogo simples como `unidades_medida`) deben incluir la columna `deleted_at` para soft deletes.

---

## 8. Endpoints / Rutas Sugeridas (estilo RESTful)

```
Auth
POST   /login
POST   /logout
POST   /register
POST   /password/forgot
POST   /password/reset

Empresas
GET    /empresas
POST   /empresas
GET    /empresas/{id}
PUT    /empresas/{id}
DELETE /empresas/{id}

Sucursales
GET    /sucursales
POST   /sucursales
GET    /sucursales/{id}
PUT    /sucursales/{id}
DELETE /sucursales/{id}

Areas
GET    /areas
POST   /areas
GET    /areas/{id}
PUT    /areas/{id}
DELETE /areas/{id}

Items
GET    /items
POST   /items
GET    /items/{id}
PUT    /items/{id}
DELETE /items/{id}
GET    /items/{id}/inventario     -- muestra en qué áreas está y cuánto hay

Movimientos
GET    /movimientos
POST   /movimientos/entrada
POST   /movimientos/salida
POST   /movimientos/traslado
POST   /movimientos/ajuste

Dashboard
GET    /dashboard

Reportes
GET    /reportes/inventario
GET    /reportes/movimientos
GET    /reportes/inventario/exportar-excel
GET    /reportes/inventario/exportar-pdf
```

---

## 9. Interfaz de Usuario (Frontend)

### 9.1 Pantallas mínimas requeridas
1. Login / Registro / Recuperar contraseña
2. Dashboard
3. Listado y gestión de Empresas (solo Super Admin)
4. Listado y gestión de Sucursales
5. Listado y gestión de Áreas (con selección de encargado)
6. Listado y gestión de Categorías, Unidades de Medida y Proveedores
7. Listado y gestión de Ítems (catálogo)
8. Vista de detalle de Ítem (muestra stock por área)
9. Formulario de Traslado de inventario (selección: ítem, área origen, área destino, cantidad)
10. Formulario de Entrada / Salida / Ajuste de inventario
11. Historial de movimientos (con filtros)
12. Reportes (con botones de exportar)

### 9.2 Buenas prácticas de UI para que el tracking sea "intuitivo"
- Usar códigos de color: verde (stock normal), amarillo (cerca del mínimo), rojo (por debajo del mínimo o agotado).
- El formulario de traslado debe mostrar en tiempo real (vía JS/AJAX) la cantidad disponible en el área origen antes de confirmar.
- Confirmaciones modales antes de eliminar cualquier recurso.
- Búsqueda y filtros en todas las tablas (por nombre, código, categoría, área, etc.).
- Paginación en todos los listados.
- Diseño responsivo (debe verse bien en celular y tablet).

---

## 10. Requerimientos No Funcionales

- **Seguridad:** protección CSRF (nativa de Laravel), validación de formularios en backend, hash de contraseñas (bcrypt/argon2 nativo de Laravel), políticas de autorización (`Policies` de Laravel) para asegurar que un usuario no pueda modificar datos de otra empresa.
- **Rendimiento:** uso de paginación en todos los listados; índices en las columnas usadas para búsquedas y llaves foráneas (`item_id`, `area_id`, `empresa_id`, `sku`).
- **Escalabilidad:** arquitectura multi-tenant por `empresa_id`, permitiendo agregar nuevas empresas sin cambios estructurales.
- **Disponibilidad de datos:** uso de soft deletes en todo el sistema; recomendable respaldo (backup) periódico de la base de datos.
- **Usabilidad:** diseño responsivo, mensajes de error y éxito claros, validaciones en tiempo real donde sea posible.
- **Mantenibilidad:** seguir convenciones estándar de Laravel (Eloquent, Form Requests para validación, Controllers delgados, lógica de negocio en Services cuando aplique).
- **Compatibilidad:** debe funcionar correctamente en los navegadores modernos más usados (Chrome, Edge, Firefox).

---

## 11. Estructura de Carpetas Sugerida (Laravel)

```
app/
├── Http/
│   ├── Controllers/
│   │   ├── Auth/
│   │   ├── EmpresaController.php
│   │   ├── SucursalController.php
│   │   ├── AreaController.php
│   │   ├── ItemController.php
│   │   ├── MovimientoInventarioController.php
│   │   ├── DashboardController.php
│   │   └── ReporteController.php
│   ├── Requests/
│   │   ├── StoreItemRequest.php
│   │   ├── StoreTrasladoRequest.php
│   │   └── ...
│   └── Middleware/
├── Models/
│   ├── Empresa.php
│   ├── Sucursal.php
│   ├── Area.php
│   ├── Item.php
│   ├── InventarioArea.php
│   ├── MovimientoInventario.php
│   ├── Categoria.php
│   ├── UnidadMedida.php
│   ├── Proveedor.php
│   └── User.php
├── Services/
│   └── InventarioService.php    -- lógica de traslados, entradas, salidas, validaciones de stock
└── Policies/
    ├── EmpresaPolicy.php
    ├── ItemPolicy.php
    └── ...

database/
├── migrations/
└── seeders/
    ├── RolesAndPermissionsSeeder.php
    ├── UnidadesMedidaSeeder.php
    └── DemoDataSeeder.php   -- datos de ejemplo para pruebas

resources/
└── views/
    ├── layouts/
    ├── auth/
    ├── dashboard/
    ├── empresas/
    ├── sucursales/
    ├── areas/
    ├── items/
    ├── movimientos/
    └── reportes/

public/
├── css/
└── js/
```

---

## 12. Datos de Prueba (Seeders)

Crear un `DemoDataSeeder` que genere:
- 1 empresa de ejemplo con 2 sucursales.
- 3–4 áreas por sucursal, cada una con un encargado distinto.
- Categorías y unidades de medida básicas (Unidad, Caja, Kg, Litro).
- 15–20 ítems de ejemplo distribuidos en distintas áreas.
- Algunos movimientos de entrada, salida y traslado ya registrados, para que el dashboard no se vea vacío al hacer las pruebas.

---

## 13. Fases de Desarrollo Sugeridas

1. **Fase 1 — Base:** instalación de Laravel 12, configuración de MySQL, migraciones base, autenticación (Breeze) y roles (spatie/permission).
2. **Fase 2 — Estructura organizacional:** CRUD de Empresas, Sucursales y Áreas.
3. **Fase 3 — Catálogo:** CRUD de Categorías, Unidades de Medida, Proveedores e Ítems.
4. **Fase 4 — Núcleo de inventario:** tabla `inventario_area`, y lógica de Entradas, Salidas, Traslados y Ajustes (con validaciones de stock).
5. **Fase 5 — Dashboard y Reportes:** resumen general, alertas de stock mínimo, exportación a Excel/PDF.
6. **Fase 6 — Pulido:** validaciones finales, diseño responsivo, pruebas con datos reales, documentación de usuario.

---

## 14. Criterios de Aceptación (resumen)

- [ ] Un usuario puede registrarse, iniciar sesión y recuperar su contraseña.
- [ ] Un Administrador de Empresa puede crear sucursales y áreas, asignando un encargado a cada área.
- [ ] Se puede registrar un ítem en el catálogo con categoría, unidad de medida y stock mínimo.
- [ ] Se puede registrar una entrada de inventario que incrementa el stock de un área.
- [ ] Se puede trasladar inventario de un área a otra, y el sistema reasigna automáticamente el responsable.
- [ ] No es posible trasladar o dar salida a más cantidad de la disponible en el área origen.
- [ ] El dashboard muestra el resumen general y alerta visualmente los ítems por debajo del stock mínimo.
- [ ] Todos los movimientos quedan registrados en el historial con usuario y fecha, sin posibilidad de edición o borrado.
- [ ] Todos los recursos (empresa, sucursal, área, ítem, etc.) permiten Crear, Editar, Eliminar (lógicamente) y Listar.
- [ ] El sistema es utilizable desde un dispositivo móvil.