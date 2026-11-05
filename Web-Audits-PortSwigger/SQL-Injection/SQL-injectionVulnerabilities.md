# SQL Injection (SQLi) — PortSwigger Web Security Academy

## 1. Introducción

SQL Injection (SQLi) es una vulnerabilidad de seguridad que ocurre cuando una aplicación incorpora datos controlados por el usuario en consultas SQL sin aplicar mecanismos adecuados de validación o parametrización.

Esta vulnerabilidad permite que un atacante manipule la lógica de las consultas ejecutadas por la base de datos, pudiendo acceder a información no autorizada, modificar registros, consultar estructuras internas e incluso comprometer otros recursos dependiendo de los privilegios del usuario de la base de datos y del motor utilizado.

El objetivo de esta práctica fue comprender los diferentes tipos de SQL Injection, identificar posibles puntos de inyección, reconocer motores de bases de datos y recuperar información mediante distintas técnicas.

**Plataforma de práctica:** PortSwigger Web Security Academy.

**Herramientas utilizadas:** Burp Suite Community Edition, principalmente Repeater e Intruder.

---

## 2. Puntos de entrada de SQL Injection

Una vulnerabilidad SQL Injection puede encontrarse en cualquier dato controlado por el usuario que termine incorporándose a una consulta SQL de manera insegura.

Entre los puntos de entrada más habituales se encuentran:

- Parámetros de URL (GET).
- Parámetros del cuerpo de las peticiones HTTP (POST).
- Cookies.
- Cabeceras HTTP, como User-Agent y Referer.
- Formularios de autenticación.
- Filtros de categorías.
- Campos de búsqueda.
- Otros parámetros utilizados internamente por la aplicación.

Un aspecto importante es que el punto de inyección no necesariamente debe ser visible en la interfaz gráfica. También puede encontrarse en elementos de las peticiones HTTP que normalmente pasan desapercibidos para el usuario.

### 2.1. Detección inicial

Una de las primeras técnicas utilizadas para identificar posibles puntos de inyección consiste en introducir caracteres especiales, como una comilla simple, en parámetros que podrían intervenir en consultas SQL.

Posteriormente se observa el comportamiento de la aplicación.

Los posibles indicios incluyen:

- Errores relacionados con la sintaxis SQL.
- Cambios inesperados en el contenido de la página.
- Diferencias en las respuestas HTTP.
- Comportamientos anormales al modificar los parámetros.

La aparición de un error SQL puede indicar una posible vulnerabilidad, pero no constituye una confirmación definitiva. De igual manera, la ausencia de errores no significa que la aplicación sea segura.

---

## 3. Manipulación de consultas y bypass de autenticación

### 3.1. Recuperación de datos ocultos

Una aplicación puede utilizar consultas SQL para recuperar registros según determinadas condiciones.

Por ejemplo, un filtro de categorías puede utilizar una condición `WHERE` para mostrar únicamente los productos correspondientes a una categoría específica.

Si la entrada no se procesa correctamente, es posible manipular la condición lógica de la consulta.

Una de las técnicas estudiadas consiste en utilizar operadores booleanos, como `OR`, para introducir una condición que siempre resulte verdadera.

Esto puede provocar que la consulta devuelva registros que originalmente estaban ocultos o restringidos por el filtro.

### 3.2. Bypass de inicio de sesión

Los formularios de autenticación también pueden ser vulnerables cuando las credenciales se incorporan directamente a una consulta SQL.

Generalmente, la aplicación comprueba si existe un usuario y si la contraseña proporcionada coincide con la almacenada.

Mediante la manipulación de la lógica de la consulta, un atacante puede intentar alterar las condiciones de autenticación para conseguir que se cumplan sin conocer la contraseña legítima.

También pueden utilizarse comentarios SQL para ignorar partes posteriores de una consulta, dependiendo de la sintaxis y del motor de base de datos.

**Conceptos estudiados:**

- Operadores lógicos `OR` y `AND`.
- Condiciones booleanas.
- Cláusula `WHERE`.
- Comentarios SQL.
- Manipulación de consultas de autenticación.

---

## 4. UNION-based SQL Injection

UNION-based SQL Injection es una técnica que permite combinar los resultados de una consulta SQL con los resultados de otra consulta mediante el operador `UNION`.

Esto puede utilizarse para recuperar información de otras tablas de la base de datos y mostrarla dentro de la respuesta normal de la aplicación.

Para realizar este tipo de ataque es necesario comprender varias condiciones.

### 4.1. Determinar el número de columnas

Las consultas combinadas mediante `UNION` deben devolver el mismo número de columnas.

Existen dos métodos principales para determinar cuántas columnas devuelve la consulta original:

**Método ORDER BY**

Consiste en modificar progresivamente el índice de ordenamiento hasta provocar un error.

El último índice válido permite deducir el número de columnas de la consulta.

**Método UNION SELECT NULL**

Consiste en realizar pruebas utilizando diferentes cantidades de valores `NULL` hasta conseguir una consulta compatible con la cantidad de columnas original.

### 4.2. Identificar columnas compatibles con texto

Después de determinar el número de columnas, es necesario identificar cuáles aceptan valores de tipo texto.

Esto se consigue sustituyendo progresivamente los valores utilizados durante las pruebas por cadenas de texto y observando la respuesta de la aplicación.

Esta información es importante porque no todas las columnas admiten los mismos tipos de datos.

### 4.3. Recuperación de información de otras tablas

Una vez identificadas las columnas compatibles, es posible utilizar `UNION` para consultar otras tablas y recuperar información que normalmente no estaría disponible en la página.

Entre los datos que pueden recuperarse se encuentran:

- Nombres de usuarios.
- Direcciones de correo electrónico.
- Contraseñas almacenadas.
- Información de otras tablas.

### 4.4. Recuperación de múltiples valores en una columna

En determinadas situaciones, únicamente una columna de la consulta original permite mostrar texto.

Para solucionar esta limitación se pueden concatenar varios valores en una sola columna.

Por ejemplo, es posible combinar un nombre de usuario y una contraseña utilizando funciones de concatenación o separadores.

La función utilizada depende del motor de base de datos.

**Conceptos estudiados:**

- `UNION`.
- `UNION ALL`.
- `ORDER BY`.
- `NULL`.
- Compatibilidad de tipos de datos.
- Concatenación de cadenas.
- Recuperación de información entre tablas.

---

## 5. Identificación del motor y versión de la base de datos

Antes de realizar consultas más complejas, es importante identificar el sistema gestor de bases de datos utilizado por la aplicación.

Cada motor presenta diferencias en sus funciones, sintaxis, tablas del sistema y mecanismos para consultar información interna.

### 5.1. MySQL

Algunas funciones y variables relevantes son:

- `VERSION()`: permite consultar la versión del servidor.
- `@@version`: devuelve información de la versión del motor.

### 5.2. Microsoft SQL Server

Una de las variables utilizadas para consultar información de versión es:

- `@@VERSION`.

Esta variable proporciona información sobre la versión y características del servidor SQL Server.

### 5.3. Oracle

Oracle utiliza mecanismos diferentes para consultar información del motor.

Entre los elementos estudiados se encuentran:

- `V$VERSION`: vista que proporciona información sobre la versión del sistema.
- `ALL_TABLES`: permite consultar las tablas accesibles para el usuario.
- `ALL_TAB_COLUMNS`: permite consultar información sobre las columnas de las tablas accesibles.

Una diferencia importante es que Oracle requiere una estructura sintáctica particular para determinadas consultas, por ejemplo, el uso de `FROM DUAL` en expresiones que necesitan una tabla de referencia.

---

## 6. Enumeración de bases de datos y estructuras internas

Una vez identificado el motor de base de datos, es posible consultar sus metadatos para conocer la estructura interna de la información almacenada.

### 6.1. INFORMATION_SCHEMA

En motores como MySQL, `INFORMATION_SCHEMA` proporciona información sobre la estructura de las bases de datos.

Entre sus tablas más relevantes se encuentran:

**INFORMATION_SCHEMA.TABLES**

Permite consultar información sobre las tablas existentes, incluyendo sus nombres y las bases de datos a las que pertenecen.

**INFORMATION_SCHEMA.COLUMNS**

Permite obtener información sobre las columnas de las tablas, incluyendo:

- Nombre de la columna.
- Nombre de la tabla.
- Tipo de dato.
- Posición de la columna.

Esta información permite conocer la estructura de la base de datos antes de intentar recuperar registros de interés.

### 6.2. Enumeración en Oracle

En Oracle se utilizan vistas del diccionario de datos.

Entre las principales se encuentran:

- `ALL_TABLES`: información sobre las tablas accesibles.
- `ALL_TAB_COLUMNS`: información sobre las columnas accesibles.
- `USER_TABLES`: tablas pertenecientes al usuario actual.
- `USER_TAB_COLUMNS`: columnas de las tablas del usuario actual.

La diferencia entre estas vistas depende del alcance de la información que permiten consultar.

---

## 7. Blind SQL Injection

Blind SQL Injection es una técnica que se presenta cuando una aplicación es vulnerable a la manipulación de consultas SQL, pero no muestra directamente los resultados de la consulta ni la información extraída.

En estos casos, la recuperación de información se realiza mediante la observación de otros comportamientos de la aplicación.

Durante los laboratorios se estudiaron diferentes modalidades.

### 7.1. Boolean-based Blind SQL Injection

Este método utiliza condiciones booleanas para deducir información de la base de datos.

La aplicación responde de manera diferente dependiendo de si una condición SQL es verdadera o falsa.

Por ejemplo:

- Una condición verdadera puede provocar que la página muestre determinado contenido.
- Una condición falsa puede provocar que ese contenido desaparezca o que la respuesta cambie.

En los laboratorios se utilizó la cookie `TrackingId` como punto de inyección.

Esta cookie era procesada por el servidor y permitía observar diferencias en el comportamiento de la aplicación.

Mediante este comportamiento fue posible recuperar información de la base de datos carácter por carácter.

**Funciones relevantes:**

- `SUBSTRING()`: permite extraer una parte de una cadena.
- `LENGTH()`: permite determinar la longitud de una cadena.

Estas funciones permiten comprobar progresivamente los caracteres de una contraseña y deducir su contenido mediante respuestas verdaderas o falsas.

### 7.2. Conditional Errors

Esta técnica utiliza errores SQL provocados de manera condicional para obtener información.

Una de las estructuras utilizadas es `CASE WHEN`.

Su funcionamiento consiste en evaluar una condición y ejecutar una expresión dependiendo del resultado.

Por ejemplo, si una condición es verdadera, puede ejecutarse una operación que provoque un error en la base de datos, como una división entre cero.

Si la condición es falsa, puede ejecutarse una operación normal.

De esta manera, es posible deducir información observando si la aplicación devuelve un error o una respuesta normal.

A diferencia del método booleano, aquí la señal utilizada para obtener información es la aparición de errores controlados.

### 7.3. Error-based SQL Injection

Esta modalidad aprovecha los mensajes de error generados por la base de datos para obtener información.

En determinadas circunstancias, los errores pueden revelar datos sobre:

- La estructura de las consultas.
- El motor de base de datos.
- Los tipos de datos.
- Expresiones evaluadas.
- Información almacenada.

La diferencia principal respecto a los ataques ciegos es que la información puede aparecer directamente en el mensaje de error, en lugar de deducirse únicamente por diferencias de comportamiento.

### 7.4. Time-based Blind SQL Injection

Esta técnica utiliza retrasos temporales para determinar si una condición SQL se cumple.

En lugar de recibir directamente un resultado verdadero o falso, se provoca un retraso en la respuesta cuando se cumple una condición específica.

Por ejemplo, el motor de base de datos puede ejecutar una función de espera temporal dependiendo del resultado de una expresión.

Si la respuesta tarda más de lo habitual, puede indicar que la condición evaluada fue verdadera.

Si responde normalmente, puede indicar que fue falsa.

Este método permite recuperar información incluso cuando la aplicación no presenta cambios visibles ni errores útiles.

**Funciones relevantes según el motor:**

- MySQL: `SLEEP()`.
- Microsoft SQL Server: `WAITFOR DELAY`.
- Oracle: mecanismos de espera temporal como `DBMS_LOCK.SLEEP()`.

### 7.5. Recuperación de información mediante retrasos temporales

En uno de los laboratorios se utilizó esta técnica para recuperar una contraseña completa.

El procedimiento consistió en:

1. Identificar el punto de inyección.
2. Comprobar manualmente el comportamiento mediante Burp Repeater.
3. Determinar la longitud de la contraseña mediante `LENGTH()`.
4. Comprobar los caracteres mediante `SUBSTRING()`.
5. Automatizar las comprobaciones utilizando Burp Intruder.
6. Interpretar los retrasos de las respuestas para reconstruir la información.

La longitud obtenida para la contraseña del laboratorio fue de exactamente **20 caracteres**.

Burp Intruder permitió automatizar las comprobaciones, evitando realizar manualmente cada petición.

Es importante diferenciar el propósito de cada función:

- `LENGTH()` permite determinar cuántos caracteres contiene una cadena.
- `SUBSTRING()` permite seleccionar posiciones concretas de esa cadena.
- Intruder permite automatizar las pruebas de diferentes posiciones y valores.

La recuperación de información mediante esta técnica requiere analizar cuidadosamente los tiempos de respuesta, ya que la latencia de red y otros factores pueden generar resultados inconsistentes.

---

## 8. Out-of-band SQL Injection (OAST)

Out-of-band SQL Injection es una modalidad en la que la información sobre la vulnerabilidad se obtiene mediante un canal diferente al utilizado para enviar la petición original.

A diferencia de los ataques booleanos o temporales, no es necesario depender exclusivamente de la respuesta HTTP de la aplicación.

### 8.1. Interacción fuera de banda

La técnica consiste en provocar una interacción externa desde el servidor vulnerable hacia un dominio controlado por el investigador.

El procedimiento general es:

1. Identificar una posible inyección SQL.
2. Introducir una expresión que provoque una interacción externa.
3. Conseguir que el servidor o el motor de base de datos intente resolver un dominio externo.
4. Observar la interacción recibida mediante una herramienta de monitorización.

Una herramienta utilizada para este propósito es Burp Collaborator.

La interacción puede producirse mediante una consulta DNS o una solicitud HTTP, dependiendo de la técnica y del entorno.

Esta modalidad resulta especialmente útil cuando la aplicación no muestra resultados, no presenta errores y tampoco permite identificar diferencias temporales fiables.

### 8.2. Exfiltración de datos fuera de banda

Esta técnica amplía el concepto anterior.

En lugar de limitarse a demostrar que existe una interacción externa, busca transmitir información de la base de datos mediante esa interacción.

Conceptualmente, determinados datos pueden incorporarse a una solicitud externa, permitiendo observarlos desde un dominio controlado por el investigador.

La diferencia es importante:

- OAST de interacción: permite demostrar que el servidor realiza una comunicación externa.
- OAST con exfiltración: busca recuperar información mediante esa comunicación.

**Estado práctico:** estos dos laboratorios quedaron pendientes. Se estudió su funcionamiento y la mecánica de las interacciones externas, pero no se completó su ejecución práctica.

La disponibilidad de Burp Collaborator y las capacidades de automatización pueden variar según la edición y las herramientas disponibles.

---

## 9. SQL Injection mediante XML Encoding

Este laboratorio aborda una técnica de evasión de filtros mediante codificación XML.

En determinados escenarios, las aplicaciones reciben datos en formato XML y los procesan antes de utilizarlos en otras operaciones.

Si existen filtros que buscan determinadas cadenas o caracteres en su representación original, puede ser posible evadirlos utilizando referencias de caracteres XML.

Estas referencias pueden expresarse mediante:

- Codificación numérica decimal.
- Codificación numérica hexadecimal.

El funcionamiento consiste en representar los caracteres del contenido mediante sus referencias XML.

Posteriormente, el analizador XML interpreta esas referencias y reconstruye los caracteres originales.

Si la aplicación realiza la decodificación antes de procesar la entrada y el filtro no contempla adecuadamente este comportamiento, puede producirse una evasión del mecanismo de protección.

Este laboratorio permitió comprender que los filtros de entrada no deben depender únicamente de buscar cadenas específicas, sino que deben considerar cómo se normalizan y procesan los datos.

---

## 10. Herramientas utilizadas

### Burp Suite Community Edition

Se utilizó Burp Suite para interceptar, modificar y analizar las peticiones HTTP durante las pruebas.

**Repeater**

Permitió modificar manualmente las peticiones y comprobar el comportamiento de la aplicación frente a diferentes entradas.

Fue especialmente útil para:

- Identificar puntos de inyección.
- Comprobar condiciones booleanas.
- Analizar respuestas HTTP.
- Evaluar errores SQL.
- Comprobar retrasos temporales.

**Intruder**

Se utilizó para automatizar solicitudes y comprobar diferentes valores.

Fue especialmente útil en la recuperación de información mediante técnicas temporales, donde era necesario evaluar múltiples posiciones y caracteres.

**Burp Collaborator**

Se estudió su funcionamiento como mecanismo de detección de interacciones externas en ataques OAST. Los dos laboratorios correspondientes quedaron pendientes de resolución práctica.

---

## 11. Registro de laboratorios

Los siguientes laboratorios fueron realizados en PortSwigger Web Security Academy.

| N.º | Laboratorio | Estado |
|---|---|---|
| 1 | Recuperación de datos ocultos mediante SQL Injection | Resuelto |
| 2 | Bypass de inicio de sesión mediante SQL Injection | Resuelto |
| 3 | Consulta de versión de Oracle | Resuelto |
| 4 | Consulta de versión de MySQL y Microsoft SQL Server | Resuelto |
| 5 | Enumeración de bases de datos no Oracle | Resuelto |
| 6 | Enumeración de bases de datos Oracle | Resuelto |
| 7 | Determinación del número de columnas mediante UNION | Resuelto |
| 8 | Identificación de columnas compatibles con texto | Resuelto |
| 9 | Recuperación de datos de otras tablas mediante UNION | Resuelto |
| 10 | Recuperación de múltiples valores en una columna | Resuelto |
| 11 | Blind SQL Injection mediante respuestas condicionales | Resuelto |
| 12 | Blind SQL Injection mediante errores condicionales | Resuelto |
| 13 | SQL Injection visible basada en errores | Resuelto |
| 14 | Blind SQL Injection mediante retrasos temporales | Resuelto |
| 15 | Recuperación de información mediante retrasos temporales | Resuelto |
| 16 | Blind SQL Injection mediante interacción fuera de banda | Pendiente |
| 17 | Blind SQL Injection con exfiltración de datos fuera de banda | Pendiente |
| 18 | SQL Injection con evasión de filtros mediante XML Encoding | Resuelto |

**Balance de laboratorios:**

- Total: 18.
- Resueltos: 16.
- Pendientes: 2.

---

## 12. Conclusiones

El desarrollo de estos laboratorios permitió comprender que SQL Injection no se limita a introducir caracteres especiales en formularios, sino que requiere analizar cómo la aplicación procesa las entradas y cómo estas terminan formando parte de las consultas ejecutadas por la base de datos.

Se estudiaron técnicas de manipulación de consultas, bypass de autenticación, recuperación de información mediante UNION, identificación de motores de bases de datos y enumeración de sus estructuras internas.

También se trabajaron diferentes modalidades de Blind SQL Injection, comprendiendo cómo es posible obtener información incluso cuando la aplicación no muestra directamente los resultados de las consultas.

El uso de Burp Suite permitió analizar las peticiones HTTP, modificar parámetros, observar respuestas y automatizar comprobaciones mediante Repeater e Intruder.

Finalmente, se estudió la evasión de filtros mediante XML Encoding y el funcionamiento conceptual de los ataques fuera de banda.

**Resultado:** 16 de los 18 laboratorios completados, con dos ejercicios de OAST pendientes de ejecución práctica.

Este conjunto de prácticas constituye una base de conocimientos sobre identificación, análisis y explotación controlada de vulnerabilidades SQL Injection en entornos de laboratorio autorizados.

---

**Plataforma:** [PortSwigger Web Security Academy](https://portswigger.net/web-security)

**Categoría:** Web Application Security / SQL Injection

