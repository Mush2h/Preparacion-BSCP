## Reto 1: SSRF básico contra el servidor local

Este laboratorio tiene una función de verificación de existencias que obtiene datos de un sistema interno.

Para resolver el laboratorio, cambie la URL de verificación de existencias para acceder a la interfaz de administración en http://localhost/adminy eliminar el usuario carlos.

Explotamos una vulnerabilidad SSRF en la funcionalidad de stock. Ya demostramos que era posible acceder al panel de administración interno, y ahora profundizamos en cómo automatizar la interacción con rutas internas específicas.

El objetivo en esta parte es enviar una solicitud manipulada desde el parámetro vulnerable para ejecutar una acción concreta: eliminar al usuario carlos desde el panel de administración. Esto demuestra cómo un SSRF básico puede escalar rápidamente en impacto si se accede a funciones internas críticas del servidor.

```
stockApi=http://localhost/admin/delete?username=carlos
```

## Reto 2: SSRF básico contra otro sistema back-end

Este laboratorio tiene una función de verificación de existencias que obtiene datos de un sistema interno.

Para resolver el laboratorio, utilice la función de verificación de existencias para escanear el interior 192.168.0.X rango para una interfaz de administrador en el puerto 8080, luego úsalo para eliminar al usuario carlos. 

Aprovechamos la funcionalidad de verificación de stock para lanzar un escaneo controlado dentro del rango 192.168.0.X, variando la última parte de la IP.

Una vez identificamos qué IP responde en el puerto 8080 con una interfaz de administración accesible, manipulamos el parámetro vulnerable para interactuar con ella y ejecutar una petición que elimina al usuario carlos.


```
stockApi=http://192.168.0.80:8080/admin/delete?username=carlos
```

## Reto 3: SSRF ciega con detección fuera de banda

Este sitio utiliza un software de análisis que obtiene la URL especificada en el encabezado Referer cuando se carga una página de producto.

Para resolver el laboratorio, utilice esta funcionalidad para generar una solicitud HTTP al servidor público Burp Collaborator. 

En esta clase continuamos con el estudio de ataques SSRF, pero en este caso nos enfrentamos a una variante ciega donde no obtenemos respuesta directa de la interacción. Aprovechamos una funcionalidad del sistema que realiza peticiones basadas en el valor del encabezado Referer, utilizado por un software de analítica interna.

Manipulamos este encabezado para incluir un dominio generado por Burp Collaborator, lo que permite detectar si el servidor realiza una solicitud saliente hacia dicho dominio. Aunque no veamos resultados en la respuesta de la aplicación, los logs de Burp Collaborator nos confirman que se ha producido una interacción, validando así la existencia de la vulnerabilidad. 

```html
Referer: https://e3in67nfsxrvz0qy88nx5ojwonuei56u.oastify.com
```

## Reto 4: SSRF con filtro de entrada basado en lista negra

Este laboratorio tiene una función de verificación de existencias que obtiene datos de un sistema interno.

Para resolver el laboratorio, cambie la URL de verificación de existencias para acceder a la interfaz de administración en http://localhost/admin y eliminar el usuario carlos.

El desarrollador ha implementado dos defensas anti-SSRF débiles que deberás evitar. 

centrándonos en cómo evadir defensas basadas en listas negras. La aplicación impide directamente ciertas direcciones como 127.0.0.1 y rutas sensibles como /admin, pero estas restricciones son débiles.

Utilizamos técnicas de evasión como redirecciones a direcciones IP alternativas (por ejemplo, 127.1) y codificación doble de caracteres (%2561 en lugar de a) para engañar al filtro.

```
stockApi=http://127.0.1/%2561dmin/delete?username=carlos
```