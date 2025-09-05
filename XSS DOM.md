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

## Reto 2:DOM XSS usando mensajes web y una URL de JavaScript

Este laboratorio demuestra una vulnerabilidad de redirección basada en DOM que se activa mediante mensajería web. Para resolver este laboratorio, cree una página HTML en el servidor de exploits que explote esta vulnerabilidad y llame al print() función.


En este laboratorio explotamos una vulnerabilidad de tipo DOM XSS basada en redirección, donde la aplicación escucha mensajes entrantes mediante postMessage y utiliza su contenido como destino para un cambio de página usando ‘location.href‘. Aunque intenta validar que la URL comienza con http o https, el uso incorrecto de indexOf permite eludir la comprobación.

Desde el exploit server, se carga la página vulnerable dentro de un iframe y se envía un mensaje que contiene un payload JavaScript válido seguido de una cadena que incluye http, lo suficiente para pasar el filtro. Al llegar al sink ‘location.href‘, se ejecuta la función print, demostrando cómo una validación mal implementada puede abrir la puerta a la ejecución de código arbitrario.

```html
<iframe src="https://0a40003a0333871280748fc800f00028.web-security-academy.net/" onload="this.contentWindow.postMessage('javascript:print()//http:','*')">
```

## Reto 3: DOM XSS usando mensajes web y JSON.parse


Este laboratorio utiliza mensajería web y analiza el mensaje como JSON. Para resolver el laboratorio, cree una página HTML en el servidor de exploits que explote esta vulnerabilidad y llame al print() función.

En este laboratorio aprovechamos una vulnerabilidad de tipo DOM XSS donde la aplicación recibe mensajes mediante postMessage, los interpreta como JSON con JSON.parse y, según su contenido, modifica dinámicamente el atributo src de un iframe interno.

El ataque consiste en enviar desde un iframe un mensaje JSON con un campo type establecido en load-channel y un campo url que contiene un payload en forma de URL JavaScript. El listener interno interpreta el mensaje, valida el tipo y actualiza el iframe sin realizar ninguna comprobación de origen ni sanitización.

```html
<iframe src=https://0a7900b303e3ee3d80bcbce100c8005e.web-security-academy.net/ onload='this.contentWindow.postMessage("{\"type\":\"load-channel\",\"url\":\"javascript:print()\"}","*")'>
```

## Reto 4: Redirección abierta basada en DOM

Este laboratorio contiene una vulnerabilidad de redirección abierta basada en DOM. Para solucionarlo, explote esta vulnerabilidad y redirija a la víctima al servidor de exploits. 

En este laboratorio nos enfrentamos a una vulnerabilidad de tipo open redirection basada en el DOM. El comportamiento vulnerable se produce al hacer clic en un enlace que evalúa un parámetro url directamente desde el fragmento del location.href sin realizar ninguna validación.

El enlace ejecuta un pequeño script que extrae con una expresión regular el valor del parámetro url y redirige al usuario allí, lo cual puede ser aprovechado para enviar a la víctima a un dominio externo controlado por un atacante.

Para explotar el fallo, construimos una URL que apunta al sitio vulnerable incluyendo como valor del parámetro url la dirección del exploit server. Cuando la víctima accede, se realiza la redirección automática hacia el servidor del atacante.

```http
https://0aa800200443f2df80d1b78b007a0078.web-security-academy.net/post?postId=10&url=https://exploit-0a8000cb04f7f2df80c1b62001860024.exploit-server.net/
```

## Reto 5:Manipulación de cookies basada en DOM

Este laboratorio demuestra la manipulación de cookies del lado del cliente basada en DOM. Para resolver este laboratorio, inyecte una cookie que cause un XSS en una página diferente y llame a la print()Función. Necesitará usar el servidor de exploits para dirigir a la víctima a las páginas correctas. 

```html
<iframe src="https://0a6500a00473d2f780d826bc00f800ab.web-security-academy.net/product?productId=1&'><script>print()</script>" onload="if(!window.x)this.src='https://0a6500a00473d2f780d826bc00f800ab.web-security-academy.net';window.x=1;">
```