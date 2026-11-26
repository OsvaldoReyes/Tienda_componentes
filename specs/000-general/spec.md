# spec.md — CompuParts (especificación general)

> Tienda en línea de componentes de PC.
> Este documento describe **qué** hace el sistema y **para quién**. Las decisiones técnicas van en `plan.md`.

---

## 1. Contexto

CompuParts es una tienda en línea especializada en componentes de computadora. Permite a los clientes consultar componentes con sus especificaciones, agregarlos a un carrito y realizar una compra con **pago simulado**. Un administrador gestiona los productos y las existencias.

Flujo principal del sistema: **registro → catálogo → carrito → pedido → ticket**, más la administración del inventario.

## 2. Usuarios

| Usuario | Descripción | Puede hacer |
|---|---|---|
| **Visitante** | Persona sin sesión iniciada | Ver el catálogo, buscar y filtrar, ver el detalle de un producto, registrarse |
| **Cliente** | Usuario registrado con rol `cliente` | Todo lo del visitante + usar el carrito, confirmar pedidos y ver su ticket |
| **Administrador** | Usuario con rol `admin` | Dar de alta, editar y dar de baja productos; controlar existencias |

## 3. Categorías de producto

Procesadores · Tarjetas madre · Tarjetas gráficas · Memorias RAM · Almacenamiento (SSD/HDD) · Fuentes de poder · Gabinetes · Enfriamiento · Periféricos

Cada producto tiene: **nombre, marca, categoría, precio, stock, imagen y especificaciones** (por ejemplo, un procesador: núcleos, frecuencia y socket; una RAM: capacidad, frecuencia y tipo).

## 4. Historias de usuario

| ID | Como… | Quiero… | Para… |
|---|---|---|---|
| HU-1 | visitante | registrarme con mi nombre, correo y contraseña | poder comprar |
| HU-2 | usuario registrado | iniciar y cerrar sesión | acceder a mis funciones |
| HU-3 | visitante o cliente | ver los componentes organizados por categoría | explorar la tienda |
| HU-4 | visitante o cliente | ver el detalle de un componente | conocer sus especificaciones antes de comprar |
| HU-5 | visitante o cliente | buscar por nombre y filtrar por categoría, marca y precio | encontrar rápido lo que necesito |
| HU-6 | cliente | agregar, quitar y cambiar cantidades en mi carrito | preparar mi compra |
| HU-7 | cliente | confirmar mi pedido con un pago simulado | completar la compra |
| HU-8 | cliente | recibir un ticket con los datos de mi pedido | tener un comprobante |
| HU-9 | administrador | dar de alta, editar y dar de baja productos, con su imagen | mantener el catálogo actualizado |
| HU-10 | administrador | actualizar las existencias de cada producto | evitar vender lo que no hay |

## 5. Requerimientos funcionales

Escritos en notación EARS: *Cuando / Si / Mientras …, el sistema deberá …*

### Usuarios y autenticación
- **RF-1** Cuando un visitante envíe el formulario de registro con nombre, correo y contraseña válidos, el sistema deberá crear una cuenta con rol `cliente`.
- **RF-2** Si el correo ya está registrado, el sistema deberá rechazar el registro y mostrar un mensaje de error.
- **RF-3** Cuando un usuario envíe correo y contraseña correctos, el sistema deberá iniciar su sesión y dirigirlo según su rol.
- **RF-4** Si las credenciales son incorrectas, el sistema deberá mostrar un mensaje de error sin indicar cuál de los dos datos falló.
- **RF-5** Mientras un usuario no tenga rol `admin`, el sistema deberá impedirle el acceso al panel de administración.
- **RF-6** El sistema deberá guardar las contraseñas cifradas, nunca en texto plano.

### Catálogo, búsqueda y filtros
- **RF-7** El sistema deberá mostrar el listado de productos activos con imagen, nombre, marca, precio y disponibilidad.
- **RF-8** Cuando el usuario seleccione un producto, el sistema deberá mostrar su detalle con todas sus especificaciones y existencias.
- **RF-9** Cuando el usuario escriba un texto en el buscador, el sistema deberá mostrar los productos cuyo nombre o marca lo contengan.
- **RF-10** Cuando el usuario aplique filtros de categoría, marca o rango de precio, el sistema deberá mostrar solo los productos que cumplan todos los filtros.
- **RF-11** Si un producto tiene stock 0, el sistema deberá mostrarlo como "Agotado" y no permitir agregarlo al carrito.

### Carrito, pedidos y pago simulado
- **RF-12** Mientras el cliente tenga la sesión iniciada, el sistema deberá permitirle agregar productos al carrito.
- **RF-13** El sistema deberá permitir quitar productos y cambiar cantidades, y recalcular el subtotal y el total.
- **RF-14** Si el cliente intenta poner una cantidad mayor al stock disponible, el sistema deberá impedirlo y avisarle.
- **RF-15** Cuando el cliente confirme el pedido con un pago simulado, el sistema deberá registrar el pedido, descontar el stock y vaciar el carrito.
- **RF-16** Cuando un pedido se registre, el sistema deberá mostrar un ticket con número de pedido, fecha, productos, cantidades, precios y total.
- **RF-17** Si al confirmar el pedido algún producto ya no tiene stock suficiente, el sistema deberá cancelar la confirmación e indicar qué producto falló.

### Panel de administración e inventario
- **RF-18** Mientras el usuario tenga rol `admin`, el sistema deberá permitirle dar de alta productos con todos sus datos.
- **RF-19** El sistema deberá permitir al administrador editar los datos de un producto existente.
- **RF-20** Cuando el administrador dé de baja un producto, el sistema deberá ocultarlo del catálogo sin borrar su historial en pedidos.
- **RF-21** Cuando el administrador actualice las existencias, el cambio deberá reflejarse de inmediato en el catálogo y el carrito.
- **RF-22** Si el administrador captura un precio o stock negativo, el sistema deberá rechazarlo.
- **RF-23** Cuando el administrador suba la imagen de un producto, el sistema deberá guardar el archivo en una carpeta del servidor y registrar en la base de datos solo la ruta de la imagen.
- **RF-24** Si la imagen subida no está en formato WebP, el sistema deberá convertirla automáticamente a WebP y guardar solo la versión convertida.
- **RF-25** Si el archivo subido no es una imagen válida (JPG, PNG o WebP) o supera el tamaño máximo permitido, el sistema deberá rechazarlo y mostrar un mensaje de error.
- **RF-26** Cuando el administrador reemplace la imagen de un producto, el sistema deberá eliminar el archivo anterior de la carpeta.

## 5.1 Requerimientos no funcionales

- **RNF-1** El sistema se ejecutará en un entorno **local** (frontend, backend y base de datos en la misma máquina). Por ahora no se contempla un despliegue en servidor.
- **RNF-2** La interfaz deberá ser **responsiva**: usable en computadora, tableta y celular.
- **RNF-3** Las imágenes de productos se almacenarán y servirán siempre en formato **WebP**.

## 6. Casos límite

- Un cliente agrega un producto y, antes de pagar, otro cliente compra las últimas unidades → aplica RF-17.
- El administrador da de baja un producto que está en carritos de clientes → al confirmar, aplica RF-17.
- Un usuario intenta entrar al panel admin escribiendo la URL directamente → aplica RF-5 (también se valida en el backend).
- Búsqueda sin resultados → mensaje "No se encontraron productos".
- Carrito vacío → no se puede confirmar el pedido.
- Producto sin imagen → se muestra una imagen genérica.
- El administrador sube un archivo que no es imagen (por ejemplo, un PDF) → aplica RF-25.
- La conversión a WebP falla → no se guarda el producto con una imagen rota; se muestra un error.

## 7. Funciones opcionales

Se implementan solo si las funciones principales ya están completas:

- Historial de pedidos del cliente
- Alerta de stock bajo para el administrador
- Panel de estadísticas de ventas

## 8. Fuera del alcance

- Pagos reales o pasarelas de pago
- Envíos y cálculo de paquetería
- Reseñas y calificaciones
- Correos automáticos
- Armador de PC con verificación de compatibilidad

## 9. Criterios de completitud

El sistema se considera completo cuando:

1. Se cumplen los requerimientos RF-1 a RF-26 y RNF-1 a RNF-3.
2. El flujo registro → catálogo → carrito → pedido → ticket funciona de principio a fin.
3. Un administrador puede gestionar productos y existencias, y los cambios se ven en el catálogo.
4. Un cliente no puede acceder a funciones de administrador, ni desde la interfaz ni desde la API.
5. Ninguna compra deja el stock en negativo.
6. Todas las imágenes de productos están guardadas en formato WebP.
7. Las pantallas principales se pueden usar en celular.

## 10. Decisiones tomadas

- Las imágenes se suben como archivo a una carpeta; en la base de datos solo se guarda su ruta. Todas se guardan en WebP (RF-23 a RF-26).
- El sistema se ejecuta en local; el despliegue en servidor queda pendiente de definir (RNF-1).
- La interfaz es responsiva (RNF-2).
