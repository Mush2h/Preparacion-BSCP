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