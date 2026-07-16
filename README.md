# wp-inspector (Lite Edition) 🔎

Script de diagnóstico (*single-file*) para la evaluación de rendimiento y seguridad en entornos WordPress.

> **Nota:** Esta es una edición simplificada para propósitos de demostración. La versión comercial incluye 34 módulos operativos (detección de malware, validación de cumplimiento normativo, entre otros). Los módulos sensibles han sido reemplazados por *stubs*, manteniendo intacta la arquitectura base del software.

## El Problema
Auditar un servidor comprometido o con alta carga de CPU instalando plugins adicionales es contraproducente. Los plugins convencionales alteran la base de datos, generan *bloat* y fallan si el CMS está inestable. El objetivo de este proyecto es diagnosticar la infraestructura sin alterar su estado actual.

## La Solución
`wp-inspector` opera estrictamente en modo de lectura (*Read-Only*). Se despliega vía SFTP, se ejecuta mediante una única petición HTTP validada y genera la telemetría requerida. Tras el análisis, el archivo se elimina. El impacto en el servidor y la base de datos es nulo.

## Arquitectura Técnica
El código está diseñado para no interrumpir su ejecución, incluso en entornos severamente degradados. Implementa los siguientes patrones operativos:

* **Tolerancia a fallos (Circuit Breaker):** Si la conexión a la base de datos falla durante la ejecución de un módulo, la excepción es capturada y aislada. El orquestador permite que el resto de los módulos (ej. análisis del sistema de archivos) continúe su ejecución sin devolver un Error 500 fatal.
* **Autenticación Zero Trust:** La ejecución requiere un token validado mediante `hash_equals` para mitigar ataques de temporización (*timing attacks*). Ante accesos no autorizados, el script devuelve un encabezado `404 Not Found` nativo para evadir escáneres automatizados.
* **Modo degradado (Shims):** Si el núcleo del CMS está corrupto, la sonda inyecta *polyfills* para emular las funciones nativas requeridas y lograr completar el diagnóstico.
* **Scoring Multi-vector:** Utiliza un sistema heurístico de pesos. Las alertas críticas solo se disparan si múltiples vectores coinciden, reduciendo la tasa de falsos positivos en el reporte final.

## Módulos de Demostración
Para ilustrar la interacción con el entorno, esta versión expone dos módulos funcionales:
1. `wpo.autoload`: Analiza el peso en bytes de la tabla `wp_options` para identificar cuellos de botella que impactan directamente el TTFB (*Time to First Byte*).
2. `sec.filesystem`: Escanea la exposición pública de directorios de control de versiones (ej. `.git`) y la presencia de archivos sensibles.

## Implementación Básica
1. Configurar un token seguro en la constante `PROBE_DEFAULT_TOKEN`.
2. Subir `wp-inspector-lite.php` a la raíz pública del servidor.
3. Ejecutar en el navegador o consola: `https://tudominio.com/wp-inspector-lite.php?token=TU_TOKEN`
4. Analizar la salida estructurada en formato JSON.
5. **Eliminar el archivo** del servidor tras finalizar la auditoría.
