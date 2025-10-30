## Reto 1: Confianza excesiva en los controles del lado del cliente

Este laboratorio no valida adecuadamente la entrada del usuario. Puedes aprovechar una falla lógica en su flujo de trabajo de compras para comprar artículos a un precio no previsto. Para resolver el laboratorio, compra una "Chaqueta ligera de cuero l33t".

Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales: wiener:peter 

vemos cómo un control excesivo en el cliente puede derivar en una lógica vulnerable. Durante el proceso de añadir un producto al carrito, observamos que el precio del artículo se incluye en el cuerpo de la petición.

Dado que el servidor no valida este dato, lo modificamos en Burp Repeater por un valor inferior al crédito disponible. Una vez ajustado, completamos la compra del artículo “Lightweight l33t leather jacket” por un precio mucho menor al real, resolviendo el laboratorio.

```
productId=1&redir=PRODUCT&quantity=1&price=13
```

## Reto 2: Vulnerabilidad lógica de alto nivel


Este laboratorio no valida adecuadamente la entrada del usuario. Puedes aprovechar una falla lógica en su flujo de trabajo de compras para comprar artículos a un precio no previsto. Para resolver el laboratorio, compra una "Chaqueta ligera de cuero l33t".

Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales: wiener:peter 

Una validación deficiente permite manipular la lógica del sistema. Tras añadir un producto barato al carrito, interceptamos la petición y cambiamos la cantidad a un valor negativo. Esto provoca que el sistema reste unidades y genere un total negativo.

Aprovechamos esta lógica para añadir la chaqueta “Lightweight l33t leather jacket” al carrito, y reducimos el precio total combinándolo con una cantidad negativa de otro artículo, logrando así completar la compra por debajo del crédito disponible y resolver el laboratorio.

## Reto 3: Controles de seguridad inconsistentes


La lógica defectuosa de este laboratorio permite que usuarios arbitrarios accedan a funciones administrativas que solo deberían estar disponibles para los empleados de la empresa. Para resolver el laboratorio, acceda al panel de administración y elimine el usuario. carlos. 

un control de seguridad mal implementado puede permitir a un atacante acceder a funciones reservadas para empleados.

Aunque el panel de administración está limitado a usuarios con email corporativo, al registrarnos con un correo legítimo y luego modificarlo por uno con dominio @dontwannacry.com, se nos concede acceso sin validar adecuadamente esta modificación. Esta incoherencia en la lógica nos permite entrar al panel de admin y eliminar al usuario carlos para resolver el laboratorio.


## Reto 4: Aplicación defectuosa de las normas empresariales

Este laboratorio tiene un fallo lógico en su proceso de compras. Para solucionarlo, aprovecha este fallo para comprar una "chaqueta de cuero ligera l33t".

Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales: wiener:peter 

un fallo lógico en la validación de cupones dentro del proceso de compra. Aunque el sistema impide aplicar el mismo cupón más de una vez consecutiva, permite alternar entre distintos cupones sin comprobar si ya fueron usados.

Al alternar repetidamente los códigos NEWCUST5 y SIGNUP30, logramos aplicar múltiples descuentos acumulativos. Así conseguimos bajar el precio de la chaqueta por debajo de nuestro crédito y completar la compra con éxito, resolviendo el laboratorio.

## Reto 5: Fallo lógico de bajo nivel

Este laboratorio no valida adecuadamente la entrada del usuario. Puedes aprovechar una falla lógica en su flujo de trabajo de compras para comprar artículos a un precio no previsto. Para resolver el laboratorio, compra una "Chaqueta ligera de cuero l33t".

Puede iniciar sesión en su propia cuenta utilizando las siguientes credenciales: wiener:peter 

Una vez provocado el desbordamiento, usamos el valor negativo resultante para ajustar cuidadosamente el contenido del carrito. Añadimos una cantidad concreta de otro producto hasta que el total del pedido esté justo por debajo de los $100 de crédito disponibles.

De este modo, conseguimos comprar la chaqueta de cuero a un precio no intencionado, completando la compra con éxito y resolviendo el laboratorio.

## Reto 6: Manejo inconsistente de entradas excepcionales

Este laboratorio no valida adecuadamente la entrada del usuario. Puedes aprovechar una vulnerabilidad en su proceso de registro de cuentas para obtener acceso a las funciones administrativas. Para completar el laboratorio, accede al panel de administración y elimina el usuario. carlos. 

validación deficiente de direcciones de correo puede permitir acceso no autorizado. La aplicación restringe el acceso al panel de administración a usuarios con correos del dominio dontwannacry.com, pero recorta cualquier dirección a 255 caracteres sin volver a validar el dominio después del truncamiento.

Al registrar una cuenta con una dirección larga que contenga @dontwannacry.com justo antes del límite, logramos que el sistema guarde una dirección aparentemente legítima y nos dé acceso al panel de administración. Desde ahí, eliminamos al usuario carlos para completar el laboratorio.

```shell
echo -n "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA@dontwannacry.com.exploit-0aef00fa03776d8d80f1e85601830045.exploit-server.net" | wc -c

```

## Reto 7: Aislamiento débil en punto final de doble uso

Este laboratorio parte de una suposición errónea sobre el nivel de privilegios del usuario basándose en su entrada. Como resultado, puedes aprovechar la lógica de sus funciones de gestión de cuentas para acceder a las cuentas de usuarios arbitrarios. Para resolver el laboratorio, accede a administratorcuenta y eliminar el usuario carlos.

Puedes iniciar sesión en tu cuenta utilizando las siguientes credenciales: wiener:peter 

vemos cómo un endpoint diseñado para el cambio de contraseña puede ser reutilizado de forma maliciosa por falta de aislamiento lógico. Aunque normalmente se solicita la contraseña actual para realizar el cambio, al eliminar este parámetro en la petición y modificar el valor del campo username a administrator, logramos modificar directamente la contraseña del administrador.

Esto ocurre porque el sistema no valida adecuadamente que la solicitud provenga del usuario autenticado. Tras cambiar la contraseña, accedemos como administrador y eliminamos al usuario carlos, completando así el laboratorio

```
csrf=u1Vq7m9mIajQJxjsnJZxS0cwlRUBHP72&username=administrator&new-password-1=123&new-password-2=123
```

## Reto 8: Validación insuficiente del flujo de trabajo

Este experimento parte de premisas erróneas sobre la secuencia de eventos en el proceso de compra. Para resolverlo, aprovecha esta errata para comprar una "chaqueta de cuero ligera l33t".

Puedes iniciar sesión en tu cuenta utilizando las siguientes credenciales: wiener:peter 
un fallo lógico en la secuencia del proceso de compra. Tras adquirir un producto asequible con el crédito disponible, observamos que el servidor valida la compra únicamente al recibir una petición concreta de confirmación.

Más adelante, añadimos un artículo de mayor valor (la chaqueta) al carrito y simplemente repetimos la misma petición de confirmación usada anteriormente. El sistema no valida si el producto ha sido efectivamente pagado, ni si el crédito cubre el importe, permitiéndonos así adquirir la chaqueta sin coste y resolver el laboratorio.

```
GET /cart/order-confirmation?order-confirmed=true
```

## Reto 9: Omisión de autenticación mediante máquina de estados defectuosa

Este laboratorio parte de suposiciones erróneas sobre la secuencia de eventos en el proceso de inicio de sesión. Para resolverlo, aprovecha esta vulnerabilidad para eludir la autenticación, acceder a la interfaz de administración y eliminar el usuario. carlos.

Puedes iniciar sesión en tu cuenta utilizando las siguientes credenciales: wiener:peter 

un fallo en la lógica de la máquina de estados del proceso de autenticación. Tras hacer login, la aplicación nos redirige a una página para seleccionar el rol del usuario. Sin embargo, si interceptamos y descartamos la solicitud de selección de rol justo después de autenticarnos, y accedemos directamente al sitio, la aplicación asume por defecto el rol de administrador.

Esto nos permite acceder al panel de administración sin autorización legítima y eliminar al usuario Carlos para completar el laboratorio.

```
GET /role-selector HTTP/2
```