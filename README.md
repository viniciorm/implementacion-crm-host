# Implementación de un CRM en hosting compartido

Bitácora de aprendizaje sobre la implementación progresiva de un CRM privado en un hosting compartido, aprovechando una interfaz existente y separando la aplicación pública, la configuración y la base de datos.

## Objetivo

Documentar decisiones, pruebas, dificultades y soluciones verificadas durante la construcción. Este repositorio contiene únicamente documentación genérica; no incluye el código del cliente, datos de prospectos, nombres, dominios, identificadores de servicios ni credenciales.

## Arquitectura prevista

- **Interfaz:** HTML, CSS y JavaScript responsive existentes.
- **Aplicación:** Laravel en una instalación independiente.
- **Datos comerciales:** base MySQL o MariaDB exclusiva.
- **Acceso:** Google OpenID Connect directo con autorización en Laravel; no se utiliza Firebase.
- **Captación:** lectura controlada de una planilla externa; sus registros se sincronizan sin modificar la fuente.
- **Ejecución periódica:** cron del hosting, previa validación del ejecutable de PHP y de los permisos.

La integración completa todavía no se ha construido ni validado en producción. Las decisiones siguientes son un plan, salvo que la bitácora indique expresamente lo contrario.

## Índice

- [Bitácora del proyecto](docs/bitacora.md)
- [Riesgos y comprobaciones](docs/riesgos-y-pruebas.md)
- [Reglas para contribuir](CONTRIBUTING.md)

## Alcance y privacidad

Los ejemplos se mantienen abstractos. Los datos reales, nombres de personas y organizaciones, dominios, direcciones de correo, capturas identificables, IDs de hojas, rutas reales de servidores, secretos y datos de clientes no deben formar parte de este repositorio ni de su historial.

## Licencia

Documentación publicada bajo Creative Commons Attribution 4.0 International (CC BY 4.0). Consulte el archivo LICENSE.
