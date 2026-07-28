# 🛒 Sistema de Punto de Venta (POS) - Offline # 

Sistema de escritorio diseñado para bodegas, tiendas de abarrotes y negocios pequeños. Desarrollado con enfoque en rendimiento, fiabilidad sin conexión a internet y facilidad de uso.

 ⚠️ **Este es un producto comercial cerrado.** Este repositorio sirve como demostración técnica de la arquitectura, diseño de interfaz y capacidades del sistema para el portafolio del desarrollador.
 
------

## 🏗️ Stack Tecnológico y Arquitectura

* **Lenguaje:** Java 17
* **Interfaz Gráfica:** JavaFX con tema moderno (AtlantaFX + CSS personalizado)
* **Base de Datos:** SQLite cifrado con SQLCipher (AES-256) vía JDBC
* **Reportes y Tickets:** JasperReports (generación de tickets 100% por código, sin archivos `.jrxml`)
* **Arquitectura:** Modelo-Vista-Controlador (MVC) + Data Access Object (DAO) con transacciones ACID

### Principios de Diseño Implementados

- **Cero eliminación**: No existe ningún `DELETE` sobre `ventas` ni `productos`. "Borrar" un producto lo desactiva (`activo = 0`); "borrar" una venta la anula (`estado = 'ANULADA'`) y repone el stock, pero la fila permanece para siempre.
- **Transacciones ACID**: `VentaDAOImpl.registrarVenta()` y `.anularVenta()` usan `connection.setAutoCommit(false)` + `commit()`/`rollback()`, para que nunca se descuente stock sin registrar la venta.
- **Auditoría total**: `logs_sistema` registra TODA acción del sistema: login/logout, ventas creadas, anulaciones, cierres de caja, exportaciones, accesos denegados, etc.
- **Blindaje por rol**: `MainController` oculta los botones según el rol, y además `verificarAdmin()` bloquea el acceso aunque alguien manipule la UI.
- **Costo histórico**: `venta_detalle` guarda `precio_costo_hist` y `precio_venta_hist` al momento de la venta, para que el reporte de ganancia real no se distorsione si los precios cambian luego.

### Estructura del Proyecto

com.tienda.pos
├── App.java                 # Punto de entrada JavaFX
├── config/
│   └── DatabaseManager.java # Singleton de conexión SQLCipher + auto-setup
├── model/                   # Entidades (Usuario, Producto, Venta, ...)
├── dao/                     # Interfaces DAO
│   └── impl/                # Implementaciones JDBC con transacciones
├── controller/              # Controladores JavaFX (uno por pantalla)
└── util/                    # SessionManager, PasswordUtil, TicketService

src/main/resources/
├── fxml/                    # Vistas (.fxml)
├── css/                     # Ajustes visuales sobre AtlantaFX
└── db/schema.sql            # Esquema completo de la base de datos

---

## 📸 Interfaz de Usuario: ADMIN (ACCESO TOTAL)

![Apertura de Caja](capturas/aperturacaja.jpg)
![Punto de Venta](capturas/caja.jpg)
![Historial y Detalle de Ventas](capturas/historialventas1.jpg)
![Historial y Detalle de Ventas](capturas/historialventas2.jpg)
![Historial y Detalle de Ventas](capturas/historialventas3.jpg)
![Fiados](capturas/fiados1.jpg)
![Fiados](capturas/fiados2.jpg)
![Fiados](capturas/fiados3.jpg)
![Fiados](capturas/fiados4.jpg)
![Inventario](capturas/inventario1.jpg)
![Inventario](capturas/inventario2.jpg)
![Inventario](capturas/inventario3.jpg)
![Entrada de Mercadería](capturas/entradamerc1.jpg)
![Entrada de Mercadería](capturas/entradamerc2.jpg)
![Reportes](capturas/reportes.jpg)
![Cierre de Caja](capturas/cierrecaja.jpg)
![Usuarios](capturas/usuarios1.jpg)
![Usuarios](capturas/usuarios2.jpg)
![Configuración](capturas/configuracion.jpg)
![Logs y Auditoría](capturas/logs1.jpg)
![Logs y Auditoría](capturas/logs2.jpg)

## 📸 Interfaz de Usuario: CAJERO

![Punto de Venta](capturas/cajaUsuario.jpg)
![Inventario](capturas/inventarioUsuario.jpg)
![Fiados](capturas/fiadosUsuario.jpg)
![Cierre de Caja](capturas/cierrecajaUsuario.jpg)

---

## 🗂️ Módulos del Sistema

### A. 🛒 Punto de Venta (Mostrador de Atención Rápida)

Caja registradora diseñada para operar con velocidad y sin errores en el mostrador.

* **Búsqueda instantánea**: por código de barras (lector óptico) o por nombre del producto registrado.
* **Venta a granel / balanza**: productos por unidad o por peso (KG) con decimales (`0.250 kg`, `1.500 kg`).
* **Modo Atajo F8**: precarga la cantidad/peso antes del escaneo sin abrir modales.
* **Teclas rápidas**: `F1` vender, `F2` cobrar, `F12` cerrar sesión.
* **Medios de pago**: Efectivo, Yape, Plin, Tarjeta y Fiado.
* **Vuelto automático**: cálculo exacto para evitar errores al dar el cambio.
* **Impresión térmica**: tickets en ticketeras de 58mm u 80mm.

---

### B. 📜 Historial de Ventas y Comprobantes

Panel de consulta interactivo que permite al administrador revisar y gestionar todas las transacciones del negocio.

* **Tarjetas KPI en tiempo real**: Total Recaudado, N° de Tickets, Ticket Promedio y Ventas Anuladas.
* **Filtros reactivos**: se actualizan automáticamente al cambiar Cajero, Método de Pago, Rango de Fechas o al escribir en la búsqueda — sin necesidad de presionar un botón.
* **Reimpresión de tickets**: reemisión física del comprobante a la ticketera.
* **Exportación a PDF**: generación del comprobante de venta en `.pdf`.
* **Exportación a CSV/Excel**: descarga limpia del historial según el FILTRO para contabilidad.
* **Anulación segura**: solo los administradores pueden anular ventas; el stock se repone automáticamente y el motivo queda registrado permanentemente.

---

### C. 📔 Cuaderno de Fiados (Créditos a Clientes)

Reemplaza el cuaderno de deudas físico con un registro digital organizado por cliente.

* **Registro de crédito**: vinculado al nombre y teléfono del cliente.
* **Listado de cobranza**: saldos pendientes acumulados por cliente.
* **Abonos parciales y totales**: actualización inmediata del saldo.

---

### D. 📦 Inventario y Entrada de Mercadería

Control total del catálogo de productos y del stock disponible.

* **Catálogo de productos**: código de barras, nombre de producto, precio de costo, precio de venta, stock actual y stock mínimo.
* **Alerta de stock bajo**: indicador en tiempo real en el menú lateral `📦 Inventario ⚠ 2`.
* **Entrada de mercadería**: módulo dedicado para actualizar stock por ingreso de productos.
* **Sin borrado de datos**: los productos se desactivan, nunca se eliminan — el historial permanece intacto.

---

### E. 📊 Reportes y Analítica Comercial

Panel de inteligencia de negocio para que el dueño entienda qué vende, qué gana y qué productos necesita atención.

* **Resumen financiero por rango de fechas**: Ingresos Totales, Ganancia Neta Real (resta el costo histórico de cada producto) y Número de Transacciones.
* **Gráficos intercambiables**:
  * 💰 **Ventas por Ingresos (S/)** — productos que más dinero generan.
  * 📦 **Ventas por Volumen** — productos con mayor rotación en mostrador.
  * 📈 **Productos más Rentables** — productos con mayor margen de ganancia neta real.
  * ⚠️ **Productos más Fiados** — análisis de riesgo de cartera de crédito.
  * 🧊 **Productos Sin Ventas** — mercadería estancada para liquidar o promocionar.

---

### F. 🔒 Control de Caja y Arqueo (Apertura y Cierre de Turno)

Sistema de control de efectivo por turno de trabajo.

* **Apertura obligatoria**: el sistema no permite realizar ventas sin apertura de caja.
* **Arqueo a ciegas para cajeros**: el cajero ingresa el conteo físico de dinero sin saber el monto esperado.
* **Reporte de descuadre para administradores**: comparativo entre el dinero esperado y el entregado, con cálculo automático de sobrante o faltante.

---

### G. 🔐 Seguridad, Roles y Auditoría

* **Roles de acceso**:
  * **Cajero**: vender, consultar stock, cobrar fiados y cerrar turno.
  * **Administrador**: acceso total al sistema: reportes, usuarios, auditoría, configuración y anulaciones.
* **Log de eventos**: historial permanente e inalterable de inicios de sesión, anulaciones, descuadres y exportaciones.
* **Base de datos cifrada**: SQLCipher AES-256 — los datos del negocio no pueden copiarse ni leerse sin la clave.

---

_-Desarrollado por [Zarcap]-_
