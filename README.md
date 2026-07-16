# wp-inspector (Lite Edition) 🔎

Un script de un solo archivo para hacer *triage* rápido de rendimiento y seguridad en entornos WordPress.

> **Nota:** Esta es una versión reducida pensada para mi portafolio. La versión completa de esta herramienta tiene 34 módulos (incluyendo detección de malware y validación legal). Para proteger ese trabajo, dejé un par de módulos funcionales aquí y el resto como cascarones (*stubs*), pero la estructura base del código es la real.

## ¿Por qué hice esto?
Cuando un servidor con WordPress está comprometido o tiene el CPU al 100%, instalarle *otro plugin* para auditarlo es una pésima idea. Los plugins modifican la base de datos, dejan basura y muchas veces ni siquiera cargan si el sitio está caído. 

Necesitaba una forma de diagnosticar sitios enfermos sin tocarlos.

## ¿Qué hace?
`wp-inspector` es una sonda de **solo lectura**. La subes por SFTP a la raíz del sitio, la ejecutas en tu navegador pasándole un token, te escupe la telemetría y la borras. Cero impacto en la base de datos y cero rastros.

## ¿Cómo funciona bajo el capó?
El código está estructurado para no romperse, incluso si el servidor está en las últimas. Implementé un par de patrones clave:

* **Tolerancia a fallos (Circuit Breaker):** Si el orquestador intenta leer la base de datos y esta no responde, el script no arroja un Error 500 fatal. Captura la excepción, aísla ese módulo y sigue escaneando el resto del sistema (como los archivos físicos).
* **Autenticación estricta:** No basta con adivinar la URL del script. Requiere un token validado con `hash_equals` (para evitar *timing attacks*). Si entras sin el token correcto, el script simula un `404 Not Found` nativo para despistar a los bots.
* **Modo de supervivencia:** Si el core del CMS está corrupto y no carga sus funciones, la sonda inyecta *polyfills* (funciones de rescate) para poder terminar el diagnóstico.
* **Anti-Falsos Positivos:** Usa un sistema de puntaje por pesos. Un problema se marca como "Crítico" solo si coinciden varios vectores al mismo tiempo, evitando que te llene de alertas inútiles.

## Módulos de demostración en este repo
Para mostrar cómo interactúa con el entorno, dejé expuestos estos dos módulos:
1. `wpo.autoload`: Revisa el peso en bytes de la tabla `wp_options` para detectar cuellos de botella en el TTFB (Time to First Byte).
2. `sec.filesystem`: Escanea si hay directorios de control de versiones (como `.git`) expuestos públicamente.

## Uso rápido
1. Edita el archivo y cambia la constante `PROBE_DEFAULT_TOKEN` por un token seguro.
2. Sube `wp-inspector-lite.php` a la carpeta `public_html` (o equivalente).
3. Abre en tu navegador: `https://tudominio.com/wp-inspector-lite.php?token=TU_TOKEN`
4. Lee el JSON devuelto.
5. **Borra el archivo** cuando termines.
