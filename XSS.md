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

<script>
    $(window).on('hashchange', function(){
        var post = $('section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')');
            if (post) post.get(0).scrollIntoView();
            });
</script>
