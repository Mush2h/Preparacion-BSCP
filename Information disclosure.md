## Reto 1: Divulgación de información en mensajes de error

Los mensajes de error detallados de este laboratorio revelan que se utiliza una versión vulnerable de un framework de terceros. Para resolver el laboratorio, obtenga y envíe el número de versión de este framework. 

interceptamos la petición que contiene el parámetro ‘productId‘. Al modificar su valor por un tipo de dato inesperado (por ejemplo, una cadena de texto), provocamos que la aplicación genere una excepción.

Esta excepción muestra una traza completa del error, lo que nos permite descubrir detalles técnicos sobre el sistema.

```
https://0a7a00c20485b31a80208a8f005800fc.web-security-academy.net/product?productId=pepe
```

```
Apache Struts 2 2.3.31
```

## Reto 2: Divulgación de información en la página de depuración


Este laboratorio contiene una página de depuración que revela información confidencial sobre la aplicación. Para resolver el laboratorio, obtenga y envíe la SECRET_KEYvariable de entorno. 

```
<tr><td class="e">SECRET_KEY </td><td class="v">ifew5c2h660kyp33tebdnx9avfmdq0bc </td></tr>
```

En un simple comentario HTML puede exponer una ruta interna a una página de depuración (/cgi-bin/phpinfo.php). Al acceder a ella, obtenemos información sensible del entorno de la aplicación, como la variable SECRET_KEY, que utilizamos para completar.

## Reto 3: Divulgación del código fuente mediante archivos de respaldo

Este laboratorio filtra su código fuente mediante archivos de respaldo en un directorio oculto. Para resolver el problema, identifique y envíe la contraseña de la base de datos, que está codificada en el código fuente filtrado. 

En esta clase exploramos cómo rutas expuestas en ‘robots.txt‘ pueden revelar directorios sensibles como ‘/backup‘. Al acceder a un archivo ‘.java.bak‘ dentro de esta ruta, obtenemos acceso al código fuente de la aplicación. Dentro de este archivo, identificamos una cadena hardcoded con la contraseña de acceso a la base de datos PostgreSQL, con la cual completamos el laboratorio.

```
private void readObject(ObjectInputStream inputStream) throws IOException, ClassNotFoundException
    {
        inputStream.defaultReadObject();

        ConnectionBuilder connectionBuilder = ConnectionBuilder.from(
                "org.postgresql.Driver",
                "postgresql",
                "localhost",
                5432,
                "postgres",
                "postgres",
                "zxolrn98vno59zn2swax6tu6nsxekfvs"
        ).withAutoCommit();
    }
```

## Reto 4: Evitar la autenticación mediante divulgación de información


La interfaz de administración de este laboratorio tiene una vulnerabilidad de omisión de autenticación, pero no es práctico explotarla sin el conocimiento de un encabezado HTTP personalizado utilizado por el front-end.

Para resolver el laboratorio, obtenga el nombre del encabezado y úselo para omitir la autenticación del laboratorio. Acceda a la interfaz de administración y elimine el usuario. carlos.

Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales: wiener:peter 

En esta clase nos encontramos con una interfaz de administración protegida por autenticación. Al enviar una petición usando el método HTTP TRACE, descubrimos que el servidor refleja una cabecera personalizada (X-Custom-IP-Authorization) con nuestra IP.

Este comportamiento revela una cabecera utilizada internamente para validar si una solicitud proviene de localhost. Al añadir manualmente esta cabecera con el valor 127.0.0.1, conseguimos acceder al panel de administración y eliminar al usuario carlos, resolviendo el laboratorio.


```
TRACE /admin HTTP/1.1
...
X-Custom-IP-Authorization: 87.221.170.8
```

```
GET /admin/delete?username=carlos HTTP/2
...
X-Custom-Ip-Authorization: 127.0.0.1
```

## Reto 5:Divulgación de información en el historial de control de versiones


Este laboratorio divulga información confidencial a través de su historial de control de versiones. Para resolverlo, obtenga la contraseña del administrator Luego, el usuario inicia sesión y elimina el usuario. carlos.  

exploramos cómo la exposición del directorio ‘.git‘ puede revelar información sensible. Al descargar el repositorio completo desde el servidor, accedemos al historial de commits y encontramos uno en el que se eliminó una contraseña hardcodeada del fichero ‘admin.conf‘.

Aunque en el código actual ya no está presente, el diff del commit aún contiene la contraseña en texto claro. Utilizamos esa información para iniciar sesión como administrador, acceder al panel de administración y eliminar al usuario carlos, completando así el laboratorio.

```
wget -r https://0a9c003e03c9c2dd83b4a05400fe00c7.web-security-academy.net/.git 
```
Cuando nos bajamos el proyecto comprobamos los logs 

```shell
git log -p ee5cc3ea9445b1e2fc338f27f07429e131a37fb6
commit ee5cc3ea9445b1e2fc338f27f07429e131a37fb6 (HEAD -> master)
Author: Carlos Montoya <carlos@carlos-montoya.net>
Date:   Tue Jun 23 14:05:07 2020 +0000

    Remove admin password from config

diff --git a/admin.conf b/admin.conf
index c7bddd3..21d23f1 100644
--- a/admin.conf
+++ b/admin.conf
@@ -1 +1 @@
-ADMIN_PASSWORD=40x00dvdt4z7ic080y1x
+ADMIN_PASSWORD=env('ADMIN_PASSWORD')
```