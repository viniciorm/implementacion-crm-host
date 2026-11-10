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
