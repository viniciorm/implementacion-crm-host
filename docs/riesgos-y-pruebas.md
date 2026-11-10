# Riesgos y pruebas previstas

| Riesgo | Respuesta prevista | Estado |
|---|---|---|
| Mezclar aplicaciones del mismo hosting | Carpetas, base y credenciales independientes; comprobar límites reales del proveedor. | Pendiente de verificar |
| Exponer configuración o credenciales | Mantener archivos privados fuera del directorio público y probar acceso denegado. | Pendiente |
| PHP web y cron distintos | Verificar versión y extensiones de ambos antes de programar tareas. | Pendiente |
| Cambio de orden de columnas | Buscar columnas por encabezado y rechazar ambigüedades. | Comprobado en pruebas unitarias locales |
| Identificadores ausentes o duplicados | Rechazar o informar conflictos; no inventar identidad. | Validación local de esquema y lote; falta prueba con base de datos |
| Asignaciones no verificables | Marcar para conciliación sin inferir responsables. | Comprobado en normalizador local |
| Sobrescribir gestiones al importar | Separar campos de origen y datos comerciales; probar sincronizaciones repetidas. | Pendiente |
| Acceso entre asesores | Aplicar restricciones en consultas y escrituras del servidor; ensayar accesos cruzados. | Regla aislada probada; falta integración |
| Acceso de una cuenta no autorizada | Validar identidad y lista de usuarios en el servidor. | Pendiente |
| Falla de fuente externa | Mostrar estado de sincronización y conservar los últimos datos disponibles sin inventar actualizaciones. | Pendiente |
| Pérdida de datos | Programar respaldos independientes y ensayar restauración. | Pendiente |

Las pruebas con datos reales no deben realizarse hasta validar permisos, HTTPS, respaldo y aislamiento en un entorno apropiado.
