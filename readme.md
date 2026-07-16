# wp-inspector (Lite Edition) 🔎

**Script de diagnóstico y auditoría *Single-File* para entornos WordPress.**

> **Nota:** Este repositorio contiene la **Edición Lite** (demostrativa) de `wp-inspector`. La versión comercial completa incluye 34 módulos de detección, firmas de malware propietarias y auditoría técnica de la Ley 21.719 (Chile). Para proteger la propiedad intelectual, los módulos heurísticos y normativos sensibles han sido reemplazados por *stubs* en esta demostración pública.

## 📌 El Problema
Las auditorías tradicionales de WordPress dependen de plugins pesados que escriben en la base de datos, dejan basura (bloat) y requieren acceso completo al panel de administración. En escenarios de respuesta a incidentes (compromiso por malware) o cuellos de botella severos, instalar más plugins degrada aún más el entorno.

## 🚀 La Solución
`wp-inspector` es una sonda de diagnóstico que opera bajo un modelo **Zero Trust** y de **Lectura Pura (Read-Only)**. Se sube vía SFTP, se ejecuta en una sola petición HTTP y genera telemetría sin modificar la base de datos.

## 🏗️ Conceptos Arquitectónicos (Ingeniería de Software)
Este proyecto no es un script procedural tradicional; implementa patrones de diseño corporativos agnósticos para garantizar resiliencia en servidores críticos:

* **Kernel Zero Trust:** Autenticación estricta por token de 64 caracteres. La validación se realiza en tiempo constante (`hash_equals`) para mitigar *timing attacks*. Si la sonda no está configurada, devuelve un `404 Not Found` nativo para evadir escáneres automatizados.
* **Pipeline Engine & Circuit Breaker:** Un orquestador central aísla la ejecución de cada módulo. Si la base de datos está caída y un módulo lanza una excepción, el *Circuit Breaker* lo captura, aísla el fallo y permite que los módulos del sistema de archivos sigan operando.
* **Scoring Multi-vector:** Sistema heurístico de acumulación de puntajes para reducir la fatiga de alertas y falsos positivos.
* **Shims Defensivos:** Soporte para operar en "Modo Degradado" inyectando *polyfills* si el core del CMS está corrupto.

## 🧩 Módulos Demostrativos Incluidos
1. `wpo.autoload`: Análisis de rendimiento y bloat en tabla de opciones (`wp_options`).
2. `sec.filesystem`: Detección de archivos de control de versiones (`.git`) y directorios sensibles expuestos públicamente.
*(Nota: Módulos de inyección de contenido y compliance están presentes como stubs).*

## ⚙️ Uso Básico
1. Configurar un token seguro en la constante `PROBE_TOKEN`.
2. Subir a la raíz del servidor vía SFTP.
3. Ejecutar: `https://midominio.com/wp-inspector-lite.php?token=TU_TOKEN`
4. Eliminar el archivo tras finalizar el triage.