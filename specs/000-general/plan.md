# plan.md — CompuParts (plan técnico general)

> Basado en `spec.md`. Este documento describe **cómo** se construye el sistema.

---

## 1. Stack tecnológico

| Capa | Tecnología | Motivo |
|---|---|---|
| Frontend | **React + Vite** | Arranque rápido y desarrollo por componentes |
| Ruteo frontend | React Router | Navegación entre pantallas y rutas protegidas |
| Backend | **Node.js + Express** | API REST sencilla, separada del frontend |
| Base de datos | **MySQL** | Datos relacionales (productos, pedidos, usuarios) |
| Conexión a BD | `mysql2` | Driver directo, sin ORM |
| Autenticación | JWT + `bcrypt` | Sesión por token; contraseñas cifradas (RF-6) |
| Comunicación | `fetch` / Axios con JSON | El frontend consume la API REST |
| Subida de archivos | `multer` | Recibe la imagen del producto (multipart/form-data) |
| Conversión de imágenes | `sharp` | Convierte JPG/PNG a WebP en el servidor (RF-24) |
| Estilos | CSS con enfoque *mobile-first* y media queries | Interfaz responsiva (RNF-2) |
| Entorno | Ejecución local | Frontend, backend y MySQL en la misma máquina (RNF-1) |

## 2. Arquitectura

```
[ Navegador ]
     │  React + Vite (puerto 5173)
     ▼
[ API REST ]  Node + Express (puerto 3000)
     │  rutas → controladores → consultas SQL
     ▼
[ MySQL ]  base de datos compuparts
```

- El frontend **nunca** se conecta directo a MySQL; todo pasa por la API.
- Las rutas de administración se protegen **en el backend** (middleware de rol), no solo ocultando elementos en el frontend.

## 3. Estructura del proyecto

```
compuparts/
├── specs/
│   ├── 000-general/              spec.md · plan.md · tasks.md
│   ├── 001-usuarios/
│   ├── 002-catalogo/
│   ├── 003-carrito-pedidos/
│   └── 004-admin-inventario/
├── frontend/
│   └── src/
│       ├── components/           componentes compartidos (Navbar, ProductCard…)
│       ├── pages/                una página por pantalla
│       ├── context/              AuthContext, CartContext
│       ├── services/             llamadas a la API (api.js)
│       └── main.jsx
├── backend/
│   ├── src/
│   │   ├── routes/               auth, products, cart, orders, admin
│   │   ├── controllers/
│   │   ├── middlewares/          auth.js (verifica JWT), isAdmin.js, upload.js (multer)
│   │   ├── utils/                imagen.js (conversión a WebP con sharp)
│   │   ├── db.js                 conexión a MySQL
│   │   └── index.js
│   ├── uploads/
│   │   └── productos/            imágenes .webp (no se suben al repo, salvo las del seed)
│   └── .env.example
├── database/
│   ├── schema.sql                creación de tablas
│   └── seed.sql                  categorías y productos de prueba
└── README.md
```

## 4. Modelo de datos

```
usuarios (id, nombre, correo UNIQUE, password_hash, rol ENUM('cliente','admin'), creado_en)

categorias (id, nombre UNIQUE)

productos (id, nombre, marca, categoria_id → categorias.id, precio DECIMAL(10,2),
           stock INT, imagen_ruta VARCHAR(255) NULL, especificaciones JSON, activo BOOLEAN DEFAULT TRUE,
           creado_en)

carrito_items (id, usuario_id → usuarios.id, producto_id → productos.id, cantidad,
               UNIQUE(usuario_id, producto_id))

pedidos (id, usuario_id → usuarios.id, fecha, total DECIMAL(10,2),
         estado ENUM('pagado') DEFAULT 'pagado')

detalle_pedido (id, pedido_id → pedidos.id, producto_id → productos.id,
                cantidad, precio_unitario DECIMAL(10,2))
```

**Decisiones del modelo:**
- `especificaciones` es JSON porque cada categoría tiene campos distintos (núcleos, capacidad, watts…).
- `activo = FALSE` en lugar de borrar el producto (RF-20).
- `precio_unitario` se copia al detalle del pedido para que el ticket no cambie si después cambia el precio.
- El carrito se guarda en la base de datos para poder validar el stock en el backend.
- `CHECK (precio >= 0)` y `CHECK (stock >= 0)` en `productos` (RF-22).
- `imagen_ruta` guarda solo la ruta relativa (ej. `/uploads/productos/12-1732650000.webp`), nunca la imagen en sí (RF-23). Si es `NULL`, el frontend muestra una imagen genérica.

## 5. API REST

Base: `http://localhost:3000/api`

| Método | Ruta | Acceso | RF |
|---|---|---|---|
| POST | `/auth/register` | Público | RF-1, RF-2 |
| POST | `/auth/login` | Público | RF-3, RF-4 |
| GET | `/auth/me` | Sesión | — |
| GET | `/categories` | Público | RF-7 |
| GET | `/products?q=&categoria=&marca=&min=&max=` | Público | RF-7, RF-9, RF-10 |
| GET | `/products/:id` | Público | RF-8 |
| GET | `/cart` | Cliente | RF-12 |
| POST | `/cart` `{productoId, cantidad}` | Cliente | RF-12, RF-14 |
| PUT | `/cart/:productoId` `{cantidad}` | Cliente | RF-13, RF-14 |
| DELETE | `/cart/:productoId` | Cliente | RF-13 |
| POST | `/orders` | Cliente | RF-15, RF-17 |
| GET | `/orders/:id` | Cliente (dueño) | RF-16 |
| POST | `/admin/products` (multipart, campo `imagen`) | Admin | RF-18, RF-22 a RF-25 |
| PUT | `/admin/products/:id` (multipart, `imagen` opcional) | Admin | RF-19, RF-22 a RF-26 |
| DELETE | `/admin/products/:id` | Admin | RF-20 |
| PATCH | `/admin/products/:id/stock` `{stock}` | Admin | RF-21, RF-22 |

**Formato de respuesta común:**
- Éxito: `{ "ok": true, "data": ... }`
- Error: `{ "ok": false, "error": "mensaje legible" }`, con código HTTP adecuado (400, 401, 403, 404, 409).

**Autenticación:** el frontend envía `Authorization: Bearer <token>` en las rutas protegidas. El token incluye `id` y `rol` del usuario.

**Pedidos:** `POST /orders` se ejecuta dentro de una **transacción** de MySQL. Si algún producto no tiene stock suficiente, se hace `ROLLBACK` y no se guarda nada (RF-17).

**Imágenes estáticas:** Express sirve la carpeta con `app.use('/uploads', express.static('uploads'))`, así el frontend muestra la imagen con `http://localhost:3000` + `imagen_ruta`.

## 5.1 Flujo de subida de imágenes

```
Formulario admin (input file)
   │  multipart/form-data
   ▼
multer (memoria) → valida tipo (image/jpeg, image/png, image/webp) y tamaño (máx. 5 MB)
   │
   ▼
sharp → .webp({ quality: 80 }) y redimensiona a máx. 800 px de ancho
   │
   ▼
guarda en uploads/productos/<idProducto>-<timestamp>.webp
   │
   ▼
UPDATE productos SET imagen_ruta = '/uploads/productos/…webp'
   │
   └─ si había imagen anterior → se borra el archivo viejo (RF-26)
```

- Aunque el archivo ya venga en WebP, también pasa por `sharp` para homogeneizar tamaño y calidad.
- Si `sharp` falla, se responde 400 y no se modifica el producto.
- El nombre del archivo lo genera el servidor; nunca se usa el nombre original que envía el usuario.

## 6. Dependencias técnicas entre módulos

| Módulo | Depende de | Qué necesita |
|---|---|---|
| Carrito y pedidos | Autenticación | Middleware `auth` que entregue `req.user.id` |
| Carrito y pedidos | Productos | Tabla `productos` con `precio`, `stock` y `activo` |
| Panel admin | Autenticación | Middleware `isAdmin` |
| Catálogo | Base de datos | Productos de prueba en `seed.sql` |
| Todos | Base del proyecto | `schema.sql`, `db.js` y layout del frontend |

## 7. Pantallas (frontend)

| Ruta | Pantalla | Acceso |
|---|---|---|
| `/` | Inicio / catálogo | Público |
| `/producto/:id` | Detalle de producto | Público |
| `/login`, `/registro` | Autenticación | Público |
| `/carrito` | Carrito | Cliente |
| `/checkout` | Confirmación y pago simulado | Cliente |
| `/pedido/:id` | Ticket | Cliente |
| `/admin/productos` | Gestión de productos e inventario | Admin |

Componentes compartidos: `Navbar`, `Footer`, `ProductCard`, `ProtectedRoute` (redirige a `/login` sin sesión y bloquea `/admin` a los clientes).

**Diseño responsivo (RNF-2):**

| Pantalla | Ancho | Catálogo |
|---|---|---|
| Celular | < 640 px | 1 columna de productos; Navbar con menú hamburguesa |
| Tableta | 640–1024 px | 2–3 columnas |
| Computadora | > 1024 px | 4 columnas; filtros en barra lateral |

- Se diseña primero para celular (*mobile-first*) y se amplía con media queries.
- El catálogo usa CSS Grid; las tablas del panel admin tienen scroll horizontal en celular.
- Las imágenes usan `width: 100%` y `object-fit: contain`.

## 8. Variables de entorno (`backend/.env.example`)

```
PORT=3000
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=
DB_NAME=compuparts
JWT_SECRET=cambia_esto
UPLOADS_DIR=uploads/productos
MAX_IMAGE_MB=5
```

El archivo `.env` real **no** se sube al repositorio. La carpeta `uploads/productos/` se agrega al `.gitignore`, excepto las imágenes de prueba del seed.

**Ejecución local (RNF-1):** MySQL en `localhost:3306`, backend en `localhost:3000` y frontend en `localhost:5173`. Se habilita CORS en el backend solo para `http://localhost:5173`.

## 9. Validaciones

| Dato | Regla |
|---|---|
| Correo | Formato válido y único |
| Contraseña | Mínimo 8 caracteres |
| Precio | Número ≥ 0 con 2 decimales |
| Stock | Entero ≥ 0 |
| Cantidad en carrito | Entero entre 1 y el stock disponible |
| Imagen de producto | JPG, PNG o WebP; máximo 5 MB; se guarda siempre como WebP |

Todas las validaciones se hacen en el backend; el frontend las repite solo para mostrar mensajes rápidos al usuario.
