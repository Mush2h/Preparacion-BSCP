# Path traversal

## Qué es un path traversal

El path traversal también conocido como directory traversal. Estas vulnerabilidades permiten a un atacante leer archivos arbitrarios en el servidor que ejecuta en una aplicación.

Esto podría incluir:

- Códigos y datos de la aplicación.
- Credenciales para sistemas del back-end.
- Archivos confidenciales del SO.

En algunos casos, un atacante podría escribir en archivos arbitrarios del servidor, lo que le permitiría modificar datos o comportamientos de la aplicacion y, en ultima instancia, tomar el control total del servidor.

## Lectura de archivos arbitrarios via path traversal
Imagine una aplicación de compras que muestra imágenes de artículos en venta. Esta podría cargar una imagen utilizando el siguiente código HTML:

```ruby
<img src="/loadImage?filename=218.png">
```

La loadImageURL toma un `filename` parámetro y devuelve el contenido del archivo especificado. Los archivos de imagen se almacenan en el disco en la ubicación `/var/www/images/`. Para devolver una imagen, la aplicación agrega el nombre de archivo solicitado a este directorio base y utiliza una API del sistema de archivos para leer el contenido del archivo. En otras palabras, la aplicación lee desde la siguiente ruta de archivo:

```ruby
/var/www/images/218.png
```

Esta aplicación no implementa defensas contra ataques de path traversal. Como resultado, un atacante puede solicitar la siguiente URL para recuperar el `/etc/passwd` archivo del sistema de archivos del servidor:

```ruby
https://insecure-website.com/loadImage?filename=../../../etc/passwd
```

Esto hace que la aplicación lea desde la siguiente ruta de archivo:

`/var/www/images/../../../etc/passwd`

La secuencia ../ es válida dentro de una ruta de archivo y significa subir un nivel en la estructura de directorios. Las tres ../ en secuencias consecutivas suben de nivel desde /var/www/images/ a la raíz del sistema de archivos, por lo que el archivo que se lee en realidad es:

`/etc/passwd`

En los sistemas operativos basados ​​en Unix, este es un archivo estándar que contiene detalles de los usuarios registrados en el servidor, pero un atacante podría recuperar otros archivos arbitrarios utilizando la misma técnica.

En Windows, tanto ../ y como ..\ son secuencias válidas de path traversal. El siguiente es un ejemplo de un ataque equivalente contra un servidor basado en Windows:

```ruby
https://insecure-website.com/loadImage?filename=..\..\..\windows\win.ini
```

Laboratorio: File path traversal , caso simple.




## Obstáculos comunes para explotar vulnerabilidades de recorrido de ruta

Muchas aplicaciones que introducen datos del usuario en rutas de archivos implementan defensas contra ataques de cruce de rutas, que a menudo se pueden eludir.

Si una aplicación elimina o bloquea las secuencias de recorrido de directorio del nombre de archivo proporcionado por el usuario, podría ser posible eludir la defensa utilizando una variedad de técnicas.

Es posible que puedas usar una ruta absoluta desde la raíz del sistema de archivos, como `filename=/etc/passwd`, para hacer referencia directa a un archivo sin utilizar ninguna secuencia de recorrido.

Es posible que puedas usar secuencias de recorrido anidadas, como `....//`o `....\/`. Estas se convierten en secuencias de recorrido simples cuando se elimina la secuencia interna.

En algunos contextos, como en una ruta URL o en el `filename` parámetro de una `multipart/form-data`solicitud, los servidores web pueden eliminar cualquier secuencia de recorrido de directorio antes de pasar su entrada a la aplicación. A veces, puede evitar este tipo de limpieza codificando la URL o incluso codificando la URL dos veces, `../` lo que da como resultado `%2e%2e%2f`y `%252e%252e%252f`respectivamente. También pueden funcionar varias codificaciones no estándar, como `..%c0%afo` `..%ef%bc%8f`.

Para los usuarios de Burp Suite Professional, Burp Intruder proporciona la lista de carga útil predefinida Fuzzing - path traversal . Esta contiene algunas secuencias de recorrido de ruta codificadas que puede probar.

Una aplicación puede requerir que el nombre de archivo proporcionado por el usuario comience con la carpeta base esperada, como `/var/www/images`. En este caso, podría ser posible incluir la carpeta base requerida seguida de secuencias de recorrido adecuadas. Por ejemplo: filename=/var/www/images/../../../etc/passwd.

Una aplicación puede requerir que el nombre de archivo proporcionado por el usuario termine con una extensión de archivo esperada, como .png. En este caso, podría ser posible usar un byte nulo para terminar efectivamente la ruta del archivo antes de la extensión requerida. Por ejemplo: filename=../../../etc/passwd%00.png.

## Cómo prevenir el ataque
La forma más eficaz de evitar vulnerabilidades de path traversal es evitar por completo el paso de datos proporcionados por el usuario a las API del sistema de archivos. Muchas funciones de aplicaciones que hacen esto se pueden reescribir para ofrecer el mismo comportamiento de una manera más segura.

Si no puede evitar pasar la información proporcionada por el usuario a las API del sistema de archivos, recomendamos utilizar dos capas de defensa para prevenir ataques:

Valide la entrada del usuario antes de procesarla. Lo ideal es comparar la entrada del usuario con una lista blanca de valores permitidos. Si eso no es posible, verifique que la entrada contenga únicamente contenido permitido, como caracteres alfanuméricos únicamente.
Después de validar la entrada proporcionada, añada la entrada al directorio base y utilice una API del sistema de archivos de la plataforma para canonizar la ruta. Verifique que la ruta canonizada comience con el directorio base esperado.
A continuación se muestra un ejemplo de un código Java simple para validar la ruta canónica de un archivo según la entrada del usuario:

File file = new File(BASE_DIRECTORY, userInput);
if (file.getCanonicalPath().startsWith(BASE_DIRECTORY)) {
    // process file
}

## Reto 1: Path traversal caso simple

Este laboratorio contiene una vulnerabilidad de recorrido de ruta en la visualización de imágenes de productos.
Para resolver el laboratorio, recupera el contenido del /etc/passwd archivo. 


La aplicación permite cargar imágenes de productos a través de un parámetro ‘filename‘. Interceptamos esta petición y abusamos de la falta de validación para escalar directorios utilizando secuencias ‘../‘.

```
GET /image?filename=../../../../../../etc/passwd
```

## Reto 2: Path traversal secuencias de recorrido bloqueadas con omisión de ruta absoluta

Este laboratorio contiene una vulnerabilidad de recorrido de ruta en la visualización de imágenes de productos.

La aplicación bloquea las secuencias de recorrido pero trata el nombre de archivo proporcionado como relativo a un directorio de trabajo predeterminado.

Para resolver el laboratorio, recupera el contenido del /etc/passwd archivo. 

Aunque la aplicación filtra las secuencias clásicas de path traversal, como ‘../‘, permite especificar rutas absolutas desde el sistema de archivos raíz.

```
GET /image?filename=/etc/passwd
```

## Reto 3: Recorrido de rutas de archivos, secuencias de recorrido eliminadas de forma no recursiva

Este laboratorio contiene una vulnerabilidad de recorrido de ruta en la visualización de imágenes de productos.

La aplicación elimina las secuencias de recorrido de ruta del nombre de archivo proporcionado por el usuario antes de usarlo.

Para resolver el laboratorio, recupera el contenido del /etc/passwd archivo. 

La aplicación intenta mitigar el path traversal eliminando las secuencias ‘../‘, pero lo hace de forma no recursiva. Esto permite evadir el filtro usando variantes como ‘….//‘, que tras la normalización del sistema de archivos siguen interpretándose como una subida de directorio.

```
GET /image?filename=....//....//....//....//....//....//etc/passwd
```

## Reto 4:Path traversal secuencias de recorrido eliminadas con decodificación de URL superflua

Este laboratorio contiene una vulnerabilidad de recorrido de ruta en la visualización de imágenes de productos.

La aplicación bloquea la entrada que contiene secuencias de recorrido de ruta. Luego, realiza una decodificación URL de la entrada antes de usarla.

Para resolver el laboratorio, recupera el contenido del /etc/passwd archivo.

En este caso, la aplicación bloquea directamente las secuencias ‘../‘, pero comete el error de decodificar la entrada una vez más después del filtrado. Aprovechamos esto utilizando ‘..%252f‘, que al ser decodificado dos veces se transforma en ‘../‘, permitiéndonos realizar un path traversal efectivo.

```
GET /image?filename=..%2f..%2f..%2f..%2f..%2f..%2fetc/passwd
```

## Reto 5: Recorrido de rutas de archivos, validación del inicio de la ruta

Este laboratorio contiene una vulnerabilidad de recorrido de ruta en la visualización de imágenes de productos.
La aplicación transmite la ruta completa del archivo a través de un parámetro de solicitud y valida que la ruta proporcionada comience con la carpeta esperada.
Para resolver el laboratorio, recupera el contenido del /etc/passwd archivo
Recorrido de rutas de archivos, validación del inicio de la ruta

En este escenario, la aplicación intenta validar que las rutas comiencen con ‘/var/www/images/‘, pero no restringe adecuadamente lo que viene después. Aprovechamos esto enviando una ruta como ‘/var/www/images/https://loquesea/../etc/passwd‘, que tras la resolución final accede al archivo sensible fuera del directorio permitido.

```
GET /image?filename=/var/www/images/../../../etc/passwd
```

## Reto 6: Recorrido de rutas de archivos, validación de la extensión de archivo con omisión de bytes nulos

Este laboratorio contiene una vulnerabilidad de recorrido de ruta en la visualización de imágenes de productos.

La aplicación valida que el nombre de archivo proporcionado termine con la extensión de archivo esperada.

Para resolver el laboratorio, recupera el contenido del /etc/passwd archivo. 

```
GET /image?filename=../../../etc/passwd%00.png
```
