## Reto 1 y Reto 2 fáciles.

## Reto 3: Lógica rota en el restablecimiento de contraseña

La función de restablecimiento de contraseña de este laboratorio es vulnerable. Para solucionar el problema, restablezca la contraseña de Carlos, inicie sesión y acceda a su página "Mi cuenta".
Sus credenciales: wiener:peter Nombre de usuario de la víctima: carlos

El servidor permite cambiar la contraseña sin validar el token enviado por email, lo que nos permite interceptar nuestra propia solicitud de cambio, eliminar el token y modificar el nombre de usuario por el de Carlos.

```
POST /forgot-password?temp-forgot-password-token= HTTP/2
temp-forgot-password-token=&username=carlos&new-password-1=test&new-password-2=test
```
## Reto 4: Enumeración de nombres de usuario mediante respuestas sutilmente diferentes

Este laboratorio es sutilmente vulnerable a ataques de fuerza bruta de enumeración de nombres de usuario y contraseñas. Tiene una cuenta con un nombre de usuario y contraseña predecibles, que se pueden encontrar en las siguientes listas de palabras:
Para resolver el laboratorio, enumere un nombre de usuario válido, fuerce la contraseña de este usuario y luego acceda a la página de su cuenta. 

veremos cómo identificar un nombre de usuario válido mediante pequeñas diferencias en los mensajes de error que devuelve el servidor, y posteriormente realizar un ataque de fuerza bruta sobre su contraseña para acceder a su cuenta.

Todo el proceso se realiza desde Burp Intruder, apoyándonos en la función Grep – Extract para detectar variaciones mínimas en la respuesta que nos permitan distinguir usuarios existentes.

## Reto 5: Enumeración de nombres de usuario mediante el tiempo de respuesta

Este laboratorio es vulnerable a la enumeración de nombres de usuario mediante sus tiempos de respuesta. Para resolver el laboratorio, enumere un nombre de usuario válido, utilice fuerza bruta para obtener su contraseña y acceda a su página de cuenta.
Sus credenciales: wiener:peter
Nombres de usuario de los candidatos
Contraseñas de candidatos

Explotar una vulnerabilidad basada en el tiempo de respuesta del servidor para identificar nombres de usuario válidos. A través de Burp Suite, lanzarás un ataque tipo timing aprovechando que el servidor tarda más en responder cuando el nombre de usuario es correcto pero la contraseña es incorrecta y larga.

Además, el laboratorio implementa un sistema de bloqueo por IP tras varios intentos fallidos, lo cual se evita manipulando la cabecera ‘X-Forwarded-For‘ para simular distintas direcciones IP. 

```
POST /login HTTP/2
X-Forwarded-For: 127.0.0.93

username=atlanta&password=AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```
## Reto 6: 
