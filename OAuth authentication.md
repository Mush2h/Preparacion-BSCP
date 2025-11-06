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