# Documentación Técnica: Vulnerabilidades de Subida de Archivos (File Upload Vulnerabilities)

**Módulo:** PortSwigger Web Security Academy  
**Autor:** Luis Felipe  
**Fecha:** Diciembre 2026  
**Propósito:** Guía técnica y base de conocimiento para auditorías Red Team y prácticas de desarrollo seguro (GitOps Knowledge Base).

---

## 📋 Resumen Ejecutivo

La carga insegura de archivos ocurre cuando un servidor web permite a los usuarios subir archivos a su sistema de archivos sin validar adecuadamente sus atributos como el nombre, la extensión, el contenido o el tamaño. La falta de estos controles puede derivar desde la lectura no autorizada de archivos sensibles hasta la **Ejecución Remota de Código (RCE)** en el servidor de aplicaciones.

---

## 🎯 Sección 1: Vectores Prácticos (Laboratorios Completados)

### Vector A: Ausencia Absoluta de Controles (Unrestricted File Upload)

#### 1. Descripción y Falla de Seguridad
El servidor no implementa ningún mecanismo de verificación en el formulario de subida ni en el procesamiento posterior. Permite la recepción directa de scripts ejecutables en el servidor (como PHP) y los almacena dentro de un directorio web accesible públicamente con permisos de ejecución activos.

#### 2. Metodología de Explotación y Payload
* **Payload PHP enviado (`exploit.php`):**
  ```php
  <?php echo file_get_contents('/home/carlos/secret'); ?>
  ```
* **Paso a paso:**
  1. Se envió una petición `POST` al endpoint de subida adjuntando el archivo `exploit.php`.
  2. El servidor respondió confirmando la subida en el directorio de almacenamiento estático (ej. `/files/avatars/exploit.php`).
  3. Se ejecutó una petición `GET` a la ruta del archivo subido:
     ```http
     GET /files/avatars/exploit.php HTTP/1.1
     Host: target-app.web-security-academy.net
     ```
  4. El intérprete de PHP ejecutó la función y devolvió el contenido del archivo secreto `/home/carlos/secret` en el cuerpo de la respuesta HTTP, logrando **RCE**.

#### 3. Impacto
Ejecución arbitraria de código del lado del servidor con los privilegios del usuario del servicio web (`www-data` o similar).

---

### Vector B: Validación Débil Basada en Metadatos de Cabecera (MIME-Type Bypass)

#### 1. Descripción y Falla de Seguridad
El servidor intenta validar la legitimidad del archivo inspeccionando únicamente la cabecera HTTP `Content-Type` de la petición multipart enviada por el cliente. No realiza una inspección del contenido real del archivo (*Magic Bytes*) ni de la extensión en el servidor.

#### 2. Metodología de Explotación y Payload
* **Interceptación y Modificación con Burp Suite:**
  Al intentar subir el archivo `exploit.php`, el navegador envía por defecto la cabecera `Content-Type: application/x-php` o `text/x-php`.
* **Modificación del Payload HTTP:**
  ```http
  POST /my-account/avatar HTTP/1.1
  Host: target-app.web-security-academy.net
  Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryX

  ------WebKitFormBoundaryX
  Content-Disposition: form-data; name="avatar"; filename="exploit.php"
  Content-Type: image/jpeg

  <?php echo file_get_contents('/home/carlos/secret'); ?>
  ------WebKitFormBoundaryX--
  ```
* **Paso a paso:**
  1. Interceptación de la petición `POST` en Burp Suite.
  2. Cambio de `Content-Type: application/x-php` a `Content-Type: image/jpeg`.
  3. Envío de la petición y posterior solicitud `GET /files/avatars/exploit.php` para desencadenar la ejecución del script.

---

### Vector C: Escape de Directorio por Sanitización Defectuosa (Path Traversal)

#### 1. Descripción y Falla de Seguridad
El servidor deshabilita la ejecución de scripts en el directorio público predeterminado donde guarda los archivos subidos. Sin embargo, intenta sanitizar el parámetro `filename` para evitar subidas a directorios superiores mediante un filtro débil que puede ser eludido con codificación de URL (*URL Encoding*) o secuencias mal formadas.

#### 2. Metodología de Explotación y Payload
* **Proceso de Bypass de la Sanitización:**
  1. Intentos iniciales con `../exploit.php` fueron stripped/sanitizados por el servidor.
  2. Secuencia codificada parcialmente con URL Encoding (`..%2fexploit.php`) eliminaba caracteres esenciales.
  3. Secuencia final utilizada en la cabecera `Content-Disposition`:
     ```http
     Content-Disposition: form-data; name="avatar"; filename="..%2fexploit.php"
     ```
* **Paso a paso:**
  1. Envío de la petición con la ruta manipulada para forzar al servidor a escribir el archivo un nivel por encima del directorio restringido (ej. escribir en `/files/exploit.php` en vez de `/files/avatars/exploit.php`).
  2. Ejecución mediante petición `GET /files/exploit.php`, logrando leer `/home/carlos/secret`.

---

### Vector D: Anulación de Listas Negras e Inyección de Configuraciones (`.htaccess`)

#### 1. Descripción y Falla de Seguridad
El servidor utiliza un enfoque defensivo basado en **listas negras** para bloquear extensiones conocidas como `.php`. Sin embargo, los servidores de archivos (como Apache) permiten la anulación de directivas de configuración por directorio mediante archivos especiales como `.htaccess`. Si no se prohíbe la subida de estos archivos de configuración, un atacante puede redefinir cómo el servidor procesa archivos con extensiones aparentemente inofensivas.

#### 2. Metodología de Explotación y Payload
* **Fase 1: Subida del archivo `.htaccess` customizado:**
  Se sube un archivo llamado `.htaccess` asignando el tipo MIME de PHP a una extensión personalizada (`.lfe`):
  ```http
  POST /my-account/avatar HTTP/1.1
  Content-Disposition: form-data; name="avatar"; filename=".htaccess"
  Content-Type: text/plain

  AddType application/x-httpd-php .lfe
  ```

* **Fase 2: Subida y Ejecución del Payload:**
  Posteriormente, se sube el payload codificado en PHP pero guardado con la extensión autorizada `.lfe`:
  ```http
  POST /my-account/avatar HTTP/1.1
  Content-Disposition: form-data; name="avatar"; filename="exploit.lfe"
  Content-Type: image/jpeg

  <?php echo file_get_contents('/home/carlos/secret'); ?>
  ```
* **Paso a paso:**
  1. Subida exitosa de `.htaccess`, modificando la configuración del motor Apache para el directorio actual.
  2. Subida de `exploit.lfe` evadiendo la lista negra de extensiones.
  3. Petición `GET /files/avatars/exploit.lfe`. El servidor procesa la extensión `.lfe` mediante el intérprete de PHP gracias a la nueva regla, logrando RCE.

---

## 🔬 Sección 2: Vectores Teóricos (Análisis de Vectores Avanzados)

### Vector E: Explotación de Condición de Carrera en Subida (Race Condition)

#### 1. Mecánica y Ventana de Oportunidad (Timing)
El servidor recibe el archivo y lo escribe temporalmente en un directorio accesible mientras ejecuta las operaciones de validación (escaneo de virus, verificación de firma, etc.) para determinar si lo borra o lo acepta.
* Existe un intervalo de tiempo perceptible entre la escritura en disco y la eliminación/movimiento del archivo.
* **Prerrequisito esencial:** Para lograr RCE, la ubicación donde se escribe temporalmente el archivo debe contar con permisos de ejecución de scripts dinámicos (PHP, CGI, etc.).

#### 2. Estrategia de Explotación
* **Envío Simultáneo / Concurrente:** Se sincroniza el envío del `POST` (subida) y el `GET` (solicitud de ejecución) de forma prácticamente idéntica utilizando herramientas avanzadas como Burp Turbo Intruder o scripts de múltiples hilos.
* **Factores Críticos:**
  * **Ancho de banda e infraestructura:** Cuanto menor sea el rendimiento/ancho de banda del servidor objetivo, mayor será la ventana de tiempo para enganchar la petición `GET`.
  * **Predecibilidad del Nombre:** El ataque depende de si el servidor mantiene el nombre original del archivo o si utiliza algoritmos de nombrado aleatorio. Si la aleatoriedad es verdaderamente fuerte (ej. UUIDv4), adivinar la ruta para el `GET` resulta inviable; si se usan algoritmos débiles (marcas de tiempo Unix, hashing predecible), el nombre es fuerza-bruteable en esa ventana.

---

### Vector F: Race Condition en Subida Basada en URL

#### 1. Diferencia Clave
A diferencia de la subida por formulario web directo donde el usuario envía los bytes del archivo desde el cliente, en este vector el atacante proporciona una **URL remota** y es el servidor objetivo el que actúa como cliente descargando el recurso externo hacia un almacenamiento temporal local.

#### 2. Manipulación e Inyección de Latencia
* **Ampliación de la Ventana de Tiempo (Latency Injection):** Para forzar al servidor a tardar varios segundos escribiendo los datos en disco, el atacante puede:
  1. Servir el payload malicioso desde un servidor propio bajo su control.
  2. Rellenar el archivo malicioso con datos de basura (*padding*) para aumentar enormemente su tamaño.
  3. Controlar la tasa de transferencia de red (throttling) durante la respuesta HTTP del servidor controlado.
* **Factores de Éxito:** Si el servidor utiliza patrones de asignación de nombres temporales débiles (basados en timestamps o el nombre del recurso de la URL) y almacena los datos progresivamente en un directorio web ejecutable, el atacante puede realizar la petición `GET` mientras el servidor todavía está ocupado descargando el resto del archivo inflado.

---

### Vector G: Explotación del Lado del Cliente y Analizadores de Archivos (Parser Exploits)

#### 1. Vectores de Ataque y Contexto del Mismo Origen (Same-Origin)
Cuando la aplicación no ejecuta directamente scripts tipo PHP en el servidor pero sirve los archivos almacenados dentro del mismo dominio/origen principal de la aplicación web, abre la puerta a vectores del lado del cliente y vulnerabilidades de renderizado:
* **Mantenimiento del Contexto:** Al servirse desde la misma entidad de origen, el archivo subido tiene acceso al DOM, `localStorage` y cookies de la aplicación principal.
* **Formatos de Archivo Flexibles:** Se explotan vectores mediante archivos `SVG`, `HTML`, `PDF` o archivos binarios procesados por el backend.

#### 2. Mecanismos de Explotación Frecuentes
* **XML External Entity (XXE) en SVG:** Los archivos vectoriales SVG se basan en la estructura XML. Si el servidor procesa o renderiza estos archivos utilizando parsers vulnerables a entidades externas, se puede exfiltrar información interna del servidor.
* **XSS Almacenado mediante SVG/HTML:** Si un archivo SVG que incluye etiquetas `<script>` es visualizado directamente en el navegador, se ejecuta JavaScript en el contexto de la sesión del usuario.
* **Explotación de Librerías de Renderizado (Parsing Binario):** Librerías del servidor en Java/C++ (ej. ImageMagick, Ghostscript, parsers de PDF/miniaturas) son blanco de exploits a nivel de memoria o inyección de comandos cuando procesan archivos con cabeceras modificadas o formatos estructurales complejos.

---

### Vector H: Subida Directa de Archivos mediante Métodos HTTP Alternativos

#### 1. Métodos HTTP Alternativos
Se evalúa el uso de métodos HTTP distintos a la petición `POST` estándar de formularios web, en particular el método **`PUT`** (diseñado estructuralmente para la creación o reemplazo directo de recursos en el servidor).

#### 2. Condiciones de Servidor y Fallas de Configuración
Para que esta subida sea funcional sin pasar por la lógica ni las validaciones de la aplicación web:
* **Directivas de Métodos Permitidos:** El servidor web (Apache, Nginx, IIS) o API debe tener habilitado explícitamente el método `PUT` en el directorio web (verificable a través de una petición `OPTIONS`).
* **Ausencia de Autenticación / Control de Acceso:** No existen controles de autorización que restrinjan el envío de este método a usuarios anónimos o no autorizados.
* **Permisos de Escritura:** El directorio objetivo en el sistema de archivos posee permisos de escritura asignados al proceso del servidor web, permitiendo crear archivos arbitrarios (incluyendo scripts ejecutables) de forma directa.

---

## 🛡️ Sección 3: Guía de Remediación y Desarrollo Seguro

Para mitigar integralmente los riesgos asociados a la carga de archivos, se deben aplicar las siguientes medidas defensivas en profundidad:

1. **Uso de Listas Blancas (Allowlists) de Extensiones:**
   * Nunca utilizar listas negras (*blacklists*).
   * Validar estrictamente la extensión del archivo contra una lista permitida explícita (ej. solo `.jpg`, `.jpeg`, `.png`).

2. **Verificación Estricta del Contenido (Magic Bytes / Headers):**
   * Validar las firmas binarias iniciales del archivo (*Magic Bytes*) independientemente de la extensión o de la cabecera `Content-Type` enviada en la petición.

3. **Re-procesamiento y Sanitización de Imágenes:**
   * Volver a procesar o re-renderizar las imágenes subidas utilizando librerías seguras. Esto destruye cualquier payload PHP o metadatos maliciosos incrustados en comentarios o campos EXIF.

4. **Almacenamiento Desvinculado y Renombrado Aleatorio:**
   * Renombrar los archivos subidos utilizando identificadores únicos globales (UUIDv4) inaccesibles a adivinación para evitar condiciones de carrera o *Path Traversal*.
   * Guardar los archivos subidos fuera de la raíz web del servidor (*Document Root*) o en servicios de almacenamiento de objetos aislados (ej. Amazon S3).

5. **Deshabilitar Permisos de Ejecución:**
   * Configurar el servidor web (Nginx, Apache) para desactivar explícitamente la ejecución de scripts (PHP, Python, CGI) dentro del directorio reservado para archivos subidos.

6. **Desactivar el Sobrescritura de Configuraciones:**
   * Configurar el servidor de aplicaciones para ignorar archivos de configuración de directorio subidos por el usuario (ej. `AllowOverride None` en Apache para impedir ataques vía `.htaccess`).
