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

## Validación de identidad offline mediante JWKS y rotación de claves

Para eliminar la dependencia de peticiones HTTP a endpoints secundarios de depuración en producción:

- Validación criptográfica offline: se sustituyó la consulta a endpoints de inspección por verificación directa mediante la biblioteca mantenida `firebase/php-jwt` (v7.2.1), verificando la firma digital RS256 con los certificados públicos oficiales del proveedor (JWKS).
- Gestión de claves y rotación dinámica:
  - Las claves públicas se obtienen del endpoint oficial de certificados y se almacenan en caché respetando el valor `max-age` de la cabecera `Cache-Control`.
  - Si un token recibido contiene un identificador de clave (`kid`) ausente en la caché (por ejemplo, debido a rotación periódica de claves por parte del proveedor), el sistema fuerza automáticamente la recarga del conjunto JWKS antes de evaluar el token.
  - Si el `kid` no existe tras la recarga, o la firma es inválida, se rechaza de inmediato.
- Comprobación estricta de claims:
  - Emisor (`iss`): coincidencia exacta con los dominios autorizados del proveedor.
  - Audiencia (`aud`): correspondencia estricta con el identificador de cliente configurado.
  - Expiración (`exp`): rechazo de tokens vencidos.
  - Correo electrónico confirmado (`email_verified`): rechazo de cuentas con verificación pendiente.
  - Identificador único (`sub`): validación contra registros previos y bloqueo de carreras concurrentes.

## Bloqueo compartido entre procesos desacoplados y control de fallos

Para garantizar que procesos web independientes y tareas programadas por línea de comandos no colisionen:

- Controlador de caché entre procesos: se seleccionó el controlador de base de datos (`database` con tablas de caché y bloqueos atómicos), garantizando coherencia transaccional entre peticiones web (FPM/servidor web) y ejecuciones de consola (CLI/Cron) en hosting compartido.
- Bloqueo compartido unificado: la sincronización manual desde la interfaz administrativa y la sincronización periódica por comando de consola comparten la misma clave de bloqueo atómico (`crm_sheets_sync_shared_lock`).
- Pruebas de contención y resiliencia:
  - Una sincronización manual es rechazada si el comando programado está en ejecución.
  - El comando programado omite su ejecución y registra evento de auditoría si la sincronización manual está activa.
  - Ante cualquier excepción imprevista (fallo de red, archivo no encontrado o respuesta corrupta de la API externa), el bloqueo se libera estrictamente en el bloque `finally`, evitando bloqueos residuales o bloqueos permanentes innecesarios.

## Reconciliación determinista con bloqueo pesimista de fila

Para proteger la integridad de las asignaciones ante operaciones simultáneas en la base de datos:

- Bloqueo exclusivo dentro de transacción: en lugar de confiar en datos leídos previamente fuera del contexto transaccional, el proceso de reconciliación vuelve a consultar y bloquea explícitamente tanto el registro del prospecto como su fila de gestión (`SELECT ... FOR UPDATE` en `leads` y `lead_management`).
- Detección de asignación manual concurrente: si un administrador o asesor asigna un prospecto manualmente mientras el proceso de reconciliación evalúa el lote, al adquirir el bloqueo exclusivo el proceso detecta que `assigned_user_id` ya no es nulo, preserva intacta la asignación manual previa y marca el prospecto como resuelto sin sobrescribir con el asesor automático.
- Prueba con dos procesos concurrentes sincronizados para MySQL / MariaDB (InnoDB):
  - Exigencia estricta de base de datos aislada: la prueba requiere explícitamente una base de datos de pruebas configurada (ej. `nombre_crm_test`) y prohíbe de forma terminante usar la base de datos principal (`DB_DATABASE`) como alternativa o destino, previniendo cualquier afectación a datos del sistema.
  - Se implementó una prueba reproducible con dos subprocesos del sistema operativo (`proc_open`) conectados a InnoDB.
  - El Proceso A inicia transacción y adquiere `SELECT ... FOR UPDATE` sobre la fila de gestión.
  - El Proceso B inicia transacción concurrente e intenta ejecutar el servicio real de reconciliación (`AdvisorAssignmentResolver::reconcilePendingLeads`), quedando físicamente retenido en `LOCK WAIT` por el motor de base de datos.
  - El Proceso A retiene el bloqueo intencionalmente por 1.4 segundos (tiempo medible), realiza la asignación manual a un asesor y ejecuta `COMMIT`.
  - El Proceso B se desbloquea inmediatamente tras el `COMMIT`, mide el tiempo de espera efectivo (demostrando que esperó en InnoDB durante la retención del bloqueo), ejecuta la reconciliación y preserva la asignación manual previa sin sobrescribirla.
  - Ejecución segura sin exposición de credenciales: las credenciales se cargan desde un archivo seguro con permisos restringidos (600) y se transmiten a los subprocesos mediante variables de entorno del sistema operativo (`proc_open`), impidiendo que aparezcan en la lista de procesos (`ps aux`), en la interfaz de tareas programadas del panel de control o en los registros de salida.
  - Distinción estricta de pruebas omitidas: si el servidor MySQL no se encuentra disponible o no se configuró una base de datos de pruebas aislada, la prueba se reporta de forma inequívoca como **OMITIDA (SKIPPED)** tanto en PHPUnit como en el ejecutable CLI (`[PRUEBA OMITIDA - NO APROBADA]`), impidiendo presentarla falsamente como aprobada en entornos SQLite en memoria.

## Mitigación de tareas extensas (>600s), renovación estricta y uso de la opción forzada

- Bloqueo compartido en base de datos (`cache_locks`): la sincronización manual (Web HTTP) y la sincronización automática (CLI Cron) utilizan explícitamente el almacén de base de datos (`DatabaseLock`), garantizando que dos procesos de sistema operativo independientes compartan el mismo cerrojo atómico y dispongan de la funcionalidad nativa de renovación (`refresh()`).
- Verificación estricta de propiedad y renovación (`refresh()`):
  - El método `refresh()` de `DatabaseLock` ejecuta una actualización atómica con condición de vigencia y propietario. Si el bloqueo expiró o fue adquirido por otro proceso, `refresh()` retorna estrictamente `false`.
  - El sistema nunca silencia un fallo de renovación. Ante `false`, registra inmediatamente el evento de auditoría `sync.lock_lost`, interrumpe el procesamiento y arroja una excepción en tiempo de ejecución, lo que dispara un `rollback` transaccional en la base de datos para prevenir colisiones o escrituras sin exclusividad.
- Renovación periódica (heartbeat) intra-hoja y entre hojas: para prevenir expiraciones durante la ingesta de grandes volúmenes de datos, el pulso de renovación se ejecuta antes de consultar cada hoja autorizada y periódicamente dentro de cada hoja (cada 25 registros procesados).
- Uso controlado de la opción `--force`:
  - Propósito: diseñada exclusivamente para recuperación administrativa cuando un proceso previo fue interrumpido abruptamente por el sistema operativo (ej. falta de memoria, reinicio de servidor) y dejó un bloqueo huérfano en la tabla `cache_locks`.
  - Acción: libera forzosamente el bloqueo residual, audita el evento `sync.forced_lock_release` y adquiere un nuevo bloqueo limpio.
  - Advertencia: jamás debe utilizarse en trabajos automáticos de Cron, ya que anularía la protección de concurrencia si una tarea legítima se encuentra ejecutándose.

## Verificación integral mediante pruebas automatizadas

Se construyó y ejecutó una suite de pruebas con 60 casos automatizados y 231 aserciones (59 aprobadas, 1 omitida por requerir MySQL/MariaDB aislado):

1. Rechazo de accesos no autenticados en todas las rutas protegidas.
2. Rechazo inmediato de usuarios inactivos o no autorizados.
3. Validación estricta de estado CSRF en el flujo de identidad.
4. Validación criptográfica offline de `id_token` mediante certificados públicos JWKS.
5. Rechazo de firmas manipuladas o inválidas.
6. Rechazo de emisores (`iss`) y audiencias (`aud`) no autorizados.
7. Rechazo de tokens expirados o con correos no confirmados.
8. Detección y manejo exitoso de rotación de claves públicas del proveedor.
9. Doble comprobación post-bloqueo en vinculación concurrente y prevención de carreras.
10. Aislamiento estricto entre asesores (HTTP 403 verificado).
11. Mapeo explícito de asesores de origen y marcado automático de casos ambiguos para conciliación.
12. Reconciliación explícita con bloqueo de fila y re-consulta transaccional.
13. Preservación estricta de asignaciones manuales concurrentes ante carreras con el proceso de reconciliación.
14. Actualización de atributos de contacto modificados en la fuente sin alteración de datos comerciales del sistema.
15. Rechazo de identificadores duplicados a nivel de lote.
16. Idempotencia y conservación de notas, llamadas, estados y seguimientos tras sincronizaciones repetidas.
17. Verificación de que el cliente de hojas realiza exclusivamente peticiones GET y nunca escrituras.
18. Bloqueo compartido entre sincronización manual y comando CLI en controlador de base de datos real.
19. Exclusión mutua comprobada entre dos procesos independientes que intentan adquirir el mismo bloqueo en `cache_locks`.
20. Comprobación de que la expiración del bloqueo en base de datos permite la adquisición por un nuevo proceso e invalida la titularidad del propietario anterior.
21. Renovación de bloqueo (heartbeat) intra-hoja y entre hojas para operaciones extensas (>600s).
22. Interrupción inmediata y registro de auditoría (`sync.lock_lost`) si la renovación del bloqueo falla durante la ejecución.
23. Reversión completa de base de datos (`rollback`) si se interrumpe la sincronización por pérdida de bloqueo.
24. Liberación forzada controlada mediante `--force` ante bloqueos huérfanos con registro de auditoría.
25. Liberación garantizada del bloqueo de sincronización ante excepciones y fallos simulados en bloque `finally`.
26. Comprobación de que la configuración, `.env` y archivos privados no residen en el directorio público.
27. Prueba de concurrencia y bloqueo InnoDB entre 2 subprocesos independientes del SO (omitida en SQLite; lista para validación en base MySQL aislada).

## Estado del desarrollo y limitaciones pendientes

- **Commit de referencia:** `9e2a55f498843941d1c7751333b68a3de92f718d` (rama de características del CRM).
- **Limitaciones pendientes:**
  - Las tareas periódicas en Cron se encuentran temporalmente desactivadas en el servidor hasta completar la validación manual inicial en producción.
  - Los asesores con asignaciones históricas no confirmadas permanecen inhabilitados; sus registros se encuentran marcados como pendientes de conciliación.
  - La prueba concurrente de dos procesos en MySQL/MariaDB requiere la provisión de la base de datos de pruebas aislada en el servidor para pasar de omitida a ejecutada en ese motor específico.
  - La ejecución de migraciones en la base de datos final y la generación de la clave criptográfica única de la aplicación se realizarán directamente en el servidor de destino previo al inicio de operaciones.
