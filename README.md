# 🔄 Automatización ETL con SSIS: Carga de Clientes y Control de Flujo

En este proyecto desarrollé un paquete de Integration Services (SSIS) para resolver un problema típico de integración de datos: automatizar la ingesta de un archivo plano de clientes hacia una base de datos relacional, asegurando la limpieza del entorno post-procesamiento y un control estricto de errores.

Diseñé el flujo dividiéndolo en una lógica de procesamiento de datos (*Data Flow*) y una secuencia de control (*Control Flow*) que reacciona según el resultado de la carga.

---

## 🛠️ ¿Cómo funciona el flujo? (Paso a Paso)

El paquete sigue una lógica secuencial y condicional muy clara, la cual estructuré de la siguiente manera:

### 1. La Ingesta y Transformación de Datos (Data Flow)
Todo el proceso inicia con la tarea principal: **`Cargar Clientes TXT a SQL`**. Si entramos al detalle de este componente, el flujo interno que diseñé realiza lo siguiente:
* **Origen (`Customers en TXT`):** Extrae la información cruda desde un archivo de texto plano.
* **Transformación (`Conversión de datos`):** Decidí aplicar este componente intermedio para estandarizar los tipos de datos en tránsito, previniendo conflictos de codificación o longitud antes de impactar la base de datos.
* **Destino (`Tabla Customers`):** Inyecta los datos limpios en la tabla final de SQL Server mediante una conexión OLE DB.

### 2. Gestión de Excepciones (El Camino del Error ❌)
Si la tarea de carga descrita arriba falla por cualquier motivo (un archivo corrupto, un cambio de estructura o pérdida de conexión), el flujo se desvía inmediatamente por las restricciones de precedencia rojas hacia dos tareas en paralelo:
* **`Mensaje Error`:** Lanza una alerta local en el sistema para auditoría inmediata.
* **`Email Error`:** Dispara una notificación vía SMTP para avisar al administrador del sistema sobre el fallo.

### 3. Post-procesamiento y Éxito (El Camino Correcto ✅)
Si la carga de datos finaliza con éxito, el paquete avanza por el camino verde, ejecutando una cadena de ordenamiento de archivos y notificaciones:
1. **`Crear Directorio`:** Primero, el sistema crea una carpeta específica para organizar los archivos históricos.
2. **`Mover TXT a procesados`:** Una vez asegurada la carpeta, muevo el archivo `.txt` original a ese nuevo directorio. Esto evita que el mismo archivo vuelva a ser procesado por error en la próxima ejecución.
3. **Cierre y Alertas:** Una vez que el archivo fue archivado de forma segura, se ejecutan en paralelo las tareas de confirmación: un archivo.txt con el nombre de  (`Mensaje Correcto`) y un correo de notificación (`Email Correcto`).

---

## 💻 Tecnologías aplicadas
* **Integration Services (SSIS)** - Diseño del flujo general y lógica ETL.
* **Visual Studio (SSDT)** - Entorno de desarrollo para la edición y depuración del proyecto ETL.
* **SQL Server / OLE DB** - Almacenamiento y destino de los datos.


---

## 🚀 Notas de Configuración
Para replicar o probar este paquete en otro entorno, es necesario ajustar los **Connection Managers** en Visual Studio:
* Cambiar la ruta del origen `Customers` para que apunte a un archivo local de prueba.
* Actualizar el destino `LocalHost.Supermercado` con las credenciales de tu instancia local de SQL Server.
* Configurar el servidor de salida en el `Administrador de conexiones SMTP` para habilitar el envío de correos.
