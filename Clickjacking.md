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
