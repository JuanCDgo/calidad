# Manual de Usuario - Control de Calidad en Planta

Este manual explica cómo utilizar la aplicación tanto desde los teléfonos móviles en la línea de producción como desde la computadora central de la fábrica.

---

## 1. Acceso a la Aplicación

### Desde la Computadora Central
1. Abrí el archivo `iniciar_servidor.bat` en el escritorio o carpeta del proyecto.
2. Abrí el navegador (Google Chrome o Edge) e ingresá a:
   `https://localhost:8000`

### Desde los Celulares en la Fábrica (Escaneo QR en 1 toque)
1. Conectá el teléfono a la red WiFi de la fábrica.
2. En la pantalla de la computadora central (Panel Admin), hacé clic en el botón azul **"QR para Celulares"**.
3. Abrí la cámara del celular y apuntá al código QR gigante en la pantalla.
4. Tocá la notificación para abrir la página en Chrome.
5. Tocá **"Instalar aplicación"** o **"Agregar a pantalla principal"**. ¡Listo! La app queda instalada con ícono propio y lista para trabajar offline.

---

## 2. Guía para el Operario (Móvil)

### Cargar un Defecto en la Línea de Producción
1. Tocá el botón grande azul **"Capturar Foto del Defecto"**. Se abrirá la cámara de tu celular.
2. Apuntá y sacá una foto clara donde se aprecie la no conformidad (madera rajada, laca corrida, esquina golpeada).
3. La aplicación optimizará la foto automáticamente en segundos para que no pese de más.
4. Seleccioná el **Modelo del Mueble** y el **Sector de Planta** donde estás trabajando.
5. En **Cantidad Afectada**, escribí de forma ágil el material dañado (ejemplo: *"2 tableros"*, *"1 frente de cajón"*, *"4 patas"*).
6. En **Descripción del Defecto**, escribí una explicación breve pero clara del problema.
7. Tocá **"Guardar Reporte"**.

### ¿Qué pasa si se corta el WiFi en el galpón?
* **No te preocupes:** la aplicación está pensada para esto. El reporte y la foto se guardan de forma 100% segura dentro de la memoria de tu celular.
* En la parte superior verás la etiqueta `🔴 Sin WiFi`.
* En la parte inferior de la pantalla aparecerá la lista de **"Pendientes de Sincronización"** mostrando las fotos y datos que aún están guardados en tu teléfono.

### ¿Cómo enviar los datos a la computadora central?
* Cuando camines hacia una zona con señal o vuelva el WiFi, la barra superior cambiará a `📶 Online`.
* Tocá el botón **"Sincronizar"** en la barra superior (que mostrará un número naranja indicando cuántos reportes tenés acumulados).
* En segundos, los reportes viajarán a la PC central y se vaciará la lista de pendientes de tu teléfono.

---

## 3. Guía para el Supervisor / Administración (PC Central)

En la parte superior derecha de la aplicación podés alternar a la vista **"Admin PC"**.

### Pestaña 1: Reportes de Calidad y Exportación
* Vas a ver todos los defectos enviados por los operarios organizados en tarjetas con sus fotos.
* Podés filtrar por **Sector de Planta** o por **Modelo de Mueble** para analizar en qué área se están repitiendo fallas.
* **Exportar a Excel:** Al tocar el botón verde **"Exportar a Excel"**, se descarga un archivo `.csv` compatible con Excel en español (con codificación UTF-8 con BOM y separador `;`), con todos los reportes filtrados, fechas, sectores, modelos, cantidades, descripciones y enlaces directos a las fotos.
* Al hacer clic sobre cualquier foto, se abre en **tamaño grande (Zoom)** para inspeccionar los detalles de la pieza dañada.
* Cada reporte muestra dos horarios en formato argentino (24 horas):
  * **Captura en planta:** Momento exacto en que el operario sacó la foto en la máquina.
  * **Sincronizado en PC:** Momento en que impactó en la computadora central.

### Pestaña 2: Dashboard / Métricas (Para Proyectar en TV)
Diseñado especialmente para reuniones de producción, sala de capacitación y auditorías:
* **Tarjetas KPI:** Muestra en números grandes el total de reportes, el sector con mayor cantidad de incidencias y el volumen de registros.
* **Filtros rápidos de fecha:** Podés alternar en 1 toque entre *Hoy*, *Últimos 7 días*, *Este Mes* o *Todos*.
* **Diagrama de Pareto (80/20):** Muestra los sectores ordenados de mayor a menor frecuencia en barras azules, y sobre ellos la curva naranja de porcentaje acumulado con la línea roja de corte al 80%. Esto permite mostrarle al equipo el 20% de las causas que generan el 80% de los problemas en planta.
* **Gráfico de Dona (Modelos):** Proporción de fallas según el modelo de mueble fabricado.
* **Modo Edición de Textos (Cero Textos Hardcodeados):**
  1. Tocá el botón **"Editar Textos"** en la barra superior del tablero.
  2. Todos los títulos, subtítulos y nombres de KPI aparecerán con un subrayado amarillo.
  3. Hacé clic sobre cualquiera de ellos y escribí la terminología exacta que querés mostrar en la reunión (ej: cambiar *"Frecuencia de Defectos"* por *"No Conformidades de Carpintería"*).
  4. Al tocar Enter o volver a pulsar "Listo", los cambios se guardan de forma permanente en tu navegador para futuras reuniones.
  5. Contás también con un botón de reinicio para volver a los textos estándar si lo deseás.

### Pestaña 2: Gestión de Modelos de Muebles
* **Agregar nuevo:** Tocá `+ Nuevo Modelo`, completá el nombre, código SKU y descripción, y tocá Guardar.
* **Editar:** Hacé clic en el ícono de lápiz para corregir nombres o descripciones.
* **Desactivar / Activar:** Tocá el botón de encendido para poner un modelo en estado "Inactivo". Los modelos inactivos **desaparecen automáticamente del celular de los operarios** para que no elijan modelos viejos o discontinuados, pero **no se borran del historial** de reportes anteriores.

### Pestaña 3: Gestión de Sectores de Planta
* La aplicación ya viene cargada con los sectores de fábrica: *Maquinado, Armado, Cabinas de Fondo, Cabinas de Terminación, Embalaje, Living y Colchones, Red*.
* Podés agregar nuevos sectores si la fábrica se expande, corregir nombres o desactivar sectores temporales.
