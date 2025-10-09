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

n este caso ‘X-Host‘— para manipular cómo el servidor genera contenido que posteriormente se almacena en caché.

Tras identificar el punto vulnerable y confirmar que la cabecera ‘X-Host‘ afecta a la generación de URLs para recursos estáticos, utilizamos dicha cabecera para referenciar un archivo JavaScript alojado en un servidor controlado por el atacante, el cual contiene un payload como ‘alert(document.cookie)‘.

Sin embargo, el laboratorio añade una capa adicional de complejidad: el uso de la cabecera ‘Vary: User-Agent‘ implica que la caché discrimina las respuestas en función del navegador. Para asegurar que el contenido envenenado impacta a la víctima, es necesario identificar su User-Agent, lo cual conseguimos mediante una petición enviada desde un comentario malicioso publicado en la web.

Una vez determinado el User-Agent de la víctima, se ajusta la petición para que coincida con dicho valor, permitiendo que el servidor entregue la versión cacheada del recurso malicioso únicamente a esa víctima.


```
User-Agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36  --> este es el User agent de la victima la he sacado en el Post 
X-Host: exploit-0acf000a03c9f40380e8026c01fc009e.exploit-server.net
```

## Reto 5: Envenenamiento de caché web a través de una cadena de consulta sin clave

Este laboratorio es vulnerable al envenenamiento de caché web porque la cadena de consulta no tiene clave. Un usuario visita regularmente la página principal de este sitio con Chrome.

Para resolver el laboratorio, envenene la página de inicio con una respuesta que se ejecute alert(1)en el navegador de la víctima. 

En este laboratorio se explota una vulnerabilidad de tipo web cache poisoning. La clave está en que los parámetros de la URL no forman parte de la clave que usa la caché para distinguir entre distintas peticiones, lo que permite que una respuesta generada con un parámetro malicioso se almacene y luego se sirva a otros usuarios aunque accedan sin ese parámetro.

El proceso consiste en añadir un parámetro con un valor que se refleje en la respuesta y permita inyectar código malicioso. Se usa una cabecera como Origin para forzar una respuesta no cacheada, y se comprueba que el contenido malicioso queda guardado

```
GET /?test=1'/><script>alert(1)</script>
```

## Reto 6:Envenenamiento de caché web a través de un parámetro de consulta sin clave

Este laboratorio es vulnerable al envenenamiento de caché web porque excluye un parámetro de la clave de caché. Un usuario visita regularmente la página principal de este sitio con Chrome.

Para resolver el laboratorio, envenene el caché con una respuesta que se ejecute alert(1)en el navegador de la víctima. 


Este laboratorio trata sobre una vulnerabilidad de web cache poisoning provocada por un parámetro que no forma parte de la clave usada por la caché, a pesar de ser procesado y reflejado en la respuesta. Esto permite que un atacante genere una versión maliciosa de la página que se almacene en caché y se entregue luego a usuarios legítimos.

Se identifica un parámetro como ‘utm_content‘ usando herramientas como Param Miner. A pesar de que el resto de la query string sí influye en la caché, este parámetro concreto no lo hace, lo que permite inyectar un payload como parte de su valor y que este se refleje en la página cacheada. Una vez envenenada la caché, cualquier usuario que acceda a la home sin ese parámetro recibirá la versión modificada. El laboratorio se resuelve cuando el usuario víctima accede y se ejecuta el contenido inyectado.

```
GET /?utm_content=asd'/><script>alert(1)</script>
```

## Reto 7: Encubrimiento de parámetros


Este laboratorio es vulnerable al envenenamiento de caché web porque excluye un parámetro de la clave de caché. Además, el análisis de parámetros entre la caché y el backend es inconsistente. Un usuario visita regularmente la página principal de este sitio web con Chrome.

Para resolver el laboratorio, utilice la técnica de encubrimiento de parámetros para envenenar el caché con una respuesta que se ejecute alert(1)en el navegador de la víctima. 

En este laboratorio se explora la técnica de parameter cloaking, una forma avanzada de web cache poisoning que aprovecha que la caché ignora ciertos parámetros mientras que el back-end sí los interpreta. Aquí, el parámetro ‘utm_content‘ es ignorado por la caché, pero no por el servidor, lo que permite camuflar un segundo parámetro (callback) tras un punto y coma para alterar la lógica sin que se vea afectada la clave de caché.

La aplicación carga un script que ejecuta una función definida por el parámetro callback. Aunque este parámetro sí forma parte de la clave de caché cuando se incluye directamente, al camuflarlo dentro de ‘utm_content‘, es procesado por el servidor pero no considerado por la caché. Esto permite envenenar el recurso compartido con una llamada a ‘alert(1)‘, que se ejecutará cada vez que un usuario cargue ese archivo JavaScript en su navegador. La clave está en mantener la caché envenenada hasta que el usuario víctima acceda.

```
GET /js/geolocate.js?callback=setCountryCookie&utm_content=1;callback=alert(1) 
```

## Reto 8:Envenenamiento de caché web mediante una solicitud GET pesada


Este laboratorio es vulnerable al envenenamiento de caché web. Acepta GETSolicitudes que tienen un cuerpo, pero que no lo incluyen en la clave de caché. Un usuario visita regularmente la página principal de este sitio con Chrome.

Para resolver el laboratorio, envenene el caché con una respuesta que se ejecute alert(1)en el navegador de la víctima. 

En este laboratorio se utiliza la técnica de fat GET request, donde el cuerpo de una petición GET es procesado por el servidor pero ignorado por la caché. Esta diferencia permite modificar el comportamiento de un recurso compartido sin alterar su clave de caché.

El objetivo es manipular la función JavaScript callback utilizada en el script ‘/js/geolocate.js‘. Aunque la clave de caché se basa en el valor del parámetro en la URL, el servidor procesa también un valor duplicado del mismo parámetro enviado en el cuerpo de la petición. Al incluir ‘alert(1)‘ como valor del parámetro en el cuerpo, el script se modifica sin que la caché lo detecte, envenenando así la respuesta que recibirán los usuarios posteriores. La cache debe mantenerse activa hasta que el usuario víctima acceda, momento en el que se resolverá el laboratorio.


```
callback=alert(1)
```