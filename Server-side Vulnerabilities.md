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


## Escalada de privilegios de horizontal a vertical
A menudo, un ataque de escalada de privilegios horizontal puede convertirse en uno vertical, comprometiendo a un usuario con más privilegios. Por ejemplo, una escalada horizontal podría permitir a un atacante restablecer o capturar la contraseña de otro usuario. Si el atacante ataca a un usuario administrativo y compromete su cuenta, puede obtener acceso administrativo y, por lo tanto, realizar una escalada de privilegios vertical.

Un atacante podría obtener acceso a la página de la cuenta de otro usuario utilizando la técnica de manipulación de parámetros ya descrita para la escalada de privilegios horizontal:

https://insecure-website.com/myaccount?id=456
Si el usuario objetivo es un administrador de la aplicación, el atacante obtendrá acceso a una página de cuenta administrativa. Esta página puede revelar la contraseña del administrador o proporcionar un medio para cambiarla, o puede proporcionar acceso directo a funciones privilegiadas.

## Vulnerabilidades de autenticación
En teoría, las vulnerabilidades de autenticación son fáciles de entender. Sin embargo, suelen ser críticas debido a la clara relación entre autenticación y seguridad.

Las vulnerabilidades de autenticación pueden permitir a los atacantes obtener acceso a datos y funciones confidenciales. También exponen una superficie de ataque adicional para futuras vulnerabilidades. Por este motivo, es importante aprender a identificar y explotar las vulnerabilidades de autenticación y a eludir las medidas de protección habituales.

En esta sección explicamos:

Los mecanismos de autenticación más comunes utilizados por los sitios web.
Posibles vulnerabilidades en estos mecanismos.
Vulnerabilidades inherentes a diferentes mecanismos de autenticación.
Vulnerabilidades típicas que se introducen por su implementación incorrecta.
Cómo hacer que sus propios mecanismos de autenticación sean lo más robustos posible

## ¿Cuál es la diferencia entre autenticación y autorización?
La autenticación es el proceso de verificar que un usuario es quien dice ser. La autorización implica verificar si un usuario tiene permiso para hacer algo.

Por ejemplo, la autenticación determina si alguien que intenta acceder a un sitio web con el nombre de usuario `Carlos123es` realmente la misma persona que creó la cuenta.

Una vez Carlos123 autenticado, sus permisos determinan lo que está autorizado a hacer. Por ejemplo, puede estar autorizado a acceder a información personal sobre otros usuarios o realizar acciones como eliminar la cuenta de otro usuario.

## Ataques de fuerza bruta
Un ataque de fuerza bruta es cuando un atacante utiliza un sistema de prueba y error para adivinar las credenciales válidas de un usuario. Estos ataques suelen estar automatizados mediante listas de nombres de usuario y contraseñas. La automatización de este proceso, especialmente mediante herramientas dedicadas, permite potencialmente que un atacante realice una gran cantidad de intentos de inicio de sesión a gran velocidad.

La fuerza bruta no siempre consiste en adivinar nombres de usuario y contraseñas de forma totalmente aleatoria. Al utilizar también lógica básica o conocimientos disponibles públicamente, los atacantes pueden afinar los ataques de fuerza bruta para realizar conjeturas mucho más fundamentadas. Esto aumenta considerablemente la eficacia de dichos ataques. Los sitios web que dependen del inicio de sesión con contraseña como único método de autenticación de los usuarios pueden ser muy vulnerables si no implementan una protección suficiente contra la fuerza bruta.

## Nombres de usuario forzados por fuerza bruta
Los nombres de usuario son especialmente fáciles de adivinar si se ajustan a un patrón reconocible, como una dirección de correo electrónico. Por ejemplo, es muy común ver inicios de sesión comerciales en el formato firstname.lastname@somecompany.com. Sin embargo, incluso si no hay un patrón obvio, a veces incluso las cuentas con privilegios elevados se crean utilizando nombres de usuario predecibles, como admino administrator.

Durante la auditoría, verifique si el sitio web revela nombres de usuario potenciales de forma pública. Por ejemplo, ¿puede acceder a los perfiles de usuario sin iniciar sesión? Incluso si el contenido real de los perfiles está oculto, el nombre utilizado en el perfil a veces es el mismo que el nombre de usuario de inicio de sesión. También debe verificar las respuestas HTTP para ver si se revela alguna dirección de correo electrónico. Ocasionalmente, las respuestas contienen direcciones de correo electrónico de usuarios con privilegios elevados, como administradores o soporte de TI.

## Fuerza bruta de contraseñas

Las contraseñas también se pueden descifrar mediante fuerza bruta, y la dificultad varía según la fortaleza de la contraseña. Muchos sitios web adoptan algún tipo de política de contraseñas que obliga a los usuarios a crear contraseñas de alta entropía que, al menos en teoría, son más difíciles de descifrar mediante fuerza bruta únicamente. Esto suele implicar la aplicación de contraseñas con:

- Un número mínimo de caracteres
- Una mezcla de letras mayúsculas y minúsculas
- Al menos un carácter especial

Sin embargo, aunque las contraseñas de alta entropía son difíciles de descifrar para las computadoras por sí solas, podemos usar un conocimiento básico del comportamiento humano para explotar las vulnerabilidades que los usuarios introducen involuntariamente en este sistema. En lugar de crear una contraseña segura con una combinación aleatoria de caracteres, los usuarios a menudo toman una contraseña que pueden recordar e intentan forzarla para que se ajuste a la política de contraseñas. Por ejemplo, si mypasswordno está permitido, los usuarios pueden intentar algo como Mypassword1!o Myp4$$w0rden su lugar.

En los casos en los que la política exige que los usuarios cambien sus contraseñas periódicamente, también es habitual que los usuarios realicen cambios menores y predecibles en su contraseña preferida. Por ejemplo, Mypassword1!se convierte Mypassword1?en oMypassword2!.

Este conocimiento de credenciales probables y patrones predecibles significa que los ataques de fuerza bruta a menudo pueden ser mucho más sofisticados y, por lo tanto, efectivos, que simplemente iterar a través de cada combinación posible de caracteres.

## Enumeración de nombres de usuario
La enumeración de nombres de usuario es cuando un atacante puede observar cambios en el comportamiento del sitio web para identificar si un nombre de usuario determinado es válido.

La enumeración de nombres de usuario suele ocurrir en la página de inicio de sesión, por ejemplo, cuando se introduce un nombre de usuario válido pero una contraseña incorrecta, o en los formularios de registro cuando se introduce un nombre de usuario que ya está en uso. Esto reduce en gran medida el tiempo y el esfuerzo necesarios para forzar un inicio de sesión, ya que el atacante puede generar rápidamente una lista de nombres de usuario válidos.

## Cómo eludir la autenticación de dos factores
A veces, la implementación de la autenticación de dos factores es tan defectuosa que puede eludirse por completo.

Si primero se le solicita al usuario que ingrese una contraseña y luego se le solicita que ingrese un código de verificación en una página separada, el usuario estará efectivamente en un estado de "inicio de sesión" antes de haber ingresado el código de verificación. En este caso, vale la pena probar para ver si puede saltar directamente a las páginas de "solo inicio de sesión" después de completar el primer paso de autenticación. Ocasionalmente, encontrará que un sitio web no verifica si completó o no el segundo paso antes de cargar la página.

## ¿Qué es la SSRF?
La falsificación de solicitud del lado del servidor es una vulnerabilidad de seguridad web que permite a un atacante hacer que la aplicación del lado del servidor realice solicitudes a una ubicación no deseada.

En un ataque SSRF típico, el atacante podría hacer que el servidor se conecte a servicios internos exclusivos dentro de la infraestructura de la organización. En otros casos, es posible que pueda obligar al servidor a conectarse a sistemas externos arbitrarios. Esto podría filtrar datos confidenciales, como credenciales de autorización.

## Ataques SSRF contra el servidor
En un ataque SSRF contra el servidor, el atacante hace que la aplicación envíe una solicitud HTTP al servidor que la aloja, a través de su interfaz de red de bucle invertido. Esto normalmente implica proporcionar una URL con un nombre de host como 127.0.0.1(una dirección IP reservada que apunta al adaptador de bucle invertido) o localhost(un nombre de uso común para el mismo adaptador).

Por ejemplo, imaginemos una aplicación de compras que permite al usuario ver si un artículo está en stock en una tienda en particular. Para proporcionar la información sobre el stock, la aplicación debe consultar varias API REST de back-end. Para ello, pasa la URL al punto final de la API de back-end correspondiente a través de una solicitud HTTP de front-end. Cuando un usuario ve el estado del stock de un artículo, su navegador realiza la siguiente solicitud:

```ruby
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://stock.weliketoshop.net:8080/product/stock/check%3FproductId%3D6%26storeId%3D1
```

Esto hace que el servidor realice una solicitud a la URL especificada, recupere el estado del stock y lo devuelva al usuario.

En este ejemplo, un atacante puede modificar la solicitud para especificar una URL local del servidor:

```ruby
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://localhost/admin
```

El servidor obtiene el contenido de la /adminURL y lo devuelve al usuario.

Un atacante puede visitar la /adminURL, pero la funcionalidad administrativa normalmente solo es accesible para usuarios autenticados. Esto significa que un atacante no verá nada de interés. Sin embargo, si la solicitud a la /adminURL proviene de la máquina local, se omiten los controles de acceso normales. La aplicación otorga acceso completo a la funcionalidad administrativa, porque la solicitud parece originarse desde una ubicación confiable.

