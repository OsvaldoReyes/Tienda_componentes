# tasks.md — CompuParts (tareas generales)

> Basado en `spec.md` y `plan.md`. Tareas para construir el sistema, en orden de dependencia.
> Cada módulo detalla sus propias tareas en su carpeta `specs/00X-…/tasks.md`.

---

## 1. Base del proyecto

- [ ] **T1** Crear el proyecto `frontend/` con React + Vite y React Router.
  - **Terminado cuando:** `npm run dev` levanta la app y se puede navegar entre rutas vacías.
- [ ] **T2** Crear el proyecto `backend/` con Express, `db.js` y `.env.example`.
  - **Terminado cuando:** `GET /api/health` responde `{ ok: true }`, la conexión a MySQL funciona, CORS permite `localhost:5173` y `/uploads` se sirve como carpeta estática.
- [ ] **T3** Escribir `database/schema.sql` con las 6 tablas del modelo de datos.
  - **Terminado cuando:** el script crea la base `compuparts` desde cero sin errores.
- [ ] **T4** Escribir `database/seed.sql` con las 9 categorías, al menos 20 productos (con imágenes .webp en `uploads/productos/`) y un usuario administrador.
  - **Terminado cuando:** después de ejecutarlo hay productos con imagen en todas las categorías.
- [ ] **T5** Crear el layout general responsivo (Navbar con menú hamburguesa en celular, Footer), los estilos base *mobile-first* y el servicio `api.js`. (RNF-2)
  - **Terminado cuando:** el layout se ve bien a 375 px, 768 px y 1280 px de ancho, y `api.js` envía el token si existe.

## 2. Usuarios y autenticación

- [ ] **T6** Endpoint `POST /auth/register` con bcrypt. (RF-1, RF-2, RF-6)
  - **Terminado cuando:** se crea un usuario con rol `cliente`, la contraseña queda cifrada y un correo repetido devuelve 409.
- [ ] **T7** Endpoint `POST /auth/login` que devuelva un JWT, y `GET /auth/me`. (RF-3, RF-4)
  - **Terminado cuando:** con credenciales correctas se recibe un token; con incorrectas, 401 con un mensaje genérico.
- [ ] **T8** Middlewares `auth` e `isAdmin`. (RF-5)
  - **Terminado cuando:** una ruta protegida responde 401 sin token y 403 con token de cliente.
- [ ] **T9** Pantallas `/login` y `/registro`, `AuthContext` y `ProtectedRoute`.
  - **Terminado cuando:** el usuario inicia sesión, ve su nombre en la Navbar, cierra sesión y un cliente no puede abrir `/admin/productos`.

## 3. Catálogo, búsqueda y filtros

- [ ] **T10** Endpoints `GET /categories`, `GET /products` y `GET /products/:id`. (RF-7, RF-8)
  - **Terminado cuando:** devuelven solo productos activos en el formato común de respuesta.
- [ ] **T11** Pantalla de catálogo con `ProductCard` y pantalla de detalle. (RF-7, RF-8, RF-11)
  - **Terminado cuando:** se ven imagen, precio y disponibilidad; los productos con stock 0 aparecen como "Agotado"; un producto sin imagen muestra la genérica; y la cuadrícula pasa de 1 a 4 columnas según el ancho de pantalla.
- [ ] **T12** Buscador y filtros por categoría, marca y rango de precio. (RF-9, RF-10)
  - **Terminado cuando:** los filtros se combinan y aparece "No se encontraron productos" cuando no hay resultados.

## 4. Carrito, pedidos y pago simulado

- [ ] **T13** Endpoints `GET/POST/PUT/DELETE /cart` con validación de stock. (RF-12, RF-13, RF-14)
  - **Terminado cuando:** no se puede agregar más cantidad que el stock y el total se calcula correctamente.
- [ ] **T14** Botón "Agregar al carrito", `CartContext` y pantalla `/carrito`.
  - **Terminado cuando:** el cliente agrega, quita y cambia cantidades y ve el total actualizado.
- [ ] **T15** Endpoint `POST /orders` con transacción. (RF-15, RF-17)
  - **Terminado cuando:** registra el pedido, descuenta el stock y vacía el carrito; si un producto no alcanza, no guarda nada e indica cuál falló.
- [ ] **T16** Pantallas `/checkout` (pago simulado) y `/pedido/:id` (ticket), y endpoint `GET /orders/:id`. (RF-16)
  - **Terminado cuando:** el ticket muestra número, fecha, productos, cantidades, precios y total, y solo lo ve el dueño del pedido.

## 5. Panel de administración e inventario

- [ ] **T17** Middleware `upload.js` (multer) y utilidad `imagen.js` (sharp) para subir y convertir imágenes a WebP. (RF-23 a RF-25)
  - **Terminado cuando:** un JPG o PNG subido queda guardado como `.webp` en `uploads/productos/`, y un archivo que no es imagen o pesa más de 5 MB se rechaza con 400.
- [ ] **T18** Endpoints `POST` y `PUT /admin/products` con validaciones e imagen. (RF-18, RF-19, RF-22, RF-26)
  - **Terminado cuando:** un admin crea y edita productos con imagen, en la BD solo queda la ruta, al reemplazar la imagen se borra el archivo anterior y se rechazan precios o stock negativos.
- [ ] **T19** Endpoints `DELETE /admin/products/:id` (baja lógica) y `PATCH /admin/products/:id/stock`. (RF-20, RF-21)
  - **Terminado cuando:** el producto dado de baja desaparece del catálogo y el nuevo stock se refleja en el catálogo y el carrito.
- [ ] **T20** Pantalla `/admin/productos` con tabla, formulario de alta y edición (con selector de imagen y vista previa) y edición de stock.
  - **Terminado cuando:** todo lo que hace el admin desde la pantalla se refleja en el catálogo público, y la tabla se puede usar en celular con scroll horizontal.

## 6. Integración y pruebas

- [ ] **T21** Probar el flujo completo: registro → catálogo → carrito → pedido → ticket.
  - **Terminado cuando:** el flujo funciona sin errores y el stock queda correcto después de la compra.
- [ ] **T22** Probar los casos límite del `spec.md` (stock agotado durante la compra, acceso a admin por URL, carrito vacío, búsqueda sin resultados, archivo que no es imagen).
  - **Terminado cuando:** cada caso se comporta como dice la especificación.
- [ ] **T23** Revisar todas las pantallas en celular, tableta y computadora (herramientas de desarrollador del navegador). (RNF-2)
  - **Terminado cuando:** ninguna pantalla tiene texto cortado, botones inalcanzables ni scroll horizontal no deseado.
- [ ] **T24** README con requisitos, instalación, creación de la base de datos y ejecución local.
  - **Terminado cuando:** alguien que no participó puede levantar el sistema siguiendo solo el README.

## 7. Opcionales

- [ ] **T25** Historial de pedidos del cliente.
- [ ] **T26** Alerta de stock bajo para el administrador.
- [ ] **T27** Panel de estadísticas de ventas.
