# Server-side vulnerabilities

## Qué es un path traversal

El path traversal también conocido como recorrido de direcctorio. EStas vulnerabilidades permiten a un atacante leer archivos arbitrarios en el servidor que ejecuta en una aplicación.

Esto podría incluir:
Códigos y datos de la aplicación.
Credenciales para sistemas del back-end.
Archivos confidenciales del SO.

En algunos casos, un atacante podría escribir en archivos arbitrarios del servidor, lo que le permitiría modificar datos o comportamientos de la aplicacion y, en ultima instancia, tomar el control total del servidor.

## Lectura de archivos a través del path traversal

Imagine una aplicación de compras que muestra imagenes de artículos en venta. Esta podría cargar una imagen utilizando el siguiente código HTML

```html
<img src="/loadImage?filename=218.png">
```

La `loadImagen` en la URL toma un `filename` como parámetro y devuelve el contenido del archivo especificado.

Los archivos de imagen se almacenan en el disco en la ubicación `/var/www/images/218.png`

Esta aplicación no implementa defensa contra ataques de path traversal. 
como resultado, un atacante puede solicitar la siguiente URL para recuperar el `/etc/password` del sistema.

`https://insecure-website.com/loadImage?filename=../../../etc/passwd`


Esto hace que la aplicación lea desde la siguiente ruta

`/var/www/images/../../../etc/passwd`

La secuencia ../ es válida dentro de un ruta de archivos y significa subir un nivel en al estructura de directorios. Las tres ../ consecutivas suben de nivel desde /var/www/images la raíz del sistema de archivos, por lo que el archivo que se lee en realidad es `/etc/passwd/`

En los sistemas operativos basados en Unix, este es un archivo estándar que contiene detalles de lso usuarios registrados en el servidor , pero un atacante podría recuperar otros archivos arbitrarios utilizando la misma técnica

En Windows tanto ../ como ..\ son secuencias válidas de path traversal . Un ejemplo de ataque equivalente contra un servidor de Windows:

`https://insecure-website.com/loadImage?filename=..\..\..\windows\win.ini`

## ¿ Qué es el control de acceso ?

El control de acceso es la aplicación de restrinciones sobre quién o qué está autorizado a realizar acciones o acceder a resursos. En el contexto de las aplicaciones web , el control de acceso depende de la autenticación y la gestión de sesiones:

- La autenticación confirma que el usuario es quien dice ser.
- La gestión de sesiones identifica qué solicitudes HTTP posteriores realiza ese mismo usuario
- El control de acceso determina si el usuario puede realizar la acción que intenta realizar.

Los controles de acceso defectuosos son habituales y suelen representar una vulnerabilidad de seguridad critica .El diseño y la gestión de controles de acceso es un problema complejo y dinámico que aplica restrinciones comerciales , organizativas y legales a una implementación técnica . Las decisiones de diseño de los controles de acceso deben ser tomadas por personas , por lo que los errores son altos.

## Escalada de privilegios verticales

Si un usuario puede acceder a una función a la que no tiene permiso ,se trata de una escalada de privilegios vertical. Por ejemplo si un usuario no administrativo puede acceder a una ágina de administración donde puede eliminar cuentas de ususario, se trata de una escalada de privilegios vertical.

## Funcionalidad desprotegida

La escalada de privilegios vertica surge cuando una aplicación no aplica ninguna protección a las fucniones confidenciales. Por ejemplo las funciones pueden estar vicnuladas desde la página de bienvenida de un administrador, pero no desde la página de bienvenida de un admin. Sin embargo , un usuario podría tneer acceso a las funciones.
Sin embargo , un usuario pdría tener acceso a las funciones administrativas navegando  hata la URL de administración correspondiente.

Ejemplo:

```ruby
https://insecure-website.com/admin
```
En este sitio web podría alojar funcionalidad confidencial en la siguiente URL.

Cualquier usuario puede acceder a esta función no solo los usuarios administradores que tienen un enlace . En algunos casos el URL administrativa puede estar disponible en otras ubicaciones como en el `robots.txt`.

En algunos casos m se ocultan funciones sensibles al asignarles una URL menos predecible. es decir "seguridad por oscuridad". Sim embargo ocultar funciones sensibles no proporciona un control de acceso efectivo porque los usuarios pueden descubrir la URL oculta de varias maneras.

Ejemplo:

Aplicación que aloja funciones administrativas en esta URL:


```ruby
https://insecure-website.com/administrator-panel-fasdfadsf
```

Es posible que un atacante no pueda adivinar esto directamente. Sin embargo la aplicacion puede filtrar la URL a los usuarios y podría revelarse en Javascript que construye la interfaz de usuario en función del rol del usuario.

```ruby
<script>
	var isAdmin = false;
	if (isAdmin) {
		...
		var adminPanelTag = document.createElement('a');
		adminPanelTag.setAttribute('https://insecure-website.com/administrator-panel-fasdfadsf');
		adminPanelTag.innerText = 'Admin panel';
		...
	}
</script>
```
Este script agrega un enlace a la interfaz de usuario del usuario si es un usuario administrador. 
Sin embargo, el script que contiene la URL es visible para todos los usuarios, independientemente de su rol.

## Métodos de control de acceso basados ​​en parámetros

Algunas aplicaciones determinan los derechos de acceso o el rol del usuario al iniciar sesión y luego almacenan esta información en una ubicación que el usuario puede controlar. 
Esta podría ser:

- Un campo oculto.
- Una galleta.
- Un parámetro de cadena de consulta preestablecido.

La aplicación toma decisiones de control de acceso en función del valor enviado. Por ejemplo:

`https://insecure-website.com/login/home.jsp?admin=true`
`https://insecure-website.com/login/home.jsp?role=1`

Este enfoque es inseguro porque un usuario puede modificar el valor y acceder a funciones a las que no está autorizado, como funciones administrativas.

## Escalada de privilegios horizontal

La escalada de privilegios horizontal se produce si un usuario puede acceder a los recursos que pertenecen a otro usuario, en lugar de a sus propios recursos de ese tipo. Por ejemplo, si un empleado puede acceder a los registros de otros empleados además de a los suyos propios, se trata de una escalada de privilegios horizontal.

Los ataques de escalada de privilegios horizontales pueden utilizar métodos de explotación similares a los de la escalada de privilegios verticales. Por ejemplo, un usuario podría acceder a la página de su propia cuenta mediante la siguiente URL:

`https://insecure-website.com/myaccount?id=123`

Si un atacante modifica el idvalor del parámetro al de otro usuario, podría obtener acceso a la página de la cuenta de otro usuario y a los datos y funciones asociados.

Nota:
Este es un ejemplo de una vulnerabilidad de referencia directa a objetos (IDOR) insegura. Este tipo de vulnerabilidad surge cuando se utilizan valores de parámetros del controlador del usuario para acceder directamente a recursos o funciones.

En algunas aplicaciones, el parámetro explotable no tiene un valor predecible. Por ejemplo, en lugar de un número creciente, una aplicación podría utilizar identificadores únicos globales (GUID) para identificar a los usuarios. Esto puede impedir que un atacante adivine o prediga el identificador de otro usuario. Sin embargo, los GUID que pertenecen a otros usuarios podrían revelarse en otras partes de la aplicación donde se hace referencia a los usuarios, como mensajes o reseñas de usuarios.





