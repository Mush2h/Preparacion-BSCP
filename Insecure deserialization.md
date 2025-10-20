# Insecure deserialization

## Reto 1: Modificación de objetos serializados

Este laboratorio utiliza un mecanismo de sesión basado en serialización y, por lo tanto, es vulnerable a la escalada de privilegios. Para solucionarlo, edite el objeto serializado en la cookie de sesión para explotar esta vulnerabilidad y obtener privilegios administrativos. A continuación, elimine al usuario. carlos.

Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales: wiener:peter 

En este laboratorio se trabaja con una aplicación que utiliza objetos PHP serializados como mecanismo de sesión. Al iniciar sesión, se observa que la cookie contiene un objeto codificado en Base64 y URL.

Al decodificarlo, se descubre una estructura con un atributo admin configurado como falso (b:0). Al modificar este atributo a verdadero (b:1) y reenviar la petición, se otorgan privilegios administrativos al usuario.

Gracias a esto, se accede al panel de administración, desde donde es posible eliminar al usuario objetivo (carlos).

```
Cookie: session=Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czo1OiJhZG1pbiI7YjoxO30%3d
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:1;}
```

## Reto 2: Modificación de tipos de datos serializados


Este laboratorio utiliza un mecanismo de sesión basado en serialización y, por lo tanto, es vulnerable a la omisión de la autenticación. Para resolver el laboratorio, edite el objeto serializado en la cookie de sesión para acceder a... administratorcuenta. Luego, elimine el usuario. carlos.

Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales: wiener:peter 

En este laboratorio, la aplicación utiliza objetos PHP serializados dentro de una cookie de sesión. Al iniciar sesión como usuario legítimo, se puede interceptar y analizar la estructura del objeto.

La vulnerabilidad se explota modificando el tipo de dato del campo ‘access_token‘, convirtiéndolo de una cadena a un entero (i:0), y cambiando el nombre de usuario a ‘administrator‘, ajustando también la longitud del valor serializado.

Gracias a este cambio, la aplicación interpreta la sesión como si perteneciera al administrador. Esto permite acceder al panel de administración y eliminar al usuario carlos, resolviendo así el laboratorio.

```
O:4:"User":2:{s:8:"username";s:13:"administrator";s:12:"access_token";b:1;}
```

## Reto 3: Uso de la funcionalidad de la aplicación para explotar la deserialización insegura

FACULTATIVO
LAB No resuelto

Este laboratorio utiliza un mecanismo de sesión basado en serialización. Una característica específica invoca un método peligroso en los datos proporcionados en un objeto serializado. Para resolver el laboratorio, edite el objeto serializado en la cookie de sesión y úselo para eliminarlo. morale.txtarchivo del directorio de inicio de Carlos.

Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales: wiener:peter

También tienes acceso a una cuenta de respaldo: gregg:rosebud  


En este laboratorio, la aplicación gestiona sesiones a través de objetos PHP serializados que incluyen atributos como la ruta al avatar del usuario.

Mediante la función legítima de eliminar cuenta, se puede explotar un comportamiento peligroso del servidor: al eliminar al usuario, el sistema también elimina el archivo indicado por el atributo ‘avatar_link‘.

Al modificar dicho atributo para que apunte al archivo ‘/home/carlos/morale.txt‘, la aplicación termina borrando ese fichero en lugar del avatar.

```
O:4:"User":3:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"ooru6xjhzzktovkjwksckzd9l9dhwhkv";s:11:"avatar_link";s:23:"/home/carlos/morale.txt";}
```

## Reto 4: Inyección de objetos arbitrarios en PHP

Este laboratorio utiliza un mecanismo de sesión basado en serialización y, por lo tanto, es vulnerable a la inyección de objetos arbitrarios. Para solucionar el problema, cree e inyecte un objeto serializado malicioso para eliminar el... morale.txtArchivo del directorio personal de Carlos. Necesitará acceder al código fuente para resolver este laboratorio.

Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales: wiener:peter 

En este laboratorio, se analiza una vulnerabilidad crítica: la inyección de objetos arbitrarios en PHP. Tras acceder al código fuente, se descubre una clase con un método mágico ‘__destruct()‘ que borra el archivo definido en el atributo ‘lock_file_path‘.

Aprovechando esta funcionalidad, se construye manualmente un objeto serializado que apunta al archivo ‘/home/carlos/morale.txt‘. Al inyectarlo como cookie de sesión y ejecutar una petición, el objeto se deserializa y el método destructivo se dispara automáticamente.

```
O:14:"CustomTemplate":1:{s:14:"lock_file_path";s:23:"/home/carlos/morale.txt";}
```

## Reto 5: Explotación de la deserialización de Java con Apache Commons

Este laboratorio utiliza un mecanismo de sesión basado en serialización y carga la biblioteca Apache Commons Collections. Aunque no tenga acceso al código fuente, puede aprovechar este laboratorio utilizando cadenas de gadgets preconstruidas.

Para resolver el laboratorio, utilice una herramienta de terceros para generar un objeto serializado malicioso que contenga una carga útil de ejecución remota de código. Luego, pase este objeto al sitio web para eliminarlo. morale.txtarchivo del directorio de inicio de Carlos.

Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales: wiener:peter 

Este laboratorio expone una deserialización insegura en una aplicación Java que utiliza la librería Apache Commons Collections. Aunque no se tiene acceso al código fuente, es posible explotar la vulnerabilidad usando cadenas de gadgets preconstruidas.

Con la herramienta `ysoserial`, se genera un objeto serializado con un payload que ejecuta un comando del sistema operativo para eliminar el archivo morale.txt de Carlos. Este objeto se inyecta en la cookie de sesión tras codificarlo adecuadamente.

Cuando el servidor deserializa el objeto, se desencadena la ejecución remota del comando. Este escenario representa una amenaza crítica frecuente en aplicaciones Java que confían en objetos serializados sin validación adecuada.

```
java -jar ysoserial-all.jar CommonsCollections4 'rm /home/carlos/morale.txt' | base64 -w 0
```

## Reto 6: Explotación de la deserialización de PHP con una cadena de gadgets preconstruida

Este laboratorio cuenta con un mecanismo de sesión basado en serialización que utiliza una cookie firmada. También utiliza un framework PHP común. Aunque no tenga acceso al código fuente, puede aprovechar la deserialización insegura de este laboratorio mediante cadenas de gadgets preconstruidas.

Para resolver el laboratorio, identifique el framework de destino y utilice una herramienta de terceros para generar un objeto serializado malicioso que contenga una carga útil de ejecución remota de código. Después, determine cómo generar una cookie firmada válida que contenga el objeto malicioso. Finalmente, transfiérala al sitio web para eliminar el objeto. morale.txtarchivo del directorio de inicio de Carlos.

Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales: wiener:peter 

analizamos el mecanismo de sesión basado en cookies firmadas con HMAC SHA-1. Al inspeccionar el contenido de la cookie, se identifica que contiene un objeto PHP serializado. Sin embargo, cualquier modificación invalida la firma, impidiendo su explotación directa.

Se encuentra un archivo de depuración expuesto (/cgi-bin/phpinfo.php) que revela que la aplicación usa Symfony 4.3.6. Además, en la salida del phpinfo, se expone la variable SECRET_KEY, que es fundamental para firmar de nuevo la cookie maliciosa con una firma válida.

Con la clave secreta descubierta, se utiliza la herramienta PHPGGC para generar un objeto serializado con un payload que elimina el archivo ‘morale.txt‘ de Carlos.

Este objeto se firma con HMAC SHA-1 utilizando la clave obtenida previamente, lo que permite generar una cookie válida que el servidor aceptará.

Finalmente, esta cookie maliciosa se inyecta en la sesión y, al ser deserializada por el backend de Symfony, se ejecuta el payload, logrando así la ejecución remota de comandos y la resolución del laboratorio.

```shell
./phpggc Symfony/RCE4 exec 'rm /home/carlos/morale.txt' | base64 -w 0
```

```php
<?php
  $object = "Tzo0NzoiU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxUYWdBd2FyZUFkYXB0ZXIiOjI6e3M6NTc6IgBTeW1mb255XENvbXBvbmVudFxDYWNoZVxBZGFwdGVyXFRhZ0F3YXJlQWRhcHRlcgBkZWZlcnJlZCI7YToxOntpOjA7TzozMzoiU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQ2FjaGVJdGVtIjoyOntzOjExOiIAKgBwb29sSGFzaCI7aToxO3M6MTI6IgAqAGlubmVySXRlbSI7czoyNjoicm0gL2hvbWUvY2FybG9zL21vcmFsZS50eHQiO319czo1MzoiAFN5bWZvbnlcQ29tcG9uZW50XENhY2hlXEFkYXB0ZXJcVGFnQXdhcmVBZGFwdGVyAHBvb2wiO086NDQ6IlN5bWZvbnlcQ29tcG9uZW50XENhY2hlXEFkYXB0ZXJcUHJveHlBZGFwdGVyIjoyOntzOjU0OiIAU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxQcm94eUFkYXB0ZXIAcG9vbEhhc2giO2k6MTtzOjU4OiIAU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxQcm94eUFkYXB0ZXIAc2V0SW5uZXJJdGVtIjtzOjQ6ImV4ZWMiO319Cg==";
  $secretKey = "76xfgpxerrpc7fk09z35o25lxmbt55sx";
  $cookie = '{"token":"' . $object . '","sig_hmac_sha1":"' . hash_hmac("sha1", $object, $secretKey) . '"}';
  echo $cookie;
?>
```