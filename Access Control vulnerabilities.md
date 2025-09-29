## Reto 1: Funcionalidad de administración sin protección

Este laboratorio tiene un panel de administración desprotegido.
Resuelva el laboratorio eliminando el usuario carlos.

El archivo ‘robots.txt’, utilizado para indicar a los motores de búsqueda qué rutas no indexar, revela accidentalmente la ubicación del panel de administración. Al acceder a la URL filtrada, encontramos una funcionalidad crítica sin ningún tipo de autenticación.

```
https://0a97000004cf93da871656a500b60090.web-security-academy.net/administrator-panel
```

## Reto 2: Funcionalidad de administración sin protección con URL impredecible

Este laboratorio tiene un panel de administración sin protección. Está ubicado en una ubicación impredecible, pero su ubicación se indica en alguna parte de la aplicación.

Resuelva el laboratorio accediendo al panel de administración y utilizándolo para eliminar al usuario. carlos. 

Aunque el panel de administración no se encuentra en una ruta estándar, su ubicación se filtra en el código JavaScript de la página principal.

```
https://0aa0001504ffa77b80d735aa00790063.web-security-academy.net/admin-pi11dl
```

## Reto 3: Rol de usuario controlado por parámetro de solicitud

Este laboratorio tiene un panel de administración en /admin, que identifica a los administradores mediante una cookie falsificable.
Resuelva el laboratorio accediendo al panel de administración y usándolo para eliminar al usuario carlos.
Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales: wiener:peter 

Durante el proceso de login, interceptamos la respuesta que establece una cookie llamada ‘Admin=false‘.

Al modificarla a ‘Admin=true‘, obtenemos privilegios de administrador y accedemos al panel oculto en /admin.

## Reto 4: El rol del usuario se puede modificar en el perfil del usuario

Este laboratorio tiene un panel de administración en /admin. Solo es accesible para usuarios registrados con una role id de 2.

Resuelva el laboratorio accediendo al panel de administración y usándolo para eliminar al usuario carlos.

Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales: wiener:peter 

Al editar nuestra dirección de correo desde la cuenta, interceptamos la petición y añadimos el campo ‘“roleid”:2‘ al cuerpo JSON.

```
{"email":"hola@peter.com",
"roleid": 2
}
```

## Reto 5: ID de usuario controlado por parámetro de solicitud

Este laboratorio tiene una vulnerabilidad de escalada de privilegios horizontal en la página de la cuenta de usuario.
Para resolver el laboratorio, obtenga la clave API del usuario carlos y presentarlo como solución.
Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales: wiener:peter 

```
https://0aee00d903b385f483ece75a000600bd.web-security-academy.net/my-account?id=carlos
```

## Reto 6: ID de usuario controlado por parámetro de solicitud, con ID de usuario impredecibles

Este laboratorio tiene una vulnerabilidad de escalada de privilegios horizontal en la página de la cuenta de usuario, pero identifica a los usuarios con GUID.

Para resolver el laboratorio, busque el GUID para carlos, luego envíe su clave API como solución.

Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales: wiener:peter 

Identificamos el GUID del usuario carlos accediendo a una publicación suya y copiando su ID desde la URL.

Luego, modificamos el parámetro id en nuestra página de cuenta con ese identificador para obtener su API key

```
https://0a9c00c703765004810135e700e800ab.web-security-academy.net/my-account?id=818d0394-ff76-4e90-b9f6-05c900b0eb02
```

## Reto 7: ID de usuario controlado por parámetro de solicitud con fuga de datos en redirección

Este laboratorio contiene una vulnerabilidad de control de acceso donde se filtra información confidencial en el cuerpo de una respuesta de redirección.

Para resolver el laboratorio, obtenga la clave API del usuario carlosy presentarlo como solución.

Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales: wiener:peter 

Modificamos el parámetro id en nuestra página de cuenta para apuntar a carlos.

Aunque el servidor nos redirige, en el cuerpo de la respuesta se filtra la API key de carlos, permitiendo obtener su información sin autorización.

```
https://0a7000c10311f1b5802e716e000200a7.web-security-academy.net/my-account?id=carlos
```


## Reto 9: Referencias directas a objetos inseguras

Este laboratorio almacena los registros de chat de los usuarios directamente en el sistema de archivos del servidor y los recupera mediante URL estáticas.

Resuelva el laboratorio encontrando la contraseña del usuario. carlos, e iniciar sesión en su cuenta. 

Tras enviar un mensaje desde la pestaña de chat, accedemos a la transcripción y analizamos la URL. Comprobamos que los nombres de archivo siguen un patrón incremental (1.txt, 2.txt, etc.), por lo que los modificamos manualmente para intentar acceder a otras conversaciones.

```
GET /download-transcript/1.txt HTTP/2
```

```
Hal Pline: Takes one to know one
You: Ok so my password is ph5kkhg4hoft49p2qkhw. Is that right?
Hal Pline: Yes it is!
You: Ok thanks, bye!
```

## Reto 10: 