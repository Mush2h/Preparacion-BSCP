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
