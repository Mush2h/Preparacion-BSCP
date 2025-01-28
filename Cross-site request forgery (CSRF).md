# Cross-site request forgery

## ¿Qué es CSRF?
La falsificación de solicitud entre sitios (también conocida como CSRF) es una vulnerabilidad de seguridad web que permite a un atacante inducir a los usuarios a realizar acciones que no tienen intención de realizar. Permite a un atacante eludir parcialmente la política de mismo origen, que está diseñada para evitar que diferentes sitios web interfieran entre sí.

## ¿Cuál es el impacto de un ataque CSRF?
En un ataque CSRF exitoso, el atacante hace que el usuario víctima realice una acción de forma no intencionada. Por ejemplo, puede ser cambiar la dirección de correo electrónico de su cuenta, cambiar su contraseña o realizar una transferencia de fondos. Según la naturaleza de la acción, el atacante podría obtener el control total de la cuenta del usuario. Si el usuario afectado tiene un rol privilegiado dentro de la aplicación, el atacante podría obtener el control total de todos los datos y funciones de la aplicación.

## ¿Cómo funciona CSRF?
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

```HTML
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
```

Si un usuario víctima visita la página web del atacante, ocurrirá lo siguiente:

- La página del atacante activará una solicitud HTTP al sitio web vulnerable.
- Si el usuario ha iniciado sesión en el sitio web vulnerable, su navegador incluirá automáticamente su cookie de sesión en la solicitud (suponiendo que no se utilicen cookies de SameSite).
- El sitio web vulnerable procesará la solicitud de forma normal, la tratará como si hubiera sido realizada por el usuario víctima y cambiará su dirección de correo electrónico.

### Nota
Aunque CSRF normalmente se describe en relación con el manejo de sesiones basado en cookies, también surge en otros contextos donde la aplicación agrega automáticamente algunas credenciales de usuario a las solicitudes, como la autenticación básica HTTP y la autenticación basada en certificados.

## Cómo construir un ataque CSRF

La creación manual del HTML necesario para un exploit CSRF puede ser complicada, en particular cuando la solicitud deseada contiene una gran cantidad de parámetros o existen otras peculiaridades en la solicitud. La forma más sencilla de construir un exploit CSRF es utilizando el generador de PoC CSRF que está integrado en Burp Suite Professional:

- Seleccione una solicitud en cualquier lugar de Burp Suite Professional que desee probar o explotar.
- Desde el menú contextual del botón derecho, seleccione Herramientas de participación / Generar PoC CSRF.
- Burp Suite generará algo de HTML que activará la solicitud seleccionada (menos las cookies, que serán agregadas automáticamente por el navegador de la víctima).
- Puede modificar varias opciones en el generador de PoC CSRF para ajustar aspectos del ataque. Es posible que deba hacer esto en algunas situaciones inusuales para lidiar con características peculiares de las solicitudes.
- Copie el HTML generado en una página web, véalo en un navegador que haya iniciado sesión en el sitio web vulnerable y pruebe si la solicitud prevista se emite con éxito y si se produce la acción deseada.

## Cómo distribuir un exploit CSRF
Los mecanismos de entrega de los ataques de falsificación de solicitud entre sitios son básicamente los mismos que los de los ataques XSS reflejados. Normalmente, el atacante coloca el HTML malicioso en un sitio web que controla y luego induce a las víctimas a visitar ese sitio web. Esto puede hacerse proporcionando al usuario un enlace al sitio web mediante un correo electrónico o un mensaje en las redes sociales. O si el ataque se coloca en un sitio web popular (por ejemplo, en un comentario de un usuario), es posible que simplemente esperen a que los usuarios visiten el sitio web.

Tenga en cuenta que algunos ataques CSRF simples emplean el método GET y pueden ser completamente autónomos con una única URL en el sitio web vulnerable. En esta situación, es posible que el atacante no necesite emplear un sitio externo y pueda proporcionar directamente a las víctimas una URL maliciosa en el dominio vulnerable. En el ejemplo anterior, si la solicitud para cambiar la dirección de correo electrónico se puede realizar con el método GET, entonces un ataque autónomo se vería así:

```ruby
<img src="https://vulnerable-website.com/email/change?email=pwned@evil-user.net">
```

## Defensas comunes contra el CSRF
En la actualidad, para encontrar y explotar vulnerabilidades CSRF con éxito, a menudo es necesario eludir las medidas anti-CSRF implementadas por el sitio web de destino, el navegador de la víctima o ambos. Las defensas más comunes con las que te encontrarás son las siguientes:

- Tokens CSRF : un token CSRF es un valor único, secreto e impredecible que genera la aplicación del lado del servidor y comparte con el cliente. Al intentar realizar una acción confidencial, como enviar un formulario, el cliente debe incluir el token CSRF correcto en la solicitud. Esto hace que sea muy difícil para un atacante crear una solicitud válida en nombre de la víctima.

- Cookies de SameSite : SameSite es un mecanismo de seguridad del navegador que determina cuándo se incluyen las cookies de un sitio web en las solicitudes que se originan en otros sitios web. Como las solicitudes para realizar acciones confidenciales suelen requerir una cookie de sesión autenticada, las restricciones adecuadas de SameSite pueden evitar que un atacante active estas acciones en varios sitios. Desde 2021, Chrome aplica `Lax` restricciones de SameSite de forma predeterminada. Como este es el estándar propuesto, esperamos que otros navegadores importantes adopten este comportamiento en el futuro.

- Validación basada en referenciadores : algunas aplicaciones utilizan el encabezado HTTP Referer para intentar defenderse de ataques CSRF, normalmente verificando que la solicitud se originó en el propio dominio de la aplicación. Esto suele ser menos eficaz que la validación de tokens CSRF.

## ¿Qué es un token CSRF?
Un token CSRF es un valor único, secreto e impredecible que genera la aplicación del lado del servidor y comparte con el cliente. Al emitir una solicitud para realizar una acción confidencial, como enviar un formulario, el cliente debe incluir el token CSRF correcto. De lo contrario, el servidor se negará a realizar la acción solicitada.

Una forma común de compartir tokens CSRF con el cliente es incluirlos como un parámetro oculto en un formulario HTML, por ejemplo:

<form name="change-email-form" action="/my-account/change-email" method="POST">
    <label>Email</label>
    <input required type="email" name="email" value="example@normal-website.com">
    <input required type="hidden" name="csrf" value="50FaWgdOhi9M9wyna8taR1k3ODOR8d6u">
    <button class='button' type='submit'> Update email </button>
</form>

Al enviar este formulario se obtiene la siguiente solicitud:
```ruby
POST /my-account/change-email HTTP/1.1
Host: normal-website.com
Content-Length: 70
Content-Type: application/x-www-form-urlencoded

csrf=50FaWgdOhi9M9wyna8taR1k3ODOR8d6u&email=example@normal-website.com

```
Cuando se implementan correctamente, los tokens CSRF ayudan a proteger contra ataques CSRF al dificultar que un atacante construya una solicitud válida en nombre de la víctima. Como el atacante no tiene forma de predecir el valor correcto del token CSRF, no podrá incluirlo en la solicitud maliciosa.

### Nota
Los tokens CSRF no tienen por qué enviarse como parámetros ocultos en una POST solicitud. Algunas aplicaciones colocan tokens CSRF en encabezados HTTP, por ejemplo. La forma en que se transmiten los tokens tiene un impacto significativo en la seguridad de un mecanismo en su conjunto. Para obtener más información, consulte Cómo prevenir vulnerabilidades CSRF.

## Fallas comunes en la validación de tokens CSRF
Las vulnerabilidades CSRF suelen surgir debido a una validación defectuosa de los tokens CSRF. En esta sección, abordaremos algunos de los problemas más comunes que permiten a los atacantes eludir estas defensas.

## La validación del token CSRF depende del método de solicitud
Algunas aplicaciones validan correctamente el token cuando la solicitud utiliza el método POST, pero omiten la validación cuando se utiliza el método GET.

En esta situación, el atacante puede cambiar al método GET para eludir la validación y lanzar un ataque CSRF:

```ruby
GET /email/change?email=pwned@evil-user.net HTTP/1.1
Host: vulnerable-website.com
Cookie: session=2yQIDcpia41WrATfjPqvm9tOkDvkMvLm
```

## La validación del token CSRF depende de la presencia del token
Algunas aplicaciones validan correctamente el token cuando está presente, pero omiten la validación si se omite el token.

En esta situación, el atacante puede eliminar todo el parámetro que contiene el token (no solo su valor) para eludir la validación y lanzar un ataque CSRF:
```ruby
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 25
Cookie: session=2yQIDcpia41WrATfjPqvm9tOkDvkMvLm

email=pwned@evil-user.net
``` 

## El token CSRF no está vinculado a la sesión del usuario
Algunas aplicaciones no validan que el token pertenezca a la misma sesión que el usuario que realiza la solicitud. En su lugar, la aplicación mantiene un grupo global de tokens que ha emitido y acepta cualquier token que aparezca en este grupo.

En esta situación, el atacante puede iniciar sesión en la aplicación usando su propia cuenta, obtener un token válido y luego pasar ese token al usuario víctima en su ataque CSRF.

## El token CSRF está vinculado a una cookie que no es de sesión
En una variación de la vulnerabilidad anterior, algunas aplicaciones vinculan el token CSRF a una cookie, pero no a la misma cookie que se utiliza para rastrear las sesiones. Esto puede ocurrir fácilmente cuando una aplicación emplea dos marcos diferentes, uno para el manejo de sesiones y otro para la protección CSRF, que no están integrados entre sí:

```ruby
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 68
Cookie: session=pSJYSScWKpmC60LpFOAHKixuFuM4uXWF; csrfKey=rZHCnSzEp8dbI6atzagGoSYyqJqTz5dv

csrf=RhV7yQDO0xcq9gLEah2WVbmuFqyOq7tY&email=wiener@normal-user.com
```

Esta situación es más difícil de explotar, pero sigue siendo vulnerable. Si el sitio web contiene algún comportamiento que permita a un atacante colocar una cookie en el navegador de la víctima, entonces es posible un ataque. El atacante puede iniciar sesión en la aplicación utilizando su propia cuenta, obtener un token válido y una cookie asociada, aprovechar el comportamiento de configuración de cookies para colocar su cookie en el navegador de la víctima y proporcionarle su token en su ataque CSRF.

### Nota
El comportamiento de configuración de cookies ni siquiera necesita existir dentro de la misma aplicación web que la vulnerabilidad CSRF. Cualquier otra aplicación dentro del mismo dominio DNS general puede potencialmente aprovecharse para configurar cookies en la aplicación que se está atacando, si la cookie que se controla tiene el alcance adecuado. Por ejemplo, una función de configuración de cookies en staging.demo.normal-website.compodría aprovecharse para colocar una cookie que se envíe a secure.normal-website.com.

## El token CSRF simplemente se duplica en una cookie
En otra variación de la vulnerabilidad anterior, algunas aplicaciones no mantienen ningún registro del lado del servidor de los tokens que se han emitido, sino que duplican cada token dentro de una cookie y un parámetro de solicitud. Cuando se valida la solicitud posterior, la aplicación simplemente verifica que el token enviado en el parámetro de solicitud coincida con el valor enviado en la cookie. Esto a veces se denomina defensa de "doble envío" contra CSRF y se recomienda porque es fácil de implementar y evita la necesidad de cualquier estado del lado del servidor:


```ruby
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 68
Cookie: session=1DQGdzYbOJQzLP7460tfyiv3do7MjyPw; csrf=R8ov2YBfTYmzFyjit8o2hKBuoIjXXVpa

csrf=R8ov2YBfTYmzFyjit8o2hKBuoIjXXVpa&email=wiener@normal-user.com
```

En esta situación, el atacante puede volver a realizar un ataque CSRF si el sitio web contiene alguna funcionalidad de configuración de cookies. En este caso, el atacante no necesita obtener un token válido propio. Simplemente inventa un token (quizás en el formato requerido, si se está verificando), aprovecha el comportamiento de configuración de cookies para colocar su cookie en el navegador de la víctima y le proporciona su token en su ataque CSRF.

## Cómo eludir las restricciones de cookies de SameSite
SameSite es un mecanismo de seguridad del navegador que determina cuándo se incluyen las cookies de un sitio web en las solicitudes que se originan en otros sitios web. Las restricciones de cookies de SameSite brindan protección parcial contra una variedad de ataques entre sitios, incluidos CSRF, fugas entre sitios y algunas vulnerabilidades CORS.

Desde 2021, Chrome aplica Laxrestricciones SameSite de forma predeterminada si el sitio web que emite la cookie no establece explícitamente su propio nivel de restricción. Se trata de un estándar propuesto y esperamos que otros navegadores importantes adopten este comportamiento en el futuro. Como resultado, es esencial tener un conocimiento sólido de cómo funcionan estas restricciones, así como de cómo se pueden eludir, para poder realizar pruebas exhaustivas en busca de vectores de ataque entre sitios.

En esta sección, primero explicaremos cómo funciona el mecanismo SameSite y aclararemos algunos términos relacionados. Luego, veremos algunas de las formas más comunes en las que puede eludir estas restricciones, lo que permite CSRF y otros ataques entre sitios en sitios web que inicialmente pueden parecer seguros.

## ¿Qué es un sitio en el contexto de las cookies de SameSite?
En el contexto de las restricciones de cookies de SameSite, un sitio se define como el dominio de nivel superior (TLD), generalmente algo como .como .net, más un nivel adicional del nombre de dominio. Esto se suele denominar TLD+1.

Al determinar si una solicitud pertenece o no al mismo sitio, también se tiene en cuenta el esquema de URL. Esto significa que la mayoría de los navegadores consideran que un enlace de `http://app.example.com` a `https://app.example.com` es entre sitios.
![alt text](image.png)
### Nota
Es posible que te encuentres con el término "dominio de nivel superior efectivo" (eTLD). Esto es simplemente una forma de dar cuenta de los sufijos multiparte reservados que se tratan como dominios de nivel superior en la práctica, como .co.uk.

## ¿Cuál es la diferencia entre un sitio y un origen?
La diferencia entre un sitio y un origen es su alcance: un sitio abarca varios nombres de dominio, mientras que un origen solo incluye uno. Aunque están estrechamente relacionados, es importante no utilizar los términos indistintamente, ya que mezclarlos puede tener graves consecuencias para la seguridad.

Se considera que dos URL tienen el mismo origen si comparten exactamente el mismo esquema, nombre de dominio y puerto. No obstante, tenga en cuenta que el puerto suele inferirse del esquema.
![alt text](image-1.png)

Como puede ver en este ejemplo, el término "sitio" es mucho menos específico, ya que solo tiene en cuenta el esquema y la última parte del nombre de dominio. Fundamentalmente, esto significa que una solicitud de origen cruzado puede seguir siendo del mismo sitio, pero no al revés.


| Request from | Request to | Same-site? | Same-origin? |
|-----------|-----------|-----------|-----------|
| https://example.com | https://example.com| Sí | Sí |
| https://app.example.com | https://intranet.example.com | Sí | No nombre de dominio no coincidente |
| https://example.com | https://example.com:8080 | Sí | No puerto no coincidente |
| https://example.com | https://example.co.uk | No eTLD no coincidente | No nombre de dominio no coincidente |
| https://example.com | http://example.com | No esquema no coincidente | No esquema no coincidente |

Esta es una distinción importante, ya que significa que cualquier vulnerabilidad que permita la ejecución arbitraria de JavaScript puede aprovecharse para eludir las defensas basadas en sitios en otros dominios que pertenecen al mismo sitio. Veremos un ejemplo de esto en uno de los laboratorios más adelante.
