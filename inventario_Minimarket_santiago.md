#  Control de Inventario
## Minimarket "Santiago"

> **Enlace al Diseño Conceptual (Draw.io):** [Ver Diagrama Entidad-Relación](https://app.diagrams.net/#G1dqWPp799ApF7jr2GaE9rdTR8FTfAHxls#%7B%22pageId%22%3A%220%22%7D)

---

## 1. Descripción del Problema

En la actualidad, el **Minimarket "Santiago"** opera bajo métodos manuales o sistemas rudimentarios para la gestión de sus actividades comerciales diarias. Esta situación genera una serie de cuellos de botella operativos, financieros y administrativos que limitan su crecimiento y eficiencia:

1. **Falta de control en tiempo real del inventario y mermas:** Al no contar con un sistema automatizado que relacione las compras y ventas mediante un kardex preciso, la administración desconoce el stock exacto de los productos, lo que provoca quiebres de stock en artículos de alta rotación o, por el contrario, acumulación de productos vencidos (especialmente en lácteos, embutidos y panadería) debido a la ausencia de un seguimiento por lotes y fechas de caducidad.
2. **Inexactitud en los arqueos de caja:** El proceso de cierre de turno se realiza de manera empírica, lo que dificulta conciliar el dinero físico existente en la gaveta con las ventas registradas. La falta de un registro formal para los gastos menores de caja (como compras imprevistas de insumos o suministros) genera diferencias monetarias y faltantes injustificados al finalizar el día.
3. **Ausencia de trazabilidad histórica en precios y transacciones:** Los cambios frecuentes en los precios de los proveedores afectan la rentabilidad si no se registran adecuadamente. Actualmente, la falta de un precio histórico de venta por cada detalle transaccional impide calcular con precisión los márgenes reales de ganancia y distorsiona los reportes financieros retrospectivos.
4. **Demoras en la atención al cliente:** La falta de integración con lectores de código de barras y la gestión manual de productos fraccionados o a granel ralentizan el proceso de cobro en el mostrador, generando filas y una experiencia de compra deficiente.

**Planteamiento de la solución:**  
Para resolver estas problemáticas, se propone el desarrollo e implementación de un sistema informático de Punto de Venta (POS) y control de inventarios estructurado bajo una arquitectura de base de datos relacional robusta (respaldada por FastAPI y MySQL). Este sistema permitirá automatizar el registro de operaciones, garantizar la trazabilidad de lotes, controlar los flujos de caja y turnos, y proveer información confiable y oportuna para la toma de decisiones gerenciales en el Minimarket "Santiago".

---

## 2. Narración y Requisitos del Sistema

Soy Don Santiago, propietario del Minimarket "Santiago". Junto con mi empleado Carlos, operamos este establecimiento comercial y necesitamos un sistema informático que automatice nuestras operaciones diarias para eliminar pérdidas y errores manuales. Actualmente requerimos gestionar un catálogo de productos organizados por **categorías**, controlados mediante **códigos de barra** y alertados por **stock mínimo**. 

Para garantizar la calidad de la mercadería y evitar pérdidas en productos perecederos, el sistema debe registrar el control de **lotes y fechas de vencimiento**. Asimismo, mantenemos un directorio unificado de **personas** que clasifica tanto a nuestros proveedores como a clientes frecuentes. En la parte operativa de caja, cada **usuario** inicia su turno registrando una caja con un monto base de efectivo (`Cash_In`). Durante la jornada se procesan **ventas** asociadas a un **método de pago** (efectivo, QR, tarjeta) y se registran salidas menores o **gastos** justificados de caja. 

Finalmente, el sistema debe registrar cada operación de venta detallando las cantidades y el **precio histórico** exacto de los productos al momento de la transacción, además de almacenar parámetros globales de **configuración** del negocio.

---

## 3. Modelo Conceptual (Entidades y Atributos)

- **USUARIO:** `ID_Usuario` (PK), Nombre, Apellido, Username, Email, Password, Estado, Es_Admin.
- **CATEGORIA:** `ID_Categoria` (PK), Nombre, Descripción.
- **PRODUCTO:** `ID_Producto` (PK), Barcode, Nombre, Descripción, Stock_Min, Precio_Costo, Precio_Venta, Unidad.
- **LOTE:** `ID_Lote` (PK), Codigo_Lote, Fecha_Vencimiento, Cant_Inicial, Cant_Actual.
- **PERSONA:** `ID_Persona` (PK), Nombre, Apellido, Empresa, Dirección, Teléfono, Tipo_Persona.
- **CAJA:** `ID_Caja` (PK), Cash_In, Cash_Out, Estado, Fecha_Apertura, Fecha_Cierre.
- **GASTO:** `ID_Gasto` (PK), Descripción, Monto, Fecha.
- **VENTA:** `ID_Venta` (PK), Total, Monto_Recibido, Descuento, Fecha.
- **DETALLE_VENTA:** `ID_Detalle` (PK), Cantidad, Precio_Historico, Subtotal *(Entidad asociativa)*.
- **METODO_PAGO:** `ID_Metodo_Pago` (PK), Nombre, Estado.
- **CONFIGURACION:** `ID_Configuracion` (PK), Clave, Valor.

### Relaciones y Cardinalidades Principales
* **CATEGORIA (1) — (N) PRODUCTO** (*CLASIFICA*)
* **USUARIO (1) — (N) PRODUCTO** (*REGISTRA*)
* **PRODUCTO (1) — (N) LOTE** (*TIENE*)
* **USUARIO (1) — (N) CAJA** (*ABRE*)
* **CAJA (1) — (N) GASTO** (*GENERA*)
* **CAJA (1) — (N) VENTA** (*REALIZA*)
* **PERSONA (1) — (N) VENTA** (*COMPRA*)
* **METODO_PAGO (1) — (N) VENTA** (*PAGA*)
* **VENTA (1) — (N) DETALLE_VENTA** (*GENERA / CONTIENE*)
* **PRODUCTO (1) — (N) DETALLE_VENTA** (*INCLUYE*)

## 4. Anexo: Entrevistas

### Entrevista 1: Al Dueño / Administrador (Don Santiago)
1. **¿Cuáles considera que son los mayores desafíos financieros o de control que enfrenta actualmente?**  
   *R:* El mayor problema es que al final del día las cuentas a veces no cuadran exactas por retiros no registrados o pagos a proveedores en efectivo.
2. **¿Cómo gestiona actualmente el control de las ganancias y los precios de venta?**  
   *R:* De forma muy empírica. Los precios suben a menudo y a veces vendemos con márgenes ajustados por no tener un registro histórico exacto de costos anteriores.
3. **¿Tiene problemas con productos vencidos o mermas?**  
   *R:* Sí, sobre todo en lácteos y embutidos, ya que sin un control por lotes a veces encontramos productos caducados en góndola.
4. **¿Cómo verifica el rendimiento de su personal o de los turnos?**  
   *R:* Con apuntes en un cuaderno, lo cual carece de trazabilidad por usuario o turnos separados.
5. **¿Qué tan seguido se queda sin stock de productos de alta rotación?**  
   *R:* Demasiado seguido; nos damos cuenta cuando el cliente ya está en la caja pidiéndolo.
6. **¿Le gustaría restringir permisos entre cajeros y administradores?**  
   *R:* Sí, los cajeros solo deben cobrar y abrir caja, sin acceso a costos, reportes gerenciales o configuraciones.
7. **¿Cómo maneja actualmente los métodos de pago (efectivo, QR, tarjetas)?**  
   *R:* Se aceptan transferencias y QR, pero al cerrar turno es difícil conciliarlos con el efectivo físico.
8. **¿Requiere reportes gerenciales de ventas y rentabilidad?**  
   *R:* Sí, para identificar qué productos dejan mayor margen y enfocar el capital de compra.
9. **¿El minimarket opera con varias sucursales?**  
   *R:* No, operamos en un único local centralizado.
10. **¿Es importante imprimir los datos de la empresa en los tickets?**  
    *R:* Sí, otorga formalidad y respaldo al cliente.
11. **¿Autoriza retiros de caja para compras menores de emergencia?**  
    *R:* Sí, pero necesito que queden estrictamente justificados por motivo, monto y usuario.
12. **¿Qué espera de la velocidad de atención en caja?**  
    *R:* Reducir filas mediante lectores de códigos de barra y automatización de cálculos a granel.
13. **¿Lleva registro formal de proveedores o clientes a crédito?**  
    *R:* Actualmente se lleva en libretas informales propensas a errores.
14. **¿Estaría de acuerdo con exigir un monto inicial de cambio ("base") por turno?**  
    *R:* Me parece excelente para transparentar el saldo inicial de la gaveta.
15. **¿Cómo definiría el éxito del sistema?**  
    *R:* Si la caja cuadra sin diferencias, conozco la ganancia exacta, evito mermas por vencimiento y la atención es rápida.

---

### Entrevista 2: Al Empleado Encargado / Cajero (Carlos)
1. **¿Cómo es el proceso actual al atender una venta con múltiples productos?**  
   *R:* Búsqueda manual de precios o cálculo a pulso en productos a granel, lo que genera filas lentas.
2. **¿Qué herramientas tecnológicas utiliza en el mostrador?**  
   *R:* Una caja registradora básica y anotaciones manuales en un cuaderno.
3. **¿Cómo maneja la venta de productos fraccionados o por peso?**  
   *R:* Cálculos mentales o reglas de tres propensas a errores humanos.
4. **¿Cómo registra los gastos menores pagados con dinero de caja?**  
   *R:* En papeles sueltos o libretas que a menudo se extravían y descuadran el arqueo.
5. **¿Qué tan fácil es identificar productos próximos a vencer en estantería?**  
   *R:* Difícil, requiere revisión visual manual uno por uno.
6. **¿Cómo procede al recibir mercadería nueva de un proveedor?**  
   *R:* Anoto en una hoja de recepción y acomodo en estantes, sin vinculación inmediata al inventario digital.
7. **¿Cómo gestiona los pagos por QR frente al efectivo?**  
   *R:* Debo verificar el comprobante visualmente en el celular del cliente y separar mentalmente el dinero.
8. **¿Qué información desearía ver en pantalla al escanear un producto?**  
   *R:* Nombre, presentación, stock disponible y precio unitario de venta.
9. **¿Cómo realiza el cierre de turno y arqueo de caja?**  
   *R:* Conteo físico total, resta de la base inicial, cálculo manual de gastos y comparación estimada.
10. **¿Ha tenido problemas con diferencias monetarias inexplicables al cerrar turno?**  
    *R:* Sí, de forma frecuente al carecer de un registro automatizado por operación.
11. **¿Qué tan ágil debe ser el inicio de sesión y apertura de caja?**  
    *R:* Inmediato, ingresando credenciales y monto base de cambio.
12. **¿Los clientes solicitan tickets de compra detallados?**  
    *R:* Sí, de manera regular para sus controles personales o empresariales.
13. **¿Qué función le ahorraría más tiempo en su jornada?**  
    *R:* El escaneo por código de barras y el descuento automático de inventario.
14. **¿Considera útil recibir alertas visuales de stock mínimo?**  
    *R:* Sí, para avisar oportunamente al administrador sobre productos agotados o por agotarse.
15. **¿Qué expectativas tiene del nuevo sistema POS?**  
    *R:* Eliminar el estrés de cálculos manuales, agilizar arqueos de caja y optimizar la atención al cliente.
