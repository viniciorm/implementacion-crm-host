# Bitácora

## Punto de partida

Se desarrolló una interfaz navegable de un CRM con HTML, CSS y JavaScript. Un prototipo de integración contemplaba Firebase Authentication, funciones Node.js y Firestore. El prototipo no llegó a validarse de extremo a extremo con datos reales.

**Aprendizaje:** una interfaz que navega no demuestra que el sistema tenga autenticación, persistencia o autorización. Conviene separar funciones reales, código preparado y componentes demostrativos.

## Decisión: backend independiente en hosting compartido

Se optó por preparar Laravel y una base de datos independiente, conservando la interfaz existente. El hosting comparte infraestructura con otras aplicaciones, por lo que se requiere aislamiento de archivos, base de datos, credenciales, rutas de despliegue y tareas programadas.

**Previsto:** servir únicamente el directorio público de Laravel y mantener código de aplicación, configuración y secretos fuera del área pública. Verificar respaldos, recuperación, configuración PHP, permisos y ejecución de tareas antes de publicar.

## Limitación: sin acceso SSH

El panel del hosting permite administrar archivos y crear trabajos de cron. Se evaluará preparar las dependencias fuera del servidor y subir un paquete adecuado, sin exponer herramientas administrativas en la web.

**Pendiente de comprobación:** versión y extensiones de PHP efectivas tanto en el sitio como en cron, disponibilidad del ejecutable adecuado y compatibilidad real de las dependencias. La configuración mostrada en el panel no demuestra por sí sola la configuración de CLI.

## Seguridad de transporte

Se verificó en el panel la existencia de un certificado válido para el subdominio de trabajo y su renovación automática. Está pendiente comprobar la respuesta HTTPS real y las redirecciones durante el despliegue.

## Cambio de autenticación

La arquitectura cambió de Firebase Authentication a **Google OpenID Connect directo con Laravel**, para reducir componentes. La identidad de Google no concede acceso por sí sola: el servidor deberá permitir solo cuentas autorizadas y aplicar permisos a cada consulta y cambio.

**Pendiente:** implementar y verificar el flujo OAuth/OIDC, la gestión segura de sesiones, la autorización y el cierre de sesión. Los asesores no se habilitarán hasta comprobar el aislamiento de sus datos.

## Fuente de prospectos

La planilla existente tiene pestañas con columnas de nombres y posiciones distintas. Una pestaña carece del encabezado de asignación. La fuente también alimenta un proceso externo de distribución y notificación que debe permanecer intacto.

**Preparado y probado localmente:** normalización PHP de cuatro esquemas de ejemplo, identificación por pestaña e ID, detección de encabezados ambiguos y de IDs ausentes, conservación de valores originales y marca de conciliación para asignaciones no verificadas. Las pruebas no demuestran que exista sincronización real ni persistencia.

Se agregó una validación previa de lote: cuando un identificador se repite en una misma pestaña, se rechazan ambas filas para revisión en vez de elegir una silenciosamente. El resultado es reproducible en las pruebas locales. Esto aún no constituye una sincronización con base de datos.

**Previsto:** lectura de solo acceso, sincronización idempotente, registro de errores sin datos personales y separación entre datos de origen y gestiones comerciales. Nunca inventar asignaciones históricas ni sobrescribir notas durante la sincronización.

## Próximo hito

Preparar e instalar dependencias de Laravel y construir el circuito completo: iniciar sesión, consultar prospectos autorizados, abrir una ficha y guardar una gestión en la base independiente. Probar primero con datos de prueba y revisar la seguridad antes de conectar datos reales.

## Preparación del acceso directo

Se preparó una regla aislada para vincular una identidad previamente verificada con una cuenta ya autorizada y activa. La vinculación debe conservar el identificador estable del proveedor, verificar el correo inicial y rechazar cambios inesperados de identidad. La primera vinculación necesitará una transacción y una restricción única para evitar carreras.

**Estado:** propuesta de código en revisión; los casos unitarios independientes pasaron en un entorno local. Esto no prueba el flujo de autenticación completo. La regla no verifica tokens ni implementa sesiones o inicio de sesión. Quedan pendientes la integración con Laravel, pruebas de extremo a extremo y la configuración del proveedor.

## Prueba local del framework

Se descargó una instalación de prueba del framework en un entorno aislado. El arranque falló al intentar escribir archivos de caché por permisos del entorno de pruebas; no se ha determinado que exista el mismo problema en el hosting. No se ha validado aún la versión definitiva de dependencias para el servidor ni se ha desplegado la aplicación.


## Diagnóstico seguro del hosting compartido sin exposición web

Para determinar las capacidades reales del servidor sin exponer información sensible ni crear endpoints administrativos públicos, se preparó un script de diagnóstico ejecutable exclusivamente vía CLI o tarea de cron temporal:

- Restricción estricta de entorno: valida que `php_sapi_name() === 'cli'`, finalizando inmediatamente si se invoca desde la web.
- Comprobación de binarios: inspecciona rutas estándar del panel (`ea-php84`, `alt-php84`, `composer`) para contrastar el PHP web con el PHP de la línea de comandos.
- Verificación de extensiones: valida la presencia de `pdo_mysql`, `openssl`, `curl`, `mbstring`, `fileinfo` y `intl`.
- Conectividad saliente: evalúa mediante conexiones seguras la disponibilidad de los servicios de identidad y hojas de cálculo.
- Registro privado: escribe los resultados únicamente en una ruta privada fuera del área pública (`public_html`), garantizando que la salida no sea accesible vía navegador.

**Aprendizaje:** en hosting compartido, nunca se deben crear scripts web temporales que ejecuten comandos del sistema o expongan `phpinfo()`. Las tareas de cron con salida a un archivo privado y retiro inmediato posterior ofrecen una vía segura y controlada para auditar el entorno.

## Implementación de Laravel con separación física de directorios

Se construyó la estructura completa de la aplicación backend dentro de la carpeta correspondiente, adoptando un diseño desacoplado para hosting compartido:

- Área privada: contiene el núcleo de la aplicación, controladores, modelos, migraciones, dependencias (`vendor/`), configuración y el archivo `.env`.
- Área pública: contiene exclusivamente el punto de entrada `index.php`, los estilos de marca y el archivo `.htaccess` configurado con `Options -Indexes` y cabeceras de seguridad (`X-Frame-Options`, `X-Content-Type-Options`).
- Resolución dinámica de rutas: `index.php` detecta automáticamente si se encuentra en un entorno de desarrollo local o en la estructura separada de hosting compartido, registrando la ruta pública efectiva sin requerir rutas absolutas cableadas en el código fuente.
- Prevención de fallos de caché: se aseguraron las carpetas internas de `storage/framework/cache/data`, `storage/framework/sessions`, `storage/framework/views` y `storage/logs` con exclusiones adecuadas de control de versiones.

## Integración de autenticación Google OpenID Connect y vinculación atómica

Se implementó el flujo completo de autenticación directa con el proveedor de identidad sin dependencias externas:

- Protección de flujo: generación de un token de estado CSRF criptográfico almacenado en sesión y verificado en el callback.
- Intercambio seguro: canje de código por tokens y verificación de identidad y correo confirmado contra el endpoint oficial del proveedor.
- Vinculación atómica y prevención de colisiones:
  - Solo se admiten correos previamente autorizados y activos en la base de datos.
  - En el primer inicio de sesión, el identificador único y estable del proveedor (`sub`) se vincula dentro de una transacción con bloqueo de fila (`lockForUpdate`), aplicando una restricción de unicidad para impedir carreras concurrentes.
  - En inicios de sesión posteriores, se exige coincidencia estricta del identificador vinculado.
- Gestión de sesiones: regeneración del ID de sesión al autenticar para mitigar ataques de fijación de sesión, y middleware para expulsar de inmediato a usuarios desactivados.

## Sincronización idempotente y persistencia desacoplada

Se implementó la lectura de las hojas de cálculo autorizadas mediante una cuenta de servicio configurada con alcance estricto de solo lectura:

- Canal seguro de lectura: firma de aserción JWT mediante algoritmo RS256 con la clave privada de la cuenta de servicio y alcance exclusivo `spreadsheets.readonly`.
- Validación por lote: detección de identificadores repetidos en una misma hoja, rechazando ambas filas para revisión manual sin aceptar duplicados silenciosamente.
- Idempotencia: cálculo de clave única basada en hoja e ID de origen; ejecuciones posteriores actualizan marcas de tiempo pero conservan intactos los estados comerciales, notas de gestión, reasignaciones internas y seguimientos.
- Trazabilidad y auditoría: registro de cada inicio de sesión, cambio de estado, reasignación, creación de gestión y sincronización en tablas dedicadas.

## Autenticación de identidad robusta y doble verificación concurrente

Para proteger el flujo de autenticación federada sin dependencias externas innecesarias:

- Verificación integral de tokens: validación criptográfica del `id_token` mediante el servicio de comprobación oficial del proveedor, verificando emisor (`iss`), audiencia coincidente con el identificador de cliente (`aud`), expiración (`exp`) y confirmación del correo electrónico (`email_verified`).
- Verificación cruzada: cotejo estricto del identificador único (`sub`) obtenido del `id_token` frente a los datos devueltos por el endpoint de perfil (`userinfo`).
- Bloqueo pesimista con revalidación post-bloqueo (doble comprobación):
  - Al recibir una solicitud de vinculación inicial, se obtiene un bloqueo exclusivo de fila (`lockForUpdate`).
  - Inmediatamente tras adquirir el bloqueo, se revalida que el usuario permanezca activo, no haya sido vinculado concurrentemente por otra solicitud paralela y no colisione con otra cuenta.
  - Esto garantiza que dos solicitudes simultáneas no puedan sobrescribir la identidad ni dejar el registro en un estado inconsistente.

## Mapeo determinista de asesores y actualización selectiva de origen

En la integración con hojas de cálculo externas:

- Preservación de autoría de origen: el valor original del asesor asignado en la hoja se almacena intacto en un atributo dedicado (`source_assigned_to`).
- Mapeo explícito y controlado: solo se vinculan prospectos a asesores del sistema formalmente registrados y activos. Todo registro con asesor no habilitado o desconocido se marca de forma transparente como pendiente de conciliación (`reconciliation_pending = true`), sin asignación interna automática ni inferencias heurísticas.
- Actualización selectiva de origen: al re-ejecutar la sincronización, los atributos de contacto que hayan mutado en la hoja de origen (nombre, teléfono, correo, campaña, estado de notificación) se actualizan en el modelo del prospecto, pero los datos comerciales generados en el sistema (estado comercial, reasignación a usuario interno, notas de actividad, llamadas y compromisos) quedan estrictamente protegidos contra sobreescritura.

## Tareas programadas seguras y compatibilidad de plataforma

- Comando CLI con bloqueo de aislamiento: se construyó un comando de consola dedicado para sincronización programada, protegido mediante bloqueo atómico (`Cache::lock`) para prevenir ejecuciones superpuestas ante tareas que tomen más tiempo del intervalo programado. Las omisiones por concurrencia se registran con severidad de auditoría.
- Fijación de plataforma PHP: en entornos de alojamiento compartido donde PHP 8.4 puede presentarse en versiones de parche como 8.4.0, se fijó explícitamente `"platform": { "php": "8.4.0" }` en la configuración del gestor de dependencias para forzar la resolución de paquetes a versiones plenamente compatibles (evitando subdependencias que exijan PHP >= 8.4.1).

## Verificación integral mediante pruebas automatizadas

Se construyó y ejecutó una suite de pruebas con 37 casos automatizados y 135 aserciones que validaron:

1. Rechazo de accesos no autenticados en todas las rutas protegidas.
2. Rechazo inmediato de usuarios inactivos o no autorizados.
3. Validación estricta de estado CSRF en el flujo de identidad.
4. Validación formal del token de identidad y rechazo de tokens expirados o con audiencia inválida.
5. Doble comprobación post-bloqueo en vinculación concurrente y prevención de carreras.
6. Aislamiento estricto entre asesores: imposibilidad de que un asesor consulte o modifique prospectos asignados a otro.
7. Mapeo explícito de asesores de origen y marcado automático de casos ambiguos para conciliación.
8. Actualización de atributos de contacto modificados en la fuente sin alteración de datos comerciales del sistema.
9. Rechazo de identificadores duplicados a nivel de lote.
10. Idempotencia y conservación de notas, llamadas, estados y seguimientos tras sincronizaciones repetidas.
11. Verificación de que el cliente de hojas realiza exclusivamente peticiones GET y nunca escrituras.
12. Prevención de ejecuciones superpuestas en el comando CLI mediante bloqueo atómico.
13. Comprobación de que la configuración, `.env` y archivos privados no residen en el directorio público.
