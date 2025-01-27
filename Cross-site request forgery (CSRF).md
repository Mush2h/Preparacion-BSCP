# Cross-site request forgery

# ¿Qué es CSRF?
La falsificación de solicitud entre sitios (también conocida como CSRF) es una vulnerabilidad de seguridad web que permite a un atacante inducir a los usuarios a realizar acciones que no tienen intención de realizar. Permite a un atacante eludir parcialmente la política de mismo origen, que está diseñada para evitar que diferentes sitios web interfieran entre sí.

# ¿Cuál es el impacto de un ataque CSRF?
En un ataque CSRF exitoso, el atacante hace que el usuario víctima realice una acción de forma no intencionada. Por ejemplo, puede ser cambiar la dirección de correo electrónico de su cuenta, cambiar su contraseña o realizar una transferencia de fondos. Según la naturaleza de la acción, el atacante podría obtener el control total de la cuenta del usuario. Si el usuario afectado tiene un rol privilegiado dentro de la aplicación, el atacante podría obtener el control total de todos los datos y funciones de la aplicación.

# ¿Cómo funciona CSRF?
Para que sea posible un ataque CSRF, deben cumplirse tres condiciones clave:

- Una acción relevante. 
Hay una acción dentro de la aplicación que el atacante tiene motivos para inducir. Puede ser una acción privilegiada (como modificar los permisos de otros usuarios) o cualquier acción sobre datos específicos del usuario (como cambiar la contraseña del usuario).
- Manejo de sesiones basado en cookies. 
Para realizar la acción se deben emitir una o más solicitudes HTTP y la aplicación se basa únicamente en las cookies de sesión para identificar al usuario que realizó las solicitudes. No existe ningún otro mecanismo para realizar el seguimiento de las sesiones o validar las solicitudes de los usuarios.
- Sin parámetros de solicitud impredecibles. 
Las solicitudes que realizan la acción no contienen ningún parámetro cuyos valores el atacante no pueda determinar o adivinar. Por ejemplo, al hacer que un usuario cambie su contraseña, la función no es vulnerable si un atacante necesita saber el valor de la contraseña existente.

Por ejemplo, supongamos que una aplicación contiene una función que permite al usuario cambiar la dirección de correo electrónico de su cuenta. Cuando un usuario realiza esta acción, realiza una solicitud HTTP como la siguiente:

```ruby
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 30
Cookie: session=yvthwsztyeQkAPzeQ5gHgTvlyxHfsAfE

email=wiener@normal-user.com
```

Esto cumple las condiciones requeridas para CSRF:

La acción de cambiar la dirección de correo electrónico de la cuenta de un usuario es de interés para un atacante. Después de esta acción, el atacante normalmente podrá activar un restablecimiento de contraseña y tomar el control total de la cuenta del usuario.
La aplicación utiliza una cookie de sesión para identificar qué usuario realizó la solicitud. No existen otros tokens ni mecanismos para realizar un seguimiento de las sesiones de los usuarios.
El atacante puede determinar fácilmente los valores de los parámetros de solicitud que se necesitan para realizar la acción.

Con estas condiciones establecidas, el atacante puede construir una página web que contenga el siguiente HTML:

<html>
    <body>
        <form action="https://vulnerable-website.com/email/change" method="POST">
            <input type="hidden" name="email" value="pwned@evil-user.net" />
        </form>
        <script>
            document.forms[0].submit();
        </script>
    </body>
</html>

Si un usuario víctima visita la página web del atacante, ocurrirá lo siguiente:

- La página del atacante activará una solicitud HTTP al sitio web vulnerable.
- Si el usuario ha iniciado sesión en el sitio web vulnerable, su navegador incluirá automáticamente su cookie de sesión en la solicitud (suponiendo que no se utilicen cookies de SameSite).
- El sitio web vulnerable procesará la solicitud de forma normal, la tratará como si hubiera sido realizada por el usuario víctima y cambiará su dirección de correo electrónico.

### Nota
Aunque CSRF normalmente se describe en relación con el manejo de sesiones basado en cookies, también surge en otros contextos donde la aplicación agrega automáticamente algunas credenciales de usuario a las solicitudes, como la autenticación básica HTTP y la autenticación basada en certificados.