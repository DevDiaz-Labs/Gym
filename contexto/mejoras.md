# Análisis y Plan de Acción: Finanzas y POS

A continuación, presento un análisis profundo del estado actual de los módulos de **Finanzas** (`finanzas.astro`) y **Punto de Venta (POS)** (`pos.astro`), identificando todo lo que está pendiente, bugs (lógica estática/hardcodeada) y los pasos para conectar correctamente estas pantallas con el backend (Google Sheets).

## ⚠️ User Review Required

Revisa el plan detallado para ambas pantallas. Hay algunas decisiones importantes que tomar (por ejemplo, si usamos una librería para las gráficas en Finanzas o si tienes los endpoints de Google Sheets listos para los productos del POS).

> [!IMPORTANT]
> **Endpoints de Google Sheets**: Para poder conectar el POS y las transacciones, asegúrate de preguntarle a tu primo si el Google Script ya tiene las funciones `action=ultimasTransacciones`, `action=catalogoProductos` y `action=registrarVentaPOS`. Si no las tiene, tendremos que decirle que las agregue.

---

## 1. Módulo de Finanzas (`finanzas.astro`)

### Estado Actual / Problemas Detectados
1. **Métricas Estáticas:**
   - Si bien los *Ingresos (Revenue)* y *Ventas (New Memberships)* ya tienen código para intentar jalar información del API (`action=ingresosHoy`), los *Gastos (Expenses)* y *Ganancias Netas (Net Profit)* están completamente escritos a mano (hardcodeados como `$14,200` y `$28,650`).
   - Los porcentajes de crecimiento ("vs last month") también son falsos.
2. **Gráficas "Faux" (Falsas):**
   - Las gráficas de líneas (Revenue Overview) y barras (Revenue by Category) están hechas puramente con CSS y HTML (SVG estáticos). No cambian sin importar los datos reales.
3. **Tabla de Transacciones Falsa:**
   - Las "Recent Transactions" muestran datos inventados de Octubre de 2023. No hay lógica conectada para leer las transacciones del gimnasio desde Sheets.
4. **Filtros y Exportaciones no funcionales:**
   - Los botones de filtros de tiempo (Day, Week, Month, Year) y Exportación (PDF/Excel) no hacen nada al darles clic.

### Pasos para Implementar (Fase 1)
- [ ] **Paso 1.1:** Ajustar el script del cliente para leer y mapear todas las métricas (Ingresos, Gastos, Ganancias).
- [ ] **Paso 1.2:** Asignarle un `id` al cuerpo de la tabla de transacciones (`<tbody>`) y crear la función en JavaScript para hacer el fetch de las transacciones recientes a Google Sheets, iterarlas e inyectarlas dinámicamente como se hizo en `miembros.astro`.
- [ ] **Paso 1.3:** Reemplazar las gráficas de CSS por una librería liviana (como Chart.js importada desde un CDN) y alimentarlas con la información real de ganancias y categorías.

---

## 2. Punto de Venta / POS (`pos.astro`)

### Estado Actual / Problemas Detectados
1. **Productos Inventados:**
   - La pestaña de "Productos" muestra una sola tarjeta falsa de "Whey Protein Isolate". No hace fetch hacia ningún catálogo en Google Sheets.
2. **Carrito de Compras Estático:**
   - El panel derecho (Shopping Cart) siempre muestra "1 Item" y un subtotal inventado. Al darle "+" o "-", o intentar eliminar el ítem, no sucede absolutamente nada.
3. **Métodos de Pago Desconectados:**
   - Puedes ver los botones de Efectivo, Tarjeta y Transferencia, pero al darles clic no cambian de estado ni guardan cuál fue seleccionado.
4. **Procesar Pago No Funciona:**
   - El gran botón verde de "Procesar Pago" no tiene ningún evento atado a él. Al pulsarlo no se envía ninguna información ni se registra venta alguna.

### Pasos para Implementar (Fase 2)
- [ ] **Paso 2.1:** Configurar el "Estado Global" del Carrito en JavaScript. Crearemos un arreglo `let cart = []` que maneje qué productos se van agregando, y funciones para renderizar el subtotal, IVA y total dinámicamente.
- [ ] **Paso 2.2:** Realizar un `fetch` hacia Google Sheets (idealmente `action=catalogoProductos`) e inyectar las tarjetas reales de los productos que vende el gimnasio (aguas, batidos, camisetas, etc.).
- [ ] **Paso 2.3:** Añadir eventos de clic a cada tarjeta de producto para que, al seleccionarla, el producto pase al arreglo del carrito y se actualice la pantalla derecha.
- [ ] **Paso 2.4:** Hacer que los botones de método de pago sean seleccionables (como hicimos con las tarjetas en `nuevo-miembro.astro`).
- [ ] **Paso 2.5:** Programar el botón "Procesar Pago" para que tome la lista de ítems del carrito y el método de pago seleccionado, envíe un `POST` al Google Web App, y así quede registrada la venta, vaciando el carrito en caso de éxito.

---

## Open Questions para el Usuario

1. **Sobre los Endpoints de Google Sheets:** ¿Tu primo ya programó las funciones en Google Sheets para pedir las transacciones y los productos de la tienda, o las va a agregar pronto?
2. **Gráficas de Finanzas:** ¿Te parece bien si usamos `Chart.js` (es rápido, gratis y se ve excelente) para reemplazar esas gráficas estáticas falsas por gráficas reales e interactivas?
3. **Exportaciones:** ¿Los botones de Exportar a PDF / Excel son vitales para hoy, o los dejamos funcionales en una segunda etapa y nos enfocamos en que salgan las métricas y la tabla primero?

¡Dime qué opinas y si apruebas empezamos a trabajar en la Fase 1 (Finanzas)!
