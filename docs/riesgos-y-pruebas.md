# Riesgos y pruebas previstas

| Riesgo | Respuesta prevista | Estado |
|---|---|---|
| Mezclar aplicaciones del mismo hosting | Carpetas, base y credenciales independientes; comprobar límites reales del proveedor. | Arquitectura desacoplada preparada; pendiente validación final en hosting |
| Exponer configuración o credenciales | Mantener archivos privados fuera del directorio público y probar acceso denegado. | Comprobado en suite de pruebas de aislamiento público/privado |
| PHP web y cron distintos | Verificar versión y extensiones de ambos antes de programar tareas. | Diagnóstico CLI seguro preparado para ejecución vía tarea programada privada |
| Cambio de orden de columnas | Buscar columnas por encabezado y rechazar ambigüedades. | Comprobado en pruebas unitarias locales |
| Identificadores ausentes o duplicados | Rechazar o informar conflictos; no inventar identidad. | Comprobado en pruebas de lote y persistencia con base de datos |
| Asignaciones no verificables | Marcar para conciliación sin inferir responsables. | Comprobado en normalizador y esquema de base de datos |
| Sobrescribir gestiones al importar | Separar campos de origen y datos comerciales; probar sincronizaciones repetidas. | Comprobado en prueba de sincronización repetida e idempotente |
| Acceso entre asesores | Aplicar restricciones en consultas y escrituras del servidor; ensayar accesos cruzados. | Comprobado en suite de pruebas de autorización (403 Forbidden verificado) |
| Acceso de una cuenta no autorizada | Validar identidad y lista de usuarios en el servidor. | Comprobado en pruebas de flujo OIDC y middleware de sesión activa |
| Falla de fuente externa | Mostrar estado de sincronización y conservar los últimos datos disponibles sin inventar actualizaciones. | Comprobado mediante registro transaccional en tabla de ejecuciones de sincronización |
| Pérdida de datos | Programar respaldos independientes y ensayar restauración. | Procedimiento documentado; pendiente comprobación operativa en panel |

Las pruebas con datos reales no deben realizarse hasta validar permisos, HTTPS, respaldo y aislamiento en un entorno apropiado.
