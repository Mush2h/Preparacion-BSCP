# path traversal

## Qué es un path traversal

El path traversal también conocido como recorrido de direcctorio. Estas vulnerabilidades permiten a un atacante leer archivos arbitrarios en el servidor que ejecuta en una aplicación.

Esto podría incluir:

- Códigos y datos de la aplicación.
- Credenciales para sistemas del back-end.
- Archivos confidenciales del SO.

En algunos casos, un atacante podría escribir en archivos arbitrarios del servidor, lo que le permitiría modificar datos o comportamientos de la aplicacion y, en ultima instancia, 
tomar el control total del servidor.

## Lectura de archivos arbitrarios a través del recorrido de ruta

Imagine una aplicación de compras que muestra imágenes de artículos en venta. Esta podría cargar una imagen utilizando el siguiente código HTML:

```ruby
<img src="/loadImage?filename=218.png">
```

La loadImageURL toma un `filename` parámetro y devuelve el contenido del archivo especificado. Los archivos de imagen se almacenan en el disco en la ubicación `/var/www/images/`. Para devolver una imagen, la aplicación agrega el nombre de archivo solicitado a este directorio base y utiliza una API del sistema de archivos para leer el contenido del archivo. En otras palabras, la aplicación lee desde la siguiente ruta de archivo:

```ruby
/var/www/images/218.png
```

Esta aplicación no implementa defensas contra ataques de cruce de ruta. Como resultado, un atacante puede solicitar la siguiente URL para recuperar el `/etc/passwd` archivo del sistema de archivos del servidor:

```ruby
https://insecure-website.com/loadImage?filename=../../../etc/passwd
```

Esto hace que la aplicación lea desde la siguiente ruta de archivo:

`/var/www/images/../../../etc/passwd`

La secuencia ../es válida dentro de una ruta de archivo y significa subir un nivel en la estructura de directorios. Las tres ../secuencias consecutivas suben de nivel desde /var/www/images/la raíz del sistema de archivos, por lo que el archivo que se lee en realidad es:

`/etc/passwd`

En los sistemas operativos basados ​​en Unix, este es un archivo estándar que contiene detalles de los usuarios registrados en el servidor, pero un atacante podría recuperar otros archivos arbitrarios utilizando la misma técnica.

En Windows, tanto ../y como ..\son secuencias válidas de recorrido de directorio. El siguiente es un ejemplo de un ataque equivalente contra un servidor basado en Windows:

```ruby
https://insecure-website.com/loadImage?filename=..\..\..\windows\win.ini
```

## Obstáculos comunes para explotar vulnerabilidades de recorrido de ruta

Muchas aplicaciones que introducen datos del usuario en rutas de archivos implementan defensas contra ataques de cruce de rutas, que a menudo se pueden eludir.

Si una aplicación elimina o bloquea las secuencias de recorrido de directorio del nombre de archivo proporcionado por el usuario, podría ser posible eludir la defensa utilizando una variedad de técnicas.

Es posible que puedas usar una ruta absoluta desde la raíz del sistema de archivos, como `filename=/etc/passwd`, para hacer referencia directa a un archivo sin utilizar ninguna secuencia de recorrido.

Es posible que puedas usar secuencias de recorrido anidadas, como `....//`o `....\/`. Estas se convierten en secuencias de recorrido simples cuando se elimina la secuencia interna.

En algunos contextos, como en una ruta URL o en el `filename` parámetro de una `multipart/form-data`solicitud, los servidores web pueden eliminar cualquier secuencia de recorrido de directorio antes de pasar su entrada a la aplicación. A veces, puede evitar este tipo de limpieza codificando la URL o incluso codificando la URL dos veces, `../` lo que da como resultado `%2e%2e%2f`y `%252e%252e%252f`respectivamente. También pueden funcionar varias codificaciones no estándar, como `..%c0%afo` `..%ef%bc%8f`.

Para los usuarios de Burp Suite Professional, Burp Intruder proporciona la lista de carga útil predefinida Fuzzing - path traversal . Esta contiene algunas secuencias de recorrido de ruta codificadas que puede probar.