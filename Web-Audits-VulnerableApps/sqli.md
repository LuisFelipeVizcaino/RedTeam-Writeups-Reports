SQLMap y SQL Injection → Shell (LFI encadenado) — Metasploitable2 / Mutillidae
1. Introducción

Esta sección documenta el uso de SQLMap como herramienta de automatización para la explotación de SQL Injection, tanto en peticiones GET como POST, y el estudio conceptual y práctico de la técnica de escalado de SQLi a ejecución de comandos (shell) mediante escritura de archivos en el servidor (INTO OUTFILE / INTO DUMPFILE) encadenada con Local File Inclusion (LFI).

Entorno de práctica: Metasploitable2 (SQLMap manual, GET/POST) y Mutillidae (cadena SQLi→shell vía LFI).

Herramienta principal: SQLMap.

2. SQLMap manual con parámetros GET
2.1. Flujo de trabajo

El procedimiento seguido para explotar una inyección SQL vía parámetros GET con SQLMap fue el siguiente:

Identificación del parámetro inyectable. Se usó -p para indicar explícitamente a SQLMap el parámetro sobre el que debía enfocar las pruebas (en este caso, id), en lugar de dejar que la herramienta probara todos los parámetros de la URL.
Confirmación de la vulnerabilidad. SQLMap valida si el parámetro indicado es efectivamente inyectable antes de continuar con la extracción de información.
Enumeración de bases de datos. Con --dbs se obtuvo el listado de bases de datos disponibles en el motor.
Selección de base de datos. Con -D <nombre> se apuntó a una base de datos específica de las listadas.
Enumeración de tablas. Con --tables se obtuvo el listado de tablas dentro de la base de datos seleccionada.
Selección de tabla. Con -T <nombre> se apuntó a una tabla específica.
Volcado de datos. Con --dump se extrajeron los valores almacenados en la tabla seleccionada.

Este flujo se repitió varias veces contra Metasploitable2, subiendo progresivamente el nivel de seguridad del objetivo entre cada repetición, para observar cómo cambiaba la dificultad de explotación.

Comando base (ejemplo de estructura):

bash
sqlmap -u "http://<objetivo>/vuln.php?id=1" -p id --dbs
sqlmap -u "http://<objetivo>/vuln.php?id=1" -D <basedatos> --tables
sqlmap -u "http://<objetivo>/vuln.php?id=1" -D <basedatos> -T <tabla> --dump
3. SQLMap con peticiones POST (-r)
3.1. Captura y uso de archivo de petición

En lugar de pasar la URL directamente, se capturó una petición HTTP completa (por ejemplo, desde Burp Suite) y se guardó en un archivo de texto. SQLMap se apuntó a ese archivo mediante -r, lo que permite que la herramienta pruebe automáticamente los parámetros contenidos en el cuerpo de la petición POST, cabeceras y cookies, en lugar de limitarse a parámetros de URL.

bash
sqlmap -r peticion.txt --dbs
3.2. Ajuste de nivel y riesgo (--level / --risk)

Contra un objetivo configurado con seguridad alta, SQLMap no detectó inyección con sus parámetros por defecto. Fue necesario aumentar:

--level 5: hace que SQLMap pruebe todos los parámetros de la petición HTTP (incluyendo cabeceras y cookies), no solo los más evidentes.
--risk 3: habilita payloads más agresivos (incluyendo los que pueden ser más invasivos o lentos de ejecutar).
bash
sqlmap -r peticion.txt --level 5 --risk 3
3.3. Resultado de la prueba contra seguridad alta

Con --level 5 --risk 3 no fue posible confirmar la inyección. La causa identificada fue que los parámetros del formulario solo aceptaban valores numéricos, lo cual limitaba severamente la superficie de ataque y los payloads que SQLMap podía inyectar sin romper el formato esperado. Este resultado se documenta como un intento sin éxito, no como una vulnerabilidad confirmada en ese escenario específico.

4. SQL Injection → Shell vía escritura de archivos (INTO OUTFILE / INTO DUMPFILE)
4.1. Concepto

Cuando una aplicación es vulnerable a SQL Injection sobre un motor con privilegios suficientes, es posible usar las cláusulas INTO OUTFILE o INTO DUMPFILE (MySQL) para escribir el resultado de una consulta como archivo en el sistema de archivos del servidor. Si ese archivo se escribe dentro del document root de una aplicación web, y luego se accede a él (directamente o mediante una vulnerabilidad de Local File Inclusion), puede lograrse ejecución de código del lado del servidor — es decir, escalar de SQLi a shell.

4.2. Condiciones necesarias para que la escritura sea posible

Para que INTO OUTFILE / INTO DUMPFILE funcione, deben cumplirse varias condiciones simultáneamente:

secure_file_priv no debe estar en NULL/restrictivo de forma que bloquee la escritura. Esta variable de MySQL controla dónde puede leer/escribir archivos el motor:
Si está vacía (""), MySQL puede escribir en cualquier ruta del sistema (sujeto a permisos del SO).
Si apunta a una ruta específica, solo se puede escribir dentro de esa ruta — y esa ruta se convierte en el objetivo del ataque.
Si está en NULL, la escritura de archivos queda deshabilitada por completo.
Permisos de escritura del lado del motor de base de datos (privilegio FILE). El usuario de MySQL con el que se ejecuta la consulta inyectada debe tener el privilegio FILE concedido, que es el que autoriza operaciones de lectura/escritura de archivos desde SQL.
Permisos de escritura a nivel de sistema operativo. Independientemente de los privilegios dentro de MySQL, el usuario del sistema operativo bajo el cual corre el proceso de MySQL (en Linux, típicamente mysql) debe tener permisos de escritura sobre el directorio/archivo destino a nivel de sistema de archivos (permisos tipo rwx). Si el proceso de MySQL no tiene ese permiso en Linux, la escritura falla aunque secure_file_priv y el privilegio FILE estén correctamente configurados.

En resumen: hace falta que coincidan tres capas — configuración del motor (secure_file_priv), privilegio del usuario de BD (FILE), y permisos del usuario del sistema operativo sobre la ruta destino.

4.3. DUMPFILE vs OUTFILE
INTO DUMPFILE: escribe una sola fila con un solo valor, sin ningún formato adicional (sin separadores de columna, sin salto de línea añadido). Es la opción usada en esta práctica, por ser más adecuada para texto plano, ya que no corrompe el contenido con caracteres de formato extra.
INTO OUTFILE: permite volcar resultados con más de una fila/columna, añadiendo separadores de campo y de línea. Es la opción típicamente recomendada para contenido binario (por ejemplo, para depositar un webshell compilado o un archivo que no deba tener bytes de formato añadidos por fila/columna), aunque en esta práctica no se llegó a usar, quedando documentada solo a nivel conceptual.
4.4. Estado práctico

La cadena completa SQLi→shell vía LFI se trabajó sobre Mutillidae y Metasploitable2, utilizando DUMPFILE para depositar contenido en texto plano. El acceso manual mediante el formulario de login de Mutillidae no tuvo éxito en esta sesión.

5. Conclusiones

El uso de SQLMap permitió automatizar el flujo completo de explotación de SQL Injection — desde la identificación del parámetro vulnerable hasta el volcado de datos — tanto en peticiones GET simples como en peticiones POST capturadas como archivo (-r). Se comprobó además que aumentar --level y --risk no garantiza la detección de la vulnerabilidad: cuando el parámetro solo acepta valores numéricos, la superficie de inyección puede quedar efectivamente cerrada para las técnicas que SQLMap prueba por defecto.

