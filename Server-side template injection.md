# Server-side template injection

## ¿Qué es server-side template injection ?
Es cuando un atacante puede usar la sintaxis de plantilla nativa para inyectar una carga maliciosa en una plantilla, que luego se ejecuta en el lado del servidor.

Los motores de plantillas están diseñados para generar páginas web combinando plantillas fijas con datos volátiles. Los ataques de inyección de plantillas del lado del servidor pueden ocurrir cuando la entrada del usuario se concatena directamente en una plantilla, en lugar de pasarla como datos. Esto permite a los atacantes inyectar directivas de plantilla arbitrarias para manipular el motor de plantillas, lo que a menudo les permite tomar el control total del servidor. Como sugiere el nombre, las cargas útiles de inyección de plantillas del lado del servidor se entregan y evalúan del lado del servidor, lo que las hace potencialmente mucho más peligrosas que una inyección de plantilla típica del lado del cliente.

## ¿Cuál es el impacto de la inyección de plantillas del lado del servidor?
Las vulnerabilidades de inyección de plantillas del lado del servidor pueden exponer los sitios web a una variedad de ataques según el motor de plantillas en cuestión y cómo lo utiliza exactamente la aplicación. En determinadas circunstancias excepcionales, estas vulnerabilidades no suponen un riesgo real para la seguridad. Sin embargo, la mayoría de las veces, el impacto de la inyección de plantillas del lado del servidor puede ser catastrófico.

En el extremo más severo de la escala, un atacante podría potencialmente lograr la ejecución remota de código, tomando el control total del servidor back-end y usándolo para realizar otros ataques a la infraestructura interna.

Incluso en casos donde no es posible la ejecución completa del código remoto, un atacante a menudo puede usar la inyección de plantillas del lado del servidor como base para muchos otros ataques, obteniendo potencialmente acceso de lectura a datos confidenciales y archivos arbitrarios en el servidor.

## ¿Cómo surgen las vulnerabilidades de inyección de plantillas del lado del servidor?
Las vulnerabilidades de inyección de plantillas del lado del servidor surgen cuando la entrada del usuario se concatena en plantillas en lugar de pasarse como datos.

Las plantillas estáticas que simplemente proporcionan marcadores de posición en los que se representa contenido dinámico generalmente no son vulnerables a la inyección de plantillas del lado del servidor. El ejemplo clásico es un correo electrónico que saluda a cada usuario por su nombre, como el siguiente extracto de una plantilla de Twig:

```ruby
$output = $twig->render("Dear {first_name},", array("first_name" => $user.first_name) );
```
Esto no es vulnerable a la inyección de plantilla del lado del servidor porque el nombre del usuario simplemente se pasa a la plantilla como datos.

Sin embargo, como las plantillas son simplemente cadenas, los desarrolladores web a veces concatenan directamente la entrada del usuario en las plantillas antes de la representación. Tomemos un ejemplo similar al anterior, pero esta vez, los usuarios pueden personalizar partes del correo electrónico antes de enviarlo. Por ejemplo, podrían elegir el nombre que se utiliza:

```ruby
$output = $twig->render("Dear " . $_GET['name']);
```
En este ejemplo, en lugar de pasar un valor estático a la plantilla, se genera dinámicamente parte de la plantilla mediante el `GET` parámetro `name`. Como la sintaxis de la plantilla se evalúa en el lado del servidor, esto potencialmente permite que un atacante coloque una carga útil de inyección de plantilla en el lado del servidor dentro del `name` parámetro de la siguiente manera:

```ruby
http://vulnerable-website.com/?name={{bad-stuff-here}}
```
A veces, este tipo de vulnerabilidades se producen por accidente debido a un diseño deficiente de las plantillas por parte de personas que no están familiarizadas con las implicaciones de seguridad. Como en el ejemplo anterior, es posible que vea diferentes componentes, algunos de los cuales contienen información del usuario, concatenados e integrados en una plantilla. En cierto modo, esto es similar a las vulnerabilidades de inyección SQL que se producen en declaraciones mal redactadas.

Sin embargo, a veces este comportamiento se implementa de manera intencional. Por ejemplo, algunos sitios web permiten deliberadamente que ciertos usuarios privilegiados, como editores de contenido, editen o envíen plantillas personalizadas por diseño. Esto claramente plantea un gran riesgo de seguridad si un atacante logra comprometer una cuenta con tales privilegios.