Control de Acceso — Access Control Vulnerabilities (PortSwigger Web Security Academy)
Resumen

Laboratorio completo de la categoría Access Control de PortSwigger: 13/13 labs resueltos (9 nivel Aprendiz, 4 nivel Profesional). Cubre las tres variantes de fallo de control de acceso (vertical, horizontal, basado en contexto/lógica) más técnicas de bypass a nivel de infraestructura (método HTTP, URL, Referer).

1. Escalada vertical de privilegios

Un usuario de bajo privilegio accede a funcionalidad reservada para un rol superior (admin).

Funcionalidad de admin no protegida: la app no revisa el rol en absoluto antes de servir el panel de admin; solo lo oculta del menú. Acceso directo por URL (/admin).
Funcionalidad de admin no protegida, URL impredecible: mismo fallo, pero la ruta de admin es un valor aleatorio/ofuscado en vez de /admin. La protección real era “seguridad por oscuridad” — se filtra el enlace vía JS del cliente, robots.txt, o comentarios HTML, y una vez conocida la URL, el control sigue sin existir.

Aprendizaje: ocultar un enlace del menú no es control de acceso. Si el backend no vuelve a validar el rol en cada petición, cualquiera que descubra la ruta entra.

2. Rol/privilegio controlado por el cliente
Rol controlado por parámetro de solicitud: el rol viaja como parámetro en la petición (ej. admin=true o roleid=2) y el backend confía en ese valor en vez de recuperarlo de la sesión del servidor.
Rol modificable en el perfil de usuario: el endpoint de actualización de perfil acepta un campo de rol que el usuario no debería poder tocar; se envía junto con los campos legítimos (nombre, email) en la petición de edición.

Aprendizaje: cualquier dato de autorización (rol, permisos) que llegue en el request en vez de derivarse del lado del servidor (sesión) es tamperable. Hay que probar agregar/modificar campos que ni siquiera están en el formulario visible.

3. Escalada horizontal (IDOR) y su combinación con vertical
ID de usuario controlado por parámetro: cambiar el id/uid en la URL para ver/editar datos de otro usuario (IDOR clásico).
ID de usuario controlado por parámetro, con IDs impredecibles: mismo fallo pero con IDs no secuenciales (UUID/hash); el ID impredecible no es protección si se puede obtener por otra vía (fuga en otra respuesta, enumeración, etc.).
ID de usuario controlado por parámetro con fuga de datos en la redirección: el servidor redirige tras denegar el acceso, pero el cuerpo de la respuesta de redirección ya incluye los datos sensibles antes de completarse el redirect — el dato se filtra aunque la “protección” final sí funcione.
ID de usuario controlado por parámetro con revelación de contraseña: el endpoint de “cambiar datos de usuario” devuelve/precarga la contraseña actual en texto plano al consultar el ID ajeno.
Referencias directas inseguras a objetos (IDOR): acceso a recursos (facturas, documentos) referenciados por un identificador directo sin validar que pertenezcan al usuario autenticado.

Aprendizaje: horizontal (IDOR) no es solo “ver datos ajenos” — puede escalar a vertical si el objeto referenciado incluye credenciales o datos que habilitan tomar la cuenta de otro usuario (incluido un admin). Siempre revisar el contenido completo de la respuesta, no solo el status code.

4. Bypass de control de acceso a nivel de infraestructura (labs Profesional)
Control de acceso basado en URL puede eludirse: el control lo aplica un componente front-end (reverse proxy) basándose únicamente en la ruta de la petición que a él le llega. El back-end, sin embargo, da prioridad al header X-Original-URL (o X-Rewrite-URL) para decidir qué recurso procesar realmente. Bypass: mandar la petición a una ruta que el front-end sí permite (ej. /), e indicar la ruta real restringida (ej. /admin/deleteUser?username=carlos) en el header X-Original-URL. El front-end valida la ruta “inocente” que ve; el back-end ejecuta la ruta real del header.
Control de acceso basado en métodos puede eludirse: el check de autorización solo está implementado para un método HTTP específico (típicamente POST). Bypass 1: repetir la misma acción con GET, pasando los parámetros como query string — el endpoint responde igual pero el filtro de autorización no cubre ese método. Bypass 2 (verb tampering): usar un método inventado tipo POSTX — el filtro de autorización compara contra el string exacto “POST” y no matchea, pero el router de la aplicación es más laxo y procesa la petición igual.
Proceso de varios pasos sin control de acceso en un solo paso: el flujo de “promover a admin” tiene dos pasos (solicitar cambio → confirmar cambio). El control de acceso solo se valida en el primer paso; el segundo (el que ejecuta el cambio real) no vuelve a chequear el rol de la sesión. Bypass: cambiar la cookie de sesión justo antes de enviar la petición de confirmación (el segundo paso), sin necesidad de pasar por el primero con esa sesión.
Control de acceso basado en Referer: el backend usa el header Referer como prueba de que la petición vino navegando desde la página de administración (ej. solo permite el cambio de rol si Referer contiene /admin). Bypass: falsificar el header Referer — es un dato enviado por el cliente y trivialmente modificable, no una prueba real de autorización.

