## Reto 1: Envenenamiento de caché web con un encabezado sin clave

Este laboratorio es vulnerable al envenenamiento de caché web porque maneja la entrada de un encabezado sin clave de forma insegura. Un usuario desprevenido visita regularmente la página de inicio del sitio. Para solucionar este laboratorio, envenene la caché con una respuesta que ejecute alert(document.cookie)en el navegador del visitante. 

En este laboratorio se analiza una vulnerabilidad de Web Cache Poisoning provocada por el uso inseguro de una cabecera HTTP que no es tenida en cuenta como parte de la clave de caché (unkeyed header). Esto permite a un atacante modificar el contenido que será servido a futuros usuarios desde la caché del servidor.

La aplicación utiliza la cabecera ‘X-Forwarded-Host’ para generar URLs absolutas de recursos JavaScript. Al modificar esta cabecera, es posible hacer que el navegador del usuario intente cargar un script desde un dominio arbitrario, como el exploit server.

Al comprobar que la respuesta modificada se almacena en caché (gracias a la cabecera ‘X-Cache: hit‘), se confirma que es posible envenenar la caché con contenido controlado por el atacante.

Luego, vuelve a realizar una petición manipulada a la home del sitio, esta vez sin parámetro de cache-busting, y utilizando de nuevo la cabecera ‘X-Forwarded-Host‘, esta vez apuntando al exploit server. Una vez confirmada la cacheabilidad de la respuesta, se espera que el próximo visitante habitual de la página reciba la versión envenenada y ejecute el código malicioso en su navegador.

## Reto 2: Envenenamiento de caché web con una cookie sin clave

Este laboratorio es vulnerable al envenenamiento de la caché web porque las cookies no están incluidas en la clave de caché. Un usuario desprevenido visita regularmente la página de inicio del sitio. Para solucionar este laboratorio, envenene la caché con una respuesta que ejecute alert(1)en el navegador del visitante.  

```
Cookie: session=z3qnHl3mE14W0WuF0GnJaanKTPUjtz2D; fehost=test"-alert(1)-"asd
```

Exploramos un escenario en el que la cookie ‘fehost‘, enviada por el cliente y reflejada en la respuesta del servidor, no se utiliza como parte de la clave de caché. Esto significa que es posible modificar su valor para introducir un payload malicioso, como un ‘alert(1)‘, y lograr que la respuesta sea cacheada con ese contenido.

Tras identificar esta condición, se utiliza una cadena especialmente construida en la cookie para romper la estructura original de la respuesta e insertar el script. Luego, al comprobar que la respuesta manipulada es almacenada en caché (X-Cache: hit), se espera a que un usuario desprevenido acceda a la página y ejecute el payload.

## Reto 3: Envenenamiento de caché web con múltiples encabezados

Este laboratorio contiene una vulnerabilidad de envenenamiento de caché web que solo se puede explotar al usar múltiples encabezados para crear una solicitud maliciosa. Un usuario visita la página de inicio aproximadamente una vez por minuto. Para solucionar este laboratorio, envenene la caché con una respuesta que ejecute alert(document.cookie)en el navegador del visitante. 

requiere el uso conjunto de varias cabeceras HTTP para llevar a cabo un ataque de Web Cache Poisoning. El servidor utiliza información de cabeceras como ‘X-Forwarded-Scheme‘ y ‘X-Forwarded-Host‘ para construir redirecciones, pero no incorpora dichas cabeceras en su clave de caché.

Manipulando estas cabeceras es posible hacer que el servidor genere una redirección hacia un dominio controlado por el atacante, apuntando a un script JavaScript alojado en un exploit server con un payload malicioso, como ‘alert(document.cookie)‘.

Una vez que la respuesta es cacheada y el indicador ‘X-Cache: hit‘ lo confirma, cualquier usuario que acceda a la página durante ese periodo recibirá la versión envenenada del recurso. Esto permite ejecutar código JavaScript en el navegador de la víctima sin interacción por su part

```
X-Forwarded-Scheme: http
X-Forwarded-Host: exploit-0a1a00ca03f56ebe80c9a28001dc001d.exploit-server.net
```

## Reto 4: Envenenamiento de caché web dirigido mediante un encabezado desconocido

Este laboratorio es vulnerable al envenenamiento de caché web. Un usuario víctima verá cualquier comentario que publiques. Para solucionar este laboratorio, debes envenenar la caché con una respuesta que se ejecute. alert(document.cookie)En el navegador del visitante. Sin embargo, también debe asegurarse de que la respuesta se muestre al subconjunto específico de usuarios al que pertenece la víctima prevista. 