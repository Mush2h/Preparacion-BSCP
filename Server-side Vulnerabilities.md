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
