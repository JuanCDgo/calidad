# Manual Técnico - Control de Calidad Fábrica de Muebles

## 1. Arquitectura del Sistema
El sistema está diseñado bajo una arquitectura **Offline-First** desacoplada pero desplegable de forma unificada:
- **Frontend (PWA Móvil y PC):** Desarrollado en React 18 con Vite y Workbox (`vite-plugin-pwa`). Gestiona almacenamiento local offline mediante IndexedDB (`idb`).
- **Backend (PC Central):** Construido sobre FastAPI (Python 3.12) y ejecutado con Uvicorn bajo HTTPS con certificados SSL locales.
- **Base de Datos:** SQLite 3 optimizada con `PRAGMA journal_mode=WAL` (Write-Ahead Logging) y `busy_timeout=5000` para soportar ráfagas de sincronización concurrente desde múltiples celulares.
- **Almacenamiento de Archivos:** Las fotos de defectos son validadas en sus bytes reales (evitando inyecciones o scripts con Pillow) y almacenadas en el disco de la PC central particionadas por fecha: `/backend/uploads/YYYY/MM/<timestamp>_<uuid>.jpg`.

---

## 2. Modelo de Datos (SQLite)

### Tabla `sectores`
| Campo | Tipo | Restricciones | Descripción |
|---|---|---|---|
| `id` | INTEGER | PRIMARY KEY AUTOINCREMENT | Identificador numérico del sector |
| `nombre` | TEXT | UNIQUE NOT NULL | Nombre del sector (Maquinado, Armado, etc.) |
| `activo` | INTEGER | NOT NULL DEFAULT 1 | 1 = Activo, 0 = Inactivo (Borrado lógico) |
| `created_at` | TEXT | NOT NULL | Fecha y hora de creación (GMT-3 Argentina) |
| `updated_at` | TEXT | NOT NULL | Fecha y hora de última modificación |

### Tabla `modelos_mueble`
| Campo | Tipo | Restricciones | Descripción |
|---|---|---|---|
| `id` | INTEGER | PRIMARY KEY AUTOINCREMENT | Identificador del modelo |
| `nombre` | TEXT | UNIQUE NOT NULL | Nombre comercial del modelo |
| `codigo` | TEXT | UNIQUE NULLABLE | Código interno o SKU |
| `descripcion` | TEXT | NULLABLE | Descripción técnica del mueble |
| `activo` | INTEGER | NOT NULL DEFAULT 1 | 1 = Activo, 0 = Inactivo (Borrado lógico) |
| `created_at` | TEXT | NOT NULL | Fecha y hora de creación (GMT-3) |
| `updated_at` | TEXT | NOT NULL | Fecha y hora de modificación |

### Tabla `reportes_calidad`
| Campo | Tipo | Restricciones | Descripción |
|---|---|---|---|
| `id` | INTEGER | PRIMARY KEY AUTOINCREMENT | ID interno del reporte |
| `client_uuid` | TEXT | UNIQUE NOT NULL (INDEX) | UUID generado en el celular (Idempotencia) |
| `modelo_id` | INTEGER | FOREIGN KEY -> modelos_mueble.id | Modelo afectado |
| `sector_id` | INTEGER | FOREIGN KEY -> sectores.id | Sector donde se detectó la falla |
| `descripcion_defecto` | TEXT | NOT NULL | Detalle de la no conformidad |
| `cantidad_afectada` | TEXT | NOT NULL | Cantidad libre (ej: "2 tableros", "1 lote") |
| `foto_path` | TEXT | NOT NULL | Ruta relativa web a la imagen guardada |
| `fecha_captura_local`| TEXT | NOT NULL | Timestamp en que el operario sacó la foto |
| `fecha_sincronizacion`| TEXT| NOT NULL | Timestamp en que impactó en la PC central |

---

## 3. Estrategia Offline y Sincronización

### IndexedDB en el Navegador Móvil
1. **`reportes_pendientes` (Key: `client_uuid`):** Guarda la foto como `Blob` binario (no base64) para ahorrar memoria y prevenir congelamiento del navegador móvil.
2. **`catalogos_cache`:** Guarda copia de los modelos y sectores activos. Si la app se abre en el fondo de la fábrica sin WiFi, los desplegables se cargan de inmediato desde IndexedDB.

### Compresión al Vuelo (Canvas API)
Antes de entrar a IndexedDB, cada foto tomada por la cámara nativa (que suele pesar entre 3 y 8 MB) se escala proporcionalmente a un máximo de **1280px** en su lado mayor y se comprime en JPEG a calidad **0.80**. El peso final queda en **250–400 KB**, garantizando sincronizaciones instantáneas incluso con WiFi débil.

### Idempotencia de Red
Si la red se corta a mitad de la subida, el operario reintentará la sincronización. El backend verifica la unicidad de `client_uuid`:
- Si ya existe: devuelve el reporte existente sin duplicar filas ni archivos.
- Si es nuevo: guarda la foto, inserta en SQLite y devuelve HTTP 201.

---

## 4. Seguridad, Auto-SSL y Cambio Dinámico de Red
Las PWAs y la API de cámara (`navigator.mediaDevices` y `capture="environment"`) exigen contextos seguros (`HTTPS`).

El archivo `backend/generate_certs.py` cuenta con **detección dinámica de cambio de red**:
1. En cada inicio (`run_server.py` o `iniciar_servidor.bat`), detecta la IP LAN actual de la máquina conectada al WiFi de la fábrica o Ethernet.
2. Compara la IP actual con `backend/certs/last_ip.txt`.
3. **Si la IP cambió (por ejemplo al mudar la PC del hogar a la fábrica):**
   * Conserva la Root CA (`rootCA.crt` y `rootCA.key`) para no invalidar instalaciones previas en los celulares.
   * Regenera automáticamente el certificado de servidor (`cert.pem`, `key.pem`) con los Subject Alternative Names (SAN) actualizados para la nueva IP de la fábrica.
   * Actualiza `last_ip.txt` y levanta FastAPI bajo HTTPS en la nueva dirección sin intervención manual.

### Endpoint `/api/server-info` y QR Dinámico
El backend expone `GET /api/server-info` informando:
- `local_ip`: IP actual de la fábrica.
- `server_url`: `https://<ip-fabrica>:8000`.
- El frontend (React) utiliza la librería `qrcode.react` (`QRCodeSVG`) para renderizar un código QR gigante en el Panel de Administración basado en esta URL real del servidor, evitando que se use `localhost` cuando el supervisor ingresa desde la PC central.

---

## 5. Dashboard de Métricas, Pareto y Exportación

### Diagrama de Pareto (ComposedChart de Recharts)
* **Cálculo:** Agrupa las fallas por Sector, las ordena de mayor a menor frecuencia, computa la suma acumulada y el porcentaje sobre el total filtrado (`(acumulado / total) * 100`).
* **Visualización:**
  * Eje Y Izquierdo: Frecuencia absoluta (Barras azules).
  * Eje Y Derecho: Porcentaje acumulado (0 a 100%, Línea ámbar).
  * `ReferenceLine`: Línea de referencia al 80% que demarca visualmente las causas vitales (Regla 80/20).

### Textos Editables en Caliente (Persistencia Local)
* Gestionado por el módulo `frontend/src/utils/dashboardConfig.js` y el componente `EditableText.jsx`.
* Los títulos de KPIs, encabezados y nombres de series se almacenan en `localStorage` bajo la clave `dashboard_textos_config_v1`.
* No existen textos fijos ni hardcodeados en el dashboard; el usuario puede modificarlos en vivo y conservarlos entre sesiones.

### Exportación a Excel (.csv compatible con Microsoft Excel)
* Implementado en `frontend/src/utils/exportExcel.js`.
* Emite un archivo con encabezado `\uFEFF` (Byte Order Mark UTF-8) y separador `;` para compatibilidad nativa con Microsoft Excel en español sin requerir asistentes de importación.

---

## 6. Procedimiento de Respaldos (Backups)
Para hacer una copia de seguridad total del sistema, solo se requieren dos carpetas:
1. `backend/data/calidad.db` (y archivos `-wal` / `-shm` si el servidor está en marcha).
2. `backend/uploads/` (donde se encuentran organizadas todas las fotos de los reportes).

Recomendación de script de backup diario: copiar `backend/data/calidad.db` y la carpeta `backend/uploads/` a un pendrive o carpeta de Google Drive sincronizada.

---

## 6. Comandos Operativos

### Iniciar Servidor Completo (PC Central)
Hacer doble clic en `iniciar_servidor.bat` o ejecutar:
```bash
backend\.venv\Scripts\python.exe backend\run_server.py
```

### Recompilar Frontend tras modificaciones
```bash
cd frontend
npm run build
```
FastAPI sirve automáticamente los archivos compilados de `frontend/dist` en la raíz `/`.
