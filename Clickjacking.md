# Clickjacking

## Reto 1: Clickjacking básico con protección de token CSRF

Este laboratorio incluye la función de inicio de sesión y un botón para eliminar cuenta, protegido por un token CSRF. Un usuario hará clic en elementos que muestren la palabra "clic" en un sitio web falso.

Para resolver el laboratorio, crea un código HTML que enmarque la página de la cuenta y engañe al usuario para que la elimine. El laboratorio se resuelve al eliminar la cuenta.

Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales: wiener:peter 

Este laboratorio muestra cómo realizar un ataque de clickjacking incluso cuando la acción sensible está protegida por un token CSRF. Se aprovecha la falta de encabezados de protección como X-Frame-Options para incrustar la página vulnerable dentro de un iframe en un sitio externo.

La estrategia consiste en superponer un elemento visual atractivo como un botón que diga Click me justo encima del botón de eliminar cuenta. Para ello se ajustan las propiedades de posición, tamaño y opacidad del iframe de forma que el usuario, al intentar hacer clic en el contenido visible del sitio señuelo, en realidad pulse sobre la acción destructiva del sitio objetivo.

Este tipo de ataque es posible cuando no existen medidas que impidan la carga del sitio objetivo en iframes de terceros, y es aún más efectivo cuando el usuario realiza la acción de manera involuntaria al seguir una instrucción aparentemente inofensiva como haz clic aquí.



```html
<style>
    iframe {
        position:relative;
        width:700px;
        height: 500px;
        opacity: 0.1;
        z-index: 2;
    }
    div {
        position:absolute;
        top:500px;
        left:60px;
        z-index: 1;
    }
</style>
<div>Click me</div>
<iframe src="https://0adf000104c2e8458060033100b60023.web-security-academy.net/my-account"></iframe>
```

## Reto 2: Clickjacking con datos de entrada de formulario precargados desde un parámetro de URL

Este laboratorio amplía el ejemplo básico de clickjacking del Laboratorio: Clickjacking básico con protección de token CSRF . El objetivo del laboratorio es cambiar la dirección de correo electrónico del usuario rellenando previamente un formulario con un parámetro URL e incitando al usuario a hacer clic inadvertidamente en el botón "Actualizar correo electrónico".

Para resolver el laboratorio, cree un HTML que enmarque la página de la cuenta y engañe al usuario para que actualice su dirección de correo electrónico haciendo clic en un señuelo de "Haz clic aquí". El laboratorio se resuelve al cambiar la dirección de correo electrónico.

Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales: wiener:peter 

En este caso, el objetivo es engañar al usuario para que pulse sobre el botón de actualizar correo electrónico, el cual ha sido rellenado automáticamente con una dirección maliciosa pasada por parámetro.

El ataque se basa en superponer un iframe con el formulario de cuenta sobre un botón señuelo que diga Click me. La URL del iframe incluye un parámetro email con el valor que se desea establecer. De este modo, el formulario se precompleta sin necesidad de interacción directa con el campo.

El usuario, al hacer clic en el señuelo, en realidad está enviando el formulario del iframe con el nuevo correo ya cargado. Este tipo de ataque demuestra que incluso sin vulnerabilidades en el backend, una interfaz mal protegida puede ser manipulada visualmente para obtener acciones críticas sin el conocimiento del usuario.



```html
<style>
    iframe {
        position:relative;
        width:700px;
        height: 500px;
        opacity: 0.1;
        z-index: 2;
    }
    div {
        position:absolute;
        top:450px;
        left:60px;
        z-index: 1;
    }
</style>
<div>Click me</div>
<iframe src="https://0a46007a041591aa80bed5b10053007b.web-security-academy.net/my-account?email=hacked@hacked.com"></iframe>
```

## Reto 3: Clickjacking con un script frame-buster

Este laboratorio está protegido por un detector de marcos que impide que el sitio web sea enmarcado. ¿Puedes sortear el detector de marcos y realizar un ataque de clickjacking que cambie la dirección de correo electrónico del usuario?

Para resolver el laboratorio, cree un HTML que enmarque la página de la cuenta y engañe al usuario para que cambie su dirección de correo electrónico haciendo clic en "Haz clic aquí". El laboratorio se resuelve al cambiar la dirección de correo electrónico.

Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales: wiener:peter 

Este laboratorio introduce una protección habitual contra ataques de clickjacking conocida como frame buster. Se trata de un script que impide que la página objetivo sea cargada dentro de un iframe, bloqueando así la superposición de contenidos engañosos. Sin embargo, el ataque sigue siendo posible si se neutraliza adecuadamente este mecanismo.

El truco consiste en usar el atributo sandbox dentro del iframe, permitiendo solo la ejecución de formularios mediante allow-forms. Esto impide que el script de ruptura del marco (normalmente basado en comprobar si ‘window.top !== window.self‘) se ejecute, ya que no tiene los permisos necesarios para acceder al contexto superior.

El atacante construye una página maliciosa que muestra un botón señuelo con el texto Click me y coloca justo debajo el iframe que contiene el formulario de actualización de email con el campo ya preestablecido a través de un parámetro en la URL.



```html`
<style>
    iframe {
        position:relative;
        width:800px;
        height: 600px;
        opacity: 0.1;
        z-index: 2;
    }
    div {
        position:absolute;
        top:450px;
        left:60px;
        z-index: 1;
    }
</style>
<div>Click me</div>
<iframe sandbox="allow-forms" src="https://0a0e003c04cc32fa80b42be800660044.web-security-academy.net/my-account?email=pwnd@hacked.com"></iframe>
```

## Reto 4: Explotación de la vulnerabilidad de clickjacking para activar XSS basado en DOM



Este laboratorio contiene una vulnerabilidad XSS que se activa con un clic. Construya un ataque de clickjacking que engañe al usuario para que haga clic en el botón "Haz clic aquí" y llame al... print() función. 

En este laboratorio se combina una vulnerabilidad de clickjacking con una XSS basada en DOM que se activa mediante un clic. El objetivo es engañar a la víctima para que pulse un botón en una página aparentemente inocente, lo que desencadena la ejecución de la función print() desde un payload XSS.

Para lograrlo, se incrusta la URL del formulario de feedback en un iframe, manipulando los parámetros de la URL para incluir un payload malicioso en el campo name. Este payload se aprovecha de una mala gestión de datos en el DOM, concretamente en la parte de la página que muestra los resultados del envío (#feedbackResult).

Mediante el uso de estilos CSS, se superpone un botón señuelo encima del botón real de “Submit feedback”. Gracias a la opacidad casi nula del iframe, la víctima no ve el contenido real del sitio y cree estar interactuando con la interfaz legítima.



```html
<style>
    iframe {
        position:relative;
        width:900px;
        height: 900px;
        opacity: 0.1;
        z-index: 2;
    }
    div {
        position:absolute;
        top:800px;
        left:90px;
        z-index: 1;
    }
</style>
<div>Click me</div>
<iframe  src="https://0aae001b04bc99e5809cad5300cd0007.web-security-academy.net/feedback?name=%3Cimg%20src=0%20onerror=print()%3E&email=hola@hola.com&subject=test&message=probando"></iframe>

```

## Reto 5:Clickjacking de varios pasos


Este laboratorio cuenta con funciones de cuenta protegidas por un token CSRF y un cuadro de diálogo de confirmación para proteger contra el secuestro de clics. Para resolver este laboratorio, cree un ataque que engañe al usuario para que haga clic en el botón de eliminar cuenta y en el cuadro de diálogo de confirmación mediante las acciones señuelo "Haz clic primero" y "Haz clic después". Necesitará usar dos elementos para este laboratorio.

Puede iniciar sesión en la cuenta usted mismo utilizando las siguientes credenciales: wiener:peter 

En este laboratorio se presenta una defensa adicional contra el clickjacking: además de un token CSRF, el botón de eliminación de cuenta incluye una ventana de confirmación. Para explotar esta funcionalidad, el ataque debe engañar a la víctima para que realice dos clics consecutivos en ubicaciones específicas: uno para activar el botón de “Eliminar cuenta” y otro para confirmar la acción.

Para lograrlo, se estructura una página con dos capas señuelo (Click me first y Click me next), alineadas con los botones reales del formulario y del cuadro de confirmación, respectivamente. El iframe que contiene la página objetivo se vuelve prácticamente invisible gracias al uso de opacidad muy baja.

Una vez que la víctima interactúa con los dos elementos señuelo, el proceso completo de eliminación de cuenta se ejecuta con éxito, incluyendo la confirmación del diálogo. Este ataque demuestra cómo el clickjacking puede seguir siendo efectivo incluso en flujos de múltiples pasos si no se implementan defensas robustas adicionales como ‘frame-ancestors‘ o ‘same-origin‘ en ‘Content-Security-Policy‘.


```html
<style>
    iframe {
        position:relative;
        width:900px;
        height: 900px;
        opacity: 0.1;
        z-index: 2;
    }
    .firstClick {
        position:absolute;
        top:500px;
        left:80px;
        z-index: 1;
    }
    .secondClick {
        position:absolute;
        top:300px;
        left:200px;
        z-index: 1;
    }
</style>
<div class="firstClick">Click me first</div>
<div class="secondClick ">Click me next</div>
<iframe  src="https://0af9009b04a020ca8824e417004600af.web-security-academy.net/my-account"></iframe>
```
